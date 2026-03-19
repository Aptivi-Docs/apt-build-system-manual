---
description: How does this system work?
icon: trowel-bricks
---

# Structure

Our build and pack system follows the structure that is outlined below to give you the ability to create a powerful build system for your projects to ensure that you provide developers short commands to build the project and to simplify the complicated build operations.

***

## <mark style="color:$primary;">Script structure</mark>

The below structure is required for every project that uses this build toolset. Select the directory to get a list of the required filesystem hierarchy.

{% hint style="info" %}
How the project calls the build scripts is entirely up to the project and not to a standard Makefile that makes use of those scripts found in the tools directory.
{% endhint %}

{% tabs %}
{% tab title="Tools directory" %}
You'll need to either clone `tools` as a submodule or as a git repository.

| File name                          | Description                                                                                                              |
| ---------------------------------- | ------------------------------------------------------------------------------------------------------------------------ |
| `build.cmd / build.sh`             | Build script that builds a project.                                                                                      |
| `clean.cmd / clean.sh`             | Clean script that cleans a project source.                                                                               |
| `common.cmd / common.sh`           | Common script to help the individual scripts execute the vendor script.                                                  |
| `docgen.cmd / docgen.sh`           | Documentation generation script that generates the project docs folder.                                                  |
| `docgen-pack.cmd / docgen-pack.sh` | Documentation packing script that creates a compressed file from the project docs folder for distribution.               |
| `increment.cmd / increment.sh`     | Version incrementation script that finds all files that contain the project version and replaces the version.            |
| `localize.cmd / localize.sh`       | Dependency vendoring script that downloads all project dependencies and configures the build system to use them locally. |
| `pack.cmd / pack.sh`               | Artifact packing script that creates a compressed file from the project output directory containing final binaries.      |
| `push.cmd / push.cmd`              | Artifact pushing script that publishes the project's final binaries online.                                              |
{% endtab %}

{% tab title="Vendor directory" %}
You'll need to create a `vnd` directory with any of the vendor script files, as desired:

| File name             | Description                                                                                |
| --------------------- | ------------------------------------------------------------------------------------------ |
| `vendor.sh`           | Vendor script for Linux containing all functions and commands for different build actions. |
| `vendor-build.sh`     | Vendor build script executed from the tool's script.                                       |
| `vendor-clean.sh`     | Vendor clean script executed from the tool's script.                                       |
| `vendor-docgen.sh`    | Vendor docs generation script executed from the tool's script.                             |
| `vendor-docpack.sh`   | Vendor docs packing script executed from the tool's script.                                |
| `vendor-increment.sh` | Vendor version incrementation script executed from the tool's script.                      |
| `vendor-localize.sh`  | Vendor dependency vendoring script executed from the tool's script.                        |
| `vendor-pack.sh`      | Vendor packing script executed from the tool's script.                                     |
| `vendor-push.cmd`     | Vendor pushing script executed from the tool's script.                                     |
{% endtab %}
{% endtabs %}

***

## <mark style="color:$primary;">Vendor scripts</mark>

The vendor script creation instructions are provided for both Windows and Unix-based operating systems, such as macOS. Select your operating system below:

<details>

<summary>Windows</summary>

You'll have to create the vendor batch files as mentioned underneath the `vnd` directory as needed. For example, if you don't want to push anything to the package registry, you can omit the `vendor-push.cmd` file and the base push script, `push.cmd`, will not see it.

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

### <mark style="color:$primary;">Commands before and after action</mark>

For Windows scripts, you can optionally add new files that have a prefix of either `vendor-pre` or `vendor-post` for all actions. The following examples are:

* `vendor-prebuild.cmd` for pre-build actions
* `vendor-postpack.cmd` for post-pack actions

</details>

<details>

<summary>Linux/macOS</summary>

For Linux and macOS systems, you'll have to create a single executable Bash script, called `vendor.sh` under the `vnd` folder. This is meant to be only sourced and not to be executed. It contains only the function definitions that can be omitted:

* `build()`
* `clean()`
* `docgenerate()`
* `docpack()`
* `increment()`
* `localize()`
* `packall()`
* `pushall()`

If a function has been omitted, they will be defined as a void function that immediately returns `0`, which means success. Those functions, therefore, will not be executed. Inside them, you can define what commands you can run.

{% hint style="info" %}
The best practice for making vendor scripts in Linux is to follow the below rules:

* For critical errors, you can use the `checkerror` function directly after the command to be executed that may throw such errors, such as dependencies.
* For continuable errors, you can use the `checkvendorerror` function directly after the command to be executed that may throw such errors, such as download errors.
* You can use the `$ROOTDIR` variable to get a path to the repository root, that is, the parent directory of the vendor script directory, `vnd`.
{% endhint %}

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

### <mark style="color:$primary;">Commands before and after action</mark>

For Unix scripts in `vendor.sh`, you can add new functions that have a prefix of either `pre` or `post` for all actions. The following examples are:

* `prebuild()` for pre-build actions
* `postpackall()` for post-pack actions

</details>
