---
description: How to get started
icon: helmet-safety
---

# Getting started

To get started with a very minimal C# application with just building and cleaning support, follow these steps (in Visual Studio Code):

{% stepper %}
{% step %}
### <mark style="color:$primary;">Create a minimal C# application</mark>

Follow the steps to create a minimal C# application [here](https://learn.microsoft.com/en-us/dotnet/core/tutorials/with-visual-studio-code). Let's assume that the root C# application name is `NewApp`.
{% endstep %}

{% step %}
### <mark style="color:$primary;">Open the terminal</mark>

Open the terminal by going to **Terminal** > **New Terminal** (or press the <kbd>CTRL</kbd> + <kbd>SHIFT</kbd> + <kbd>\`</kbd> keys)
{% endstep %}

{% step %}
### <mark style="color:$primary;">Clone the tools repository</mark>

Execute the `git clone https://github.com/Aptivi/tools` command.

{% hint style="info" %}
If you've initialized the Git repository for your project, you'll have to clone it as a submodule inside the root directory of your project using the `git submodule add https://github.com/Aptivi/tools` command and to push the commit.
{% endhint %}
{% endstep %}

{% step %}
### <mark style="color:$primary;">Create a directory structure</mark>

First, create the `vnd` directory. Then, underneath the `vnd` directory, create three files:

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
### <mark style="color:$primary;">Test the scripts</mark>

Open the terminal emulator on your target platform, change the working directory to the project directory, and execute the build and the clean operations using the following commands:

*   If you're running Linux:

    ```console
    $ ./tools/build.sh
    $ ./tools/clean.sh
    ```
*   If you're running Windows:

    ```
    .\tools\build.cmd
    .\tools\clean.cmd
    ```
{% endstep %}
{% endstepper %}

If there are no errors, you've created your minimal build system that builds the binary and cleans up old build files!

{% hint style="info" %}
In order to explore more, take a look at the build system structure to learn more!
{% endhint %}
