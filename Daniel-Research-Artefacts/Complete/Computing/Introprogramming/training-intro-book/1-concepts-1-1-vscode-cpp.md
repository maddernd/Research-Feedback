---
title: Configuring VS Code
sidebar:
  label: " - Configuring VS Code"
---

Getting the program compiling and running in the terminal will mean that you now know the steps an editor like Visual Studio Code would need to automate for us.

Visual Studio Code uses [extensions](https://code.visualstudio.com/docs/editor/extension-marketplace) to extend its capabilities to support developers working in a range of languages. To get support with C/C++ we need to install the associated extensions and then configure their settings to get them to link in the required libraries.

The extension you need to install is called the [C/C++ Extension Pack](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cpptools-extension-pack) by Microsoft. Visual Studio Code should automatically show you this in the extensions' pane if you have the program.cpp file open. Click the **install** button to install this extension in your editor.

![Visual Studio Code with extensions showing the C/C++ extension to install](./images/vscode-extension.png)

:::note
Don't panic if it takes a little time for this extension to install and configure itself. It may take a few minutes until the extension is fully activated. The debug button and configuration cog will appear once the extension is fully activated.
:::

With the extension installed, you now need to add some files to configure this to work with your code. The extension will have added a quick configure button that will help get you started. With your **program.cpp** file selected, click the **debug/play** icon near the top right of the window as highlighted in the following image. This button can be used to compile and run your program.

When you click the *debug/play* icon, it will open a prompt where you can select the compiler you want to use. You can choose either the **g++** or the **clang++** compiler. While the clang compiler is our preferred compiler, it is newer and isn't yet fully integrated into VS Code across all platforms. As a result, you will need to use g++ and its debugger (**gdb**) on Linux and Windows. On macOS, you should use clang++ and its debugger (**lldb**).

The image below shows the options on macOS. This lists both the clang++ and g++ compiler options. Select the appropriate option based on the operating system you are using.

![The settings icon to add the C++ build task](./images/vscode-build-task.png)

This will create a **tasks.json** files in a hidden **.vscode** folder, and will then *attempt* to build your program. At the moment, this will fail as we need to tell the compiler to link in the SplashKit library. You should get an error as shown in the image below. Click the **Abort** or the **Show Errors** button, and then you can adjust the settings to get this to compile. You should be able to see the error message in the Terminal, so you can check what went wrong there.

![Error message shown when you try to run](./images/build-error.png)

:::tip[Hidden Folders]
Remember that folders that start with a `.` are hidden in unix. They do not appear in `ls` commands unless you add the `-a` option.
:::

### Configuring the Build Task

The **tasks.json** file contains various settings related to compiling the program, as shown below. We need to provide this with the additional options we provided to the compiler when we compiled our program through the terminal. The **args** setting is a list of the values to pass as arguments to the compiler. You need to extend this with extra options to link the SplashKit library as shown below.

```json
{
  "tasks": [
    {
      "type": "cppbuild",
      "label": "C/C++: g++ build active file",
      "command": "/usr/bin/g++", // It will be clang++ here on macOS
      "args": [
        "-fdiagnostics-color=always",
        "-g",
        "${file}",
        "-l", // add the -l switch
        "SplashKit", // followed by the SplashKit library
        "-o",
        "${fileDirname}/${fileBasenameNoExtension}"
      ],
      "options": {
        "cwd": "${fileDirname}"
      },
      "problemMatcher": [
        "$gcc"
      ],
      "group": {
        "kind": "build",
        "isDefault": true
      },
      "detail": "Task generated by Debugger."
    }
  ],
  "version": "2.0.0"
}
```

Each argument is added as a separate string within quotes, separated by commas.
So, we added the following lines to our **tasks.json**:

```json
        "-l",
        "SplashKit",
```

:::note[MSys?]
For MSys, you will need to change the path in the command attribute to the use the Windows path to your g++.exe program. This will be in your msys64, mingw64 folder. The snippet below shows the path to where this is installed by default:

```json
{
  "tasks": [
    {
      // as before
      "command": "C:\\msys64\\mingw64\\bin\\g++.exe",
      // as before
    }
  ],
  "version": "2.0.0"
}
```

:::

These settings tell VS Code how to build a selected C++ file that uses the SplashKit library. With your *program.cpp* file selected (i.e. open and with the cursor in it) you can build the program using **ctrl-shift-b** (**cmd-shift-b** on macOS) or selecting **Tasks: Run Build Task** from the [command palette](https://code.visualstudio.com/docs/getstarted/userinterface#_command-palette).

In the terminal window in VS Code you should see the output from this, as shown below. Make sure that you can see that this ran successfully.

```zsh
 *  Executing task: C/C++: g++ build active file 

Starting build...
/usr/bin/g++ -std=gnu++14 -fcolor-diagnostics -fansi-escape-codes -g /Users/acain/Documents/Code/Test2/program.cpp -o /Users/acain/Documents/Code/Test2/program -l SplashKit

Build finished successfully.
 *  Terminal will be reused by tasks, press any key to close it. 
```

At this point, you have Visual Studio Code set up to build the currently selected file into a program.

Going forward, you can edit the tasks.json to compile multiple files. To do this you need to change the argument that passes in what to compile, and the output option in the following ways:

- Change **"${file}"** to **"*.cpp"**. This will then compile all cpp files.
- The **"-o"** argument is followed by the name of the program to create. This is **"${fileDirname}/${fileBasenameNoExtension}"** which names the program after the selected file. When building multiple files you can provide a fixed value here like **"program"**. This will then always build the same output regardless of which source file is selected.

### Configuring Launch

If you have your computer setup correctly, you should **_not_** need to configure the way VS Code launches the debugger. However, should you need to adjust any of these settings you can. You can create the file manually in the *.vscode* folder, or click the **cog** next to the *debug/play* button and select the compiler you selected previously. Using the cog button will create an empty **launch.json** file (and will also create *tasks.json* if that does not exist). You use the *launch.json* file to configure how to run your program.

:::caution

If you use the cog icon to create the *launch.json* file, it may reset *tasks.json*. If you try to run and it fails to build, check *tasks.json* to see that the libraries are still being linked.

:::

Configuring VS Code to run your C/C++ program is much simpler than configuring the settings needed for compiling. When you open the *launch.json* it may have defaults already included, which are likely to be ok. If not, you click the **Add Configuration** button and select the appropriate C/C++ option to launch the program using the debugger. In the image below we have selected to launch with *lldb*, but you shuld choose *gdb* if you are using Windows and Linux.

![Adding a configuration to launch.json](./images/launch-json.png)

We have an example configuration using *gdb* below. The important settings to check are the **program** setting - which should have the same details as you set in the output option of the compiler.

```json
{
  "configurations": [
    {
      "name": "C/C++: g++ build and debug active file",
      "type": "cppdbg",
      "request": "launch",
      "program": "${fileDirname}\\${fileBasenameNoExtension}.exe",
      "args": [],
      "stopAtEntry": false,
      "cwd": "${fileDirname}",
      "environment": [],
      "externalConsole": false,
      "MIMode": "gdb",
      "miDebuggerPath": "C:\\msys64\\mingw64\\bin\\gdb.exe",
      "setupCommands": [
        {
          "description": "Enable pretty-printing for gdb",
          "text": "-enable-pretty-printing",
          "ignoreFailures": true
        },
        {
          "description": "Set Disassembly Flavor to Intel",
          "text": "-gdb-set disassembly-flavor intel",
          "ignoreFailures": true
        }
      ],
      "preLaunchTask": "C/C++: clang++ build active file"
    }
  ],
  "version": "2.0.0"
}
```

:::note[macOS and lldb]
If you are using lldb on macOS, then the *launch.json* is a little simpler. An example is shown below. As with the gdb settings, the main thing to pay attention to is the *program* setting.

```json
{
    "configurations": [
        {
            "name": "C/C++: clang++ build and debug active file",
            "type": "cppdbg",
            "request": "launch",
            "program": "${fileDirname}/${fileBasenameNoExtension}",
            "args": [],
            "stopAtEntry": false,
            "cwd": "${fileDirname}",
            "environment": [],
            "externalConsole": false,
            "MIMode": "lldb",
            "preLaunchTask": "C/C++: clang++ build active file"
        }
    ],
    "version": "2.0.0"
}
```

:::

## Running in the Debugger

To run your code in the debugger, use the "debug/play" or click the "Run and Debug" icon in the debugger pane. You should be able to add breakpoints and step through the code as before.

Notice that unlike C#, the output does not go into the terminal window. To include terminal input and output, you need to change the "externalConsole" setting to `false`. When you run the debugger now, it will open the default terminal program and allow you to use it for input and output.

Remember, if you change the "-o" setting in *tasks.json*, you will also need to change it in *launch.json*.
