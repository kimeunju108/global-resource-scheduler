load("@io_bazel_rules_go//go:def.bzl", "go_binary", "go_library")

go_library(
    name = "go_default_library",
    srcs = ["main.go"],
    importpath = "k8s.io/kubernetes/globalscheduler/grpc/cluster/grpc_test/state_client",
    visibility = ["//visibility:private"],
    deps = [
        "//globalscheduler/grpc/cluster/proto:go_default_library",
        "//vendor/google.golang.org/grpc:go_default_library",
        "//vendor/k8s.io/klog:go_default_library",
    ],
)

go_binary(
    name = "state_client",
    embed = [":go_default_library"],
    visibility = ["//visibility:public"],
)

filegroup(
    name = "package-srcs",
    srcs = glob(["**"]),
    tags = ["automanaged"],
    visibility = ["//visibility:private"],
)

filegroup(
    name = "all-srcs",
    srcs = [":package-srcs"],
    tags = ["automanaged"],
    visibility = ["//visibility:public"],
)
