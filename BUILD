package(default_visibility = ["//visibility:public"])
#load("@rules_cc//cc:cc_binary.bzl", "cc_binary")
load("@rules_cc//cc:cc_library.bzl", "cc_library")
#load("@rules_cc//cc:cc_shared_library.bzl", "cc_shared_library")
#load("@rules_cc//cc:cc_test.bzl", "cc_test")
load("@sonic_rules//:shared_library.bzl", "sonic_shared_library", "sonic_shared_library_versioned")
load("@sonic_rules//:sonic_deb.bzl", "sonic_deb")

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

cc_library(
    name = "common_flags",
    cxxopts = CXX_COMMON,
    visibility = ["//visibility:public"],
)

exports_files(["stub.pl"])

# for now we make config value fixed
genrule(
    name = "config_h_gen",
    srcs = [],
    outs = ["config.h"],
    cmd = """
        SAIREDIS_REV=$$(cd $$(dirname $(RULEDIR)) && git rev-parse --short HEAD 2>/dev/null || echo 0000000)
        SAI_REV=$$(cd $$(dirname $(RULEDIR))/SAI && git rev-parse --short HEAD 2>/dev/null || echo 0000000)
        cat > $(OUTS) << 'ENDOFFILE'

#define HAVE_SAI_BULK_OBJECT_CLEAR_STATS 1
#define HAVE_SAI_BULK_OBJECT_GET_STATS 1
#define HAVE_SAI_QUERY_STATS_ST_CAPABILITY 1
#define HAVE_SAI_TAM_TELEMETRY_GET_DATA 1

ENDOFFILE
        printf '#define SAIREDIS_GIT_REVISION "%s"\n' "$$SAIREDIS_REV" >> $(OUTS)
        printf '#define SAI_GIT_REVISION "%s"\n' "$$SAI_REV" >> $(OUTS)
    """,
    visibility = ["//visibility:public"],
)

# cc_library wrapper so targets can depend on config.h via deps
cc_library(
    name = "config_h",
    hdrs = [":config_h_gen"],
    include_prefix = ".",
    visibility = ["//visibility:public"],
)

# SAI header files used by stub.pl to generate sai_vs.cpp / sai_redis.cpp
filegroup(
    name = "sai_headers",
    srcs = glob([
        "SAI/inc/sai*.h",
        "SAI/experimental/sai*.h",
    ]),
    visibility = ["//visibility:public"],
)
# need doxygen,graphviz,aspell
genrule(
    name = "saimeta_gen",
    srcs = glob(["SAI/inc/**/*", "SAI/experimental/**/*", "SAI/meta/**/*", "SAI/custom/**/*"]),
    outs = [
        "SAI/meta/saimetadata.c",
        "SAI/meta/saimetadata.h",
        "SAI/meta/saimetadatatest.c",
        "SAI/meta/saiswig.i",
        "SAI/meta/saiattrversion.h"
    ],
    cmd = """
        METADIR=$$(dirname $(execpath SAI/meta/saimetadatautils.c))
        make -C $$METADIR saimetadata.c &> /dev/null
        rm -rf $$METADIR/xml $$METADIR/html
        mkdir -p $(@D)/SAI/meta
        mv $$METADIR/saimetadata.c $$METADIR/saimetadatatest.c $$METADIR/saiswig.i $(@D)/SAI/meta
        mv $$METADIR/saimetadata.h $$METADIR/saiattrversion.h $(@D)/SAI/meta
        """,
)

filegroup(
    name = "sai_meta_headers",
    srcs = glob(["SAI/meta/sai*.h",]) + [
        "SAI/meta/saimetadata.h",
        "SAI/meta/saiattrversion.h"
    ],
    visibility = ["//visibility:public"],
)

sonic_shared_library_versioned(
    name = "saimetadata",
    srcs = [
        "SAI/meta/saimetadata.c",
        "//:SAI/meta/saimetadatautils.c",
        "//:SAI/meta/saiserialize.c"
    ],
    hdrs = glob(["SAI/inc/*.h", "SAI/experimental/*.h", "SAI/meta/*.h"]) + ["SAI/meta/saimetadata.h"],
    includes = [
        "SAI/inc",
        "SAI/experimental",
        "SAI/meta",
    ],
    soversion = "0",
    version = "0.0.0",
    visibility = ["//visibility:public"],
)

sonic_deb(
    name = "libsaimetadata_1.0.0_amd64.deb",
    architecture = "amd64",
    version = "1.0.0",
    package = "libsaimetadata",
    maintainer = "songqing.sq@alibaba-inc.com",
    description = "This package contains SAI-Metadata implementation for SONiC project.",
    content = {
       "/usr/lib/x86_64-linux-gnu":  ["//meta:saimeta_files", "//:saimetadata_files"],
    },
    changelog = "debian/changelog",
    section = "libs",
)

sonic_deb(
    name = "libsaimetadata-dev_1.0.0_amd64.deb",
    content = {
        "/usr/include/sai:*": [":sai_meta_headers"],
        "/usr/include/sai": ["//meta:saimeta_hdr_files"],
        "/usr/lib/x86_64-linux-gnu" : ["//meta:saimeta_dev_link", "//:saimetadata_dev_link"],
    },
    architecture = "amd64",
    version = "1.0.0",
    package = "libsaimetadata-dev",
    maintainer = "songqing.sq@alibaba-inc.com",
    description = "This package contains development files for SAI-Metadata.",
    changelog = "debian/changelog",
    section = "libdevel",
)

sonic_deb(
    name = "libsairedis_1.0.0_amd64.deb",
    content = {
        "/usr/lib/x86_64-linux-gnu" : ["//lib:sairedis_files"]
    },
    architecture = "amd64",
    version = "1.0.0",
    package = "libsairedis",
    maintainer = "songqing.sq@alibaba-inc.com",
    description = "This package contains SAI-Redis implementation for SONiC project.",
    changelog = "debian/changelog",
    section = "libs",
)

sonic_deb(
    name = "libsairedis-dev_1.0.0_amd64.deb",
    content = {
        "/usr/include/sai" : ["//lib:sairedis.h"],
        "/usr/lib/x86_64-linux-gnu" : ["//lib:sairedis_dev_link"]
    },
    architecture = "amd64",
    version = "1.0.0",
    package = "libsairedis-dev",
    maintainer = "songqing.sq@alibaba-inc.com",
    description = "This package contains development files for SAI-Redis.",
    changelog = "debian/changelog",
    section = "libdevel",
)
sonic_deb(
    name = "syncd_1.0.0_amd64.deb",
    content = {
        "/usr/bin:*:0755": [
            "//syncd:syncd_bin",
            "//syncd:syncd_request_shutdown",
            "//syncd:syncd_tests",
            "//saiasiccmp:saiasiccmp",
            "//saidiscovery:saidiscovery",
            "//saidump:saidump",
            "//saiplayer:saiplayer",
            "//saisdkdump:saisdkdump",
        ],
        "/usr/bin:scripts:0755": ["//syncd:syncd_scripts"],
        "/usr/lib/x86_64-linux-gnu": ["//syncd:MdioIpcClient_files"],
    },
    architecture = "amd64",
    version = "1.0.0",
    package = "syncd",
    maintainer = "songqing.sq@alibaba-inc.com",
    description = "This package contains sync daemon for SONiC project.This sync daemon syncs the ASIC_DB in Redis database and the real ASIC via SAI.",
    changelog = "debian/changelog",
    conflicts = ["syncd-rpc", "syncd-vs"],
)

sonic_deb(
    name = "syncd-vs_1.0.0_amd64.deb",
    content = {
        "/usr/bin:*:0755": [
            "//syncd:syncd_vs_bin",
            "//syncd:syncd_request_shutdown",
            "//syncd:syncd_tests",
            "//saiasiccmp:saiasiccmp",
            "//saidiscovery:saidiscovery",
            "//saidump:saidump",
            "//saiplayer:saiplayer",
            "//saisdkdump:saisdkdump",
        ],
        "/usr/bin:scripts:0755": ["//syncd:syncd_scripts"],
        "/usr/lib/x86_64-linux-gnu": ["//syncd:MdioIpcClient_files"],
    },
    architecture = "amd64",
    version = "1.0.0",
    package = "syncd-vs",
    maintainer = "songqing.sq@alibaba-inc.com",
    description = "This package contains sync daemon for SONiC project.This sync daemon syncs the ASIC_DB in Redis database and the real ASIC via SAI.",
    changelog = "debian/changelog",
    conflicts = ["syncd-rpc", "syncd"],
)

sonic_deb(
    name = "libsaivs_1.0.0_amd64.deb",
    content = {
        "/usr/lib/x86_64-linux-gnu" : ["//vslib:saivs_files"],
    },
    architecture = "amd64",
    version = "1.0.0",
    package = "libsaivs",
    maintainer = "songqing.sq@alibaba-inc.com",
    description = "This package contains SAI-VirtualSwitch implementation for SONiC project.",
    changelog = "debian/changelog",
    section = "libs",
)
sonic_deb(
    name = "libsaivs-dev_1.0.0_amd64.deb",
    content = {
        "/usr/include/sai:SAI/inc" : glob(["SAI/inc/sai*.h"]),
        "/usr/include/sai:SAI/experimental": glob(["SAI/experimental/sai*.h"]),
        "/usr/lib/x86_64-linux-gnu" : ["//vslib:saivs_dev_link"],
    },
    architecture = "amd64",
    version = "1.0.0",
    package = "libsaivs-dev",
    maintainer = "songqing.sq@alibaba-inc.com",
    description = "This package contains development files for SAI-VirtualSwitch.",
    changelog = "debian/changelog",
    section = "libdevel",
)
