java_binary(
    name = "jschecker",
    srcs = [
        "JsChecker.java",
        "JsCheckerFirstPass.java",
        "JsCheckerPassConfig.java",
        "JsCheckerSecondPass.java",
        "JsCheckerState.java",
    ],
    main_class = "com.google.javascript.jscomp.JsChecker",
    visibility = ["//visibility:public"],
    deps = ["@closure_compiler//jar"],
)