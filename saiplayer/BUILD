load("@rules_cc//cc:cc_binary.bzl", "cc_binary")
load("@rules_cc//cc:cc_library.bzl", "cc_library")

package(default_visibility = ["//visibility:public"])

cc_library(
    name = "SaiPlayer",
    srcs = [
        "CommandLineOptions.cpp",
        "CommandLineOptionsParser.cpp",
        "SaiPlayer.cpp",
    ],
    hdrs = glob(["*.h"]),
    deps = [
        "//lib:SaiRedis",
        "//syncd:Syncd",
    ],
)

cc_binary(
    name = "saiplayer",
    srcs = ["saiplayer_main.cpp"],
    dynamic_deps = [
        "@sai//meta:saimetadata_shared",
        "//meta:saimeta_shared",
        "@sonic-swss-common//:swsscommon_shared",
        "@libnl3//:libnl_3_shared",
        "@libnl3//:libnl_nf_3_shared",
        "@libnl3//:libnl_route_3_shared",
        "@libyang3//:yang_shared",
    ],
    deps = [
        ":SaiPlayer",
        "@sairedis_deps//libhiredis-dev:libhiredis",
        "@sairedis_deps//libzmq3-dev:libzmq3",
        "@sonic_build_infra//system_libs:pthread",
    ],
)
