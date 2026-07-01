load("@rules_cc//cc:cc_library.bzl", "cc_library")
load("@sonic_build_infra//shared_library:shared_library.bzl", "sonic_shared_library_versioned")
load("@sonic_build_infra//swig:gen.bzl", "swig_gen")

package(default_visibility = ["//visibility:public"])

# Hand-written __init__.py lives in the source tree under pyext/py3/; expose
# it as a source label so //pyext:py3/__init__.py resolves.
exports_files(["py3/__init__.py"])

# =============================================================================
# SWIG code generation. Mirrors pyext/py3/Makefile.am:
#   pysairedis_wrap.cpp: $(SWIG_SRC) $(SOURCES)
#       $(SWIG) -Wall -c++ -python -keyword -DSWIGWORDSIZE64 \
#               $(SAIINC) -I$(top_srcdir)/lib -I.. -o $@ $<
#
# pysairedis.i does `%include "../SAI/meta/saiswig.i"`; that file is generated
# by parse.pl (see SAI/meta/BUILD.bazel :saimetadata_sources genrule) and
# lands in Bazel's output tree at external/sai+/meta/saiswig.i. In the Make
# world both sit in the source tree side by side; in the Bazel sandbox they
# don't — so we stage a rewritten copy of pysairedis.i whose %include uses
# just `saiswig.i` (which swig_gen resolves via the -I dirs propagated from
# `hdrs`).
# =============================================================================
genrule(
    name = "pysairedis_i_staged",
    srcs = ["pysairedis.i"],
    outs = ["staged/pysairedis.i"],
    cmd = """
        sed -e 's|../SAI/meta/saiswig.i|saiswig.i|' \\
            $(execpath pysairedis.i) > $(OUTS)
    """,
)

swig_gen(
    name = "pysairedis_wrap_gen",
    interface = ":pysairedis_i_staged",
    cpp_out = "py3/pysairedis_wrap.cpp",
    python_out = "py3/pysairedis.py",
    wordsize64 = True,
    hdrs = [
        "pysairedis.h",
        # saiswig.i and saimetadata.h are pulled in transitively by the
        # `deps` cc_library, but SWIG's %include reads relative paths in the
        # source tree, so we also expose them here.
        "@sai//meta:saiswig.i",
        "@sai//meta:saimetadata.h",
        # Empty placeholder needed because the parse.pl-generated saiswig.i
        # ends with `%include "saiattrversion.h"`.
        "@sai//meta:saiattrversion.h",
    ],
    deps = [
        ":pysairedis_cc",
        "//lib:sairedis_h",
        "@sai//meta:saimetadata",
        "@sai//inc:sai_inc",
        "@sai//experimental:sai_experimental",
    ],
)

# Hand-written support headers that must be compiled into the wrapper. Kept as
# a cc_library so `swig_gen`'s CcInfo-based include propagation picks up the
# pyext/ dir automatically.
cc_library(
    name = "pysairedis_cc",
    hdrs = ["pysairedis.h"],
    includes = ["."],
    visibility = ["//visibility:public"],
)

# =============================================================================
# _pysairedis native extension. Mirrors pyext_py3__pysairedis_la in
# pyext/py3/Makefile.am: produces _pysairedis.so.0.0.0 with soname
# _pysairedis.so.0 (autotools default -version-info 0:0:0).
#
# Links libpython3.13 (the system Python on trixie) via rules_python's
# @rules_python//python/cc headers/libs — same approach as
# sonic-swss-common's _swsscommon extension.
# =============================================================================
sonic_shared_library_versioned(
    name = "_pysairedis",
    srcs = [
        "pysairedis.cpp",
        ":pysairedis_wrap_gen",
    ],
    copts = [
        "-fPIC",
        "-Wno-cast-align",
        "-Wno-cast-qual",
        "-Wno-shadow",
        "-Wno-redundant-decls",
        "-Wno-deprecated-declarations",
        "-Wno-missing-field-initializers",
        "-Wno-unused-parameter",
    ],
    dynamic_deps = [
        "//lib:sairedis_shared",
        "//meta:saimeta_shared",
        "@sai//meta:saimetadata_shared",
        "@sonic-swss-common//:swsscommon_shared",
        "@libnl3//:libnl_3_shared",
        "@libnl3//:libnl_nf_3_shared",
        "@libnl3//:libnl_route_3_shared",
        "@libyang3//:yang_shared",
    ],
    deps = [
        ":pysairedis_cc",
        "//lib:SaiRedis",
        "//meta:saimeta",
        "@sai//meta:saimetadata",
        "@sairedis_deps//libhiredis-dev:libhiredis",
        "@sairedis_deps//libzmq3-dev:libzmq3",
        "@sonic-swss-common//:swsscommon",
        "@rules_python//python/cc:current_py_cc_headers",
        "@rules_python//python/cc:current_py_cc_libs",
    ],
    soversion = "0",
    version = "0.0.0",
    output_name = "_pysairedis",
    # Python C extensions leave interpreter-provided symbols undefined; also
    # matches Make's absence of `-Wl,-z,defs`.
    allow_undefined = True,
    visibility = ["//visibility:public"],
)
