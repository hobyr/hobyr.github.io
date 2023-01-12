% Setup an OpenGL project on Linux with Vim, CMake, GLFW and GLAD (part 1)

# Introduction

I like to do a lot of things manually, and in this case, I wanted to set
CMake up for my project that uses OpenGL.

Since OpenGL is a reference and not a library, I have to use two libraries
so I can work with it in C++.

Thanks the [The Cherno's Sparky Engine
series](https://www.youtube.com/playlist?list=PLlrATfBNZ98fqE45g3jZA_hLGUrD4bo6_)
on YouTube, I found out about GLFW, which handles window and controller
initialization for OpenGL.

However, I'm running on Ubuntu 20.04 so I won't be using Visual Studio.
Instead, I use Vim as my editor and CMake to configure the project build.
All of the setup is done using the terminal and CMake.

In this first part, I'll describe how to GLFW to work with Vim and CMake.

## Setting up the files

I roughly followed the folder structure The Cherno used for his Sparky
Engine project.

I first created a project folder with a `src/` subfolder in which I'll put
all my source files.

Another subfolder `Dependencies/` will welcome all the third-party
libraries I'll use. This include GLFW and GLAD.

My first source file is obviously `main.cpp` which is a simple "Hello,
World!" project.

I also have a `CMakeLists.txt` at the root folder which will tell how to
configure the project build. Its contents are very basic for a simple
compilation.

I quickly built my project by running `cmake -H . -B build/` from the root
folder, then went into `build/` and ran `make`.

Finally executed `./main` to run the binary and the terminal output
`Hello, world!` as expected.

It's now time to add the third-party libraries, namely GLFW and GLAD.

## Download GLFW

On Linux distributions, GLFW isn't available as a pre-compiled binary, we
have to compile from source.

So the first thing I had to do was clone the Github repository on my
system. I copied the contents of the repository in my project folder,
specifically in the `Dependencies/` sub-folder. I had this:
`<project_root>/Dependencies/GLFW/[...]`.

Before using the library in my project, I had to compile it first. The
official GLFW issued
a [guide](https://www.glfw.org/docs/latest/compile_guide.html), but simply
put, I went into the GLFW source folder (`Dependencies/GLFW/`) and ran
`cmake -S . -B build/`.

This will create the `build/` folder which will contain the Makefile.

Next, I moved to the `build/` folder and ran `make` to _actually_ compile
the library. And now I can use the library.

## Make Clangd aware of GLFW

By adding `#include <GLFW/glfw3.h>` to my source code, I can now use GLFW
functions or so I thought. Clangd immediately warned my that the header
couldn't be found.

I couldn't even add the `glfwInit()` function so I had to go into the main
`CMakeLists.txt` and configure the project build.

By following the [CMake
tutorial](https://cmake.org/cmake/help/latest/guide/tutorial/Adding%20a%20Library.html)
about adding a library, I simply added the following lines to my
`CMakeLists.txt`: 
```
add_subdirectory(Dependencies/GLFW)

target_link_libraries(main PUBLIC glfw)

target_include_directories(main PUBLIC "Dependencies/GLFW/include")
```
I simply ran `cmake` and `make` once to build the project again and now
Clangd could find the `glfw3.h` header.

## Verifying the functionality of GLFW

It's now time to write a simple GLFW initialization program. I've written
code based on [this part of the official
documentation](https://www.glfw.org/docs/latest/quick_guide.html#quick_init_term):
```
if (!glfwInit())
{
  std::cout << "Initialization failed" << std::endl;
}
else
{
  std::cout << "Initialization succeeded" << std::endl;
}
```

At this point, Clangd was showing suggestions of GLFW-related functions
when writing code. The setup was working.

I ran `make` again within the `build/` folder and executed `./main`. The
terminal showed: `Initialization succeeded` so I knew everything was in
working order.

## Conclusion

Now that GLFW is working with Vim, in the next post in this series, I'll
show how to setup GLAD for the same tools, make a window and draw
something simple in a window.
