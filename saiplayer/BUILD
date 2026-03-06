package(default_visibility = ["//visibility:public"])
load("@rules_cc//cc:cc_binary.bzl", "cc_binary")
load("@rules_cc//cc:cc_library.bzl", "cc_library")

cc_library(
    name = "SaiPlayer",
    hdrs = glob(["*.h"]),
    srcs = [
        "CommandLineOptions.cpp",
        "CommandLineOptionsParser.cpp",
        "SaiPlayer.cpp",
    ],
    deps = ["//lib:SaiRedis", "//syncd:Syncd"]
)
cc_binary(
    name = "saiplayer",
    srcs = ["saiplayer_main.cpp"],
    deps = [":SaiPlayer"],
    dynamic_deps = ["//:saimetadata", "//meta:saimeta", "@sonic-swss-common//:swsscommon"],
    linkopts = ["-lhiredis", "-lpthread", "-lzmq"],
)