package(default_visibility = ["//visibility:public"])

load("@rules_cc//cc:cc_binary.bzl", "cc_binary")
load("@rules_cc//cc:cc_library.bzl", "cc_library")
load("@sonic_rules//:shared_library.bzl", "sonic_shared_library", "sonic_shared_library_versioned")
load("@rules_cc//cc:cc_test.bzl", "cc_test")


Syncd_SRCS = [
        "AsicOperation.cpp",
        "AsicView.cpp",
        "AttrVersionChecker.cpp",
        "BestCandidateFinder.cpp",
        "BreakConfig.cpp",
        "BreakConfigParser.cpp",
        "CommandLineOptions.cpp",
        "CommandLineOptionsParser.cpp",
        "ComparisonLogic.cpp",
        "DisabledRedisClient.cpp",
        "FlexCounter.cpp",
        "FlexCounterManager.cpp",
        "GlobalSwitchId.cpp",
        "HardReiniter.cpp",
        "MdioIpcServer.cpp",
        "MetadataLogger.cpp",
        "NotificationHandler.cpp",
        "NotificationProcessor.cpp",
        "NotificationQueue.cpp",
        "PortMap.cpp",
        "PortMapParser.cpp",
        "PortStateChangeHandler.cpp",
        "RedisClient.cpp",
        "RedisNotificationProducer.cpp",
        "RequestShutdownCommandLineOptions.cpp",
        "SaiAttr.cpp",
        "SaiDiscovery.cpp",
        "SaiObj.cpp",
        "SaiSwitch.cpp",
        "SaiSwitchInterface.cpp",
        "ServiceMethodTable.cpp",
        "SingleReiniter.cpp",
        "SwitchNotifications.cpp",
        "Syncd.cpp",
        "TimerWatchdog.cpp",
        "VendorSai.cpp",
        "VidManager.cpp",
        "VirtualOidTranslator.cpp",
        "WarmRestartTable.cpp",
        "WatchdogScope.cpp",
        "Workaround.cpp",
        "ZeroMQNotificationProducer.cpp",
        "syncd_main.cpp",
]

sonic_shared_library_versioned(
    name = "dummy_sai",
    srcs = ["dummy_sai.c"],
    deps = ["//:saimetadata_objs"],
    dynamic_deps = ["//:saimetadata"],
    soversion = "0",
    version = "0.0.0",
    output_name = "libsai",
)

# need vslib/saivs.h
cc_library(
    name = "Syncd",
    srcs = Syncd_SRCS,
    hdrs = glob(["*.h"]),
    includes = ["."],
    deps = ["//lib:SaiRedis", "//:saimetadata_objs", "//vslib:saivs_hdrs", "//:common_flags"],
    cxxopts = ["-Wno-stack-protector", "-Wno-format-nonliteral"],

)

cc_binary(
    name = "syncd",
    srcs = ["main.cpp"],
    deps = [":Syncd", "//lib:SaiRedis", "//:common_flags"],
    dynamic_deps = ["//meta:saimeta", ":dummy_sai", "@sonic-swss-common//:swsscommon"],
    linkopts = ["-ldl", "-lhiredis", "-lzmq", "-lpthread"],
)

# Rename for packaging: both syncd and syncd-vs install as /usr/bin/syncd
genrule(
    name = "syncd_bin",
    srcs = [":syncd"],
    outs = ["real/syncd"],
    cmd = "cp $(execpath :syncd) $(OUTS)",
)

cc_binary(
    name = "syncd-vs",
    srcs = ["main.cpp"],
    deps = [":Syncd", "//lib:SaiRedis", "//:common_flags", "//vslib:saivs_objs"],
    dynamic_deps = ["//meta:saimeta", "//vslib:saivs", "@sonic-swss-common//:swsscommon"],
    linkopts = ["-ldl", "-lhiredis", "-lzmq", "-lpthread"],
)

genrule(
    name = "syncd_vs_bin",
    srcs = [":syncd-vs"],
    outs = ["vs/syncd"],
    cmd = "cp $(execpath :syncd-vs) $(OUTS)",
)

cc_library(
    name = "SyncdRequestShutdown",
    srcs = [
        "RequestShutdown.cpp",
        "RequestShutdownCommandLineOptions.cpp",
        "RequestShutdownCommandLineOptionsParser.cpp",
    ],
    hdrs = glob(["RequestShutdown*.h"]),
    deps = ["//lib:SaiRedis"],
)

cc_binary(
    name = "syncd_request_shutdown",
    srcs = ["syncd_request_shutdown.cpp"],
    deps = ["SyncdRequestShutdown"],
    dynamic_deps = ["@sonic-swss-common//:swsscommon"],
    linkopts = ["-lhiredis", "-lpthread"],
)

sonic_shared_library_versioned(
    name = "MdioIpcClient",
    srcs = ["MdioIpcClient.cpp"],
    hdrs = ["MdioIpcClient.h", "MdioIpcCommon.h"],
    deps = ["//lib:SaiRedis"],
    soversion = "0",
    version = "0.0.0",
)

cc_binary(
    name = "syncd_tests",
    srcs = ["tests.cpp"],
    deps = [
        ":Syncd",
        "//lib:sairedis_objs",
        "@sonic-swss-common//:swsscommon_objs",
    ],
    dynamic_deps = ["@sonic-swss-common//:swsscommon"],
    linkopts = ["-lhiredis", "-lpthread", "-lzmq"],
)
filegroup(
    name = "syncd_scripts",
    srcs = glob(["scripts/*"]),
)