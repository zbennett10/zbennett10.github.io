---
layout: post
title:  "Generating Isolines From a GeoTiff With Web Assembly - Part II"
date:   2020-1-11 06:22:27 -0600
categories:
- geospatial
- algorithms
---

I originally had intended on writing blog posts as I went, but this project was too fun to stop and write a blog post about it... 

So instead, I'll walk through the *almost finished* project and I'll update this post at a later date with links to the GitHub repo after I opensource the project's code.

### Setting Up a Web Assembly Project

I came into this project as a complete WASM noob. I am now a hardened veteran... Not really... still a noob, just a... hardened?... noob.

The first thing I did was Google, "How the EFF do I do a WASM thing?" As a part of the corresponding results, I eventually landed on [this guide](https://developer.mozilla.org/en-US/docs/WebAssembly/C_to_wasm) which shows you how to compile a new C/C++ Module to WebAssembly (thank you Mozilla Developer Network).

These steps essentially had me install the Emscripten SDK and use the `emcc` binary that is provided with the SDK to generate three things based on some C code:

- WASM - What our C code is compiled to. Emscripten takes LLVM bytecode and compiles it into JavaScript although we will be using it to get WASM code 
- HTML - Basic wrapper project that can be used to run your WASM code for testing
- JavaScript - Essentially a nice "glue" file that is generated so that you have easy access to call the WASM functions from JavaScript.

Once I got this up and running, I just followed the guide and had some C code that looked like this...

```c
#include <stdio.h>

int main(int argc, char ** argv) {
    printf("Hello World\n");
}
```

Ahh.. the obligatory "Hello World". The world is OK now.


From here, I was feeling like superman. Boom, WASM code running in the browser from C. AKA, the fastest running "Hello World" in the universe!


### Creating C Project, Dependency/Linking Hell and Other Shenanigans

And More Coming soon....
