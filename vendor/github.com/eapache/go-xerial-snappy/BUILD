load("@io_bazel_rules_go//go:def.bzl", "go_library")

go_library(
    name = "go_default_library",
    srcs = ["snappy.go"],
    importmap = "k8s.io/kubernetes/vendor/github.com/eapache/go-xerial-snappy",
    importpath = "github.com/eapache/go-xerial-snappy",
    visibility = ["//visibility:public"],
    deps = ["//vendor/github.com/golang/snappy:go_default_library"],
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
