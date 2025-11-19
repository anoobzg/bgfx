cmake_minimum_required(VERSION 3.15)

project(bgfx
    VERSION 1.0.0
    LANGUAGES CXX C
    DESCRIPTION "Cross-platform rendering library"
)

# Set C++ standard
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

# Options
option(BGFX_BUILD_TOOLS "Build bgfx tools" ON)
option(BGFX_BUILD_EXAMPLES "Build bgfx examples" OFF)
option(BGFX_BUILD_SHARED_LIB "Build bgfx as shared library" OFF)
option(BGFX_BUILD_AMALGAMATED "Build bgfx as amalgamated" OFF)
option(BGFX_BUILD_WITH_SDL "Build with SDL entry" OFF)
option(BGFX_BUILD_WITH_GLFW "Build with GLFW entry" OFF)
option(BGFX_BUILD_WITH_PROFILER "Build with profiler" OFF)

# Find dependencies
set(BX_DIR "${CMAKE_CURRENT_SOURCE_DIR}/../bx" CACHE PATH "Path to bx library")
set(BIMG_DIR "${CMAKE_CURRENT_SOURCE_DIR}/../bimg" CACHE PATH "Path to bimg library")

if(NOT EXISTS "${BX_DIR}")
    message(FATAL_ERROR "bx not found at \"${BX_DIR}\". Set BX_DIR or clone https://github.com/bkaradzic/bx")
endif()

if(NOT EXISTS "${BIMG_DIR}")
    message(FATAL_ERROR "bimg not found at \"${BIMG_DIR}\". Set BIMG_DIR or clone https://github.com/bkaradzic/bimg")
endif()

# Include directories
include_directories(
    ${CMAKE_CURRENT_SOURCE_DIR}/include
    ${CMAKE_CURRENT_SOURCE_DIR}/3rdparty
    ${BIMG_DIR}/include
)

# Platform-specific includes
if(UNIX AND NOT APPLE)
    include_directories(
        ${CMAKE_CURRENT_SOURCE_DIR}/3rdparty/directx-headers/include/directx
        ${CMAKE_CURRENT_SOURCE_DIR}/3rdparty/directx-headers/include
        ${CMAKE_CURRENT_SOURCE_DIR}/3rdparty/directx-headers/include/wsl/stubs
        ${CMAKE_CURRENT_SOURCE_DIR}/3rdparty/khronos
    )
endif()

if(WIN32 OR MINGW)
    include_directories(
        ${CMAKE_CURRENT_SOURCE_DIR}/3rdparty/directx-headers/include/directx
    )
endif()

# Configure renderer backends based on platform
include(cmake/ConfigureRenderers.cmake)

# Apply renderer defines (will be set after target creation)
set(BGFX_COMPILE_DEFINITIONS_TEMP ${BGFX_COMPILE_DEFINITIONS})

# Source files
set(BGFX_SOURCES
    src/bgfx.cpp
    src/topology.cpp
    src/vertexlayout.cpp
    src/shader.cpp
    src/shader_dxbc.cpp
    src/shader_spirv.cpp
    src/dxgi.cpp
    src/nvapi.cpp
    src/debug_renderdoc.cpp
)

# Renderer sources (conditionally added)
if(BGFX_CONFIG_RENDERER_DIRECT3D11)
    list(APPEND BGFX_SOURCES src/renderer_d3d11.cpp)
endif()

if(BGFX_CONFIG_RENDERER_DIRECT3D12)
    list(APPEND BGFX_SOURCES src/renderer_d3d12.cpp)
endif()

if(BGFX_CONFIG_RENDERER_OPENGL OR BGFX_CONFIG_RENDERER_OPENGLES)
    list(APPEND BGFX_SOURCES src/renderer_gl.cpp)
    
    if(WIN32)
        list(APPEND BGFX_SOURCES src/glcontext_wgl.cpp)
    elseif(UNIX AND NOT APPLE)
        list(APPEND BGFX_SOURCES src/glcontext_egl.cpp)
    elseif(APPLE)
        list(APPEND BGFX_SOURCES src/glcontext_egl.cpp)
    endif()
endif()

if(BGFX_CONFIG_RENDERER_VULKAN)
    list(APPEND BGFX_SOURCES src/renderer_vk.cpp)
endif()

if(BGFX_CONFIG_RENDERER_METAL)
    list(APPEND BGFX_SOURCES src/renderer_mtl.mm)
endif()

if(BGFX_CONFIG_RENDERER_NOOP)
    list(APPEND BGFX_SOURCES src/renderer_noop.cpp)
endif()

# Handle amalgamated build mode
if(BGFX_BUILD_AMALGAMATED)
    # Remove individual renderer files when using amalgamated
    set(AMALGAMATED_EXCLUDE
        src/bgfx.cpp
        src/debug_renderdoc.cpp
        src/dxgi.cpp
        src/glcontext_egl.cpp
        src/glcontext_html5.cpp
        src/glcontext_wgl.cpp
        src/nvapi.cpp
        src/renderer_d3d11.cpp
        src/renderer_d3d12.cpp
        src/renderer_gl.cpp
        src/renderer_vk.cpp
        src/shader_dxbc.cpp
        src/shader_spirv.cpp
        src/topology.cpp
        src/vertexlayout.cpp
    )
    foreach(EXCLUDE_FILE ${AMALGAMATED_EXCLUDE})
        list(REMOVE_ITEM BGFX_SOURCES ${EXCLUDE_FILE})
    endforeach()
    
    # Add amalgamated file
    if(APPLE)
        list(APPEND BGFX_SOURCES src/amalgamated.mm)
    else()
        list(APPEND BGFX_SOURCES src/amalgamated.cpp)
    endif()
endif()

# Header files (for IDE support)
file(GLOB BGFX_HEADERS
    include/bgfx/*.h
    src/*.h
)

# Create bgfx library
if(BGFX_BUILD_SHARED_LIB)
    add_library(bgfx SHARED ${BGFX_SOURCES} ${BGFX_HEADERS})
    target_compile_definitions(bgfx PRIVATE BGFX_SHARED_LIB_BUILD=1)
else()
    add_library(bgfx STATIC ${BGFX_SOURCES} ${BGFX_HEADERS})
endif()

# Apply renderer defines
target_compile_definitions(bgfx PRIVATE ${BGFX_COMPILE_DEFINITIONS_TEMP})

target_include_directories(bgfx
    PUBLIC
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
        $<INSTALL_INTERFACE:include>
    PRIVATE
        ${CMAKE_CURRENT_SOURCE_DIR}/src
        ${CMAKE_CURRENT_SOURCE_DIR}/3rdparty
        ${BIMG_DIR}/include
)

# Link bx library
find_package(bx QUIET)
if(NOT bx_FOUND)
    add_subdirectory(${BX_DIR} ${CMAKE_BINARY_DIR}/bx EXCLUDE_FROM_ALL)
endif()
target_link_libraries(bgfx PUBLIC bx)

# Link bimg library
find_package(bimg QUIET)
if(NOT bimg_FOUND)
    add_subdirectory(${BIMG_DIR} ${CMAKE_BINARY_DIR}/bimg EXCLUDE_FROM_ALL)
endif()
target_link_libraries(bgfx PUBLIC bimg)

# Platform-specific settings
if(WIN32)
    target_link_libraries(bgfx PRIVATE gdi32 psapi)
    if(BGFX_BUILD_SHARED_LIB)
        set_target_properties(bgfx PROPERTIES
            WINDOWS_EXPORT_ALL_SYMBOLS ON
        )
    endif()
elseif(UNIX AND NOT APPLE)
    target_link_libraries(bgfx PRIVATE X11 GL pthread)
    if(BGFX_BUILD_SHARED_LIB)
        target_compile_options(bgfx PRIVATE -fPIC)
    endif()
elseif(APPLE)
    find_library(COCOA_LIB Cocoa)
    find_library(IOKIT_LIB IOKit)
    find_library(OPENGL_LIB OpenGL)
    find_library(QUARTZCORE_LIB QuartzCore)
    target_link_libraries(bgfx PRIVATE
        ${COCOA_LIB}
        ${IOKIT_LIB}
        ${OPENGL_LIB}
        ${QUARTZCORE_LIB}
    )
    if(BGFX_CONFIG_RENDERER_METAL)
        find_library(METAL_LIB Metal)
        find_library(METALKIT_LIB MetalKit)
        target_link_libraries(bgfx PRIVATE
            ${METAL_LIB}
            ${METALKIT_LIB}
        )
    endif()
    # Enable Objective-C++ for .mm files
    set_source_files_properties(src/renderer_mtl.mm PROPERTIES
        COMPILE_FLAGS "-x objective-c++"
    )
endif()

# Android specific
if(ANDROID)
    target_link_libraries(bgfx PRIVATE EGL GLESv2)
endif()

# Compiler-specific options
if(MSVC)
    target_compile_options(bgfx PRIVATE
        /W4
        /wd4127  # conditional expression is constant
        /wd4201  # nameless struct/union
        /wd4100  # unreferenced formal parameter
    )
else()
    target_compile_options(bgfx PRIVATE
        -Wall
        -Wextra
        -Wno-microsoft-enum-value
        -Wno-microsoft-const-init
    )
endif()

# Clang-specific
if(CMAKE_CXX_COMPILER_ID MATCHES "Clang")
    target_compile_options(bgfx PRIVATE
        -Wno-microsoft-enum-value
        -Wno-microsoft-const-init
    )
endif()

# Build tools
if(BGFX_BUILD_TOOLS)
    add_subdirectory(tools)
endif()

# Build examples
if(BGFX_BUILD_EXAMPLES)
    add_subdirectory(examples)
endif()

# Install
install(TARGETS bgfx
    EXPORT bgfxTargets
    LIBRARY DESTINATION lib
    ARCHIVE DESTINATION lib
    RUNTIME DESTINATION bin
    INCLUDES DESTINATION include
)

install(DIRECTORY include/bgfx
    DESTINATION include
    FILES_MATCHING PATTERN "*.h"
)

install(EXPORT bgfxTargets
    FILE bgfxTargets.cmake
    NAMESPACE bgfx::
    DESTINATION lib/cmake/bgfx
)

