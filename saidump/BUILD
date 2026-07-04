load("@rules_cc//cc:cc_binary.bzl", "cc_binary")
load("@rules_cc//cc:cc_library.bzl", "cc_library")

package(default_visibility = ["//visibility:public"])

cc_library(
    name = "Saidump",
    srcs = [
        "SaiDump.cpp",
        "SaiDump.h",
    ],
    deps = [
        "//lib:sairedis_hdrs",
        "//meta:saimeta",
        "@sonic-swss-common//:swsscommon",
    ],
)

cc_binary(
    name = "saidump",
    srcs = ["main.cpp"],
    dynamic_deps = [
        "//meta:saimeta_shared",
        "@sai//meta:saimetadata_shared",
        "@sonic-swss-common//:swsscommon_shared",
        "//lib:sairedis_shared",
        "@libnl3//:libnl_3_shared",
        "@libnl3//:libnl_nf_3_shared",
        "@libnl3//:libnl_route_3_shared",
        "@libyang3//:yang_shared",
    ],
    deps = [
        ":Saidump",
        "@sai//meta:saimetadata",
        "//meta:saimeta",
        "@sairedis_deps//libhiredis-dev:libhiredis",
        "@sairedis_deps//libzmq3-dev:libzmq3",
        "@sonic_build_infra//system_libs:pthread",
    ],
)
