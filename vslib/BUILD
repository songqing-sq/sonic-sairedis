package(default_visibility = ["//visibility:public"])
load("@rules_cc//cc:cc_binary.bzl", "cc_binary")
load("@rules_cc//cc:cc_library.bzl", "cc_library")
load("@sonic_rules//:shared_library.bzl", "sonic_shared_library_versioned")


SaiVS_SOURCES = [
        "Buffer.cpp",
        "ContextConfigContainer.cpp",
        "ContextConfig.cpp",
        "Context.cpp",
        "CorePortIndexMapContainer.cpp",
        "CorePortIndexMap.cpp",
        "CorePortIndexMapFileParser.cpp",
        "Event.cpp",
        "EventPayloadNetLinkMsg.cpp",
        "EventPayloadNotification.cpp",
        "EventPayloadPacket.cpp",
        "EventQueue.cpp",
        "FdbInfo.cpp",
        "HostInterfaceInfo.cpp",
        "LaneMapContainer.cpp",
        "LaneMap.cpp",
        "LaneMapFileParser.cpp",
        "MACsecAttr.cpp",
        "MACsecFilterStateGuard.cpp",
        "MACsecEgressFilter.cpp",
        "MACsecFilter.cpp",
        "MACsecForwarder.cpp",
        "MACsecIngressFilter.cpp",
        "MACsecManager.cpp",
        "NetMsgRegistrar.cpp",
        "RealObjectIdManager.cpp",
        "ResourceLimiterContainer.cpp",
        "ResourceLimiter.cpp",
        "ResourceLimiterParser.cpp",
        "SaiAttrWrap.cpp",
        "Sai.cpp",
        "SaiEventQueue.cpp",
        "SaiFdbAging.cpp",
        "SaiUnittests.cpp",
        "SelectableFd.cpp",
        "Signal.cpp",
        "SwitchBCM56850.cpp",
        "SwitchBCM56971B0.cpp",
        "SwitchBCM81724.cpp",
        "SwitchConfigContainer.cpp",
        "SwitchConfig.cpp",
        "SwitchContainer.cpp",
        "Switch.cpp",
        "SwitchMLNX2700.cpp",
        "SwitchNvdaMBF2H536C.cpp",
        "SwitchStateBase.cpp",
        "SwitchStateBaseFdb.cpp",
        "SwitchStateBaseHostif.cpp",
        "SwitchStateBaseMACsec.cpp",
        "SwitchState.cpp",
        "TrafficFilterPipes.cpp",
        "TrafficForwarder.cpp",
        "VirtualSwitchSaiInterface.cpp",
        "VirtualSwitchSaiInterfaceFdb.cpp",
        "VirtualSwitchSaiInterfacePort.cpp",
]

cc_library(
    name = "SaiVS",
    srcs = SaiVS_SOURCES,
    hdrs = glob(["*.h"]),
    includes = ["."],
    deps = ["//meta:saimeta_objs",  "@sonic-swss-common//:swsscommon_objs", "//:common_flags"],
    cxxopts = ["-Wno-stack-protector", "-I/usr/include/libnl3"],
    linkopts = ["-lnl-route-3"],
    visibility = ["//visibility:public"],
)

genrule(
    name = "sai_vs_gen",
    srcs = [
        "//:stub.pl",
        "//:sai_headers",       # SAI/inc/sai*.h + SAI/experimental/sai*.h
        "//:saimeta_gen",       # provides SAI/meta/saimetadata.c (ensures saimeta_gen has run)
    ],
    outs = ["sai_vs.cpp"],
    cmd = """
        STUBPL=$(execpath //:stub.pl)
        SAIDIR=$$(dirname $$STUBPL)/SAI
        perl $$STUBPL -d $$SAIDIR -c Sai -n saivs -f $(OUTS) -s stub &> /dev/null
    """,
)

sonic_shared_library_versioned(
    name = "saivs",
    srcs = [":sai_vs_gen"],
    hdrs = glob(["*.h"]),
    includes = ["."],
    deps = [":SaiVS", "//meta:saimeta_objs", "//:saimetadata_objs"],
    dynamic_deps = ["//meta:saimeta", "//:saimetadata", "@sonic-swss-common//:swsscommon"],
    linkopts = ["-lhiredis", "-lnl-route-3"],
    soversion = "0",
    version = "0.0.0",
    visibility = ["//visibility:public"],
)

# mirrors Makefile.am: bin_PROGRAMS = tests
# tests_LDADD = -lhiredis -lswsscommon -lpthread libsaivs.la -lsaimetadata -lsaimeta -lzmq
cc_binary(
    name = "tests",
    srcs = ["tests.cpp"],
    deps = [":SaiVS", "//meta:saimeta_objs", "//:saimetadata_objs"],
    dynamic_deps = [":saivs", "//meta:saimeta", "//:saimetadata", "@sonic-swss-common//:swsscommon"],
    cxxopts = ["-Wno-stack-protector", "-I/usr/include/libnl3"],
    linkopts = ["-lhiredis", "-lpthread", "-lzmq"],
    visibility = ["//visibility:public"],
)
