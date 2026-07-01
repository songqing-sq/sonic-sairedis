load("@rules_cc//cc:cc_library.bzl", "cc_library")
load("@sonic_build_infra//shared_library:shared_library.bzl", "sonic_shared_library_versioned")

package(default_visibility = ["//visibility:public"])

# Sources mirror vslib/Makefile.am's libsaivs_la_SOURCES.
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
    "SwitchNokiaVS.cpp",
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
    cxxopts = ["-Wno-stack-protector"],
    includes = ["."],
    linkopts = ["-pthread"],
    visibility = ["//visibility:public"],
    deps = [
        "//:common_flags",
        "//meta:saimeta",
        "@libnl3//:libnl_3",
        "@libnl3//:libnl_route_3",
        "@sonic-swss-common//:swsscommon",
    ],
)

genrule(
    name = "sai_vs_gen",
    srcs = [
        "//:stub.pl",
        # See lib/BUILD :sai_redis_gen for the SAIDIR derivation rationale.
        "@sai//:sai_files",
    ],
    outs = ["sai_vs.cpp"],
    cmd = """
        STUBPL=$(execpath //:stub.pl)
        ANY=$$(echo "$(execpaths @sai//:sai_files)" | awk '{print $$1}')
        SAIDIR=$$(dirname $$(dirname $$ANY))
        perl $$STUBPL -d $$SAIDIR -c Sai -n saivs -f $(OUTS) -s stub &> /dev/null
    """,
)

sonic_shared_library_versioned(
    name = "saivs",
    srcs = [":sai_vs_gen"],
    hdrs = glob(["*.h"]),
    dynamic_deps = [
        "//meta:saimeta_shared",
        "@libnl3//:libnl_3_shared",
        "@libnl3//:libnl_nf_3_shared",
        "@libnl3//:libnl_route_3_shared",
        "@libyang3//:yang_shared",
        "@sai//meta:saimetadata_shared",
        "@sonic-swss-common//:swsscommon_shared",
    ],
    includes = ["."],
    soversion = "0",
    version = "0.0.0",
    visibility = ["//visibility:public"],
    deps = [
        ":SaiVS",
        "@sai//meta:saimetadata",
        "//meta:saimeta",
        "@sairedis_deps//libhiredis-dev:libhiredis",
    ],
)

exports_files(
    ["saivs.h"],
    visibility = ["//visibility:public"],
)
