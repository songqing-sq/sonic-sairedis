load("@rules_cc//cc:cc_binary.bzl", "cc_binary")
load("@rules_cc//cc:cc_test.bzl", "cc_test")

CXX_COMMON =   [
    "-std=c++14",
    "-Wall",
    "-Wcast-align=strict",
    "-Wcast-qual",
    "-Wconversion",
    "-Wdisabled-optimization",
    "-Werror",
    "-Wextra",
    "-Wfloat-equal",
    "-Wformat=2",
    "-Wformat-nonliteral",
    "-Wformat-security",
    "-Wformat-y2k",
    "-Wimport",
    "-Winit-self",
    "-Wno-inline",
    "-Winvalid-pch",
    "-Wmissing-field-initializers",
    "-Wmissing-format-attribute",
#    "-Wmissing-include-dirs",
    "-Wmissing-noreturn",
    "-Wno-aggregate-return",
    "-Wno-padded",
    "-Wno-switch-enum",
    "-Wno-unused-parameter",
    "-Wpacked",
    "-Wpointer-arith",
    "-Wredundant-decls",
    "-Wshadow",
    "-Wstack-protector",
    "-Wstrict-aliasing=3",
    "-Wswitch",
    "-Wswitch-default",
    "-Wunreachable-code",
    "-Wunused",
    "-Wvariadic-macros",
    "-Wwrite-strings",
    "-Wno-switch-default",
    "-Wconversion",
    "-Wno-psabi",
    "-Wno-cast-function-type"
]

cc_binary(
    name = "vssyncd",
    srcs = ["main.cpp"],
    deps = ["//syncd:Syncd", "//lib:SaiRedis", "//:saimetadata_objs", "//meta:saimeta_objs"],
    dynamic_deps = ["//vslib:sai_vs", "//meta:saimeta", "//:saimetadata", "@sonic-swss-common//:swsscommon"],
    linkopts = ["-lhiredis", "-lswsscommon", "-lpthread", "-ldl", "-lzmq"],
)

cc_test(
    name = "tests",
    srcs = ["tests.cpp"],
    deps = ["//lib:sairedis_objs", "//lib:SaiRedis", "//syncd:Syncd", "//:saimetadata_objs", "//meta:saimeta_objs",
            "@sonic-swss-common//:swsscommon_objs"],
    linkopts = ["-lzmq", "-lpthread", "-lhiredis"],
    cxxopts = CXX_COMMON,
    size = "small",
)
cc_test(
    name = "testclient",
    srcs = ["TestClient.cpp", "testclient_main.cpp"],
    deps = ["//lib:sairedis_objs", "//lib:SaiRedis", "//syncd:Syncd", "//:saimetadata_objs", "//meta:saimeta_objs",
            "@sonic-swss-common//:swsscommon_objs"],
    linkopts = ["-lzmq", "-lpthread", "-lhiredis"],
    cxxopts = CXX_COMMON,
    timeout = "long",
)
cc_test(
    name = "testdash_gtest",
    srcs = ["TestDashMain.cpp", "TestDash.cpp", "TestDashEnv.cpp"],
    deps = ["//lib:sairedis_objs", "//lib:SaiRedis", "//syncd:Syncd", "//:saimetadata_objs", "//meta:saimeta_objs",
            "@sonic-swss-common//:swsscommon_objs",
            "@googletest//:gtest",
            "@googletest//:gtest_main",
            ],
    linkopts = ["-lzmq", "-lpthread", "-lhiredis"],
    cxxopts = CXX_COMMON,
)