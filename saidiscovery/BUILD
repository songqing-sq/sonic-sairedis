load("@rules_cc//cc:cc_binary.bzl", "cc_binary")

package(default_visibility = ["//visibility:public"])

cc_binary(
    name = "saidiscovery",
    srcs = ["saidiscovery.cpp"],
    linkopts = ["-Wl,--enable-new-dtags"],
    dynamic_deps = [
        "//vslib:saivs_shared",
        "//meta:saimeta_shared",
        "@sai//meta:saimetadata_shared",
        "@sonic-swss-common//:swsscommon_shared",
        "@libnl3//:libnl_3_shared",
        "@libnl3//:libnl_nf_3_shared",
        "@libnl3//:libnl_route_3_shared",
        "@libyang3//:yang_shared",
    ],
    deps = [
        "//syncd:Syncd",
        "//vslib:saivs",
        "//meta:saimeta",
    ],
)
