load("@rules_cc//cc:cc_binary.bzl", "cc_binary")
load("@rules_cc//cc:cc_library.bzl", "cc_library")
load("@sonic_build_infra//shared_library:shared_library.bzl", "sonic_shared_library_versioned")

package(default_visibility = ["//visibility:public"])

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
    "FlowDump.cpp",
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
    "SelectablesTracker.cpp",
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
    dynamic_deps = ["@sai//meta:saimetadata_shared"],
    output_name = "libsai",
    soversion = "0",
    version = "0.0.0",
    deps = ["@sai//meta:saimetadata"],
)

cc_library(
    name = "Syncd",
    srcs = Syncd_SRCS,
    hdrs = glob(["*.h"]),
    copts = [
        "-Wno-stack-protector",
        "-Wno-format-nonliteral",
        "-Wno-error=format",
    ],
    includes = ["."],
    deps = [
        "//:common_flags",
        "@sai//meta:saimetadata",
        "//lib:SaiRedis",
        "//vslib:saivs_hdrs",
        "@sairedis_deps//zlib1g-dev:zlib1g",
    ],
)

cc_binary(
    name = "syncd-vs",
    srcs = ["main.cpp"],
    copts = [
        "-Wno-stack-protector",
        "-Wno-format-nonliteral",
        "-Wno-error=format",
    ],
    linkopts = ["-rdynamic"],
    dynamic_deps = [
        "//meta:saimeta_shared",
        "//vslib:saivs_shared",
        "@sonic-swss-common//:swsscommon_shared",
        "@libnl3//:libnl_3_shared",
        "@libnl3//:libnl_nf_3_shared",
        "@libnl3//:libnl_route_3_shared",
        "@libyang3//:yang_shared",
    ],
    deps = [
        ":Syncd",
        "//:common_flags",
        "//lib:SaiRedis",
        "//vslib:saivs",
        "@sairedis_deps//libhiredis-dev:libhiredis",
        "@sairedis_deps//libzmq3-dev:libzmq3",
        "@sonic_build_infra//system_libs:dl",
        "@sonic_build_infra//system_libs:pthread",
    ],
)

genrule(
    name = "syncd_vs_bin",
    srcs = [":syncd-vs"],
    outs = ["vs/syncd"],
    cmd = "cp $(execpath :syncd-vs) $(OUTS)",
)

# syncd_dash — the DASH-SAI-linked syncd (Make: syncd/Makefile.am `syncd_dash`,
# built when INCLUDE_VS_DASH_SAI=y). Same main.cpp/Syncd sources as syncd-vs, but
# the SAI implementation is the DASH libsai.so (@dash-sai//:libsai_shared), loaded
# at runtime instead of the statically-linked vslib saivs. libsai.so is built
# allow_undefined=True and leaves p4::v1 (P4Runtime) / protobuf / gpr symbols
# undefined; Make resolves them via `-lsai -lprotobuf -lpiprotobuf -lpiprotogrpc
# -lgrpc++` (Makefile.am:133). We mirror that: link @p4lang's libpiprotobuf /
# libpiprotogrpc (cc_import); their transitive protobuf/grpc/gpr/absl deps are the
# Debian apt libs already in the image. -Wl,--allow-shlib-undefined lets those
# transitive symbols resolve at runtime (matching Make, which also relies on the
# dynamic loader). syncd/scripts/syncd_init_common.sh prefers /usr/bin/syncd_dash.
cc_binary(
    name = "syncd_dash_elf",
    srcs = ["main.cpp"],
    copts = [
        "-Wno-stack-protector",
        "-Wno-format-nonliteral",
        "-Wno-error=format",
    ],
    linkopts = [
        "-rdynamic",
        "-Wl,--allow-shlib-undefined",
    ],
    dynamic_deps = [
        "//meta:saimeta_shared",
        "@dash-sai//:libsai_shared",
        "@sonic-swss-common//:swsscommon_shared",
        "@libnl3//:libnl_3_shared",
        "@libnl3//:libnl_nf_3_shared",
        "@libnl3//:libnl_route_3_shared",
        "@libyang3//:yang_shared",
    ],
    deps = [
        ":Syncd",
        "//:common_flags",
        "//lib:SaiRedis",
        "@p4lang//:libpiprotobuf",
        "@p4lang//:libpiprotogrpc",
        "@sairedis_deps//libhiredis-dev:libhiredis",
        "@sairedis_deps//libzmq3-dev:libzmq3",
        "@sonic_build_infra//system_libs:dl",
        "@sonic_build_infra//system_libs:pthread",
    ],
)

genrule(
    name = "syncd_dash",
    srcs = [":syncd_dash_elf"],
    outs = ["dash/syncd_dash"],
    cmd = "cp $(execpath :syncd_dash_elf) $(OUTS)",
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
    dynamic_deps = [
        "@sonic-swss-common//:swsscommon_shared",
        "@libnl3//:libnl_3_shared",
        "@libnl3//:libnl_nf_3_shared",
        "@libnl3//:libnl_route_3_shared",
        "@libyang3//:yang_shared",
    ],
    deps = [
        "SyncdRequestShutdown",
        "@sairedis_deps//libhiredis-dev:libhiredis",
        "@sonic_build_infra//system_libs:pthread",
    ],
)

sonic_shared_library_versioned(
    name = "MdioIpcClient",
    srcs = ["MdioIpcClient.cpp"],
    hdrs = [
        "MdioIpcClient.h",
        "MdioIpcCommon.h",
    ],
    copts = ["-Wno-format-truncation"],
    dynamic_deps = ["@sonic-swss-common//:swsscommon_shared"],
    includes = ["."],
    soversion = "0",
    version = "0.0.0",
    deps = [
        "@sai//:sai_hdrs",
        "@sonic-swss-common//:swsscommon",
    ],
)

cc_binary(
    name = "syncd_tests",
    srcs = ["tests.cpp"],
    dynamic_deps = [
        "@sonic-swss-common//:swsscommon_shared",
        "@libnl3//:libnl_3_shared",
        "@libnl3//:libnl_nf_3_shared",
        "@libnl3//:libnl_route_3_shared",
        "@libyang3//:yang_shared",
    ],
    deps = [
        ":Syncd",
        "//lib:sairedis",
        "@sairedis_deps//libhiredis-dev:libhiredis",
        "@sairedis_deps//libzmq3-dev:libzmq3",
        "@sonic-swss-common//:swsscommon",
        "@sonic_build_infra//system_libs:pthread",
    ],
)

filegroup(
    name = "syncd_scripts",
    srcs = glob(["scripts/*"]),
)
