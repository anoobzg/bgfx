# CMake 构建指南

本文档说明如何使用 CMake 构建系统来构建 bgfx 项目。

## 前置要求

1. **CMake 3.15 或更高版本**
   - Windows: 从 https://cmake.org/download/ 下载安装
   - Linux: `sudo apt-get install cmake` 或 `sudo yum install cmake`
   - macOS: `brew install cmake`

2. **C++11 兼容的编译器**
   - Windows: Visual Studio 2019 或更高版本
   - Linux: GCC 11+ 或 Clang 11+
   - macOS: Xcode 12+ 或 Clang 12+

3. **依赖库**
   - **bx**: https://github.com/bkaradzic/bx
   - **bimg**: https://github.com/bkaradzic/bimg

## 快速开始

### 1. 克隆依赖库

```bash
# 假设 bgfx 在 bgfx/ 目录
cd ..
git clone https://github.com/bkaradzic/bx.git
git clone https://github.com/bkaradzic/bimg.git
cd bgfx
```

### 2. 配置构建

```bash
mkdir build
cd build
cmake ..
```

如果依赖库不在默认位置（`../bx` 和 `../bimg`），可以指定路径：

```bash
cmake -DBX_DIR=/path/to/bx -DBIMG_DIR=/path/to/bimg ..
```

### 3. 构建

```bash
# 使用 CMake 构建
cmake --build .

# 或使用平台特定的构建工具
# Windows (Visual Studio):
#   打开生成的 .sln 文件
# Linux/macOS:
make
```

## CMake 选项

### 主要选项

- `BGFX_BUILD_TOOLS` (默认: ON)
  - 构建 bgfx 工具（shaderc, geometryc 等）

- `BGFX_BUILD_EXAMPLES` (默认: OFF)
  - 构建 bgfx 示例程序

- `BGFX_BUILD_SHARED_LIB` (默认: OFF)
  - 构建共享库而不是静态库

- `BGFX_BUILD_AMALGAMATED` (默认: OFF)
  - 使用合并构建（将所有源文件合并为一个文件）

- `BGFX_BUILD_WITH_SDL` (默认: OFF)
  - 使用 SDL 作为窗口系统

- `BGFX_BUILD_WITH_GLFW` (默认: OFF)
  - 使用 GLFW 作为窗口系统

- `BGFX_BUILD_WITH_PROFILER` (默认: OFF)
  - 启用性能分析器支持

### 使用示例

```bash
# 构建示例
cmake -DBGFX_BUILD_EXAMPLES=ON ..

# 构建共享库
cmake -DBGFX_BUILD_SHARED_LIB=ON ..

# 使用 GLFW 构建示例
cmake -DBGFX_BUILD_EXAMPLES=ON -DBGFX_BUILD_WITH_GLFW=ON ..
```

## 平台特定说明

### Windows

1. 安装 Visual Studio 2019 或更高版本
2. 确保安装了 Windows SDK
3. DirectX SDK（用于 D3D11/D3D12 渲染器）

```bash
# 使用 Visual Studio 生成器
cmake -G "Visual Studio 16 2019" -A x64 ..
cmake --build . --config Release
```

### Linux

安装必要的开发库：

```bash
# Ubuntu/Debian
sudo apt-get install libx11-dev libgl1-mesa-dev libpthread-stubs0-dev

# Fedora
sudo dnf install libX11-devel mesa-libGL-devel
```

```bash
cmake ..
make -j$(nproc)
```

### macOS

1. 安装 Xcode Command Line Tools: `xcode-select --install`
2. 使用 Homebrew 安装依赖（可选）: `brew install cmake`

```bash
cmake ..
make -j$(sysctl -n hw.ncpu)
```

### Android

需要 Android NDK：

```bash
export ANDROID_NDK=/path/to/android-ndk
cmake -DCMAKE_TOOLCHAIN_FILE=$ANDROID_NDK/build/cmake/android.toolchain.cmake ..
```

## 渲染器后端

CMake 会根据目标平台自动配置渲染器后端：

- **Direct3D 11/12**: Windows, Linux (WSL)
- **OpenGL**: Linux, Windows
- **OpenGL ES**: Android, Emscripten
- **Vulkan**: Linux, Windows, macOS, Android
- **Metal**: macOS, iOS

## 安装

安装到系统目录：

```bash
cmake --build . --target install
```

默认安装路径：
- Linux: `/usr/local`
- Windows: `C:\Program Files`
- macOS: `/usr/local`

自定义安装路径：

```bash
cmake -DCMAKE_INSTALL_PREFIX=/custom/path ..
cmake --build . --target install
```

## 在其他项目中使用

安装后，可以在你的 CMake 项目中使用：

```cmake
find_package(bgfx REQUIRED)
target_link_libraries(your_target bgfx::bgfx)
```

## 故障排除

### 找不到 bx 或 bimg

确保依赖库已克隆，或使用 `-DBX_DIR` 和 `-DBIMG_DIR` 指定路径。

### 编译错误

1. 确保使用支持的编译器版本
2. 检查所有依赖库是否正确构建
3. 查看 CMake 输出中的错误信息

### 链接错误

1. 确保所有必要的系统库已安装
2. 检查平台特定的依赖（如 Windows 的 DirectX SDK）

## 与 Genie 构建系统的对比

CMake 构建系统提供了与原始 Genie 构建系统类似的功能：

| 功能 | Genie | CMake |
|------|-------|-------|
| 静态库 | ✅ | ✅ |
| 共享库 | ✅ | ✅ |
| 工具构建 | ✅ | ✅ |
| 示例构建 | ✅ | ✅ |
| 多平台支持 | ✅ | ✅ |
| 渲染器配置 | ✅ | ✅ |

CMake 的优势：
- 更广泛的项目集成支持
- 更好的 IDE 支持
- 标准的跨平台构建系统

## 贡献

如果发现 CMake 构建系统的问题，请提交 Issue 或 Pull Request。

