# CPP-Compile-Commands-Generator

CMake file that automatically generates a `compile_commands.json` in your project root for your IDE's LSP. Works with both MSVC and GCC/CLANG. I originally made this to fix LSP for including headers in Neovim while compiling with Visual Studio.

## Usage

Clone this somewhere convenient , e.g., `~/scripts/`. Then in your C++ project root where you have your main `CMakeLists.txt` , include the file in this repo:

```cmake
include(~/scripts/CMakeLists.txt)
```

Or for best practices , use an intermediate `local.cmake` file that has the above. In the main `CMakeLists.txt` have the following instead:

```cmake
include(${CMAKE_CURRENT_SOURCE_DIR}/local.cmake OPTIONAL)
```

Note that the expected format for your project for this file to work is as follows:

```
project/
├── CMakeLists.txt
├── src/
├── include/
└── build/
```

The CMake file in this repo will recursively add **all** .cpp in `src/` , and **all** .h files in `include/` to the `compile_commands.json` file. This **will only affect LSP** as far as I know.
