# Copyright 2016 The Closure Rules Authors. All rights reserved.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#    http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

package(default_testonly = True)

licenses(["notice"])  # Apache 2.0

load("//closure/private:file_test.bzl", "file_test")
load("//closure:defs.bzl", "closure_js_binary")
load("//closure:defs.bzl", "closure_js_deps")
load("//closure:defs.bzl", "closure_js_library")

# This package tests the behavior of closure_js_deps. This package also tests
# the behavior runfiles with regard to closure_js_library, closure_js_binary,
# and closure_js_deps.
#
# The concepts of deps files and runfiles are intertwined. If one wishes to
# develop in raw sources mode, one needs a deps file. In order for a deps file
# to be useful, one must also assemble and serve the runfiles data=[...] it
# references.
#
# Filegroups should not needed to assemble the raw sources, because the closure
# rules themselves can behave like smart filegroups. The tests in this package
# demonstrate how the sources and data files propagate through them.
#
# tl;dr: java_binary() data attribute only needs to reference the deps rule.

closure_js_library(
    name = "external_library",
    externs = ["external_library.js"],
)

closure_js_library(
    name = "hyperion_lib",
    srcs = ["hyperion.js"],
    data = ["data1.txt"],
    deps = [":external_library"],
)

closure_js_deps(
    name = "hyperion_deps",
    deps = [":hyperion_lib"],
)

file_test(
    name = "noTransitiveDeps_containsSingleLineRelativeToClosureLibraryBaseJs",
    file = "hyperion_deps.js",
    regexp = "^goog.addDependency('../../../../closure/compiler/test/closure_js_deps/hyperion.js', \\['hyperion'\\], \\[\\], false);$",
)

closure_js_library(
    name = "goblin",
    srcs = ["goblin.js"],
    deps = ["//closure/library"],
)

closure_js_deps(
    name = "goblin_deps",
    deps = [":goblin"],
)

file_test(
    name = "module_usesTrueArgument",
    file = "goblin_deps.js",
    regexp = "^goog.addDependency('../../../../closure/compiler/test/closure_js_deps/goblin.js', \\['goblin'\\], \\['goog.dom'\\], true);$",
)

file_test(
    name = "googDomDependency_includesClosureLibraryStuffInDepsFile",
    file = "goblin_deps.js",
    regexp = "^goog.addDependency('\\./dom/dom\\.js', \\['goog\\.dom'",
)

file_test(
    name = "externs_dontGoInDepsFile",
    file = "hyperion_deps.js",
    invert = True,
    regexp = "^external_library$",
)

################################################################################
# closure_js_deps() ingests sources transitively

closure_js_library(
    name = "hyperion2_lib",
    srcs = ["hyperion2.js"],
    data = ["data2.txt"],
    deps = [":hyperion_lib"],
)

closure_js_deps(
    name = "hyperion2_deps",
    data = ["data4.txt"],
    deps = [":hyperion2_lib"],
)

file_test(
    name = "hasTransitiveDep_showsUpInDepsFile",
    file = "hyperion2_deps.js",
    regexp = "^goog.addDependency('../../../../closure/compiler/test/closure_js_deps/hyperion.js', \\['hyperion'\\], \\[\\], false);$",
)

################################################################################
# srcs referencing closure_js_library() ingests nothing

genrule(
    name = "hyperion2_srcs",
    srcs = [":hyperion2_lib"],
    outs = ["hyperion2_srcs.txt"],
    cmd = "echo -n $(SRCS) >$@",
)

file_test(
    name = "srcsReferencingJsLibrary_getsNothing",
    content = "",
    file = "hyperion2_srcs.txt",
)

################################################################################
# data referencing closure_js_library() ingests non-externs sources transitively

sh_test(
    name = "dataAttributeReferencingJsLibrary_getsJsSourcesAndDataTransitively",
    srcs = ["hyperion2_lib_runfiles_test.sh"],
    data = [":hyperion2_lib"],
)

################################################################################
# filegroup(data) referencing closure_js_library() ingests sources transitively

filegroup(
    name = "hyperion2_lib_runfiles",
    data = [":hyperion2_lib"],
)

sh_test(
    name = "filegroupReferencingJsLibrary_getsJsSourcesAndDataTransitively",
    srcs = ["hyperion2_lib_runfiles_test.sh"],
    data = [":hyperion2_lib_runfiles"],
)

################################################################################
# closure_js_binary() includes raw sources in runfiles which .js.map references

closure_js_binary(
    name = "hyperion2_bin",
    data = ["data3.txt"],
    entry_points = ["goog:hyperion2"],
    pedantic = True,
    deps = [":hyperion2_lib"],
)

sh_test(
    name = "dataAttributeReferencingJsBinary_onlyGetsCompiledJsAndMap",
    srcs = ["hyperion2_bin_runfiles_test.sh"],
    data = [":hyperion2_bin"],
)

################################################################################
# data referencing closure_js_deps() gives you the deps file AND the sauce

sh_test(
    name = "dataAttributeReferencingJsDeps_providesDepsFileSourcesAndData",
    srcs = ["hyperion2_deps_runfiles_test.sh"],
    data = [":hyperion2_deps"],
)

################################################################################
# srcs referencing closure_js_binary() gives only the binary

genrule(
    name = "hyperion2_bin_srcs",
    srcs = [":hyperion2_bin"],
    outs = ["hyperion2_bin_srcs.txt"],
    cmd = "echo -n $(SRCS) >$@",
)

file_test(
    name = "srcsReferencingJsBinary_getsJsBinaryAndMapNotData",
    content = "bazel-out/local-fastbuild/bin/closure/compiler/test/closure_js_deps/hyperion2_bin.js bazel-out/local-fastbuild/bin/closure/compiler/test/closure_js_deps/hyperion2_bin.js.map",
    file = "hyperion2_bin_srcs.txt",
)
