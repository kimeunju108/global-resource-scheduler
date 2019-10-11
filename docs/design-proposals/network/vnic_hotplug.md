# vnic hotplug

## Feature brief
vnic hotplug is the ability to allow admin of VM workload able to insert (plug in) or remove (plug out) vnics, while VM is alive. It does not cover adding ip alias to existing vnics, which is vnic update feature, and out of scope of vnic hotplug.

In Alkaid, hotplug is limited to nics other than the primary one (usually eth0), and only applicable to VM workloads.

Admin expresses the intention of hotplug by adding/removing pod.spec.vnics elements.

Assuming there is a VM pod having pod spec like below
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu01
spec:
  VirtualMachine:
    name: ubuntu18
    image: www.foo.com/vmimages/ubuntu18.04
    flavor: m4.large
  vpc: vpc-1a2b3c4d
  nics:
      - portID: a2815656-76f7-46d4-9d11-057063db1a14
      - portID: 33da18a3-1ad1-428a-90f5-9d906dc498fe
```

Deleting portID: 33da18a3-1ad1-428a-90f5-9d906dc498fe would cause eth1 disappear from cni netns, the network resource be released; the corresponding nic inside vm disapper. 

On the contrast, appending one more portID element would lead to network resource provision and a new nic appear in the vm. If DHCP is configured properly, the new nic in vm should get its ip address automatically.

Updating portID would be treated as deletion plus a new addition, and the new nic name *won't* be the one to be deleted. In our case in the cni netns, eth1 would be gone, while eth2 appear. Inside the vm, the old nic will be removed, and a new nic shows up, accordingly.

## Impact Analysis
Hotplug not only requires new functionalities from existent componenets, it also significantly changes some aspects of existent core behaviors.

### pod type augment
pod.status needs to have individual nic state data

### network controller (if applicable)
It needs new ability to detect update of pod.spec.nics, and engages backend network provider for necessary network provision/decommision.

### kubelet
kubelet needs to change the behavior on detection of pod.spec.nics update: creates plugin/plugout messages and send to the proper runtime.
Also, kubelet may need to collect nic status and send to api server as part of status update.

### CRI (VRI?) interface
augment to ensure the support plugin/plugout actions. For now, we decided to add a few necessary methods on top of CRI interface.

Following are proposed as extension of CRI:
#### interface PodSandboxDeviceManager
* method AttachDevice

| parameter | type | description |
| --- | --- | --- |
| podSandboxID | string | identifies the podSandbox |
| deviceConfig | message DeviceConfig | the detail of device |

| results | type | description |
| --- | --- | --- |
| err | error | whether the method succeed or not |

* method DetachDevice

| parameter | type | description |
| --- | --- | --- |
| podSandboxID | string | identifies the podSandbox |
| deviceFilter | message DeviceFilter | the filter condition of device |

| results | type | description |
| --- | --- | --- |
| err | error | whether the method succeed or not |

* method ListDevices

| parameter | type | description |
| --- | --- | --- |
| podSandboxID | string | identifies the podSandbox |
| deviceFilter | message DeviceFilter | the filter condition of device |

| results | type | description |
| --- | --- | --- |
| devConfig | (list) message DevConfig | detail of devices |
| err | error | whether the method succeed or not |

#### message DeviceConfig

| field | type | description |
| --- | --- | --- |
| type | string | nic, or volume? |
| config | string | json encoded type specific device configuration |

e.g. for nic type, the config could be (see [vnic-type](https://github.com/futurewei-cloud/cniplugins/blob/master/vnic/types.go) for formal definition)
```json
{
  "vpc": "vpc-demo",
  [ 
    { "portid": "port123456", "name": "eth9" }
  ]
}
```

#### message DeviceFilter

| field | type | description |
| --- | --- | --- |
| type | string | nic, or volume? |
| name | string | device name |


### runtime (virtlet)
* new functionality to implement the hot plugin/plgout by eventually calling into libvirt; particulally it needs to update internal cache of workload network info - this is critical for cni properly release network resource on pod termination; it also delegetes network attach/detach inside cni netns to the cni plugin, ensuring the proper nic names are specified.
* code change of existent workflow of pod creation to record full set of nic name/portID pair in the internal cache;

### cni plugin
augment cni plugin to support incremental add/del op. The key factor for its proper result is meaningful nic name besides the port ID, as we lose the implicit rule to derive the nic name (except for the first time Add op)

## Blocking Technical Issue
### virtlet libvirt support
The required nic hotplug feature support of libvirt provided by virtlet seems incomplete/buggy. 

Following virsh command gets back none nic interface, though nic  obvisiously exists.  
```bash
virsh domiflist <vm-domain-id>
```

Another example, following command on the virtlet enabled onebox cluster 
```bash
ip tuntap add dev mytap0 mode tap
ip link set dev mytap0 up
cat tap.xml
<interface type='ethernet'>
  <mac address='52:54:00:e1:d8:2d'/>
  <target dev='mytap0' managed='no'/>
  <model type='virtio'/>
</interface>

virsh attach-device <vm-domain-id> ./tap.xml
```
got error:
```
error: internal error: unable to execute QEMU command 'device_add': Duplicate ID 'net0' for device
```

## Summary
There are 2 significant things we need to address for vnic hotplu feature:
* blocking issue - support of nic hotplug provided by virtlet/libvirt is incomplete & buggy (cost estimate unknown)
* pending architectureal decision - CRI/VNI interface

Given the above 2 issues resolved, the estimate of development costs is as below

| subcomponent | estimated line of code | estimated cost (man-weeks) |
| ---:      |  ---:     | ---: |
| network controller | 100 | 1 |
| kubelet | 200 | 2 | 
| runtime | 400 | 4 |
| cni plugin | 50 | 0.5 |
| total | 750 | ~2 man-months |