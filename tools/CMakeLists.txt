# Tools CMakeLists.txt

# Shaderc tool
add_executable(shaderc
    shaderc/shaderc.cpp
    shaderc/shaderc_glsl.cpp
    shaderc/shaderc_hlsl.cpp
    shaderc/shaderc_metal.cpp
    shaderc/shaderc_pssl.cpp
    shaderc/shaderc_spirv.cpp
    ${CMAKE_SOURCE_DIR}/src/vertexlayout.cpp
    ${CMAKE_SOURCE_DIR}/src/shader.cpp
    ${CMAKE_SOURCE_DIR}/src/shader_dxbc.cpp
    ${CMAKE_SOURCE_DIR}/src/shader_spirv.cpp
)

target_include_directories(shaderc PRIVATE
    ${CMAKE_SOURCE_DIR}/include
    ${CMAKE_SOURCE_DIR}/3rdparty
    ${CMAKE_SOURCE_DIR}/3rdparty/fcpp
    ${CMAKE_SOURCE_DIR}/3rdparty/glslang
    ${CMAKE_SOURCE_DIR}/3rdparty/glslang/glslang/Public
    ${CMAKE_SOURCE_DIR}/3rdparty/glslang/glslang/Include
    ${CMAKE_SOURCE_DIR}/3rdparty/glsl-optimizer/include
    ${CMAKE_SOURCE_DIR}/3rdparty/glsl-optimizer/src/glsl
    ${CMAKE_SOURCE_DIR}/3rdparty/spirv-cross
    ${CMAKE_SOURCE_DIR}/3rdparty/spirv-tools/include
    ${BIMG_DIR}/include
)

target_link_libraries(shaderc PRIVATE
    bx
    bimg
)

# Note: Building shaderc requires many third-party libraries (glslang, spirv-tools, etc.)
# For a complete build, these would need to be added as subdirectories or found packages.
# This is a simplified version that links against the necessary bgfx source files.

if(WIN32)
    target_link_libraries(shaderc PRIVATE psapi)
endif()

if(APPLE)
    find_library(COCOA_LIB Cocoa)
    target_link_libraries(shaderc PRIVATE ${COCOA_LIB})
endif()

if(UNIX AND NOT APPLE)
    target_link_libraries(shaderc PRIVATE pthread)
endif()

# Geometryc tool
add_executable(geometryc
    geometryc/geometryc.cpp
)

target_include_directories(geometryc PRIVATE
    ${CMAKE_SOURCE_DIR}/include
    ${CMAKE_SOURCE_DIR}/3rdparty
    ${BIMG_DIR}/include
)

target_link_libraries(geometryc PRIVATE
    bx
    bimg
)

# Install tools
install(TARGETS shaderc geometryc
    RUNTIME DESTINATION bin
)

