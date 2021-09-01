# open3d-crv
Below specifies instructions for building open3d [w/ headless rendering flags](http://www.open3d.org/docs/latest/tutorial/Advanced/headless_rendering.html) on crv, **without sudo**.

This requires OSMesa. With sudo it's very easy to install. Without sudo it's still doable but requires manually building LLVM and several other packages. The benefit is that once you do this you can render headlessly w/o having to open an extra viewer like turboVNC.

### Step 1: Installing cmake (without sudo)
crv's default cmake is too outdated for building Open3D. We need to install a newer version.
1. Download cmake binary distributions from [here](https://cmake.org/download/). It should be a `tar.gz` file.
2. Unzip the downloaded `tar.gz` file and add `cmake-XXX/bin` folder to `PATH`.
3. Check to see that `which cmake` outputs the folder you just unzipped, not the default cmake.

### Step 2: Installing ninja (without sudo)
We need ninja for building LLVM (step 3) and Open3d.
1. Download ninja binary distributions from [here](https://github.com/ninja-build/ninja/releases). It should be named `ninja-linux.zip`.
2. Unzip `ninja-linux.zip`, it should have an executable called ninja in the zip file. Add the folder containing this executable to `PATH`.
3. Check to see that `which ninja` outputs the folder that has the ninja executable.

### Step 3: Installing LLVM (without sudo)
The pre-built OSMesa we use (in step 4) requires `libLLVM-6.0.so`, which is obtained by building LLVM 6.0.x. 

We build LLVM 6.0.1. We adapt the instructions from [here](https://releases.llvm.org/6.0.1/docs/CMake.html) (Following the instructions in the URL does not produce `libLLVM-6.0.so` -- we need additional build flags.)
1.  Download the LLVM 6.0.1 source code from [here](https://releases.llvm.org/download.html). By the time of writing this corresponds to this link:  `https://releases.llvm.org/6.0.1/llvm-6.0.1.src.tar.xz`
2.  Unzip the `tar.xz` file. It should give you a folder called `llvm-6.0.1.src`. Moving forward we use `/path/to/llvm-src/` to denote the path to this folder.
3.  Create a directory in which you want to put the build folder and the built libraries. Moving forward we use `/path/to/llvm` to denote the path to this folder (**Note: /path/to/llvm must be different from /path/to/llvm-src**). Create a `build` folder under `/path/to/llvm`. cd to this directory:
````
    mkdir /path/to/llvm
    mkdir /path/to/llvm/build
    cd /path/to/llvm/build
````
5. Generate build files and build using ninja
````
cmake /path/to/llvm-src \
      -DCMAKE_INSTALL_PREFIX=/path/to/llvm          \
      -DLLVM_ENABLE_FFI=ON                          \
      -DCMAKE_BUILD_TYPE=Release                    \
      -DLLVM_BUILD_LLVM_DYLIB=ON                    \
      -DLLVM_LINK_LLVM_DYLIB=ON                     \
      -DLLVM_BUILD_TESTS=ON                         \
      -DLLVM_ENABLE_RTTI=ON                         \
      -Wno-dev -G Ninja .. && 
      ninja      
````

6. Move the built files into `/path/to/llvm` by running `ninja install`.
7. To confirm that build is successful, we check that `libLLVM-6.0.so` is generated and has all its dependencies:
````
cd /path/to/llvm/lib
ldd libLLVM-6.0.so
````
The output of `ldd` should indicate that all dependencies of `libLLVM-6.0.so` are found.

