load("@rules_cc//cc:cc_library.bzl", "cc_library")
load("@sonic_build_infra//shared_library:shared_library.bzl", "sonic_shared_library_versioned")

package(default_visibility = ["//visibility:public"])

# Sources mirror meta/Makefile.am's libsaimeta_la_SOURCES. The public
# sonic-dzf tree carries a few extra source files not present in the internal
# reference (DashMeta.cpp for bypassValidation/shouldCacheAttribute/logDashPolicy,
# plus three Notification* concrete classes referenced from NotificationFactory
# dispatch); include them here so the link stage doesn't leave undefined
# references.
saimeta_SRCS = [
    "AttrKeyMap.cpp",
    "DashMeta.cpp",
    "Globals.cpp",
    "Meta.cpp",
    "MetaKeyHasher.cpp",
    "Notification.cpp",
    "NotificationFactory.cpp",
    "NotificationFdbEvent.cpp",
    "NotificationFlowBulkGetSessionEvent.cpp",
    "NotificationMacsecPostStatus.cpp",
    "NotificationNatEvent.cpp",
    "NotificationPortStateChange.cpp",
    "NotificationQueuePfcDeadlock.cpp",
    "NotificationSwitchAsicSdkHealthEvent.cpp",
    "NotificationSwitchMacsecPostStatus.cpp",
    "NotificationSwitchShutdownRequest.cpp",
    "NotificationSwitchStateChange.cpp",
    "NotificationBfdSessionStateChange.cpp",
    "NotificationIcmpEchoSessionStateChange.cpp",
    "NotificationHaSetEvent.cpp",
    "NotificationHaScopeEvent.cpp",
    "NotificationTwampSessionEvent.cpp",
    "NotificationPortHostTxReadyEvent.cpp",
    "NotificationTamTelTypeConfigChange.cpp",
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
    "DummySaiInterface.cpp",
    "ZeroMQSelectableChannel.cpp",
]

sonic_shared_library_versioned(
    name = "saimeta",
    srcs = saimeta_SRCS + glob(["*.h"]),
    hdrs = glob([
        "sai*.h",
        "Sai*.h",
        "Notification*.h",
    ]) + [
        "AttrKeyMap.h",
        "Globals.h",
        "Meta.h",
        "MetaKeyHasher.h",
        "OidRefCounter.h",
        "PortRelatedSet.h",
    ],
    cxxopts = ["-Wno-stack-protector"],
    dynamic_deps = [
        "@sai//meta:saimetadata_shared",
        "@sonic-swss-common//:swsscommon_shared",
    ],
    includes = ["."],
    soversion = "0",
    version = "0.0.0",
    visibility = ["//visibility:public"],
    # OidIndexGenerator/VirtualObjectIdManager are needed to compile saimeta
    # itself but are NOT part of its public ABI — keep them private so the
    # full lib/ include dir doesn't propagate and shadow vslib's same-named
    # headers in downstream consumers.
    implementation_deps = [
        "//lib:sairedis_hdrs",
    ],
    deps = [
        "//:common_flags",
        "@sai//meta:saimetadata",
        # sai_serialize.h (public) #include "sairedis.h" — this is true
        # interface leak, so it goes in normal deps (and is intentionally
        # scoped to just that one header, not the whole lib/).
        "//lib:sairedis_h",
        "@sairedis_deps//libhiredis-dev:libhiredis",
        "@sairedis_deps//libzmq3-dev:libzmq3",
        "@sonic-swss-common//:swsscommon",
        "@sonic_build_infra//system_libs:pthread",
    ],
)
