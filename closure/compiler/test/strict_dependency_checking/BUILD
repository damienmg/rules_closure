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
load("//closure/private:rule_test.bzl", "rule_test")
load("//closure:defs.bzl", "closure_js_binary")
load("//closure:defs.bzl", "closure_js_library")

################################################################################
# Assume we're using traditional Google namespaces.

closure_js_library(
    name = "a",
    srcs = ["a.js"],
)

closure_js_library(
    name = "b",
    srcs = ["b.js"],
    deps = [":a"],
)

closure_js_library(
    name = "c",
    srcs = ["c.js"],
    deps = [
        ":a",
        ":b",
    ],
)

closure_js_library(
    name = "t",
    testonly = True,
    srcs = ["t.js"],
)

rule_test(
    name = "a_rule_test",
    provides = {
        "js_language": "^\"ECMASCRIPT5_STRICT\"$",
        "js_exports": "^\\[\\]$",
        "transitive_js_srcs": "/a.js\\]",
        "transitive_js_externs": "",
    },
    rule = ":a",
)

rule_test(
    name = "b_rule_test",
    provides = {
        "js_language": "^\"ECMASCRIPT5_STRICT\"$",
        "js_exports": "^\\[\\]$",
        "transitive_js_srcs": "/a\\.js.*/b\\.js\\]",
        "transitive_js_externs": "",
    },
    rule = ":b",
)

rule_test(
    name = "c_rule_test",
    provides = {
        "js_language": "^\"ECMASCRIPT5_STRICT\"$",
        "js_exports": "^\\[\\]$",
        "transitive_js_srcs": "/a\\.js.*/b\\.js.*/c\\.js\\]",
        "transitive_js_externs": "",
    },
    rule = ":c",
)

rule_test(
    name = "t_rule_test",
    provides = {
        "js_language": "^\"ECMASCRIPT5_STRICT\"$",
        "js_exports": "^\\[\\]$",
        "transitive_js_srcs": "/t\\.js\\]",
        "transitive_js_externs": "",
    },
    rule = ":t",
)

file_test(
    name = "fileWithGoogProvide_providesOnlyGoogProvides",
    content = "goog:a\n",
    file = "a-provided.txt",
)

file_test(
    name = "a_noWarnings",
    content = "",
    file = "a-stderr.txt",
)

file_test(
    name = "b_noWarnings",
    content = "",
    file = "b-stderr.txt",
)

file_test(
    name = "c_noWarnings",
    content = "",
    file = "c-stderr.txt",
)

file_test(
    name = "t_noWarnings",
    content = "",
    file = "t-stderr.txt",
)

closure_js_library(
    name = "didNotSpecifyTransitiveDep",
    srcs = ["c.js"],
    internal_expect_failure = True,
    tags = ["manual"],
    deps = [":b"],  # oops I forgot :a, it doesn't matter if :b depends on :a
)

file_test(
    name = "didNotSpecifyTransitiveDep_causesMissingProvideError",
    file = "didNotSpecifyTransitiveDep-stderr.txt",
    regexp = "CR_NOT_PROVIDED",
)

closure_js_library(
    name = "b2",
    srcs = ["b.js"],
    exports = [":a"],
    deps = [":a"],
)

closure_js_library(
    name = "exportMadeTransitiveDepDirect",
    srcs = ["c.js"],
    deps = [":b2"],  # but if :b exports :a, then we're ok
)

file_test(
    name = "exportedDependency_wormsAroundStrictsDependencyChecking",
    content = "",
    file = "exportMadeTransitiveDepDirect-stderr.txt",
)

closure_js_library(
    name = "duplicateProvidesWithinALibrary",
    srcs = [
        "a.js",
        "a2.js",
    ],
    internal_expect_failure = True,
    tags = ["manual"],
)

file_test(
    name = "duplicateProvidesWithinALibrary_causesDuplicateProvideError",
    file = "duplicateProvidesWithinALibrary-stderr.txt",
    regexp = "CR_DUPLICATE_PROVIDES",
)

closure_js_library(
    name = "duplicateProvideFromDeps",
    srcs = ["a2.js"],
    internal_expect_failure = True,
    tags = ["manual"],
    deps = [":a"],
)

file_test(
    name = "duplicateProvideFromDeps_causesDuplicateProvideError",
    file = "duplicateProvideFromDeps-stderr.txt",
    regexp = "CR_REDECLARED_PROVIDES",
)

closure_js_library(
    name = "setTestOnlyInLibraryWithoutTestOnly",
    testonly = False,  # <-- oops I forgot this line
    srcs = ["t.js"],
    internal_expect_failure = True,
    tags = ["manual"],
)

file_test(
    name = "setTestOnlyInLibraryWithoutTestOnly_causesDuplicateProvideError",
    file = "setTestOnlyInLibraryWithoutTestOnly-stderr.txt",
    regexp = "CR_INVALID_SETTESTONLY",
)

################################################################################
# Assume we're using ECMASCRIPT6 namespaces.

closure_js_library(
    name = "es6_a",
    srcs = ["es6_a.js"],
    language = "ECMASCRIPT6_STRICT",
)

closure_js_library(
    name = "es6_b",
    srcs = ["es6_b.js"],
    language = "ECMASCRIPT6_STRICT",
    deps = [":es6_a"],
)

closure_js_library(
    name = "es6_c",
    srcs = ["es6_c.js"],
    language = "ECMASCRIPT6_STRICT",
    deps = [
        ":es6_a",
        ":es6_b",
    ],
)

file_test(
    name = "fileWithoutGoogProvideInEs6Mode_providesShortPathWithoutJs",
    content = "closure/compiler/test/strict_dependency_checking/es6_a\n",
    file = "es6_a-provided.txt",
)

closure_js_library(
    name = "es6_didNotSpecifyTransitiveDep",
    srcs = ["es6_c.js"],
    internal_expect_failure = True,
    language = "ECMASCRIPT6_STRICT",
    tags = ["manual"],
    deps = [":es6_b"],  # oops I forgot :es6_a, it doesn't matter if :es6_b depends on :es6_a
)

file_test(
    name = "es6_didNotSpecifyTransitiveDep_causesMissingProvideError",
    file = "es6_didNotSpecifyTransitiveDep-stderr.txt",
    regexp = "CR_NOT_PROVIDED",
)

closure_js_library(
    name = "es6_requireWithinSameLibrary_works",
    srcs = [
        "es6_a.js",
        "es6_c.js",
    ],
    language = "ECMASCRIPT6_STRICT",
    deps = [":es6_b"],
)

closure_js_library(
    name = "hybrid_lib",
    srcs = [
        "a.js",
        "es6_a.js",
    ],
    language = "ECMASCRIPT6_STRICT",
)

file_test(
    name = "hybridGoogAndEs6Library_onlyProvidesPathForSrcsWithoutGoogProvide",
    content = "closure/compiler/test/strict_dependency_checking/es6_a\ngoog:a\n",
    file = "hybrid_lib-provided.txt",
)

genrule(
    name = "es6_g_generate",
    outs = ["es6_g.js"],
    cmd = "echo 'export function g() { console.log(\"hi\"); }' >$@",
)

closure_js_library(
    name = "es6_g",
    srcs = ["es6_g.js"],
    language = "ECMASCRIPT6_STRICT",
)

file_test(
    name = "generatedEs6Module_providedNamespaceDoesntHaveBazelJunk",
    content = "closure/compiler/test/strict_dependency_checking/es6_g\n",
    file = "es6_g-provided.txt",
)

genrule(
    name = "es6_g2_generate",
    outs = ["es6_g2.js"],
    cmd = "echo 'import { g } from \"closure/compiler/test/strict_dependency_checking/es6_g\"; g();' >$@",
)

closure_js_library(
    name = "es6_g2",
    srcs = ["es6_g2.js"],
    language = "ECMASCRIPT6_STRICT",
    deps = ["es6_g"],
)

closure_js_binary(
    name = "es6_g2_bin",
    pedantic = True,
    deps = [":es6_g2"],
)

file_test(
    name = "es6ModulesWithRootPaths_compileCorrectlyWhenReferencedWithoutRootPrefix",
    content = "console.log(\"hi\");\n",
    file = "es6_g2_bin.js",
)
