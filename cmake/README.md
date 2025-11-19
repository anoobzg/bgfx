# CMake Build System for bgfx

This directory contains CMake configuration files for building bgfx.

## Prerequisites

- CMake 3.15 or higher
- C++11 compatible compiler
- bx library (https://github.com/bkaradzic/bx)
- bimg library (https://github.com/bkaradzic/bimg)

## Quick Start

```bash
# Clone dependencies (if not already present)
cd ..
git clone https://github.com/bkaradzic/bx.git
git clone https://github.com/bkaradzic/bimg.git

# Build bgfx
cd bgfx
mkdir build
cd build
cmake ..
cmake --build .
```

## CMake Options

- `BGFX_BUILD_TOOLS`: Build bgfx tools (shaderc, geometryc, etc.) - Default: ON
- `BGFX_BUILD_EXAMPLES`: Build bgfx examples - Default: OFF
- `BGFX_BUILD_SHARED_LIB`: Build bgfx as shared library - Default: OFF
- `BGFX_BUILD_AMALGAMATED`: Build bgfx as amalgamated - Default: OFF
- `BGFX_BUILD_WITH_SDL`: Build with SDL entry - Default: OFF
- `BGFX_BUILD_WITH_GLFW`: Build with GLFW entry - Default: OFF
- `BGFX_BUILD_WITH_PROFILER`: Build with profiler - Default: OFF

## Custom Dependency Paths

If bx or bimg are not in the default location (../bx, ../bimg), you can specify their paths:

```bash
cmake -DBX_DIR=/path/to/bx -DBIMG_DIR=/path/to/bimg ..
```

## Platform-Specific Notes

### Windows
- Requires DirectX SDK for D3D11/D3D12 renderers
- Visual Studio 2019 or later recommended

### Linux
- Requires X11, OpenGL development libraries
- Install: `sudo apt-get install libx11-dev libgl1-mesa-dev`

### macOS
- Requires Xcode Command Line Tools
- Metal renderer is enabled by default

### Android
- Requires Android NDK
- Set `ANDROID_NDK` environment variable

## Renderer Backends

The CMake build system automatically configures renderer backends based on the target platform:

- **Direct3D 11/12**: Windows, Linux (WSL)
- **OpenGL**: Linux, Windows
- **OpenGL ES**: Android, Emscripten
- **Vulkan**: Linux, Windows, macOS, Android
- **Metal**: macOS, iOS

## Building Examples

To build examples:

```bash
cmake -DBGFX_BUILD_EXAMPLES=ON ..
cmake --build .
```

## Building Tools

Tools are built by default. To disable:

```bash
cmake -DBGFX_BUILD_TOOLS=OFF ..
```

## Installation

```bash
cmake --build . --target install
```

This will install:
- bgfx library (static or shared)
- Header files
- CMake configuration files

## Integration with Other Projects

After installation, you can use bgfx in your CMake project:

```cmake
find_package(bgfx REQUIRED)
target_link_libraries(your_target bgfx::bgfx)
```

