package(default_visibility = ["//visibility:public"])

load("@rules_cc//cc:cc_binary.bzl", "cc_binary")

cc_binary(
    name = "saidiscovery",
    srcs = ["saidiscovery.cpp"],
    linkopts = ["-Wl,--enable-new-dtags"],
    deps = ["//syncd:Syncd", "//syncd:dummy_sai_objs", "//meta:saimeta_objs"],
    dynamic_deps = ["//syncd:dummy_sai", "//meta:saimeta", "//:saimetadata", "@sonic-swss-common//:swsscommon"],
)