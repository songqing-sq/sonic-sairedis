package(default_visibility = ["//visibility:public"])

load("@rules_cc//cc:cc_binary.bzl", "cc_binary")
load("@rules_cc//cc:cc_library.bzl", "cc_library")

cc_library(
    name = "libsaidump",
    srcs = ["SaiDump.cpp"],
    hdrs = glob(["*.h"]),
    deps = ["//meta:saimeta_objs", "//:saimetadata_objs"],
)

cc_binary(
    name = "saidump",
    srcs = ["main.cpp"],
    deps = [":libsaidump", "//meta:saimeta_objs", "//:saimetadata_objs"],
    dynamic_deps = ["//meta:saimeta", "//:saimetadata", "@sonic-swss-common//:swsscommon", "//lib:sairedis"],
    linkopts = ["-lhiredis", "-lpthread", "-lzmq"],
)
