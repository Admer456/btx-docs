
# BurekTech X API documentation

Admittedly I am far, far from a web dev, but I managed to scramble up a little documentation website for the engine. This repository hosts the relevant engine sources & scripts used to generate the docs.

## Where is it?
If you wanna see the documentation itself, go here now: (todo: link pls)

## I wanna build it myself
In case you want an offline copy, or contribute to the docs, here's how to generate (after cloning **with** submodules initialised):  
```bash
# Configure CMake
mkdir build
cd build
cmake ..

# Build the API docs
# The Sphinx target will run Doxygen to generate XML,
# then Sphinx+Breathe to generate HTML from that
# The output will be in docs/
cd..
cmake --build . --target Sphinx
```

If you're familiar with Sphinx already, great! You know it better than I do. If not, well, read a bit about *reStructuredText* and Sphinx itself, and look at `source/index.rst`.

That's also where the Doxygen file is located. Have fun!
