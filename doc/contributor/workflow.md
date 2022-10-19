# The TruffleRuby Contributor Workflow

## Requirements

See the [Dependencies in the README](../../README.md#dependencies).

The requirements include a [C compiler](../user/installing-llvm.md). Because it's a common issue, we remind macOS users
they might need to add `export SDKROOT=$(xcrun --show-sdk-path)` to their shell profile.

Additionally, you will need:

* Ruby >= 2.3 (we stick at this version as it is available all the way back to for example Ubuntu 16.04)
* [CMake](https://cmake.org/), for building Sulong (GraalVM's LLVM support)
  CMake can be installed via the usual methods: `dnf`, `apt-get`, `brew`, ...)

## Workspace directory

We recommend creating an extra directory for building TruffleRuby:

```bash
mkdir truffleruby-ws
cd truffleruby-ws
```

You can then clone the repository:
```bash
git clone https://github.com/oracle/truffleruby.git
cd truffleruby
```

## Developer tool

We then use a Ruby script to run most commands.

```bash
bin/jt --help
```

Most of us add an alias to our shell profile file so that it can be run with
just `jt`. To allow this to run from any path, add this to your `~/.bash_profile`:

```bash
export SYSTEM_RUBY=/path/to/mri/bin/ruby
alias jt=/path/to/truffleruby/bin/jt
```

```bash
jt --help
```

## Pre-Commit Hook

Please install this `pre-commit` hook which runs the fast lint checks.
In our experience, it is way more efficient to use this hook than to wait for the CI,
and it also results in cleaner commits in the first place.

```bash
$ cp tool/hooks/lint-check.sh .git/hooks/pre-commit
```

It is also possible to use a `pre-push` hook instead (`cp tool/hooks/lint-check.sh .git/hooks/pre-push`).
That way the lint check runs only before `git push`.
However, that might result in extra "Fix style" commits, which sometimes can be `git rebase -i` away, but sometimes not.

## Building

```bash
jt build
```

By default the `jt build` command builds a small JVM-only (no native images)
GraalVM containing only the Ruby language. The built GraalVM can be found in the
`mxbuild/truffleruby-jvm` directory.

There are multiple *build configurations* available to build TruffleRuby:
* `jvm`: the default, JVM-only, no GraalVM compiler (Ruby code is always interpreted)
* `jvm-ce`: JVM-only, with the GraalVM compiler
* `native`: Builds a native image of TruffleRuby using SubstrateVM, including the GraalVM compiler

All `jvm*` build configurations can only run the `--jvm` [runtime configuration][rt-confs].  
The `native` build configuration can run both the `--native` and `--jvm` [runtime configurations][rt-confs].

[rt-confs]: ../../README.md#TruffleRuby-Runtime-Configurations

To build one of these build configurations, pass `--env` to the build command:
```bash
jt build [--env BUILD_CONFIGURATION]
```

You can create a new build configuration by creating an [mx env file] in `mx.truffleruby`.

[mx env file]: https://github.com/graalvm/mx/blob/master/README.md#environment-variable-processing

Builds are created in the `mxbuild/truffleruby-${BUILD_NAME}` directory. By default, the *build name* is the
name of the build configuration, but you can specify a different name with `--name BUILD_NAME`. This enables
you to store multiple builds that use the same configuration. Note that if you want to compare multiple builds using
the same `--env` then you need to use `--name` for all of them.
Without `--name`, `mxbuild/truffleruby-${env}` is just a symlink and will be reused by any build of the same `--env`.

Note that build information such as the date and Git revision hash will not be
updated when you build for a second time. Releases should always be built from
scratch.

### Using the correct version of the graal repository

TruffleRuby needs the `truffle` and `sulong` suites from the `graal` repository.
`jt build` will automatically clone the repository but not enforce a specific version (commit).
When running `jt build`, you might see an early warning:
```
$ jt build
...
NOTE: Set env variable JT_IMPORTS_DONT_ASK to always answer 'no' to this prompt.

WARNING: imported version of sulong in truffleruby (ae65c10142907329e03ad8e3fa17b88aca42058d) does not match parent (1bf42ddef0e4961cbb92ebc31019747fd1c15f1a)
Do you want to checkout the supported version of graal as specified in truffleruby's suite.py? (runs `mx sforceimports`) [y/n]
...
```

This warning is important.

- If you did not create new commits in `graal`, this means the graal import was bumped in `suite.py` and you need
  to answer `y` to this prompt, which will be equivalent to running `jt mx sforceimports` before proceeding.
- If you did create new `graal` commits, you should answer `n` or set `JT_IMPORTS_DONT_ASK` (to any value) to
  automatically do so.
- If you want to set the `suite.py` import to that checked out in `graal` (unlikely), you should run 
  jt mx scheckimports` beforehand.

### Building C Extensions more quickly

Speeding up compilation of C extensions can be achieved by using *native* toolchain launchers,
instead of the Bash toolchain launchers used with the `jvm*` build configurations.

For bundled C extensions (that is, extensions under `src/main/c`),
you can `export JT_CACHE_TOOLCHAIN=true` and that will then use native toolchain launchers for the
[bootstrap toolchain](https://github.com/oracle/graal/blob/master/sulong/docs/contributor/TOOLCHAIN.md#using-a-prebuilt-graalvm-as-a-bootstrapping-toolchain).
This has no effect after `jt build`.

For C extensions installed after `jt build` by e.g. `gem install` or `bundle install`,
one can use the `jvm-ce-ntl` build configuration which includes *native* toolchain launchers,
or use one of the `native*` build configuration (which also builds a native truffleruby launcher).

## Editors and IDEs

* [Using the IntelliJ IDE for Development](using-intellij.md) (recommended)
* [Using the Eclipse IDE for Development](using-eclipse.md)

## Running

`jt ruby` runs TruffleRuby. You can use it exactly as you'd run the MRI `ruby`
command. Additionally, `jt ruby` sets a couple of extra options to help you when
developing, such as loading the core library from disk rather than the JAR.
`jt ruby` prints the real command it's running as it starts.

If you are running a Ruby environment manager like `rvm`, `rbenv`, or `chruby`
please run `rvm use system`, `rbenv system`, or `chruby system` to clear their
environment variables, so that the correct gems are picked up.

By default, `jt ruby` runs the `jvm` build of TruffleRuby (that is, built with the `--jvm` [build
configuration](#building) and using the default build name) and aborts if this build doesn't exist.

You can also use `jt` to run other TruffleRuby builds (see the [Building](#building) section), just
pass the build name after `--use`:

```bash
jt --use BUILD_NAME ruby ...
```

You can also pass the path to a Ruby executable after `--use`, e.g.:

```bash
jt --use /usr/bin/ruby ruby ...
```

## Testing

We have 'specs' which come from [the Ruby Spec Suite](https://github.com/ruby/spec).
These are usually high quality, small tests, and are our priority at the moment.
We also have MRI's unit tests, which are often very complex and we aren't
actively working on now. Finally, we have tests of our own. The integration
tests test more macro use of Ruby. The ecosystem tests test commands related to
Ruby. The gems tests test a small number of key Ruby 3rd party modules.

The basic test to run every time you make changes is the "fast specs", a subset
of specs which runs in reasonable time.

```bash
jt [--use BUILD_CONFIGURATION] test fast
```

Other tests take longer and require more setup, so we don't normally run them
locally unless we're working on that functionality (instead, the CI runs them).

Specs under `spec/ruby` are supposed to pass on both Truffle and MRI.
To run the tests or specs on MRI, pass `--use ruby`:
```bash
jt --use ruby test path/to/spec.rb # assumes you have MRI in your PATH
jt --use /full/path/to/bin/ruby test path/to/spec.rb
```

## Options

Specify JVM options with `--vm.option`.

```bash
jt ruby --vm.Xmx1G test.rb
```

TruffleRuby options are set with `--name=value`. For example
`--exceptions-print-java=true` to print Java exceptions before translating them
to Ruby exceptions. You can leave off the value to set the option to `true`.

To see all options run `jt ruby --help:languages`.

Ruby command line options and arguments can also be set in `RUBYOPT` or
`TRUFFLERUBYOPT`.

## Running with Graal

To build TruffleRuby with the GraalVM CE compiler, use:
```bash
jt build --env jvm-ce
```

Then, run TruffleRuby with:
```bash
jt --use jvm-ce ruby ...
```

We have flags in `jt` to set some options, such as `--trace` for
`--engine.TraceCompilation`.

## Running with Polyglot

Under [mx.truffleruby](../../mx.truffleruby), there are build configurations to build a GraalVM with TruffleRuby and
other Truffle languages, such as `jvm-js` and `jvm-py`.
One can of course also make their own env file.

Let's look at the example of building with [graaljs](https://github.com/graalvm/graaljs) to be able to evaluate JavaScript code from TruffleRuby.

Building is as simple as cloning the repository and using the right env file:
```bash
cd truffleruby-ws/truffleruby
git clone https://github.com/graalvm/graaljs.git ../graaljs
jt build --env jvm-js
```

Similar for building with [graalpython](https://github.com/graalvm/graalpython):
```
cd truffleruby-ws/truffleruby
git clone https://github.com/graalvm/graalpython.git ../graalpython
jt build --env jvm-py
```

Then, run TruffleRuby with `--polyglot` support and evaluate some JavaScript:

```bash
$ jt --use jvm-js ruby --polyglot
> Polyglot.eval('js', 'var a = 1; a + 1')
=> 2
```

See the [Polyglot](../user/polyglot.md) and [Truffle Interop](interop.md) documentation for details about polyglot programming.

## Testing with Graal

The basic test for Graal is to run our compiler tests. This includes tests that
things partially evaluate as we expect, that things optimise as we'd expect,
that on-stack-replacement works and so on.

```bash
jt test compiler
```

## How to fix a failing spec

We usually use the `jt untag` command to work on failing specs. It runs only
specs that are marked as failing.

```bash
jt untag spec/ruby/core/string
```

When you find a spec that you want to work on, it can be easier to look at the
spec's source (for example look in `spec/ruby/core/string`) and recreate it
as a standalone Ruby file for simplicity.

Then you probably want to run with `--exceptions-print-java` if you see a Java
exception.

When the spec is fixed the `untag` command will remove the tag and you can
commit the fix and the removal of the tag.

## Running specs for Ruby 3.2 features

TruffleRuby currently targets Ruby 3.1. However, we welcome pull requests for
Ruby 3.2 features as long as they don't conflict significantly with
Ruby 3.1 semantics.

It is possible to run specs for Ruby 3.1 features by setting
`PRETEND_RUBY_VERSION`:

```bash
PRETEND_RUBY_VERSION=3.2.0 jt test spec/ruby/.../some_spec.rb
```

This also works for `jt tag`/`jt untag`.

When working on a feature from the next version of Ruby, add the spec file in
the corresponding file list (`:next`) in `spec/truffleruby.mspec` so that the
specs are run in CI too.

## How to fix a failing MRI test

Remove the exclusion of either the file (`test/mri/failing.exclude`) or the
individual method (`test/mri/excludes/...`) and run the individual file
`jt test mri test/mri/tests/file.rb` to see any errors.

As with specs, you probably then want to recreate the test in a standalone Ruby
file to fix it.

You can also recompute the tags automatically for an entire test file with
`jt retag test/mri/tests/file.rb`.

## Building the parser

TruffleRuby uses the Jay parser generator. A copy of this is located in
`tool/jay`. The command `jt build parser` will build Jay, if needed, and then
regenerate the parser. We check the generated parser into the source repository.
