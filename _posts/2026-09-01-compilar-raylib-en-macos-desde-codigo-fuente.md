---
layout: default
title:  Compiling Raylib on macOS from source
date:   2026-09-01 15:00:00 -0600
description: Compiling Raylib on macOS from source
categories: GameDev
permalink: /compiling-raylib-on-macos-from-source/
---

# Compiling Raylib on macOS from source

We're going to install raylib on mac from the repo, that way you can have the most up-to-date code in case you don't want to install it from homebrew. Another advantage is that when you install from the repository you can pass it whatever parameters you want, in case you'd like to use SDL or the new software renderer, or to optimize it.

You can find these parameters in the documentation. For now we're going to install it with the most common ones, but you can experiment with the different options.

## Prerequisites

First of all, we need the Xcode tools in order to compile. You don't need to download Xcode itself, but you do need to install the `Xcode Command Line Tools`.

```bash
xcode-select --install
```

Now we need to download cmake. The easiest way is through homebrew, which is going to come in handy for other installs too. We could also install raylib with homebrew, but we're going to compile it for the reason I already told you.

If we go to [https://brew.sh/](https://brew.sh/) it'll show us the instructions.

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

Once that's done, we need to add homebrew to our path, so let's edit the `.zshrc` file, which you can open with vim: `vim .zshrc`, or any other editor you like. In there, put the following export at the end of the document.

```plaintext
export PATH=.:$HOME/.local/bin:/opt/homebrew/bin:/usr/local/bin:$PATH
```

With this we're adding the most common code directories to the path.

We restart the terminal to install cmake

```bash
brew install cmake
```

Now we're ready to compile raylib.

## Compiling raylib

First, we need to download the repository, into whatever folder you want.

```bash
git clone https://github.com/raysan5/raylib.git
cd raylib
git checkout 6.0   # switch to whatever tag you need, or you can stay on main
```

Now that we have the code, we're going to compile with cmake

```bash
mkdir build && cd build

cmake .. \
  -DCMAKE_BUILD_TYPE=Debug \
  -DBUILD_SHARED_LIBS=OFF \
  -DCMAKE_INSTALL_PREFIX=/usr/local

make -j$(sysctl -n hw.logicalcpu)
sudo make install
```

First we create the build folder and move into it, we run cmake with the parameters we need, and here we can see all the options we can pass it: [https://github.com/raysan5/raylib/wiki/CMake-Build-Options](https://github.com/raysan5/raylib/wiki/CMake-Build-Options)

The part that says `make -j$(sysctl -n hw.logicalcpu)` is to compile using all the cores of our CPU so it takes less time, and at the end we install into `/usr/local`, which is the parameter we passed it.

This installs into:

| **File**              | **Destination**                |
| --------------------- | ------------------------------ |
| Header `raylib.h`     | `/usr/local/include/`          |
| Library `libraylib.a` | `/usr/local/lib/`              |
| CMake config          | `/usr/local/lib/cmake/raylib/` |

And we can verify the installation

```bash
ls /usr/local/include/raylib.h
ls /usr/local/lib/libraylib.a
```

## Testing raylib

Now we're going to make a small demo to see if what we've done works. This time I'll use Visual Studio Code since it's the most common one.

We create a folder and in there create a file called `main.c` and write the code from the raylib examples.

```c
#include <raylib.h>


int main()
{
    const int screenWidth = 800;
    const int screenHeight = 450;

    InitWindow(screenWidth, screenHeight, "raylib [core] example - basic window");
    SetTargetFPS(60);

    while (!WindowShouldClose())
    {
        BeginDrawing();


        ClearBackground(RAYWHITE);
        DrawText("Congrats! You created your second window!", 190, 200, 20, LIGHTGRAY);

        EndDrawing();
    }

    CloseWindow();
}


```

And we create a `Makefile`

```plaintext
build:
	clang -g -O0 main.c -o game \
		-lraylib \
		-framework OpenGL \
		-framework Cocoa \
		-framework IOKit \
		-framework CoreAudio \
		-framework CoreVideo

run:
	./game

clean:
	rm game

```

This is going to help us compile. Now we can simply run `make` in the terminal, in the folder where the code lives, and it'll compile the game, and then `make run` to run the example. But we're going to make use of Visual Studio so we can use the debugger and set breakpoints and see what's going on with our code.

Create a folder called `.vscode` (don't forget the dot) and create two files, one called `launch.json` and another one `tasks.json`. In the `launch.json` file write the following:

```json
{
    // Use IntelliSense to learn about possible attributes.
    // Hover to view descriptions of existing attributes.
    // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "name": "(lldb) Launch",
            "type": "cppdbg",
            "request": "launch",
            "program": "${workspaceFolder}/game",
            "args": [],
            "stopAtEntry": false,
            "cwd": "${workspaceFolder}",
            "environment": [],
            "externalConsole": false,
            "MIMode": "lldb",
            "preLaunchTask": "build-debug"
        }

    ]
}
```

And in `tasks.json` you're going to put the following

```json
{
    "version": "2.0.0",
    "tasks": [
        {
            "label": "build-debug",
            "type": "shell",
            "command": "make",
            "group": {
                "kind": "build",
                "isDefault": true
            },
            "problemMatcher": ["$gcc"]
        }
    ]
}
```

And now you can run it with `F5`. If you want to try out the debugger, set a breakpoint and you'll see if it works.

With all this, you're ready to start programming with raylib.
