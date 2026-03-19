---
description: How to get started
icon: helmet-safety
---

# Getting started

If you're familiar with the structure of our build system, navigate to the Structure page or use this as a quick reference. If you're new to our build system, you can get started by reading this first, then diving deep to the build system structure.

***

## <mark style="color:$primary;">A very minimal C# application</mark>

To get started with a very minimal C# application with just building and cleaning support, follow these steps (in Visual Studio Code):

{% stepper %}
{% step %}
### <mark style="color:$primary;">Create a minimal C# application</mark>

Follow the steps to create a minimal C# application [here](https://learn.microsoft.com/en-us/dotnet/core/tutorials/with-visual-studio-code). Let's assume that the root C# application name is `NewApp`.
{% endstep %}

{% step %}
### <mark style="color:$primary;">Open the terminal and clone the tools</mark>

Open the terminal by going to **Terminal** > **New Terminal** (or press the <kbd>CTRL</kbd> + <kbd>SHIFT</kbd> + <kbd>\`</kbd> keys).

Then, execute the `git clone -b x/archived/legacy https://github.com/Aptivi/tools` command.

{% hint style="info" %}
If you've initialized the Git repository for your project, you'll have to clone it as a submodule inside the root directory of your project using the `git submodule add -b x/archived/legacy https://github.com/Aptivi/tools` command and to push the commit.
{% endhint %}
{% endstep %}

{% step %}
### <mark style="color:$primary;">Create a vendor directory structure</mark>

First, create the `vnd` directory. Then, underneath the same directory, create three files:

1. `vendor.sh`: This will store functions that define actions
2. `vendor-build.cmd`: This will store commands to initiate the build
3. `vendor-clean.cmd`: This will store commands to initiate the cleanup process, such as wiping older build files
{% endstep %}

{% step %}
### <mark style="color:$primary;">Paste script contents (vendor.sh)</mark>

Replace the contents of `vendor.sh` with:

```bash
#!/bin/bash

build()
{
    dotnet build "$ROOTDIR/NewApp.sln" -p:Configuration=Release
}

clean()
{
    rm -rf "$ROOTDIR/NewApp/bin"
    rm -rf "$ROOTDIR/NewApp/obj"
}
```
{% endstep %}

{% step %}
### <mark style="color:$primary;">Paste script contents (\*.cmd)</mark>

Replace the contents of `vendor-build.cmd` with:

```batch
set ROOTDIR=%~dp0\..
dotnet build "%ROOTDIR%\NewApp.sln" -p:Configuration=Release
```

Replace the contents of `vendor-clean.cmd` with:

```batch
set ROOTDIR=%~dp0\..
rd /s /q "%ROOTDIR%\NewApp\bin"
rd /s /q "%ROOTDIR%\NewApp\obj"
```
{% endstep %}

{% step %}
### <mark style="color:$primary;">Verify the actions work</mark>

Open the terminal emulator on your target platform, change the working directory to the project directory, and execute the build and the clean operations using the following commands:

<details>

<summary>Linux</summary>

```shellscript
$ ./tools/build.sh
$ ./tools/clean.sh
```

</details>

<details>

<summary>Windows</summary>

```
.\tools\build.cmd
.\tools\clean.cmd
```

</details>
{% endstep %}
{% endstepper %}

If there are no errors, you've created your minimal build system that builds the binary and cleans up old build files!

***

## <mark style="color:$primary;">Explore more</mark>

In order to explore more, take a look at the build system structure to learn more!
