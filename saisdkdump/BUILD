load("@rules_cc//cc:cc_binary.bzl", "cc_binary")

package(default_visibility = ["//visibility:public"])

cc_binary(
    name = "saisdkdump",
    srcs = ["saisdkdump.cpp"],
    linkopts = ["-Wl,--enable-new-dtags"],
    dynamic_deps = [
        "//syncd:dummy_sai_shared",
        "//meta:saimeta_shared",
        "@sai//meta:saimetadata_shared",
        "@sonic-swss-common//:swsscommon_shared",
        "@libnl3//:libnl_3_shared",
        "@libnl3//:libnl_nf_3_shared",
        "@libnl3//:libnl_route_3_shared",
        "@libyang3//:yang_shared",
    ],
    deps = [
        "//syncd:dummy_sai",
        "//meta:saimeta",
        "@sonic-swss-common//:swsscommon",
    ],
)
