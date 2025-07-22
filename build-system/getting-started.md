---
description: How to get started
icon: helmet-safety
---

# Getting started

If you're familiar with the structure of our build system, navigate to the Structure page or use this as a quick reference. If you're new to our build system, you can get started by reading this first, then diving deep to the build system structure.

To get started with a very minimal C# application with just building and cleaning support, follow these steps (in Visual Studio Code):

1. Follow the steps to create a minimal C# application [here](https://learn.microsoft.com/en-us/dotnet/core/tutorials/with-visual-studio-code). Let's assume that the root C# application name is `NewApp`.
2. Open the terminal by going to **Terminal** > **New Terminal** (or press the <kbd>CTRL</kbd> + <kbd>SHIFT</kbd> + <kbd>\`</kbd> keys)
3. Execute the `git clone https://gitlab.com/aptivi/main/tools` command
   1. If you've initialized the Git repository for your project, you'll have to clone it as a submodule inside the root directory of your project using the `git submodule add https://gitlab.com/aptivi/main/tools` command and to push the commit.
4. Create the `vnd` directory
5. Underneath the `vnd` directory, create three files:
   1. `vendor.sh`: This will store functions that define actions
   2. `vendor-build.cmd`: This will store commands to initiate the build
   3. `vendor-clean.cmd`: This will store commands to initiate the cleanup process, such as wiping older build files
6.  Replace the contents of `vendor.sh` with:

    ```sh
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
7.  Replace the contents of `vendor-build.cmd` with:

    ```batch
    set ROOTDIR=%~dp0\..
    dotnet build "%ROOTDIR%\NewApp.sln" -p:Configuration=Release
    ```
8.  Replace the contents of `vendor-clean.cmd` with:

    ```batch
    set ROOTDIR=%~dp0\..
    rd /s /q "%ROOTDIR%\NewApp\bin"
    rd /s /q "%ROOTDIR%\NewApp\obj"
    ```
9. Open the terminal emulator on your target platform, change the working directory to the project directory, and execute the build and the clean operations using the following commands:
   1.  If you're running Linux:

       ```shell-session
       $ ./tools/build.sh
       $ ./tools/clean.sh
       ```
   2.  If you're running Windows:

       ```
       .\tools\build.cmd
       .\tools\clean.cmd
       ```

If there are no errors, you've created your minimal build system that builds the binary and cleans up old build files!

In order to explore more, take a look at the build system structure to learn more!
