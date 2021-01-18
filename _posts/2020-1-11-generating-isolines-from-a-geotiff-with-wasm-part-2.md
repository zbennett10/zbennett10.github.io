---
layout: post
title:  "Generating Isolines From a GeoTiff With Web Assembly - Part II"
date:   2020-1-11 06:22:27 -0600
categories:
- geospatial
- algorithms
---

I originally had intended on writing blog posts as I went, but this project was too fun to stop and write a blog post about it... 

So instead, I'll walk through the *pretty much finished* project and I'll update this post at a later date with links to the GitHub repo after I opensource the project's code.

### Setting Up a Web Assembly Project

I came into this project as a complete WASM noob. I am now a hardened veteran... Not really... still a noob, just a... hardened?... noob.

The first thing I did was Google, "How the EFF do I do a WASM thing?" As a part of the corresponding results, I eventually landed on [this guide](https://developer.mozilla.org/en-US/docs/WebAssembly/C_to_wasm) which shows you how to compile C/C++ to WebAssembly (thank you Mozilla Developer Network).

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


### Creating Our C Library

With some C code running in the browser by way of WebAssembly, it was time to port rveciana's [amazing JavaScript code](https://github.com/rveciana/raster-marching-squares/blob/master/src/marchingsquares-isolines.js) to C. 

Essentially, we need this library to provide a function that takes in a matrix consisting of raster data and an array of threshold values and spits out GeoJSON that consists of MultiLineStrings. Our library will do this by generating a grid of cells based on the raw, raster data where each cell will consist of a different "contour state". The library will then take this contour grid and then create georeferenced GeoJSON MultiLineString's based on these contour states.

The function signature will need to look like this:

```c
char* generate_isolines_geojson(
    int raster_data_rows,
    int raster_data_cols,
    double raster_data[raster_data_rows][raster_data_cols],
    double geotransform[6],
    int interval_count,
    double intervals[interval_count]
) {
    ...
```

Our function will need to know the size of the multidimensional array `raster_data` so that it can properly traverse it. It will also need to accept an array of six values consisting of the values needed to successfully georeference the generated paths that our algorithm creates. The raster data point of reference is in a pixel format - the `geotransform` array's values will enable us to successfully convert this point of reference to the geographic point of reference we need by way of an affine transformation. For more information on this, [check out this link](https://gdal.org/user/raster_data_model.html#affine-geotransform).

And lastly, we of course need our function to take in the size of our `intervals` array along with the `intervals` array itself so that we can generate isolines for an indeterminate amount of intervals.

You can check out the [full code here](https://github.com/zbennett10/wasm-marching-squares/blob/master/lib/marching-squares.c)!

### Unit Tests - For Sanity!

To make unit testing easier such that it I didn't have to link in any Emscripten libraries, I split out our library function from our WASM external API. The file `marching-squares-wasm-api.c` looks like this. 

```c

#include <stdio.h>
#include <string.h>
#include <emscripten/emscripten.h>
#include "marching-squares.h"

/**
 * Exported WASM API defined below
 **/

EMSCRIPTEN_KEEPALIVE char* generate_isolines_geojson_(
    int raster_data_rows,
    int raster_data_cols,
    double raster_data[raster_data_rows][raster_data_cols],
    double geotransform[6],
    int interval_count,
    double intervals[interval_count]
) {
    return generate_isolines_geojson(
        raster_data_rows,
        raster_data_cols,
        raster_data,
        geotransform,
        interval_count,
        intervals
    );
}
```

This makes it easier for us to unit test and also makes our external, WASM API a bit more explicit. The `EMSCRIPTEN_KEEPALIVE` macro definition does some nifty *emscripten* things that ensure that this function is not removed during the building of the library.

With the library created, I then did the *right* thing and created some unit tests for my own sanity! Fortunately, rveciana's original library had some test cases I could port over to C and start using. I used the simple and easy to use [greatest](https://github.com/silentbicycle/greatest) library to write my unit tests. Here is an example of one such, simple test case. 

```c
TEST marching_squares_lib_should_generate_geojson_isolines_for_more_complex_data(void) {
    double raster_data[7][7] = {
        {5,5,5,5,5,5,5},
        {5,12,12,12,12,12,5},
        {5,12,5,5,5,12,5},
        {5,12,5,18,5,12,5},
        {5,12,5,5,5,12,5},
        {5,12,12,12,12,12,5},
        {5,5,5,5,5,5,5},
    };
    double geotransform[6] = { 10, 1, 0, 10, 0, -1 };
    double intervals[1] = { 9 };

    char *geojson = generate_isolines_geojson(
        7,
        7,
        raster_data,
        geotransform,
        1,
        intervals
    );

    char* expected = "{\"type\":\"FeatureCollection\",\"features\":[{\"type\":\"Feature\",\"geometry\":{\"type\":\"MultiLineString\",\"coordinates\":[[[11.0,9.4285714285714288],[10.571428571428571,9.0],[10.571428571428571,8.0],[10.571428571428571,7.0],[10.571428571428571,6.0],[10.571428571428571,5.0],[11.0,4.5714285714285712],[12.0,4.5714285714285712],[13.0,4.5714285714285712],[14.0,4.5714285714285712],[15.0,4.5714285714285712],[15.428571428571429,5.0],[15.428571428571429,6.0],[15.428571428571429,7.0],[15.428571428571429,8.0],[15.428571428571429,9.0],[15.0,9.4285714285714288],[14.0,9.4285714285714288],[13.0,9.4285714285714288],[12.0,9.4285714285714288],[11.0,9.4285714285714288]],[[11.428571428571429,8.0],[12.0,8.5714285714285712],[13.0,8.5714285714285712],[14.0,8.5714285714285712],[14.571428571428571,8.0],[14.571428571428571,7.0],[14.571428571428571,6.0],[14.0,5.4285714285714288],[13.0,5.4285714285714288],[12.0,5.4285714285714288],[11.428571428571429,6.0],[11.428571428571429,7.0],[11.428571428571429,8.0]],[[13.0,7.6923076923076925],[12.307692307692307,7.0],[13.0,6.3076923076923075],[13.692307692307693,7.0],[13.0,7.6923076923076925]]]},\"properties\":{\"threshold\":9.0}}]}"; 
    ASSERT(!strcmp(geojson, expected));

    free(geojson);

    PASS();
}
```

Pretty straightforward! I create the inputs that I need, call the function `generate_isolines_geojson`, and then assert that the generated GeoJSON is as expected...

I am now sane.

### OK, WASM

It's to build our library, but for real. No more easy "Hello World". This is one of the parts that I spent the most time on... ah CMake... ah linking... what fun. I'll spare you the gritty details. Eventually, I created a build script that consisted of this command:

```bash
emcc -O3 \
     lib/marching-squares-wasm-api.c \
     lib/marching-squares.c \
     include/jansson-2.13.1/emcc-lib/lib/libjansson.a \
     -Iinclude/jansson-2.13.1/emcc-lib/include \
     -s EXPORTED_FUNCTIONS='["_generate_isolines_geojson_"]' \
     -s "EXTRA_EXPORTED_RUNTIME_METHODS=['ccall', 'cwrap', 'UTF8ToString', 'addOnPreMain']" \
     -s WASM=1 \
     -s ALLOW_MEMORY_GROWTH=1 \
     -s NO_EXIT_RUNTIME=0 \
     -s LLD_REPORT_UNDEFINED \
     -s ASSERTIONS=1 \
     -s EXPORT_ES6=1 \
     -s MODULARIZE=1 \
     -s STRICT=1 \
     -s MALLOC=emmalloc \
     --pre-js 'lib/pre-emcc-build.js' \
     --post-js 'lib/post-emcc-build.js' \
     -o dist/wasm-marching-squares.js \
     && cp dist/* test/
```

There's *a lot* going on there... Let's enter in to the matrix.

The `-O3` tells `emcc` (the Emscripten C compiler) to forcefully optimize the JavaScript output. 

This portion:
```c
lib/marching-squares-wasm-api.c \
lib/marching-squares.c \
include/jansson-2.13.1/emcc-lib/lib/libjansson.a \
-Iinclude/jansson-2.13.1/emcc-lib/include \
```
passes in the C library files we need to compile and also links in the static, shared library that we need (jansson) along with the library's header file.

The rest of the options passed to `emcc` allow us to configure the WASM and JavaScript output. You can easily tell what all of these options do by checking out the [documentation](https://emscripten.org/docs/tools_reference/emcc.html).

After running this, you get a `.wasm` file and a `.js` file. The WASM file is the juicy goodness that generates the GeoJSON that we need. The JavaScript file is the "glue code" that makes our WASM library easier to work with. More on that later... It's tricky to configure this JavaScript glue code and I'm still not fully done getting it right!

### Let's See Things and Stuff Now

Alright time for some visuals! Don't get me wrong, I love code but part of the beauty of programming in a geospatial context is that you eventually get to see some pretty visuals on a map.

>
> TO BE CONTINUED...
>