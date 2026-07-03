load("@bazel_lib//lib:expand_template.bzl", "expand_template_rule")
load("@doxygen//:doxygen.bzl", "doxygen")
load("@rules_cc//cc:cc_library.bzl", "cc_library")
load("@rules_perl//perl:perl.bzl", "perl_binary", "perl_library")
load("@rules_shell//shell:sh_binary.bzl", "sh_binary")
load("@sonic_build_infra//shared_library:shared_library.bzl", "sonic_shared_library_versioned")

package(default_visibility = ["//visibility:public"])

# =============================================================================
# SAI upstream tarball (opencomputeproject/SAI v1.18.1) — single root BUILD.
#
# All targets live in this root package so that the wrapper module
# (src/sonic-sairedis/SAI-bazel) can consume everything through one
# build_file_template as-provided by the stock sonic_http_archive rule.  The
# wrapper's own per-subpackage alias BUILDs re-export these root labels as
# @sai//inc:..., @sai//experimental:..., @sai//meta:... to preserve the label
# contract downstream consumers use.
# =============================================================================

# ---- inc/ (SAI public headers) ----------------------------------------------

filegroup(
    name = "sai_inc_files",
    srcs = glob(["inc/*.h"]),
)

cc_library(
    name = "sai_inc",
    hdrs = [":sai_inc_files"],
    includes = ["inc"],
)

# ---- experimental/ (SAI experimental headers, incl. DASH) -------------------

filegroup(
    name = "sai_experimental_files",
    srcs = glob(["experimental/*.h"]),
)

cc_library(
    name = "sai_experimental",
    hdrs = [":sai_experimental_files"],
    includes = ["experimental"],
)

# ---- Aggregated header contract (@sai//:sai_hdrs / :sai_files) --------------

filegroup(
    name = "sai_files",
    srcs = [
        ":sai_experimental_files",
        ":sai_inc_files",
    ],
)

cc_library(
    name = "sai_hdrs",
    deps = [
        ":sai_experimental",
        ":sai_inc",
    ],
)

# ---- meta/ (parse.pl generator + libsaimetadata) ----------------------------

# Patch upstream meta/Doxyfile so rules_doxygen can use it as a template:
#   * OUTPUT_DIRECTORY must be the sandboxed Doxyfile dir, not the empty
#     upstream value (which would write into the source tree).
#   * The hardcoded INPUT block points at ../inc/, ../experimental/, ...
#     which don't exist in the sandbox; swap it for the {{INPUT}} marker so
#     rules_doxygen injects the actual sandbox dirs of `srcs` instead.
expand_template_rule(
    name = "prepare_doxyfile_for_bazel",
    out = "meta/Doxyfile.bazel",
    substitutions = {
        "OUTPUT_DIRECTORY       =": "# {{OUTPUT DIRECTORY}}",
        "\n".join([
            "INPUT                  = ../inc/",
            "INPUT                  += ../experimental/",
            "INPUT                  += ../custom/",
            "INPUT                  += saimetadatatypes.h",
            "INPUT                  += saimetadatautils.h",
            "INPUT                  += saimetadatalogger.h",
            "INPUT                  += saiserialize.h",
        ]): "# {{INPUT}}\n# {{ADDITIONAL PARAMETERS}}",
    },
    template = "meta/Doxyfile",
)

doxygen(
    name = "doxygen",
    srcs = [
        "meta/saimetadatalogger.h",
        "meta/saimetadatatypes.h",
        "meta/saimetadatautils.h",
        "meta/saiserialize.h",
        ":sai_experimental_files",
        ":sai_inc_files",
    ],
    outs = [
        "xml",
    ],
    doxyfile_template = ":prepare_doxyfile_for_bazel",
)

perl_library(
    name = "perl_lib",
    srcs = [
        "meta/cap.pm",
        "meta/serialize.pm",
        "meta/style.pm",
        "meta/test.pm",
        "meta/utils.pm",
        "meta/xmlutils.pm",
    ],
    includes = ["meta"],
)

perl_binary(
    name = "parse",
    srcs = ["meta/parse.pl"],
    main = "meta/parse.pl",
    deps = [":perl_lib"],
)

# Just the xml/ TreeArtifact from :doxygen (whose default outputs include both
# the generated Doxyfile and xml/). parse.pl needs only the xml dir.
filegroup(
    name = "doxygen_xml",
    srcs = [":doxygen"],
    output_group = "xml",
)

# parse.pl is heavily cwd-coupled: $XMLDIR="xml", $INCLUDE_DIR="../inc/" and
# $EXPERIMENTAL_DIR="../experimental/" are all hardcoded relative paths, the
# four CONSTHEADERS are picked up via opendir("."), and outputs are written to
# cwd. To avoid patching upstream we stage a SAI/{meta,inc,experimental,custom}
# layout in a tmpdir, cd to meta/, run the hermetic :parse perl_binary, then
# copy the four declared outputs back.
#
# saiattrversion.h is git-derived; the Makefile path runs attrversion.sh from a
# git checkout. Inside this sandboxed action we have no git, so we stage an
# empty placeholder. Without it parse.pl emits per-attr "no version defined"
# warnings (not fatal) and saimetadata.c omits the SAI_METADATA_ATTR_VERSION_*
# entries. A separate non-hermetic target should populate it when version
# tracking is needed.
genrule(
    name = "saimetadata_sources",
    srcs = [
        "meta/saimetadatatypes.h",
        "meta/saimetadatautils.h",
        "meta/saimetadatalogger.h",
        "meta/saiserialize.h",
        "meta/acronyms.txt",
        "meta/aspell.en.pws",
        ":sai_inc_files",
        ":sai_experimental_files",
        ":doxygen_xml",
    ],
    outs = [
        "meta/saimetadata.h",
        "meta/saimetadata.c",
        "meta/saimetadatatest.c",
        "meta/saiswig.i",
    ],
    cmd = """
set -eu
PARSE=$$(realpath $(execpath :parse))
XML_DIR=$$(realpath $(execpath :doxygen_xml))

STAGE=$$(mktemp -d)
trap 'rm -rf "$$STAGE"' EXIT
mkdir -p "$$STAGE/SAI/meta" "$$STAGE/SAI/inc" \\
         "$$STAGE/SAI/experimental" "$$STAGE/SAI/custom"

for f in $(execpath meta/saimetadatatypes.h) $(execpath meta/saimetadatautils.h) \\
         $(execpath meta/saimetadatalogger.h) $(execpath meta/saiserialize.h) \\
         $(execpath meta/acronyms.txt) $(execpath meta/aspell.en.pws); do
    cp -L "$$f" "$$STAGE/SAI/meta/"
done

cp -RL "$$XML_DIR" "$$STAGE/SAI/meta/xml"

for f in $(execpaths :sai_inc_files); do
    cp -L "$$f" "$$STAGE/SAI/inc/"
done
for f in $(execpaths :sai_experimental_files); do
    cp -L "$$f" "$$STAGE/SAI/experimental/"
done

: > "$$STAGE/SAI/meta/saiattrversion.h"

# parse.pl -S skips the aspell-based style check; the hermetic build doesn't
# ship aspell. Drop -S if /usr/bin/aspell is available and you want lint.
( cd "$$STAGE/SAI/meta" && "$$PARSE" -S )

cp "$$STAGE/SAI/meta/saimetadata.h"     $(execpath meta/saimetadata.h)
cp "$$STAGE/SAI/meta/saimetadata.c"     $(execpath meta/saimetadata.c)
cp "$$STAGE/SAI/meta/saimetadatatest.c" $(execpath meta/saimetadatatest.c)
cp "$$STAGE/SAI/meta/saiswig.i"         $(execpath meta/saiswig.i)
""",
    tools = [":parse"],
)

# saiswig.i (generated above) contains `%include "saiattrversion.h"` at the
# @-version-tracking line. SWIG needs the file to exist even if empty; supply
# a zero-byte placeholder, same as the placeholder we drop inside
# :saimetadata_sources' staging tmpdir.
genrule(
    name = "saiattrversion_h_stub",
    srcs = [],
    outs = ["meta/saiattrversion.h"],
    cmd = ": > $(OUTS)",
)

# All meta public headers (hand-written + generated). For downstream consumers
# that want every sai*.h under meta/ as one filegroup (e.g. .deb packaging).
filegroup(
    name = "meta_hdrs",
    srcs = [
        "meta/saimetadatalogger.h",
        "meta/saimetadatatypes.h",
        "meta/saimetadatautils.h",
        "meta/saiserialize.h",
        "meta/saimetadata.h",  # generated by :saimetadata_sources
        ":saiattrversion_h_stub",  # empty placeholder for saiswig.i include
    ],
)

# Self-contained metadata C library, packaged as a SONiC-style versioned .so.
# Bundles the two hand-written .c files plus the parse.pl-generated
# saimetadata.c, exposes all meta sai*.h headers, and re-exports the
# inc/experimental header trees via deps.
sonic_shared_library_versioned(
    name = "saimetadata",
    srcs = [
        "meta/saimetadatautils.c",
        "meta/saiserialize.c",
        "meta/saimetadata.c",  # generated by :saimetadata_sources
    ],
    hdrs = [":meta_hdrs"],
    includes = ["meta"],
    soversion = "0",
    version = "0.0.0",
    deps = [
        ":sai_experimental",
        ":sai_inc",
    ],
)
