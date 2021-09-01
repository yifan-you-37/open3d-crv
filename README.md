# open3d-crv
Below specifies instructions for building open3d [w/ headless rendering flags](http://www.open3d.org/docs/latest/tutorial/Advanced/headless_rendering.html) on crv, **without sudo**.

This requires OSMesa. With sudo it's very easy to install. Without sudo it's still doable but requires manually building LLVM and several other packages. The benefit is that once you do this you can render headlessly w/o having to open an extra viewer like turboVNC.

**NOTE:** in the installation process below we set a lot of environment variables in the terminal (`PATH, LD_LIBRARY_PATH, CPLUS_INCLUDE_PATH, C_INCLUDE_PATH`). If you change to a different terminal then you will need to set these variables again. If you want the variables to always be set, add the corresponding commands to `~/.bashrc`.

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
The pre-built OSMesa we use (in step 4) requires `libLLVM-6.0.so.1`, which is obtained by building LLVM 6.0.x. 

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
7. To confirm that build is successful, we check that `libLLVM-6.0.so` is generated and can find all its dependencies:
````
cd /path/to/llvm/lib
ldd libLLVM-6.0.so
````
The output of `ldd` should indicate that all dependencies of `libLLVM-6.0.so` are found.

8. We need to make sure that OSMesa can find `libLLVM-6.0.so.1`. The built files do not contain `libLLVM-6.0.so.1`, but contain `libLLVM-6.0.so`. So we need to first make a soft link from `libLLVM-6.0.so.1` to `libLLVM-6.0.so`:
````
ln -s /path/to/llvm/lib/libLLVM-6.0.so /path/to/llvm/lib/libLLVM-6.0.so.1
````
We then need to make sure that the folder containing `libLLVM-6.0.so.1` is added to `LD_LIBRARY_PATH` so OSMesa can find it:
```
export LD_LIBRARY_PATH=/path/to/llvm/lib:$LD_LIBRARY_PATH
ldconfig
```
Note: you might want to add `export LD_LIBRARY_PATH=/path/to/llvm/lib:$LD_LIBRARY_PATH` to your `~/.bashrc` so you don't have to manually type it in every time you start a new terminal and try to run Open3D.

### Step 4: Installing OSMesa (without sudo)
Open3D w/ headless rendering requires us to build OSMesa. I tried building OSMesa from source but encountered more errors and packages not found. So instead I chose to use the `.deb` package someone provided in [their instructions](https://pyrender.readthedocs.io/en/latest/install/index.html#installmesa) of building OSMesa. 

Note that the usual way of installing `.deb`, which is to run `dpkg -i`, does not work without sudo. Therefore we choose to extract the package first and then manually add the necessary paths for building Open3d.

1. Create a folder under which you want to put the OSMesa files. Moving forward we use `/path/to/mesa` to denote the path to this folder.
2. Download the `.deb` file and unzip using `dpkg -x`:
```
wget https://github.com/mmatl/travis_debs/raw/master/xenial/mesa_18.3.3-0.deb
dpkg -x mesa_18.3.3-0.deb /path/to/mesa
 ``` 
Note that `dpkg -x` creates `usr/local` folders under the unzipped folder by default. The mesa libraries are in `/path/to/mesa/usr/local/lib`, and the mesa header files are in `/path/to/mesa/usr/local/include`.

3. Verify that `libOSMesa.so` can find all its dependencies (including the `libLLVM-6.0.so.1` we added from installing LLVM):
```
ldd /path/to/mesa/usr/local/lib/libOSMesa.so
```
The output of ldd should indicate that all dependencies are found.

4. Add the folder that contains mesa header files so that `gcc` and `g++` can find them:
```
CPLUS_INCLUDE_PATH=/path/to/mesa/usr/local/lib:$CPLUS_INCLUDE_PATH
C_INCLUDE_PATH=/path/to/mesa/usr/local/lib:$C_INCLUDE_PATH
```
### Step 5: Finally! Installing Open3d (without sudo)



