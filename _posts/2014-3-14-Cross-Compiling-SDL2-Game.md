---
layout: post
title: Crosscompiling a C based SDL2 Game
---

So, you want to develop SDL2 games in C on Linux, and would like to crosscompile them games for Windows too (since compiling natively in Windows sucks).


Here's a very basic hello world that proves you've got SDL2 and SDL_image working:
```
#include <stdio.h>
#include <SDL.h>

int main(int argc, char *argv[]) {
    if(SDL_Init(SDL_INIT_EVERYTHING) != 0) {
        fprintf(stderr, "SDL Init failed: %s\n", SDL_GetError());
        return 1;
    }
    SDL_Window *window = SDL_CreateWindow("Hello World", SDL_WINDOWPOS_CENTERED,SDL_WINDOWPOS_CENTERED, 640, 480, 0);
    int flags=IMG_INIT_JPG|IMG_INIT_PNG;
    if(IMG_Init(flags)&flags != flags) {
        fprintf(stderr, "IMG_Init failed: %s\n", IMG_GetError());
        return 1;
    }
    SDL_Delay(3000);
    SDL_DestroyWindow(window);
    SDL_Quit();

    return 0;
}
```


###Get your Compilers

####Get a Linux C compiler:

`sudo apt-get install build-essential`

####For cross-compiling to Windows (both x86 and x64):

`sudo apt-get install gcc-mingw32`


Target Architecture | C Compiler Binary | Directory containing /lib /include
Linux Native | gcc | /usr/
Windows x86 | i686-w64-mingw32-gcc | /usr/i686-w64-mingw32/
Windows x64 | x86_64-w64-mingw32-gcc | /usr/x86_64-w64-mingw32/


###Get SDL2 for Linux

If you're on an Ubuntu >= saucy, on x86/x64, its easy to get SDL2:

`sudo apt-get install libsdl2-2.0-0 libsdl2-dev libsdl2-image-2.0-0 libsdl2-mixer-2.0-0 libsdl2-mixer-dev libsdl2-image-dev`

(if you're on ARM, you'll probably have issues compiling libsdl2-image's dependencies, and its not in the main repositories yet)


###Get SDL2 for Windows (in Linux)

Crosscompiling SDL2 itself didn't work out well, so download the SDL2-devel-2.0.1-mingw archive.

Also download the SDL2_image-devel-2.0.0-mingw.tar.gz

Extract both of these to: `/usr/x86_64-w64-mingw32/`


##Building the Game

A Makefile will drastically simplify the development process, heres an example Makefile:

```
EXECUTABLE=game
SOURCES=test.c
EXTRALIBS=-lSDL2_image

#Linux build settings

CC=gcc
CFLAGS:=$(shell sdl2-config --cflags)
LIBS=$(shell sdl2-config --libs) $(EXTRALIBS)

#Windows build settings
win32: WINFOLDER:=/usr/x86_64-w64-mingw32/
win32: CC:=x86_64-w64-mingw32-gcc
win32: EXECUTABLE:=$(EXECUTABLE).exe
win32: CFLAGS:=-I$(WINFOLDER)include/SDL2 -Dmain=SDL_main
win32: LIBS=-L$(WINFOLDER)lib -lmingw32 -lSDL2main -lSDL2 -mwindows $(EXTRALIBS)

all: gcc_$(EXECUTABLE)
warn: CFLAGS += -Wall
warn: all
win32: all
run: all
        ./$(EXECUTABLE)
compile: headers
        $(CC) $(CFLAGS) -o $(EXECUTABLE) $(SOURCES) $(LIBS)
```
Linux build: `make`

Linux build & run if no errors: `make run`

Windows build: `make win32`


##Troubleshooting

####(Windows)  "The application was unable to start correctly: 0xc000007b"
Might have a 32bit/64bit issue. Check that your SDL2.dll, SDL2_image.dll, libjpeg-9.dll, and your game executable are all the same architecture.

####(Building) "fatal error: SDL2.h: No such file or directory"
SDL2 isn't installed in a place gcc is currently looking. The example Makefile looks in /usr/include/, and apt-get should be installing it there. make install might not.

####(Building) "undefined reference to `IMG_Init'"
You're missing the gcc argument -lSDL2_image

####(Building) " error: ‘IMG_INIT_JPG’ undeclared (first use in this function)"
Your source file is missing #include <SDL2/SDL_image.h>
