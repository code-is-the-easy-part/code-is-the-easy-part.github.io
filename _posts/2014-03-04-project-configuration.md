---
layout: post
title: Project Configuration
---

Project Configuration
================================================================================

What?
--------------------------------------------------------------------------------
How to configure code in dev, qa, and prod environments.

Responsibilities of configuration:
--------------------------------------------------------------------------------
Clarity: Configured properties tell us something about the intent of the program. Values that are configurable are expected to be changed.
Safety: Updating text in a config file is safer than updating code.
Consolidated responsibility: Don't have to hunt through source code to update database name.

dev - development. This is usually run on a developers desktop or laptop. Source code is edited and tested in the environment.
qa - Artifacts are deployed to this environment to allow for access to people other than the developer.
test - Automated tests are run in this environment. This can be a continuous build environment.
prod - End-user facing.

Why?
--------------------------------------------------------------------------------
Every developer needs to think about configuration. Configuration is one of the first things I address when building a new codebase. Configuration can tell your program
* Where to find your database
* What port to bind to
* How long to wait until timing out
* Who to email when there's a problem
* What remote API server to connect to
* What domain name the web server is using
* So much more...

You can run multiple instances of your program on a single host.

A good codebase will have solutions for all kinds of configuration needs. Some things to consider:

* What should the default (no-config) value be?
Code should work out-of-the-box (e.g. straight from the source repository) for development. So the default configuration should always be set for a developer's laptop or desktop environment. After installing all required software, a developer should be able to build and run the software without any special configuration. I recommend this for a few reasons
   * Your deployment process should take care of configuring your program in remote environments. In a developer's environment (dev), there is no deployment (usually), and therefore, no chance to set configuration
   * Money. Engineering talent is expensive and we don't want to waste time with needless configuration
   * It would not be good for a program running in a dev environment program interfere with production data (e.g. a dev program connect to a prod database)

* How can a program be configured in development *and* in deployed environments?
If it is the case that in dev, there is no deployment, we still need a way to configure the program in dev. The approach I recommend here is
   * There is always an app home directory
   * This directory contains an `etc` directory that contains override configuration files
   * In remote (deployed) environments the app is typically deployed into the app home directory
   * In dev, The app home dir only contains the override config

* What is the presedent order for configuration sources?

* Can I apply configuration changes at run-time, or do I need to restart the process?
This is a choice that should be driven by business needs. How often is this needed? Can you get by with full-process restarts to apply new configuration? If you neeed more flexibility, does *all* configuration need to be mutable at runtime?

* What format do my configuration files use? How do I transform configuration data to program data.
   * Java Properties
   * JSON
   * YAML
   * XML
   * Scripting language
The first four (Java Properties, JSON, YAML, XML) are all in the same class. Java Properties are strictly name/value pairs. On the JVM, Java Properties are a great choice because of their easy serialization and deserialization using the `java.util.Properties` class. JSON is great because of it's simplicity and well-known spec. For more structured data, JSON has a big leg up over Java Properties. YAML is similar to JSON. I would not recommend XML as it's most likely overkill.

Using a scripting language for configuration give the added advantage of allowing for dynamic and calculated configuration values. I have never done this, and my guess is that there aren't many cases where this would be a good choice. One place where it would be a good choice would be where responsibilities are separated. Say dev-ops only has access to the local config, they could be given more flexibility by using a scripting language for configuration.

* How to give developers unique values for shared resources E.g. S3 buckets.
There are two general directions to go here:
   * Manual
   Require the developer to specify a unique value before startup. This unique value can be stored in the filesystem and read at startup.

   At Amazon we used our phone extension. Email addresses work well here too.

   * Automatic
   Same as manual, except the unique value is automatically generated. E.g. md5 checksum of the current timestamp. The main difference between the two options (besides manual requiring developer time/thought) is that debugging may be easier with manual as the unique value may be more recognizable.

* How can I be sure test/qa/prod configuration is correct?
* Dynamically driven config. E.g. `controller.model.__key = value`


* Variable expansion in configuration
This is a nice-to-have bordering on necessary. It can cut down on duplication and make configuration updates easier (because of the same principals of splitting large functions into smaller, reusable functions). An example (using Java Properties and Spring's `PropertyPlaceholderHelper`) is:

```
domain = example.com
email = info@${domain}
```

At runtime, email would be expanded to `info@example.com`.

Configuration should live in four different places:
1 Version controlled source in the codebase. E.g. `.properties` files

I typically put these files in a `/src/main/resources` directory whos contents end up in the root of the constructed jar. Any configuration in these files should be suitable for development. E.g. database URLs should point to localhost, S3 buckets should be named in such a way to know that they're for development (and they should probably not conflict with other developers' buckets), etc..

1 Environment-specific version controlled source. E.g. overrides for test, qa, and prod environments

These files live in directories like `/config/qa/etc`, `/config/prod/etc`, etc.. These files are deployed, depending on on the environment being deployed to (i.e. files under `/config/prod` are deployed when deploying to production). Some common fields that are overridden here are API tokens, database URLs, S3 buckets, email addresses, timeout values, and caching policies.

1 Local non-version controlled override files. These files will exist on development laptops and desktops, and qa/test/prod servers.

This is usually reserved for host-specific configuration and quick-fix updates.

1 Command line options. E.g. `-Dxxx.home=/opt/xxx`

One should always be able to override any configuration parameter via the command line at JVM start time.

TODO:
How to share config between shell and runtime.

tl;dr There are at least two environments to configure: `dev` and `prod`. Configuration should be applied in the following order: version controlled source, version controlled environment-specific source, host-specific, start-time (in order of least to most prescedence).
