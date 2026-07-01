load("@rules_cc//cc:cc_library.bzl", "cc_library")
load("@sonic_build_infra//shared_library:shared_library.bzl", "sonic_shared_library_versioned")

package(default_visibility = ["//visibility:public"])

# Sources mirror lib/Makefile.am's libSaiRedis_la_SOURCES (autotools default
# libtool version 0:0:0 -> libsairedis.so.0.0.0 / soname libsairedis.so.0).
SaiRedis_SRCS = [
    "Channel.cpp",
    "ClientConfig.cpp",
    "ClientSai.cpp",
    "ClientServerSai.cpp",
    "Context.cpp",
    "ContextConfig.cpp",
    "ContextConfigContainer.cpp",
    "Recorder.cpp",
    "RedisChannel.cpp",
    "RedisRemoteSaiInterface.cpp",
    "RedisVidIndexGenerator.cpp",
    "Sai.cpp",
    "ServerConfig.cpp",
    "ServerSai.cpp",
    "SkipRecordAttrContainer.cpp",
    "Switch.cpp",
    "SwitchConfig.cpp",
    "SwitchConfigContainer.cpp",
    "SwitchContainer.cpp",
    "Utils.cpp",
    "VirtualObjectIdManager.cpp",
    "ZeroMQChannel.cpp",
]

# stub.pl emits lib/sai_redis.cpp — the entry point that dlopens the vendor
# SAI shared library. The Makefile.am recipe runs it via SUBDIRS from
# top-level; we replicate it here as a hermetic genrule.
genrule(
    name = "sai_redis_gen",
    srcs = [
        "//:stub.pl",
        # SAI inc + experimental sai*.h: stub.pl reads $SAIDIR/{inc,experimental}/sai*.h.
        "@sai//:sai_files",
    ],
    outs = ["sai_redis.cpp"],
    cmd = """
        STUBPL=$(execpath //:stub.pl)
        # @sai//:sai_files lands at external/sai+/SAI/{inc,experimental}/sai*.h;
        # SAIDIR is two dirnames above any of those files.
        ANY=$$(echo "$(execpaths @sai//:sai_files)" | awk '{print $$1}')
        SAIDIR=$$(dirname $$(dirname $$ANY))
        perl $$STUBPL -d $$SAIDIR -c ClientServerSai -n sairedis -f $(OUTS) -s stub &> /dev/null
    """,
)

# Object-code layer used by downstream binaries (syncd, saiplayer, ...) that
# need to statically link against the SaiRedis implementation. The shared-lib
# form below re-exports these symbols via `exports_filter`.
cc_library(
    name = "SaiRedis",
    srcs = SaiRedis_SRCS,
    hdrs = glob(["*.h"]),
    cxxopts = ["-Wno-missing-field-initializers"],
    visibility = ["//visibility:public"],
    deps = [
        ":sairedis_hdrs",
        "//:common_flags",
        "//:config_h",
        "//meta:saimeta",
        "@sairedis_deps//libhiredis-dev:libhiredis",
        "@sairedis_deps//libzmq3-dev:libzmq3",
        "@sonic-swss-common//:swsscommon",
        "@sonic_build_infra//system_libs:pthread",
    ],
)

sonic_shared_library_versioned(
    name = "sairedis",
    srcs = [":sai_redis_gen"],
    hdrs = glob(["*.h"]),
    cxxopts = ["-Wno-missing-field-initializers"],
    dynamic_deps = [
        "//meta:saimeta_shared",
        "@sai//meta:saimetadata_shared",
        "@sonic-swss-common//:swsscommon_shared",
    ],
    # SaiRedis is a transitive static dep of :sairedis (via :sairedis -> :SaiRedis).
    # syncd/BUILD targets depend on //lib:SaiRedis directly as well; without
    # declaring it exported, cc_shared_library refuses to let downstream link
    # both forms. Matches autotools: libsairedis.so exports the full SaiRedis
    # symbol set, consumers resolve at runtime.
    exports_filter = ["//lib:SaiRedis"],
    includes = ["."],
    soversion = "0",
    version = "0.0.0",
    visibility = ["//visibility:public"],
    deps = [
        ":SaiRedis",
        "//:common_flags",
    ],
)

# Minimal interface header: pure constants + extern "C" wrapping of sai.h.
# saimeta's public header meta/sai_serialize.h does #include "sairedis.h",
# so this needs to propagate to saimeta's consumers — but exposing the full
# lib/ dir via :sairedis_hdrs would also leak ContextConfigContainer.h etc.
# and shadow the same-named vslib headers. Keep this library tightly scoped.
cc_library(
    name = "sairedis_h",
    hdrs = ["sairedis.h"],
    includes = ["."],
    visibility = ["//visibility:public"],
    deps = ["@sai//meta:saimetadata"],
)

exports_files(
    ["sairedis.h"],
    visibility = ["//visibility:public"],
)
