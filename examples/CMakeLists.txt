# Examples CMakeLists.txt
# This is a simplified version - a full implementation would build all examples

# Example common library
add_library(example-common STATIC
    common/example-glue.cpp
    common/imgui/imgui.cpp
    common/imgui/imgui_demo.cpp
    common/imgui/imgui_draw.cpp
    common/imgui/imgui_tables.cpp
    common/imgui/imgui_widgets.cpp
    # Add other common source files as needed
)

target_include_directories(example-common PUBLIC
    ${CMAKE_SOURCE_DIR}/include
    ${CMAKE_SOURCE_DIR}/3rdparty
    ${CMAKE_SOURCE_DIR}/examples/common
    ${BIMG_DIR}/include
)

target_compile_definitions(example-common PRIVATE USE_ENTRY=1)

target_link_libraries(example-common PUBLIC
    bx
    bimg
    bgfx
)

if(BGFX_BUILD_WITH_SDL)
    find_package(SDL2 REQUIRED)
    target_compile_definitions(example-common PRIVATE ENTRY_CONFIG_USE_SDL=1)
    target_link_libraries(example-common PUBLIC SDL2::SDL2)
endif()

if(BGFX_BUILD_WITH_GLFW)
    find_package(glfw3 REQUIRED)
    target_compile_definitions(example-common PRIVATE ENTRY_CONFIG_USE_GLFW=1)
    target_link_libraries(example-common PUBLIC glfw)
endif()

# Platform-specific settings
if(WIN32)
    target_link_libraries(example-common PRIVATE
        gdi32
        psapi
    )
endif()

if(UNIX AND NOT APPLE)
    target_link_libraries(example-common PRIVATE
        X11
        GL
        pthread
    )
endif()

if(APPLE)
    find_library(COCOA_LIB Cocoa)
    find_library(IOKIT_LIB IOKit)
    find_library(OPENGL_LIB OpenGL)
    find_library(QUARTZCORE_LIB QuartzCore)
    target_link_libraries(example-common PRIVATE
        ${COCOA_LIB}
        ${IOKIT_LIB}
        ${OPENGL_LIB}
        ${QUARTZCORE_LIB}
    )
endif()

# Example: Hello World
function(add_bgfx_example name)
    add_executable(${name}
        ${name}/${name}.cpp
    )
    
    target_include_directories(${name} PRIVATE
        ${CMAKE_SOURCE_DIR}/examples/common
    )
    
    target_compile_definitions(${name} PRIVATE ENTRY_CONFIG_IMPLEMENT_MAIN=1)
    
    target_link_libraries(${name} PRIVATE
        example-common
    )
    
    # Set working directory for examples
    set_target_properties(${name} PROPERTIES
        VS_DEBUGGER_WORKING_DIRECTORY "${CMAKE_SOURCE_DIR}/examples/runtime"
    )
endfunction()

# Build specific examples (can be extended)
if(BGFX_BUILD_EXAMPLES)
    # Add examples here
    # add_bgfx_example(00-helloworld)
    # add_bgfx_example(01-cubes)
    # etc.
endif()

