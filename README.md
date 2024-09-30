# bazelbuild: A PEP 517 build backend for Bazel

This repository hosts bazelbuild (name not final), a PEP 517 build backend to build Python packages (wheels and sdists) using Bazel.

To find out more about PEP 517, refer to [the official document](https://peps.python.org/pep-0517/) on the Python Enhancement Proposals index.

## Motivation

Currently, building Python projects with Bazel is not easy. Some projects (mainly at Google) hand-roll a setuptools extension class, which is brittle, and breaks pretty frequently with new setuptools releases.
I am maintaining such a configuration on the side of my current job for [Google Benchmark](https://github.com/google/benchmark), and it is pretty tedious.

On top of that, [rules_python](https://github.com/bazelbuild/rules_python), the canonical ruleset for Python in Bazel, has some bugs that make it impossible to use in some cases.
One such example is building stable ABI wheels on Windows, which is currently not possible with any Python toolchain sourced via `rules_python`, possibly due to a misconfigured C++ library target.

## Installation

Currently only from git:

```shell
python -m pip install git+https://github.com/nicholasjng/bazelbuild
```

A PyPI release is planned.

## Goals

My main goal is to get rid of brittle `setup.py` configurations by implementing PEP 517 for Bazel.
Once this project matures, building wheels and sdists reproducibly with Bazel should be as easy as this:

```toml
[build-system]
requires = ["bazelbuild"]
build-backend = "bazelbuild.core"  # name not final here, either

[...]

[tool.bazelbuild]
option = "value"
```

Any other build information, such as the target platform, Bazel flags, or environment variables could be given as build configuration options.
Through the right mix of these command line options and (probably) some configuration-as-TOML (as seen in the bazelbuild tool block in the above example), it should be possible to build Python wheels for a variety of platforms with very little effort.

As a tangent: In my experience, while packaging pure Python code is not that hard (TM) to do reproducibly, things look different once C/C++ extensions are thrown into the mix.
A good example that I am working with is [nanobind](https://github.com/wjakob/nanobind), a Python<->C++ bindings generator.
Such bindings extensions end up as shared object (SO) files in built wheels, but may lack information on the build process such as:

* Which nanobind version was used?
* Which compiler flags were used to build the library?
* Are there debug symbols or not?
* What Python ABI is this extension targeting?
* Was the wheel built in release configuration, or something else?

Bazel is fairly good at delivering these answers, whereby you get a more comprehensive picture of your builds through its dependency/action graph.

Some secondary goals of this project are:

* Cross-compilation support, e.g. for building Linux wheels on a Mac.
* Configuring Python and C++ toolchains to build wheels for different Python interpreters other than the invoker, or with different C++ compilers.
* Offering rich configuration options. For example, quickly building wheels with `--compilation_mode=dbg` during high turnover development can make for a much better development experience in complex projects.
* Using Bazel for remote package builds, i.e., support for wheel build farms.

## How to get involved

This is my own side project. I am not paid to develop it, nor am I affiliated with Google or the Bazel GitHub organization.

That being said, if you are a Bazel user with Python projects and feel the same way, I very much welcome your input, either as feature requests, discussions, design ideas, or through other means.
If you'd like to help develop the project, you can send patches as pull requests against the `master` branch.
