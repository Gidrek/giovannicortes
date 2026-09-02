---
layout: default
title:  Compilar Raylib en macOS desde el código fuente
date:   2026-09-01 15:00:00 -0600
description: Compilar Raylib en macOS desde el código fuente
categories: GameDev
permalink: /compilar-raylib-desde-codigo-fuente/
---

# Compilar Raylib en macOS desde el código fuente

Vamos a instalar raylib en mac desde los repositorios, así vas a poder tener el código más actualizado si es no lo quieres instalar desde homebrew, 
otra ventaja es que al instalar desde  el repositorio puedes pasarle los parámetros que deseses por si quieres usar SDL o el nuevo render por 
software u optimizarlo. 

Estos parámetros lo puedes ver en la documentación, por ahora vamos a instalarlo con los parámetros más comunes, pero puedes experimentar con 
varias opciones.

## Prerequisitos

Primero que nada, necesitamos las herramientas de Xcode para compilar, no es necesario que descargues XCode, pero si  que instales las herramientas de `Xcode Command Line Tool`

```bash
xcode-select --install
```

Ahora, debemos descargar cmake, aunque lo más sencillo es hacerlo por medio de homebrew, que de igual manera nos va a servir para otras 
instalaciones, también podemos instalar raylib con homebrew, pero nosotros lo vamos a compilar por lo que ya te dije. 

Si vamos a [https://brew.sh/](https://brew.sh/) nos indicará las instrucciones.

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

Una vez hecho esto, debemos agregar homebrew a nuestro path, así que editemos el archivo `.zshrc` , que puedes abrilo con vim: `vim .zshrc`  u o
tro editor que desees, ahí pon el siguiente export al final del documento.

```plaintext
export PATH=.:$HOME/.local/bin:/opt/homebrew/bin:/usr/local/bin:$PATH
```

Con esto agregamos al path los directorios más comunes de código.

Reiniciamos la terminal para instalar cmake

```bash
brew install cmake
```

Ahora si, ya estamos listo para compilar raylib.

## Compilar raylib

Primero debemos bajar el repositorio, en alguna carpeta que queramos.

```bash
git clone https://github.com/raysan5/raylib.git
cd raylib
git checkout 6.0   # cambia al tag que necesites o te puedes quedar en main
```

Ya que tenemos el código, vamos a compilar con cmake

```bash
mkdir build && cd build

cmake .. \
  -DCMAKE_BUILD_TYPE=Debug \
  -DBUILD_SHARED_LIBS=OFF \
  -DCMAKE_INSTALL_PREFIX=/usr/local

make -j$(sysctl -n hw.logicalcpu)
sudo make install
```

Primero creamos la carpeta de build y nos pasamos a la carpeta, hacemos el cmake con los parámetros que necesitamos, aquí podemos ver todas 
las opciones que le podemos pasar: [https://github.com/raysan5/raylib/wiki/CMake-Build-Options](https://github.com/raysan5/raylib/wiki/CMake-Build-Options)

La parte que dice `make -j$(sysctl -n hw.logicalcpu)` es para compilar usando todos los cores de nuestra CPU y tarde menos, y al final 
instalamos en `/usr/local` que es el parámetro que le pasamos.

Esto instala en:

| **Archivo**            | **Destino**                    |
| ---------------------- | ------------------------------ |
| Header `raylib.h`      | `/usr/local/include/`          |
| Librería `libraylib.a` | `/usr/local/lib/`              |
| CMake config           | `/usr/local/lib/cmake/raylib/` |

Y podemos verificar la instalación

```bash
ls /usr/local/include/raylib.h
ls /usr/local/lib/libraylib.a
```

## Probar raylib

Ahora vamos a hacer un pequeño demo para ver si lo que hemos hecho funciona. En esta ocasión usaré Visual Studio Code ya que es el más común.

Creamos un folder y ahí crea un archivo `main.cpp`  y escribimos el código de los ejemplos de raylib.

```cpp
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

Y creamos un archivo `Makefile`

```plaintext
build:
	clang -g -O0 main.cpp -o game \
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

Esto nos va a ayudar a compilar. Ahora simplemente podemos hacer `make`  en la terminal, en la carpeta donde está el código, y compilará el juego, 
y luego `make run` para correr el ejemplo, pero vamos a hacer uso de Visual Studio para que podamos usar el debugger y así hacer breakpoints y 
ver qué pasa con nuestro código.

Crea una carpeta llamado `.vscode` (no te olvides del punto) y crea dos archivos, uno llamado `launch.json` y otro `tasks.json` . 
En el archivo `launch.json` escribe lo siguiente:

```json
{
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

Y en el `tasks.json` vas poner lo siguiente

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

Y ahora puedes correrlo con `F5` , si quieres probar el debugger, pon un break point y así vas a ver si funciona.

Con todo esto, ya tienes listo para poder programar con raylib.
