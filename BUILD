load("@bazel_skylib//rules:run_binary.bzl", "run_binary")
load("@rules_python//python:defs.bzl", "py_binary")

py_binary(
    name = "hello",
    srcs = ["hello.py"],
    main = "hello.py",
)

run_binary(
    name = "hello_run_binary",
    outs = ["hello_run_binary.out"],
    args = ["$(location hello_run_binary.out)"],
    tool = ":hello",
)

genrule(
    name = "hello_genrule",
    outs = ["hello_genrule.out"],
    cmd = "$(PYTHON3) $(location :hello) $@",
    toolchains = ["@rules_python//python:current_py_toolchain"],
    tools = [":hello"],
)
