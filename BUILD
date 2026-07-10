load("@rules_cc//cc:cc_library.bzl", "cc_library")
load("@sonic_build_infra//sonic_deb:sonic_deb.bzl", "sonic_deb")

package(default_visibility = ["//visibility:public"])

# Mirrors autotools CXXFLAGS_COMMON from configure.ac. Note -Werror is
# retained so warnings surfaced by newer GCCs fail the build the same way
# the Make/dpkg-buildpackage recipe does.
CXX_COMMON = [
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
    "-Wno-cast-function-type",
]

cc_library(
    name = "common_flags",
    cxxopts = CXX_COMMON,
    visibility = ["//visibility:public"],
)

exports_files(["stub.pl"])

# For now we make config.h values fixed. autotools' configure.ac probes SAI
# for these symbols at configure time; hard-coding them matches Debian-shipped
# libsairedis builds where all four macros are always defined on trixie.
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

# cc_library wrapper so targets can depend on config.h via deps.
cc_library(
    name = "config_h",
    hdrs = [":config_h_gen"],
    include_prefix = ".",
    visibility = ["//visibility:public"],
)

# =============================================================================
# libsaimetadata .deb — corresponds to $(LIBSAIMETADATA) in rules/sairedis.mk.
# Ships both meta/libsaimeta.so.* and SAI/meta/libsaimetadata.so.*, matching
# debian/libsaimetadata.install.
# =============================================================================
sonic_deb(
    name = "libsaimetadata_1.0.0.deb",
    changelog = "debian/changelog",
    content = {
        "${LIBDIR}": [
            "//meta:saimeta_files",
            "@sai//meta:saimetadata_files",
        ],
    },
    depends = [
        "libc6 (>= 2.38)",
        "libgcc-s1 (>= 3.0)",
        "libstdc++6 (>= 13.1)",
        "libswsscommon (>= 1.0.0)",
    ],
    description = "This package contains SAI-Metadata implementation for SONiC project.",
    gen_dbg = True,
    maintainer = "SONiC Maintainers",
    package = "libsaimetadata",
    section = "libs",
    version = "1.0.0",
)

sonic_deb(
    name = "libsaimetadata-dev_1.0.0.deb",
    changelog = "debian/changelog",
    content = {
        "/usr/include/sai:*": ["@sai//meta:meta_hdrs"],
        "/usr/include/sai": ["//meta:saimeta_hdr_files"],
        "${LIBDIR}": [
            "//meta:saimeta_dev_link",
            "@sai//meta:saimetadata_dev_link",
        ],
    },
    depends = ["libsaimetadata (= 1.0.0)"],
    description = "This package contains development files for SAI-Metadata.",
    maintainer = "SONiC Maintainers",
    package = "libsaimetadata-dev",
    section = "libdevel",
    version = "1.0.0",
)

# =============================================================================
# libsairedis .deb — corresponds to $(LIBSAIREDIS) in rules/sairedis.mk.
# Ships lib/libsairedis.so.* per debian/libsairedis.install.
# =============================================================================
sonic_deb(
    name = "libsairedis_1.0.0.deb",
    changelog = "debian/changelog",
    content = {
        "${LIBDIR}": ["//lib:sairedis_files"],
    },
    depends = [
        "libc6 (>= 2.38)",
        "libgcc-s1 (>= 3.0)",
        "libstdc++6 (>= 13.1)",
        "libswsscommon (>= 1.0.0)",
    ],
    description = "This package contains SAI-Redis implementation for SONiC project.",
    gen_dbg = True,
    maintainer = "SONiC Maintainers",
    package = "libsairedis",
    section = "libs",
    version = "1.0.0",
)

sonic_deb(
    name = "libsairedis-dev_1.0.0.deb",
    changelog = "debian/changelog",
    content = {
        "/usr/include/sai:*": ["//lib:sairedis.h"],
        "${LIBDIR}": ["//lib:sairedis_dev_link"],
    },
    depends = [
        "libsairedis (= 1.0.0)",
        "libzmq5-dev",
    ],
    description = "This package contains development files for SAI-Redis.",
    maintainer = "SONiC Maintainers",
    package = "libsairedis-dev",
    section = "libdevel",
    version = "1.0.0",
)

# =============================================================================
# libsaivs .deb — corresponds to $(LIBSAIVS) in rules/sairedis.mk.
# =============================================================================
sonic_deb(
    name = "libsaivs_1.0.0.deb",
    changelog = "debian/changelog",
    content = {
        "${LIBDIR}": ["//vslib:saivs_files"],
    },
    depends = [
        "libc6 (>= 2.38)",
        "libgcc-s1 (>= 3.0)",
        "libsaimetadata (>= 1.0.0)",
        "libstdc++6 (>= 13.1)",
    ],
    description = "This package contains SAI-VirtualSwitch implementation for SONiC project.",
    gen_dbg = True,
    maintainer = "SONiC Maintainers",
    package = "libsaivs",
    section = "libs",
    version = "1.0.0",
)

sonic_deb(
    name = "libsaivs-dev_1.0.0.deb",
    changelog = "debian/changelog",
    content = {
        "/usr/include/sai:*": [
            "@sai//inc:sai_inc_files",
            "@sai//experimental:sai_experimental_files",
            "//vslib:saivs.h",
        ],
        "${LIBDIR}": ["//vslib:saivs_dev_link"],
    },
    depends = ["libsaivs (= 1.0.0)"],
    description = "This package contains development files for SAI-VirtualSwitch.",
    maintainer = "SONiC Maintainers",
    package = "libsaivs-dev",
    section = "libdevel",
    version = "1.0.0",
)

# =============================================================================
# python3-pysairedis .deb — corresponds to $(PYTHON3_PYSAIREDIS) in
# rules/sairedis.mk. Ships _pysairedis.so (SWIG native ext) + pysairedis.py
# under /usr/lib/python3/dist-packages/sairedis/, matching
# debian/python3-pysairedis.install which lists
#   usr/lib/python3/dist-packages/sairedis/*.
# =============================================================================
sonic_deb(
    name = "python3-pysairedis_1.0.0.deb",
    changelog = "debian/changelog",
    content = {
        "/usr/lib/python3/dist-packages/sairedis:*": [
            "//pyext:py3/__init__.py",
            "//pyext:py3/pysairedis.py",
            "//pyext:_pysairedis_files",
            "//pyext:_pysairedis_dev_link_direct",
        ],
    },
    depends = [
        "libc6 (>= 2.14)",
        "libgcc-s1 (>= 3.0)",
        "libpython3.13 (>= 3.13.0~rc3)",
        "libsaimetadata (>= 1.0.0)",
        "libsairedis (>= 1.0.0)",
        "libstdc++6 (>= 13.1)",
        "libswsscommon (>= 1.0.0)",
    ],
    description = "This package contains Switch State Service sairedis Python3 library.",
    gen_dbg = True,
    maintainer = "SONiC Maintainers",
    package = "python3-pysairedis",
    section = "libs",
    version = "1.0.0",
)

# =============================================================================
# syncd-vs .deb — corresponds to $(SYNCD_VS) in platform/vs/syncd-vs.mk.
# Ships syncd binary (linked with vslib), companion tools, scripts, and
# libMdioIpcClient shared library.
# =============================================================================
sonic_deb(
    name = "syncd-vs_1.0.0.deb",
    changelog = "debian/changelog",
    conflicts = [
        "syncd",
        "syncd-rpc",
    ],
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
        "${LIBDIR}": ["//syncd:MdioIpcClient_files"],
    },
    depends = [
        "init-system-helpers (>= 1.54~)",
    ],
    description = "This package contains sync daemon for SONiC project linked with virtual switch.\n  This sync daemon syncs the ASIC_DB in Redis database and the real ASIC via SAI.",
    gen_dbg = True,
    maintainer = "SONiC Maintainers",
    package = "syncd-vs",
    version = "1.0.0",
)
