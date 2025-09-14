---
description: How does this system work?
icon: trowel-bricks
---

# Structure

Our build and pack system follows the structure that is outlined below to give you the ability to create a powerful build system for your projects to ensure that you provide developers short commands to build the project and to simplify the complicated build operations.

The following structure is required for every project that uses this build toolset:

* Project directory
  * `tools` (directory, cloned as a git submodule)
    * `build.cmd` (file, Build script for Windows)
    * `build.sh` (file, Build script for Linux and macOS)
    * `clean.cmd` (file, Clean script for Windows)
    * `clean.sh` (file, Clean script for Linux and macOS)
    * `common.cmd` (file, Common script for Windows, used by all scripts)
    * `common.sh` (file, Common script for Linux and macOS, used by all scripts)
    * `docgen.cmd` (file, Documentation generation script for Windows)
    * `docgen.sh` (file, Documentation generation script for Linux and macOS)
    * `docgen-pack.cmd` (file, Documentation packing script for Windows)
    * `docgen-pack.sh` (file, Documentation packing script for Linux and macOS)
    * `increment.cmd` (file, Version incrementation script for Windows)
    * `increment.sh` (file, Version incrementation script for Linux and macOS)
    * `localize.cmd` (file, Dependency vendoring script for Windows)
    * `localize.sh` (file, Dependency vendoring script for Linux and macOS)
    * `pack.cmd` (file, Artifact packing script for Windows)
    * `pack.sh` (file, Artifact packing script for Linux and macOS)
    * `push.cmd` (file, Artifact pushing script for Windows)
    * `push.sh` (file, Artifact pushing script for Linux and macOS)
  * `vnd` (directory, you'll have to create the below files and this directory yourself)
    * `vendor.sh` (file, vendor script for Linux and macOS)
    * `vendor-build.cmd` (file, list of commands for build for Windows)
    * `vendor-clean.cmd` (file, list of commands for cleanup for Windows)
    * `vendor-docgen.cmd` (file, list of commands for documentation generation for Windows)
    * `vendor-docpack.cmd` (file, list of commands for documentation packing for Windows)
    * `vendor-increment.cmd` (file, list of commands for version incrementation for Windows)
    * `vendor-localize.cmd` (file, list of commands for dependency vendoring for Windows)
    * `vendor-pack.cmd` (file, list of commands for artifact packing for Windows)
    * `vendor-push.cmd` (file, list of commands for pushing to package registry for Windows)

For Windows scripts, you can optionally add new files that have a prefix of either `vendor-pre` or `vendor-post` for all actions, such as `vendor-prebuild.cmd` for pre-build actions, or `vendor-postpack.cmd` for post-pack actions.

{% hint style="info" %}
Dependency vendoring may be required for offline builds, especially .NET projects that use NuGet to fetch their dependencies. In this case, you'll have to implement the `localize` function.
{% endhint %}

How the project calls the build scripts is entirely up to the project and not to a standard Makefile that makes use of those scripts found in the tools directory.

### Vendor scripts

The following vendor scripts are provided for both Windows and Unix-based operating systems, such as macOS:

#### Windows

For Windows systems, you'll have to create the vendor batch files as mentioned underneath the `vnd` directory as needed. For example, if you don't want to push anything to the package registry, you can omit the `vendor-push.cmd` file and the base push script, `push.cmd`, will not see it.

When an error occurs, store the error code value, print a helpful error message, then exit with the same error code. For example, BassBoom's `vendor-push.cmd` uses the following statements to stop pushing the rest of the packages upon encountering any error:

```batch
if !errorlevel! neq 0 (
    set error=!errorlevel!
    echo !package_path[%%i]! (!error!^)
    exit /b !error!
)
```

Inside those batch files, the least you'll need is this:

```batch
@echo off

set ROOTDIR=%~dp0\..

REM Your commands here
```

For example, BassBoom has a vendor script for building the project, called `vendor-build.cmd`, that is defined like this:

```batch
@echo off

REM This script builds and packs the artifacts.
set releaseconfig=%1
if "%releaseconfig%" == "" set releaseconfig=Release

set buildoptions=%*
call set buildoptions=%%buildoptions:*%1=%%
if "%buildoptions%" == "*=" set buildoptions=

REM Turn off telemetry and logo
set DOTNET_CLI_TELEMETRY_OPTOUT=1
set DOTNET_NOLOGO=1

set ROOTDIR=%~dp0\..

echo Building with configuration %releaseconfig%...
"%ProgramFiles%\dotnet\dotnet.exe" build "%ROOTDIR%\BassBoom.sln" -p:Configuration=%releaseconfig% %buildoptions%
```

{% hint style="info" %}
You can process the arguments as usual, because the scripts found in the `tools` directory handle the arguments to customize the build further by the end developer. Just follow the instructions on how to get the arguments.
{% endhint %}

#### Linux/macOS

For Linux and macOS systems, you'll have to create a single executable Bash script, called `vendor.sh` under the `vnd` folder. This is meant to be only sourced and not to be executed. It contains only the function definitions that can be omitted:

* `build()`
* `clean()`
* `docgenerate()`
* `docpack()`
* `increment()`
* `localize()`
* `packall()`
* `pushall()`

{% hint style="info" %}
For Linux scripts, you can optionally add new functions that have a prefix of either `pre` or `post` for all actions, such as `prebuild()` for pre-build actions, or `postpack()` for post-pack actions.
{% endhint %}

If a function has been omitted, they will be defined as a void function that immediately returns `0`, which means success. Those functions, therefore, will not be executed. Inside them, you can define what commands you can run.

The best practice for making vendor scripts in Linux is to follow the below rules:

* For critical errors, you can use the `checkerror` function directly after the command to be executed that may throw such errors, such as dependencies.
* For continuable errors, you can use the `checkvendorerror` function directly after the command to be executed that may throw such errors, such as download errors.
* You can use the `$ROOTDIR` variable to get a path to the repository root, that is, the parent directory of the vendor script directory, `vnd`.

Afterwards, you can define the functions inside the `vendor.sh` file:

```sh
#!/bin/bash

prebuild() {
    # Your commands here (can be omitted)
}

build() {
    # Your commands here (can be omitted)
}

postbuild() {
    # Your commands here (can be omitted)
}

# ...and so on.
```

For example, BassBoom defines the build function like this:

```sh
#!/bin/bash

build() {
    # Determine the release configuration
    releaseconf=$1
    if [ -z $releaseconf ]; then
	    releaseconf=Release
    fi

    # Now, build.
    echo Building with configuration $releaseconf...
    "$dotnetpath" build "$ROOTDIR/BassBoom.sln" -p:Configuration=$releaseconf ${@:2}
    checkvendorerror $?
}
```

{% hint style="info" %}
You can process the arguments as usual, because the scripts found in the `tools` directory handle the arguments to customize the build further by the end developer. Just follow the instructions on how to get the arguments.
{% endhint %}

{% hint style="warning" %}
Beware that the vendor script should contain the shebang at the top of the script.
{% endhint %}
