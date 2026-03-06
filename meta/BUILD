load("@rules_cc//cc:cc_library.bzl", "cc_library")
load("@sonic_rules//:shared_library.bzl", "sonic_shared_library", "sonic_shared_library_versioned")


saimeta_SRCS = [
        "AttrKeyMap.cpp",
        "DummySaiInterface.cpp",
        "Globals.cpp",
        "Meta.cpp",
        "MetaKeyHasher.cpp",
        "Notification.cpp",
        "NotificationFactory.cpp",
        "NotificationFdbEvent.cpp",
        "NotificationNatEvent.cpp",
        "NotificationPortStateChange.cpp",
        "NotificationQueuePfcDeadlock.cpp",
        "NotificationSwitchAsicSdkHealthEvent.cpp",
        "NotificationSwitchShutdownRequest.cpp",
        "NotificationSwitchStateChange.cpp",
        "NotificationBfdSessionStateChange.cpp",
        "NotificationIcmpEchoSessionStateChange.cpp",
        "NotificationHaSetEvent.cpp",
        "NotificationHaScopeEvent.cpp",
        "NotificationTwampSessionEvent.cpp",
        "NotificationPortHostTxReadyEvent.cpp",
        "NotificationTamTelTypeConfigChange.cpp",
        "NotificationSwitchMacsecPostStatus.cpp",
        "NotificationMacsecPostStatus.cpp",
        "NumberOidIndexGenerator.cpp",
        "OidRefCounter.cpp",
        "PerformanceIntervalTimer.cpp",
        "PortRelatedSet.cpp",
        "RedisSelectableChannel.cpp",
        "SaiAttrWrapper.cpp",
        "SaiAttributeList.cpp",
        "SaiInterface.cpp",
        "SaiObject.cpp",
        "SaiObjectCollection.cpp",
        "SaiSerialize.cpp",
        "SelectableChannel.cpp",
        "ZeroMQSelectableChannel.cpp",
]

sonic_shared_library_versioned(
    name = "saimeta",
    srcs = saimeta_SRCS + glob(["*.h"]),
    hdrs = glob(["sai*.h", "Sai*.h", "Notification*.h"]) +
    [
        "Meta.h",
        "AttrKeyMap.h",
        "MetaKeyHasher.h",
        "OidRefCounter.h",
        "PortRelatedSet.h",
        "Globals.h",
    ],
    includes = ["."],
    # saimeta depends on some headers from lib
    deps = ["//:saimetadata_objs", "//lib:sairedis_hdrs", "@sonic-swss-common//:swsscommon_objs", "//:common_flags"],
    cxxopts = ["-Wno-stack-protector"],
    dynamic_deps = ["//:saimetadata", "@sonic-swss-common//:swsscommon"],
    linkopts = ["-lhiredis", "-lzmq", "-lpthread"],
    soversion = "0",
    version = "0.0.0",
    visibility = ["//visibility:public"],
)

cc_library(
    name = "meta_testlib",
    srcs = [
        "DummySaiInterface.cpp",
        "MetaTestSaiInterface.cpp",
    ] + glob(["*.h"]),
    deps = ["//:saimetadata_objs", "@sonic-swss-common//:swsscommon_objs", "//lib:sairedis_hdrs", "//:common_flags"],
    visibility = ["//visibility:public"],
)
