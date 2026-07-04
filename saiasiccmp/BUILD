load("@rules_cc//cc:cc_binary.bzl", "cc_binary")
load("@rules_cc//cc:cc_library.bzl", "cc_library")

package(default_visibility = ["//visibility:public"])

cc_library(
    name = "AsicCmp",
    srcs = [
        "AsicCmp.cpp",
        "CommandLineOptions.cpp",
        "CommandLineOptionsParser.cpp",
        "SaiSwitchAsic.cpp",
        "View.cpp",
        "ViewCmp.cpp",
    ],
    hdrs = glob(["*.h"]),
    deps = [
        "//:common_flags",
        "//meta:saimeta",
        "//syncd:Syncd",
    ],
)

cc_binary(
    name = "saiasiccmp",
    srcs = ["main.cpp"],
    dynamic_deps = [
        "//meta:saimeta_shared",
        "@sonic-swss-common//:swsscommon_shared",
        "@libnl3//:libnl_3_shared",
        "@libnl3//:libnl_nf_3_shared",
        "@libnl3//:libnl_route_3_shared",
        "@libyang3//:yang_shared",
    ],
    deps = [":AsicCmp"],
)
