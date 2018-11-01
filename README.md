# alglib-cmake
Set of CMake files to download and build [ALGLIB](http://www.alglib.net/) as a standalone project. 
From the [ALGLIB](http://www.alglib.net/), here a description of the library:
> ALGLIB is a cross-platform numerical analysis and data processing library. It supports several programming languages (C++, C#, Delphi) and several operating systems (Windows and POSIX, including Linux). 
ALGLIB features include:
> - Data analysis (classification/regression, statistics)
> - Optimization and nonlinear solvers
> - Interpolation and linear/nonlinear least-squares fitting
> - Linear algebra (direct algorithms, EVD/SVD), direct and iterative linear solvers
> - Fast Fourier Transform and many other algorithms

## Build
With `make` facilities:
```bash
$ git clone https://github.com/S-Dafarra/alglib-cmake.git
$ cd alglib-cmake
$ mkdir build && cd build
$ cmake ..
$ make
$ [sudo] make install
```

With IDE build tool facilities, such as Visual Studio or Xcode
```bash
$ git clone https://github.com/S-Dafarra/alglib-cmake.git
$ cd alglib-cmake
$ mkdir build && cd build
$ cmake ..
$ cmake --build . --target ALL_BUILD --config Release
$ cmake --build . --target INSTALL --config Release
```

If you need more help on how to build CMake-based projects, please check [CGold's First step](https://cgold.readthedocs.io/en/latest/first-step.html) section.

### Link
Once the library is installed, you can link it using `CMake` with as little effort as writing the following line of code in your project's `CMakeLists.txt`:
```cmake
...
find_package(ALGLIB REQUIRED)
...
target_link_libraries(<target> PRIVATE ${ALGLIB_LIB})
...
target_include_directories(<target> PRIVATE ${ALGLIB_INCLUDE_DIRS})
...
```

Note that unless you did not use the default value of `CMAKE_INSTALL_PREFIX`, the `<prefix>` in which you installed ALGLIB will need to be appended to the `CMAKE_PREFIX_PATH` enviromental
variable to ensure that `find_package` can find your ALGLIB installation. Alternatively, you can specify the environmental variable
```bash
export ALGLIB_DIR=path/where/alglib-cmake/is/installed
```

See [CMake's reference documentation](https://cmake.org/cmake/help/latest/) if you need more info on the [`find_package`](https://cmake.org/cmake/help/latest/command/find_package.html) or [`target_link_libraries`](https://cmake.org/cmake/help/latest/command/target_link_libraries.html) CMake commands.

## License 
alglib-cmake is licensed under either the GNU Lesser General Public License v3.0 : 

https://www.gnu.org/licenses/lgpl-3.0.html

or the GNU Lesser General Public License v2.1 :

https://www.gnu.org/licenses/old-licenses/lgpl-2.1.html

at your option.
