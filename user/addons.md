---
title: Build Addons
layout: en
permalink: addons/
---

Travis CI allows you to set up some build tools using settings right in your
.travis.yml file.

<div id="toc"></div>

## Sauce Connect

[Sauce Connect][sauce-connect] securely proxies browser traffic between Sauce
Labs' cloud-based VMs and your local servers. Connect uses ports 443 and 80 for
communication with Sauce's cloud. If you're using Sauce Labs for your Selenium
tests, this makes connecting to your webserver a lot easier.

[sauce-connect]: https://saucelabs.com/connect

First, [sign up][sauce-sign-up] with Sauce Labs if you haven't already (it's
[free][open-sauce] for Open Source projects), and get your access key from your
[account page][sauce-account]. Once you have that, add this to your .travis.yml
file:

    addons:
      sauce_connect:
        username: "Your Sauce Labs username"
        access_key: "Your Sauce Labs access key"

[sauce-sign-up]: https://saucelabs.com/signup/plan/free
[sauce-account]: https://saucelabs.com/account
[open-sauce]: https://saucelabs.com/signup/plan/OSS

If you don't want your access key publicly available in your repository, you
can encrypt it with `travis encrypt "your-access-key"` (see [Encryption Keys][encryption-keys]
for more information on encryption), and add the secure string as such:

    addons:
      sauce_connect:
        username: "Your Sauce Labs username"
        access_key:
          secure: "The secure string output by `travis encrypt`"

You can also add the `username` and `access_key` as environment variables if you
name them `SAUCE_USERNAME` and `SAUCE_ACCESS_KEY`, respectively. In that case,
all you need to add to your .travis.yml file is this:

    addons:
      sauce_connect: true

[encryption-keys]: http://docs.travis-ci.com/user/encryption-keys/

To allow multiple tunnels to be open simultaneously, Travis CI opens a
Sauce Connect [Identified Tunnel][identified-tunnels]. Make sure you are sending
the `TRAVIS_JOB_NUMBER` environment variable when you are opening the connection
to Sauce Labs' selenium grid, as the desired capability `tunnel-identifier`,
or it will not be able to connect to the server running on the VM.

[identified-tunnels]: https://saucelabs.com/connect#tunnel-identifier

How this looks will depend on the client library you're using, in
Ruby's [selenium-webdriver][ruby-bindings] bindings:

    caps = Selenium::WebDriver::Remote::Capabilities.firefox({
      'tunnel-identifier' => ENV['TRAVIS_JOB_NUMBER']
    })
    driver = Selenium::WebDriver.for(:remote, {
      url: 'http://username:access_key@ondemand.saucelabs.com/wd/hub',
      desired_capabilities: caps
    })

[ruby-bindings]: https://code.google.com/p/selenium/wiki/RubyBindings

## Firefox

Our VMs come preinstalled with some recent version of Firefox, but sometimes you
need a specific version to be installed. The Firefox addon allows you to specify
any version of Firefox and the binary will be downloaded and installed before
running your build script (as a part of the before_install stage).

If you need version 17.0 of Firefox to be installed, add the following to your
.travis.yml file:

    addons:
      firefox: "17.0"

Please note that this downloads binaries that are only compatible with our
64-bit Linux VMs, so this won't work on our Mac VMs.

## Hosts

If your requires setting up custom hostnames, you can specify a single host or a
list of them in your .travis.yml. Travis CI will automatically setup the
hostnames in `/etc/hosts` for both IPv4 and IPv6.

    addons:
      hosts:
        - travis.dev
        - joshkalderimis.com

## PostgreSQL

Our Linux VMs come preinstalled with three versions of PostgreSQL:
9.1 (default), 9.2 and 9.3. The PostgreSQL addon allows you to specify the
version of PostgreSQL to be started before running your build script (as a part
of the before_install stage).

If you want to use PostgreSQL 9.3 in your tests, add the following to your
.travis.yml file:

    addons:
      postgresql: "9.3"

Please note that this addon is only compatible with our 64-bit Linux VMs,
so this won't work on our Mac VMs.

## Coverity Scan

[Coverity Scan](http://scan.coverity.com) is a free static code analysis tool for Java, C, C++, and C#. It analyzes every line of code and potential execution path and produces a list of potential code defects. By augmenting your CI flow with Coverity Scan, you'll gain further insight into the quality of your code, beyond that which is covered by your automated tests.

This addon leverages the Travis-CI infrastructure to automatically run code analysis on your GitHub projects.

### What is static analysis?

Static analysis is a set of processes for finding source code defects and vulnerabilities.

In static analysis, the code under examination is not executed. As a result, test cases and specially designed input datasets are not required. Examination for defects and vulnerabilities is not limited to the lines of code that are run during some number of executions of the code, but can include all lines of code in the codebase.

Additionally, Coverity's implementation of static analysis can follow all the possible paths of execution through source code (including interprocedurally) and find defects and vulnerabilities caused by the conjunction of statements that are not errors independent of each other.

See more details about Coverity Scan in the [FAQ](https://scan.coverity.com/faq).

### Build Submission Frequency

It's probably overkill to run static analysis on each and every commit of your project. To increase availability of the free service to more projects, the addon is designed by default to run analysis on a per-branch basis. We recommend you create a branch named `coverity_scan`, which you can merge into whenever you would like to trigger analysis. See the [FAQ](https://scan.coverity.com/faq#frequency) for information about build submission frequency.

### Step-by-step Configuration

1. [Sign up](http://scan.coverity.com/users/sign_up) with Coverity Scan using your GitHub account if you haven't already.

1. If necessary, create a public repo on [GitHub](https://github.com) for your project.

1. If necessary, register for [Travis CI](https://travis-ci.org/) and configure your project by following the [Getting Started](http://docs.travis-ci.com/user/getting-started/) guide.

1. Sign in to Scan, and then add your project. Be sure to add it as a [GitHub Project](https://scan.coverity.com/projects/new?tab=github).

1. (Optional) The first time you use Coverity Scan with your project, you may want to do a build on a development machine of your own to be sure everything completes properly. This is optional but it will ease any necessary debugging. Consult the Coverity Scan [download page](https://scan.coverity.com/download) for instructions.

1. From the Coverity Scan Dashboard, click `Project Settings`. Then, on the right, click the `Submit build` button.

1. Assuming the project is properly registered via GitHub, you'll see a tab for `Configure Travis CI`. Visit that panel.

1. On the Travis CI Confuration page, you'll see a sample `.travis-yml` file. It will have auto-generated several of the necessary project-specific fields, including the encrypted Coverity Scan token (necessary to upload results). Copy the configuration from there into your `.travis.yml` file, making the changes in the next steps.

1. Fill in `build_command_prepend` with any configuration commands that are necessary to prepare your project to be built. For example `build_command_prepend: ./configure`

1. Fill in `build_command` with the command to build your project. This will be supplied as an argument to the `cov-build` command. For example: `build_command: make`

1. Commit those changes to the `coverity_scan` branch.

1. Push the `coverity_scan` branch to GitHub. Wait a few moment for Travis CI to see the commit, and for it to begin the build.

1. Visit [Travis CI](https://travis-ci.org) directly, or by clicking the button on your `Project Settings` page, which will appear once the project is activated on Travis CI.

#### travis.yml

From your project page on Coverity Scan, select the Travis CI tab. You'll see a snippet of YAML to be copied over to your `.travis-ci` file. Note that this is an example, and might require some tweaking for the build to run properly.

    env:
      global:
        # COVERITY_SCAN_TOKE
        # ** specific to your project **
        - secure: "xxxx"

    addons:
      coverity_scan:

        # GitHub project metadata
        # ** specific to your project **
        project:
          name: my_github/my_project
          version: 1.0
          description: My Project

        # Where email notification of build analysis results will be sent
        notification_email: scan_notifications@example.com

        # Commands to prepare for build_command
        # ** likely specific to your build **
        build_command_prepend: ./configure

        # The command that will be added as an argument to "cov-build" to compile your project for analysis,
        # ** likely specific to your build **
        build_command: make

        # Pattern to match selecting branches that will run analysis. We recommend leaving this set to 'coverity_scan'.
        # Take care in resource usage, and consider the build frequency allowances per
        #   https://scan.coverity.com/faq#frequency
        branch_pattern: coverity_scan

The project settings should be self-explanatory, and should match the values for the project configuration on Coverity Scan. The `branch_pattern` is a regular expression for the branches on which you want to run Coverity Scan. Please refer to the [FAQ](https://scan.coverity.com/faq) regarding build submission limits before enabling additional branches. We recommend leaving this set to `coverity_scan`.

The COVERITY_SCAN_TOKEN is encrypted and is obtained by using the [Travis CI CLI](https://github.com/travis-ci/travis). Coverity Scan provides this information on your Project's Travis CI tab for convenience, but you may also run it manually (see [Encryption Keys](http://docs.travis-ci.com/user/encryption-keys/) for more information on encryption).

    gem install travis
    cd my_project
    travis encrypt COVERITY_SCAN_TOKEN=project_token_from_coverity_scan

Then copy the resulting line as shown in the YAML example.

### Execution

The next time you commit to the appropriate branch, the Coverity Scan build process will automatically run analysis and upload the results. Please note that this analysis takes the place of the normal CI run. You should merge the same changes to another branch to run your tests.

#### Disabling the Subsequent Test Run

Due to the way that Travis CI addons operate, your standard script stage (i.e. your tests) will run after the Coverity Scan analysis completes. In order to avoid this, you can modify your `script` directive in `.travis.yml`.

The `COVERITY_SCAN_BRANCH` environment variable will be set to `1` when the Coverity Scan addon is in operation. Therefore, you might change your script from

    script: make

to

    script: if [ ${COVERITY_SCAN_BRANCH} != 1 ]; then make ; fi

Be sure to replace `make` with your standard CI build command.