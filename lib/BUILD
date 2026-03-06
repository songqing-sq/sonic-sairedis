load("@rules_cc//cc:cc_library.bzl", "cc_library")
load("@sonic_rules//:shared_library.bzl", "sonic_shared_library", "sonic_shared_library_versioned")


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

genrule(
    name = "sai_redis_gen",
    srcs = [
        "//:stub.pl",
        "//:sai_headers",       # SAI/inc/sai*.h + SAI/experimental/sai*.h
        "//:saimeta_gen",       # provides SAI/meta/saimetadata.c (ensures saimeta_gen has run)
    ],
    outs = ["sai_redis.cpp"],
    cmd = """
        STUBPL=$(execpath //:stub.pl)
        SAIDIR=$$(dirname $$STUBPL)/SAI
        perl $$STUBPL -d $$SAIDIR -c ClientServerSai -n sairedis -f $(OUTS) -s stub &> /dev/null
    """,
)

cc_library(
    name = "SaiRedis",
    srcs =  SaiRedis_SRCS,
    hdrs = glob(["*.h"]),
    deps = ["//:config_h", "//meta:saimeta_objs", ":sairedis_hdrs", "@sonic-swss-common//:swsscommon_objs", "//:common_flags"],
    cxxopts =  ["-Wno-missing-field-initializers"],
    linkopts = ["-lhiredis", "-lzmq", "-lpthread"],
    visibility = ["//visibility:public"],
)

sonic_shared_library_versioned(
    name = "sairedis",
    srcs = [":sai_redis_gen"],
    hdrs = glob(["*.h"]),
    includes = ["."],
    deps = [":SaiRedis", "//:common_flags"],
    cxxopts = ["-Wno-missing-field-initializers"],
    dynamic_deps = ["//meta:saimeta", "//:saimetadata", "@sonic-swss-common//:swsscommon"],
    soversion = "0",
    version = "0.0.0",
    visibility = ["//visibility:public"],
)

exports_files(
    ["sairedis.h"],
    visibility = ["//visibility:public"],
)