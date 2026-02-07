Python binaries: run with `genrule()` or `run_binary()` ?

The BUILD file defines a `:hello` python binary. It is run with `run_binary()`
and with `genrule()`.

```python
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
```

The MODULE.bazel uses the current `skylib` and `rules_python`

```python
module(
    name="python-run_binary-issue",
    version = "0.1",
)

bazel_dep(name = "bazel_skylib", version = "1.9.0")
bazel_dep(name = "rules_python", version = "1.8.3")
```

If this is on a pure system (nothing but bazel on the system), the `genrule()`
**will work** as it uses the toolchain.

The `run_binary()` **will not work**, it looks like it does not run the binary
with the Python toolchain but maybe attempts to find a local Python of sorts ?

To test, let's set up a pure system with only bazel installed (here bazel 8,
but same problem is with bazel 7), in particular no python3 visible.

To achive the purity, we're using NixOS:

```bash
$ nix-shell --pure
$ bazel --version
bazel 8.5.0- (@non-git)
$ python3
bash: python3: command not found
```
The `run_binary()` invocation fails:

```bash
$ bazel build :hello_run_binary
INFO: Invocation ID: 0497f3ad-c2b9-452b-ac42-b73f1f1425f8
INFO: Analyzed target //:hello_run_binary (2 packages loaded, 2313 targets configured).
ERROR: /my_run_dir/python-run_binary-issue/BUILD:10:11: RunBinary hello_run_binary.out failed: (Exit 127): hello failed: error executing RunBinary command (from target //:hello_run_binary) bazel-out/k8-opt-exec-ST-d57f47055a04/bin/hello

Use --sandbox_debug to see verbose messages from the sandbox and retain the sandbox build root for debugging
env: 'python3': No such file or directory
Target //:hello_run_binary failed to build
Use --verbose_failures to see the command lines of failed build steps.
INFO: Elapsed time: 0.862s, Critical Path: 0.08s
INFO: 2 processes: 8 action cache hit, 2 internal.
ERROR: Build did NOT complete successfully
```

The `genrule()` invocation works

```bash
$ bazel build :hello_genrule
INFO: Invocation ID: ba02e350-cdf6-4743-9b64-6d12af04fc08
INFO: Analyzed target //:hello_genrule (0 packages loaded, 6 targets configured).
INFO: Found 1 target...
Target //:hello_genrule up-to-date:
  bazel-bin/hello_genrule.out
INFO: Elapsed time: 0.123s, Critical Path: 0.00s
INFO: 1 process: 1 action cache hit, 1 internal.
INFO: Build completed successfully, 1 total action

$ cat bazel-bin/hello_genrule.out
Hello world!
```

Now question: is `run_binary()` supposed to invoke the `py_binary()` with
the toolchain, or is this expected to fail ?
