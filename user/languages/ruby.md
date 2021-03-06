---
title: Building a Ruby Project
layout: en
permalink: ruby/
---
<div id="toc">
</div>

### What This Guide Covers

This guide covers build environment and configuration topics specific to Ruby
projects. Please make sure to read our [Getting
Started](/user/getting-started/) and [general build
configuration](/user/build-configuration/) guides first.

### Supported Ruby Versions

Currently pre-installed on our systems are the latest patchlevel releases of the
following Rubies:

- MRI 2.1.0
- MRI 2.0.0
- MRI 1.9.3
- MRI 1.9.2
- MRI 1.8.7 
- JRuby 1.7.9 in Ruby 1.8 mode
- JRuby 1.7.9 in Ruby 1.9 mode
- Ruby Enterprise Edition 1.8.7 2012.02
- Ruby (latest master)
- JRuby (latest master)

Ruby and JRuby master are rebuilt automatically every six hours. These and a
couple more Rubies are available for automatic download via
<https://rubies.travis-ci.org>.

The same service also provides Ruby Enterprise Edition 1.8.7 2011.12.

### Choosing Ruby versions and implementations to test against

The Ruby environment on Travis CI uses [RVM](https://rvm.io/) to provide many
Ruby implementations and versions your projects can be tested against.

To specify them, use `rvm:` key in your `.travis.yml` file, for example:

    language: ruby
    rvm:
      - 1.9.3
      - jruby-18mode # JRuby in 1.8 mode
      - jruby-19mode # JRuby in 1.9 mode
      - rbx-2.1.1
      - 1.8.7

A more extensive example:

    language: ruby
    rvm:
      - 2.1.0
      - jruby-18mode
      - jruby-19mode
      - rbx
      - ruby-head
      - jruby-head
      - ree

As time goes, new releases come out and we upgrade both RVM and Rubies, aliases
like `1.9.3` or `jruby` will float and point to different exact versions, patch
levels and so on. For full up-to-date list of provided Rubies, see our [CI
environment guide](/user/ci-environment/).

If you don't specify a version, Travis CI will use MRI 1.9.3 as the default.

### Choosing Rubies that aren't installed

While we pre-install some Rubies, you can install other versions by way of RVM's
binary installation feature.

As long as they're available as a binary for Ubuntu 12.04, you can specify
custom patchlevels.

    language: ruby
    rvm:
      - 2.0.0-p247

Note that this binds you to potentially unsupported releases of Rubies. It also
extends your build time as downloading and installing a custom Ruby can add an
extra 60 seconds or more to your build.

### Rubinius

[Rubinius](http://rubini.us) releases frequent updates which can be found on the
Rubinius [downloads](http://rubini.us/downloads/) page. Binaries for Travis CI
should be available for every release from 2.1.1 onwards. To test against a
release of Rubinius, add `rbx-X.Y.Z` to your `.travis.yml`, where X.Y.Z
specifies a Rubinius release.

Note that the syntax of `rbx-19mode` isn't supported anymore.

Rubinius will be installed via a download currently.

As of Rubinius 2.2, some special treatment is required in your build
configuration to make Ruby projects work.

In particular, Rubinius relies on a Ruby Standard Library installed as a
RubyGem. It's not bundled with Rubinius anymore and needs to be installed
separately.

To get your project ready for Rubinius, you'll need to add some libraries to
your Gemfile. You can specify a separate platform so they're only installed for
Rubinius:

    platforms :rbx do
      gem 'racc'
      gem 'rubysl', '~> 2.0'
      gem 'psych'
    end

Thanks to [Benjamin
Fleischer](http://www.benjaminfleischer.com/2013/12/05/testing-rubinius-on-travis-ci/)
for collecting information about how to get Rubinius up and running with the
recent changes in the project.

On [his blog
post](http://www.benjaminfleischer.com/2013/12/05/testing-rubinius-on-travis-ci/),
you'll find more examples specific to what your library might require from
Ruby's standard set of tools.

### JRuby: C extensions support is disabled

Please note that **C extensions are disabled for JRuby** on Travis CI. The
reason for doing so is to bring it to developers attention that their project
may have dependencies that should not be used on JRuby in production. Using C
extensions on JRuby is technically possible but is not a good idea performance
and stability-wise and we believe continuous integration services like Travis
should highlight it.

So if you want to run CI against JRuby, please check that your Gemfile takes
JRuby into account. Most of popular C extensions these days also have Java
implementations (json gem, nokogiri, eventmachine, bson gem) or Java
alternatives (like JDBC-based drivers for MySQL, PostgreSQL and so on).

## Default Test Script

Travis CI runs `rake` by default to execute your tests. Please note that **you
need to add rake to your Gemfile** (adding it to just the `:test` group should
be sufficient).

## Dependency Management

### Travis CI uses Bundler

Travis CI uses [Bundler](http://bundler.io/) to install your project's
dependencies.

The default command run by Travis CI is:

    bundle install

Note that this is only run when we detect a Gemfile in the project's root
directory, or if the Gemfile specified via the build matrix exists.

If a Gemfile.lock exists in your project's root directory, we add the
`--deployment` flag.

If you want to use a different means of handling your Ruby project's
dependencies, you can override the `install` command.

    install: gem install rails

By default, gems are installed into vendor/bundle in your project's root
directory.

### Caching Bundler

Bundler installation can take a while, slowing down your build. You can tell
[Travis CI to cache the installed bundle](/user/caching/).

On your first build, we warm the cache. On the second one, we'll pull in the
cache, making `bundle install` only take seconds to run.

Note that this feature is currently only available for private projects.

### Speed up your build by excluding non-essential dependencies

Lots of project include libraries like `ruby-debug`, `unicorn` or `newrelic_rpm`
in their default set of gems.

This slows down the installation process quite a lot, and commonly, those
libraries aren't needed when running your tests. This also includes libraries
that compile native code, slowing down the installation and overall test times
even more.

On top of that, libraries that implicitly pull in `ruby_core_source` or
`linecache19`, are bound to fail when Travis CI upgrades Ruby versions and
patchlevels.

The same is true for gems that you only need in production, like Unicorn, the
New Relic library, and the like.

You can speed up your installation process by moving these libraries to a
separate section in your .travis.yml, e.g. `production`:

    group :production do
      gem 'unicorn'
      gem 'newrelic_rpm'
    end

Adjust your Bundler arguments to explicitly exclude this group:

    bundler_args: --without production

Enjoy a faster build, which is also less prone to compilation problems.

### Custom Bundler arguments and Gemfile locations

You can specify a custom Gemfile name:

    gemfile: gemfiles/Gemfile.ci

Unless specified, the worker will look for a file named "Gemfile" in the root of
your project.

You can also set <a
href="http://bundler.io/v1.3/man/bundle-install.1.html">extra arguments</a> to
be passed to `bundle install`:

    bundler_args: --binstubs

You can also define a script to be run before 'bundle install':

    before_install: some_command

For example, to install and use the pre-release version of bundler:

    before_install: gem install bundler --pre

### Testing against multiple versions of dependencies (Ruby on Rails, EventMachine, etc)

Many projects need to be tested against multiple versions of Rack, EventMachine,
HAML, Sinatra, Ruby on Rails, you name it. It is easy with Travis CI. What you
have to do is this:

* Create a directory in your project's repository root where you will keep
  gemfiles (./gemfiles is a commonly used name)
* Add one or more gemfiles to it
* Instruct Travis CI to use those gemfiles using the *gemfile* option in your
  .travis.yml

For example, amqp gem is [tested against EventMachine 0.12.x and 1.0
pre-releases](https://github.com/ruby-amqp/amqp/blob/master/.travis.yml):

    gemfile:
      - Gemfile
      - gemfiles/eventmachine-pre

Thoughtbot's Paperclip is [tested against multiple ActiveRecord
versions](https://github.com/thoughtbot/paperclip/blob/master/.travis.yml):

    gemfile:
      - gemfiles/rails2.gemfile
      - gemfiles/rails3.gemfile
      - gemfiles/rails3_1.gemfile

An alternative to this is to use environment variables and make your test runner
use them. For example, [Sinatra is tested against multiple Tilt and Rack
versions](https://github.com/sinatra/sinatra/blob/master/.travis.yml):

    env:
      - "rack=1.3.4"
      - "rack=master"
      - "tilt=1.3.3"
      - "tilt=master"

ChefSpec is [tested against multiple Opscode Chef
versions](https://github.com/acrmp/chefspec/blob/master/.travis.yml):

    env:
      - CHEF_VERSION=0.9.18
      - CHEF_VERSION=0.10.2
      - CHEF_VERSION=0.10.4

Same technique is often applied to test against multiple databases, templating
engines, [hosted] service providers and so on.

## Testing against multiple JDKs (JRuby)

It is possible to test projects against multiple JDKs, namely

 * OpenJDK 7
 * Oracle JDK 7
 * Oracle JDK 8 EA
 * OpenJDK 6

To do so, use the `jdk` key in your `.travis.yml`, for example:

    jdk:
      - oraclejdk7
      - openjdk7

or all 3:

    jdk:
      - openjdk7
      - oraclejdk7
      - openjdk6

Each JDK you test against will create permutations with all other
configurations, so to avoid running tests for, say, CRuby 1.9.3 multiple times
you need to add some matrix excludes (described in our general [Build
Configuration guide](/user/build-configuration/)):

    language: ruby
    rvm:
      - 1.9.2
      - jruby-18mode
      - jruby-19mode
      - jruby-head
    jdk:
      - openjdk6
      - openjdk7
      - oraclejdk7
    matrix:
      exclude:
        - rvm: 1.9.2
          jdk: openjdk6
        - rvm: 1.9.2
          jdk: openjdk7
        - rvm: 1.9.2
          jdk: oraclejdk7

For example, see
[travis-support](https://github.com/travis-ci/travis-support/blob/master/.travis.yml).

## Upgrading RubyGems

The RubyGems version installed on Travis CI's Ruby environment depends on what's
installed by the newest Bundler/RubyGems combination.

We try to keep it as up-to-date as possible.

However, should you require the latest version of RubyGems, you can add the
following to your .travis.yml:

    before_install:
      - gem update --system
      - gem --version


If you need to downgrade to a specific version, you can use the following steps:

    before_install:
      - gem update --system 2.1.11
      - gem --version

Note that this will impact your overall test time, as additional network
downloads and installations are required.

## Build Matrix

For Ruby projects, `env`, `rvm`, `gemfile`, and `jdk` can be given as arrays to
construct a build matrix.
