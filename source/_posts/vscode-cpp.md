---
title: Getting Started with C/C++ in Visual Studio Code
date: 2019-04-29 16:08:00
tags: 
    - vscode
    - c++
category: 
    - Tutorials
img: /images/screenshot-vscode-tutorial-1.png
---

In the [previous post](https://shineyruan.github.io/2019/03/15/vscode-tutorials/) we have known that Visual Studio Code is a lightweight editor for small coding projects. However, you might still wonder how to actually take advantage of this small but powerful editor. In this post, we will see together how to write & compile C/C++ codes in Visual Studio Code.

<!-- more -->

# VS Code and C/C++

C/C++ has been the most popular programming languages for decades, and almost all programming around the world know something about it. Although C/C++ programs might be a little bit trickier to compile in Windows system, VS code provides a nice solution for it.

This section was written with two Chinese blogs on Zhihu (知乎) as references. I really thank them for their instructions.

1. [Visual Studio Code 如何编写运行 C、C++ 程序？](https://www.zhihu.com/question/30315894)
2. [用VSCode调试C/C++代码 (通过WSL)](https://zhuanlan.zhihu.com/p/44337349)

## Compiling C/C++ in Windows

As for finding a compiler for C/C++ program, I personally recommend *Clang for Windows* and *MinGW-w64*. In fact, each of them is a standalone C/C++ compiler for Windows, but why do we need both? This is because it is found that **there are no header files in Clang**, and we have to **add header files from MinGW-w64**.

1. Download **Windows (64-bit)** from the Pre-Built Binaries section [here](http://releases.llvm.org/download.html). `sig` file is not needed.
2. Download MinGW-w64 for 32 and 64-bit Windows [here](https://sourceforge.net/projects/mingw-w64/).
3. Install Clang for Windows. Remember to select **Add LLVM to the system PATH for all users**. I personally recommend `C:\LLVM` as the destination folder.
4. Install MinGW-w64 anywhere (we're going to **uninstall it** after moving all its files away). Remember to select `x86-64` architecture when installing.
5. Move all files from `mingw64` to `LLVM` (Image source: cite [here](https://www.zhihu.com/question/30315894)).

![](/images/screenshot-vscode-tutorial-5.jpg)

6. Uninstall MinGW-w64 by clicking `uninstall.exe`.
7. Open Command Prompt, type in `clang -v`, and you should see the corresponding version. If you see something like "not recognized as the name of a cmdlet, function, script file, or operable program", you may have to add `C:\LLVM\bin` to the system PATH manually.
8. Open VS Code, and install the following extensions from extension pack:
   * C/C++
   * C/C++ Clang Command Adapter (optional)
   * Code Runner (compile single source file with a right click)
   * One Dark Pro (optional, coding theme for VS Code)
9.  Create 3 JSON configuration files in a folder called `.vscode`:
   * `tasks.json`
   * `launch.json`
   * `settings.json` (if you want to use these settings forever, copy the content of this file into the global user settings JSON file.)
11. Create an empty folder on your computer.
12. Open VS Code, and select "Open Folder...". Open the empty folder you just created.
13. Create a folder called `.vscode` in VS Code.
14. Create a JSON file called `tasks.json` in the .vscode folder.
15. Copy the following content into your `tasks.json`:

```json
// https://code.visualstudio.com/docs/editor/tasks
{
    "version": "2.0.0",
    "tasks": [
        {
            "label": "Compile", // name of task
            "command": "g++", // either g++ or clang++ is okay.
                              // NOTICE: If you're compiling C program,
                              // change it to gcc or clang.
                              // The args should also be the C cmd args.
            "args": [
                // Additional arguments: 
                // -Wall -Wextra -Wvla -Wconversion -pedantic
                // THIS IS JUST FOR CPP!!!
                "${file}",
                "-o",
                "${fileDirname}/${fileBasenameNoExtension}.exe",
                "-g",
                "-Wall", // extra warnings
                "-Wextra", // extra warnings
                "-Wvla", // extra warnings
                "-Wconversion", // extra warnings
                "-pedantic", // extra warnings
                "-static-libgcc", // static linking
                "-std=c++17" // cpp standard
            ], 
            "type": "shell", // can be shell or process. shell
                             // means inputting the args in the command line.
            "group": {
                "kind": "build",
                "isDefault": true
            },
            "presentation": {
                "echo": true,
                "reveal": "always", // can be always, silent, never.
                "focus": false,
                "panel": "shared"
            },
            "problemMatcher":"$gcc" // if you use clang++, delete this line
        }
    ]
}
```

15. Create a JSON file called `launch.json` in the .vscode folder.
16. Copy the following content into your `launch.json`:

```json
// https://github.com/Microsoft/vscode-cpptools/blob/master/launch.md
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "(gdb) Launch", // Config name
            "type": "cppdbg", // only cppdbg
            "request": "launch", // can be launch or attach, 
                                 // but launch here only
            "program": "${fileDirname}/${fileBasenameNoExtension}.exe",
            "args": [], // modified by user. 
                        // Arguments passed to the program in cmmand line.

            "stopAtEntry": false, // if you wish to set a breakpoint
                                  // at the entry of the program, set it to true

            "cwd": "${workspaceFolder}", // workspace folder when debugging
            "environment": [],
            "externalConsole": false, // if you wish to use external prompt-out
                                      // window to show output, set it to true
            "internalConsoleOptions": "neverOpen",
            "MIMode": "gdb",
            "miDebuggerPath": "gdb.exe", // path for debugging tool
            "setupCommands": [
                {
                    "description": "Enable pretty-printing for gdb",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": false
                }
            ],
            "preLaunchTask": "Compile" // matches the config name of tasks.json
        }
    ]
}
```

17. Create a JSON file called `settings.json` in the .vscode folder. If you wish to use these configurations for all projects, ignore this line.
18. Copy the following contents into the `settings.json` either in the .vscode folder or global user settings:

```json
{
    "files.defaultLanguage": "cpp",
    "editor.formatOnType": true,
    "editor.snippetSuggestions": "top",

    "code-runner.runInTerminal": true,
    "code-runner.executorMap": {
        "c": "cd $dir && clang $fileName -o $fileNameWithoutExt.exe -Wall -g -Og -static-libgcc -fcolor-diagnostics --target=x86_64-w64-mingw -std=c11 && $dir$fileNameWithoutExt",
        "cpp": "cd $dir && clang++ $fileName -o $fileNameWithoutExt.exe -Wall -g -Og -static-libgcc -fcolor-diagnostics --target=x86_64-w64-mingw -std=c++17 && $dir$fileNameWithoutExt"
    }, // set cmd args for code runner
    "code-runner.saveFileBeforeRun": true,
    "code-runner.preserveFocus": true,
    "code-runner.clearPreviousOutput": false,

    "C_Cpp.clang_format_sortIncludes": true, // when formatting codes, 
                                             // sort the include files order.
    "C_Cpp.intelliSenseEngine": "Default",
    "C_Cpp.errorSquiggles": "Disabled",
    "C_Cpp.autocomplete": "Disabled", // enable it if you did not use Clang Adapter

    "clang.cflags": [
        "--target=x86_64-w64-mingw",
        "-std=c11",
        "-Wall"
    ],
    "clang.cxxflags": [
        "--target=x86_64-w64-mingw",
        "-std=c++17",
        "-Wall"
    ],
    "clang.completion.enable":true // set it to true and
                                   // C_Cpp autocomplete to false
                                   // if you installed Clang Adapter
}
```

19. Create an arbitrary C++ program.
20. Press `Ctrl+Shift+B` to run build tasks.
21. Press `Ctrl+F5` to run program; `F5` to debug program.

**Note. The `.vscode` folder with config JSON files has to be put in EVERY project root folder you're working on to activate C/C++ compiling.**

## Compiling C/C++ in *Windows Subsystem for Linux* (WSL)

Another solution for compiling C/C++ codes is using WSL, which I am currently using for school work. The installation procedure for WSL is rather easy and has a lot of tutorials online, so I'm going to skip this part. We also just need a `.vscode` folder in each project folder we're working on with `tasks.json` and `launch.json` configuration files. These files are generally the same as the previous ones.

1. `tasks.json`:

```json
// https://code.visualstudio.com/docs/editor/tasks
{
    "version": "2.0.0",
    "tasks": [
        {
            "label": "Compile",
            "command": "g++", // only g++ here!
            "args": [
                // Additional arguments:
                // -Wall -Wextra -Wvla -Wconversion -pedantic
                "-O3", // speeds up building
                "-g",
                "-Wall",
                "-Wextra",
                "-Wvla",
                "-Wconversion",
                "-pedantic",
                "-static-libgcc",
                "-std=c++1z",
                "${fileBasenameNoExtension}.cpp",
                "-o",
                "${fileBasenameNoExtension}",
            ],
            "type": "shell",
            "group": {
                "kind": "build",
                "isDefault": true
            },
            "presentation": {
                "echo": true,
                "reveal": "always",
                "focus": false,
                "panel": "shared"
            }
        },
        {
            "label": "Clean",
            "command": "rm",
            "args": [
                "-rf",
                "*.o",
                "${fileBasenameNoExtension}"
                "./*libgcc" // removes -static-libgcc
            ],
            "type": "shell",
            "group": {
                "kind": "build",
                "isDefault": true
            },
            "presentation": {
                "echo": true,
                "reveal": "always",
                "focus": false,
                "panel": "shared"
            }
        }
    ]
}
```

Notice here I add one more task, which is `clean`, which is used for cleaning all the object files and the executable files. You can modify the arguments if you wish.

2. `launch.json`:

```json
{
    // Use IntelliSense to learn about possible attributes.
    // Hover to view descriptions of existing attributes.
    // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "name": "(gdb) Bash on Windows Launch",
            "type": "cppdbg",
            "request": "launch",
            "program": "${fileBasenameNoExtension}",
            "args": [],
            "stopAtEntry": false,

            // TODO: Change this!
            "cwd": "", // this should be modified to be the workspace folder,
                       // in Linux format.
                       // Notice that /mnt/c/ in WSL is
                       // equivalent to C:\ in Windows

            "miDebuggerArgs": "",
            "environment": [],
            "externalConsole": false,
            "sourceFileMap": {
                "/mnt/c/": "C:\\"
            },
            "pipeTransport": {
                "debuggerPath": "/usr/bin/gdb",
                "pipeProgram": "c:\\windows\\sysnative\\bash.exe",
                "pipeArgs": ["-c"],
                "pipeCwd": ""
            },
            "setupCommands": [
                {
                    "description": "Enable pretty-printing for gdb",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": true
                }
            ],
            "preLaunchTask": "Compile"
        }
    ]
}
```

The others remain the same. Now you can create a C++ source file and play around with it. Remember that `Ctrl+Shift+P` opens the command palette of VS Code, `Ctrl+Shift+B` builds the program, `Ctrl+F5` runs it, and `F5` debugs it.

Happy coding!