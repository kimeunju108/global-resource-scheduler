load("@io_bazel_rules_go//go:def.bzl", "go_library")

go_library(
    name = "go_default_library",
    srcs = [
        "counter.go",
        "debug.go",
        "ewma.go",
        "gauge.go",
        "gauge_float64.go",
        "graphite.go",
        "healthcheck.go",
        "histogram.go",
        "json.go",
        "log.go",
        "meter.go",
        "metrics.go",
        "opentsdb.go",
        "registry.go",
        "runtime.go",
        "runtime_cgo.go",
        "runtime_gccpufraction.go",
        "runtime_no_cgo.go",
        "runtime_no_gccpufraction.go",
        "sample.go",
        "syslog.go",
        "timer.go",
        "writer.go",
    ],
    importmap = "k8s.io/kubernetes/vendor/github.com/rcrowley/go-metrics",
    importpath = "github.com/rcrowley/go-metrics",
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
