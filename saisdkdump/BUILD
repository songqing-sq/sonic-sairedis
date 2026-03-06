package(default_visibility = ["//visibility:public"])
load("@rules_cc//cc:cc_binary.bzl", "cc_binary")

cc_binary(
    name = "saisdkdump",
    srcs = ["saisdkdump.cpp"],
    linkopts = ["-Wl,--enable-new-dtags"],
    deps = ["//syncd:dummy_sai_objs", "//meta:saimeta_objs", "@sonic-swss-common//:swsscommon_objs"],
    dynamic_deps = ["//syncd:dummy_sai", "//meta:saimeta", "//:saimetadata", "@sonic-swss-common//:swsscommon"],
)