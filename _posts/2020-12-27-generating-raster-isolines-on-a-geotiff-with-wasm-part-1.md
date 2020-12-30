---
layout: post
title:  "Generating Raster Isolines On a GeoTiff With Web Assembly - Part I"
date:   2020-12-27 06:22:27 -0600
categories:
- GIS
- Geospatial
- WASM
- Algorithms
---

I've always loved maps and cartography. I remember finding one of my dad's old history textbooks when I was 8 years old. I spent a lot of time poring over the different maps filled with ancient geography. This lust for maps continued into my teenage years as I spent countless hours playing Sid Meier's Civilization III and IV. Fast-forward to today and I've been fortunate to land some great jobs as a software engineer working on different mapping applications. Cesium, Leaflet, OpenLayers, ESRI, GDAL, QGIS - the standard acronyms of GIS programming made their way into my life! I love it all!

I've been wanting to scratch this Web Assembly itch I've had for a while... so here goes. I don't know how long this will take exactly or how many parts there will be... but this blog series will cover my journey into creating an isoline generator using Web Assembly. I'm still kind of thinking it out... but *I think* the steps to get a naive implementation working will consist of the following:

1. Set up the Web Assembly project scaffolding
2. Write some C code that will generate some GeoJSON using the [jansson](https://github.com/akheron/jansson) library
3. Create a simple test application that will generate some GeoJSON from C code
4. Nail down the inputs for the marching squares algorithm (what we send to the WASM code)
5. Implement the marching squares algorithm in C and generate GeoJSON isolines from the algorithm's output
6. Call the appropriate WASM function from our test JavaScript application and display the GeoJSON produced 


So far, I have steps 1-3 complete so my next blog post will cover these steps... but that was the easy part - although getting the jansson library compiled and linked into my C code using Emscripten took more time than I wanted it too!


A lot of inspiration for this project was garnered from the [raster-marching-squares](https://github.com/rveciana/raster-marching-squares) project. My hope is to pretty much create a WASM implementation of this library and then open source it! I hope that it ends up becoming something useful to the GIS community and that it is a springboard for me as I attempt to learn *all the things* GIS (not possible).

Stay tuned!
