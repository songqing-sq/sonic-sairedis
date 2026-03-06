package(default_visibility = ["//visibility:public"])

load("@rules_cc//cc:cc_binary.bzl", "cc_binary")
load("@rules_cc//cc:cc_library.bzl", "cc_library")
load("@sonic_rules//:shared_library.bzl", "sonic_shared_library", "sonic_shared_library_versioned")


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
    deps = ["//meta:saimeta_objs", "//syncd:Syncd", "//:common_flags"],
    hdrs = glob(["*.h"]),
)
cc_binary(
    name = "saiasiccmp",
    srcs = ["main.cpp"],
    deps = [":AsicCmp"],
    dynamic_deps = ["//meta:saimeta", "@sonic-swss-common//:swsscommon"],
)
