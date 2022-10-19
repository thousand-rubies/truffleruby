---
layout: docs-experimental
toc_group: ruby
link_title: Ruby Managers and Installers
permalink: /reference-manual/ruby/RubyManagers/
---
# Ruby Managers and Installers

If you have [installed GraalVM](installing-graalvm.md), it is recommended to add TruffleRuby to a Ruby manager.
See
[Configuring Ruby managers for the full GraalVM distribution](#configuring-ruby-managers-for-the-full-graalvm-distribution) below.

## Installing TruffleRuby with `rvm`, `ruby-build`, `asdf` or `ruby-install`

TruffleRuby is supported by the 3 major Ruby installers.

### `rvm`

Upgrade `rvm` to let `rvm` know about the latest TruffleRuby release:

```bash
rvm get head
```

Install the latest TruffleRuby standalone release with:

```bash
rvm install truffleruby
```

You can also install the latest nightly standalone build of TruffleRuby with:

```bash
rvm install truffleruby-head
```

### `ruby-build` and `rbenv`

We assume you already have [`ruby-build`](https://github.com/rbenv/ruby-build) installed as a plugin for [`rbenv`](https://github.com/rbenv/rbenv).

First, you need to upgrade `ruby-build` to get the latest TruffleRuby definition.
See [`ruby-build`'s instructions for upgrading](https://github.com/rbenv/ruby-build#upgrading).

On macOS, if `ruby-build` is installed via Homebrew and you do not see the [latest TruffleRuby release](https://github.com/oracle/truffleruby/releases/latest), you might need to install the HEAD version of `ruby-build` with:

```bash
brew reinstall --HEAD ruby-build
```

Check the latest available version of TruffleRuby with:

```bash
rbenv install --list
```

Then install the latest TruffleRuby standalone release with:

```bash
rbenv install truffleruby-[LATEST_VERSION]
```

You can also install the latest nightly standalone build of TruffleRuby with:

```bash
rbenv install truffleruby-dev
```

You can also install TruffleRuby+GraalVM with:

```bash
rbenv install truffleruby+graalvm-[LATEST_VERSION] OR truffleruby+graalvm-dev
```

### `asdf` (with `asdf-ruby` plugin)

See https://github.com/asdf-vm/asdf-ruby for installing and updating `asdf-ruby`.

You can install a TruffleRuby standalone release or nightly build with:

```bash
asdf install ruby truffleruby-VERSION OR truffleruby-dev
```

You can install TruffleRuby+GraalVM with:

```bash
asdf install ruby truffleruby+graalvm-VERSION OR truffleruby+graalvm-dev
```

### `ruby-install` and `chruby`

See https://github.com/postmodern/ruby-install#install for installing and updating `ruby-install`.

First, ensure you have the latest `ruby-install` release.
Check your version with:

```bash
ruby-install --version
```

And compare to the [latest tag](https://github.com/postmodern/ruby-install/tags).
If it is older, you should update to the latest `ruby-install` (e.g. 0.8.4 is necessary for aarch64 support).
Follow the [installation instructions](https://github.com/postmodern/ruby-install#install), since the steps for upgrading `ruby-install` are the same as the steps for installing it.

Then install the latest TruffleRuby standalone release with:

```bash
ruby-install --latest
ruby-install truffleruby
```

You can also install TruffleRuby+GraalVM with:

```bash
ruby-install truffleruby-graalvm
```

`ruby-install` does not support installing nightly builds.
Please use `ruby-build` (which also works with `chruby`) if you want to install nightly builds:

```bash
ruby-build truffleruby-dev ~/.rubies/truffleruby-dev
OR
ruby-build truffleruby+graalvm-dev ~/.rubies/truffleruby+graalvm-dev
```

There are also instructions on the [chruby wiki](https://github.com/postmodern/chruby/wiki/TruffleRuby) if you prefer to install TruffleRuby manually.

## Configuring Ruby Managers for the Full GraalVM Distribution

When [installing GraalVM and Ruby](installing-graalvm.md), it is recommended to add TruffleRuby to a Ruby manager for ease of use.

Make sure you ran the post-install script *before* adding GraalVM to Ruby managers.

Then follow these steps to integrate GraalVM with your Ruby manager.

The first step is to find the TruffleRuby home.
On macOS, run:
```bash
ruby_home=$(path/to/graalvm/Contents/Home/bin/ruby -e 'print RbConfig::CONFIG["prefix"]')
```

On Linux and other platforms, run:
```bash
ruby_home=$(path/to/graalvm/bin/ruby -e 'print RbConfig::CONFIG["prefix"]')
```

### `rbenv`

To add TruffleRuby to `rbenv`, create a symbolic link in the `versions` directory of `rbenv`:

```bash
ln -s "$ruby_home" "$RBENV_ROOT/versions/truffleruby"
rbenv shell truffleruby
ruby --version
```

### `chruby`

To add TruffleRuby to `chruby`, create a symbolic link to the `$HOME/.rubies` directory:

```bash
ln -s "$ruby_home" "$HOME/.rubies/truffleruby"
chruby truffleruby
ruby --version
```

### `rvm`

`rvm` has a command for adding a precompiled Ruby to the list of available rubies:

```bash
rvm mount "$ruby_home" -n truffleruby
rvm use ext-truffleruby
ruby --version
```

### `asdf` (with `asdf-ruby` plugin)

To add TruffleRuby to `asdf`, create a symbolic link in the `.installs/ruby` directory.
You also need to reshim:

```bash
ln -s "$ruby_home" "$HOME/.asdf/installs/ruby/truffleruby"
asdf reshim ruby truffleruby
asdf local ruby truffleruby
ruby --version
```

## Using TruffleRuby without a Ruby Manager

If you are using a Ruby manager like `rvm`, `rbenv`, or `chruby` and wish not to add TruffleRuby to one of them, make sure that the manager does not set environment variables `GEM_HOME` and `GEM_PATH`.
The variables are picked up by TruffleRuby (as any other Ruby implementation would do) causing TruffleRuby to pickup the wrong gem home instead of its own.

It can be fixed for the current terminal by clearing the environment with one of the following commands:

```bash
rbenv system
chruby system
rvm use system
# Or manually:
unset GEM_HOME GEM_PATH
```
