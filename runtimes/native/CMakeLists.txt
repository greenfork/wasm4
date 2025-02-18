cmake_minimum_required(VERSION 3.12)
project(WASM4)

# Prevent BUILD_SHARED_LIBS and other options from being cleared by vendor CMakeLists
# https://stackoverflow.com/a/66342383
set(CMAKE_POLICY_DEFAULT_CMP0077 NEW)

# Enable LTO in release builds
set(CMAKE_POLICY_DEFAULT_CMP0069 NEW)
if (CMAKE_BUILD_TYPE STREQUAL "Release")
    include(CheckIPOSupported)
    check_ipo_supported()
    set(CMAKE_INTERPROCEDURAL_OPTIMIZATION TRUE)
endif ()

# GLFW options
set(BUILD_SHARED_LIBS OFF)
# set(GLFW_BUILD_DOCS OFF CACHE BOOL "" FORCE)
# set(GLFW_BUILD_TESTS OFF CACHE BOOL "" FORCE)
# set(GLFW_BUILD_EXAMPLES OFF CACHE BOOL "" FORCE)

# MiniFB options
set(MINIFB_BUILD_EXAMPLES OFF)
set(USE_OPENGL_API OFF)
# set(USE_METAL_API OFF)

# add_subdirectory(vendor/glfw)
add_subdirectory(vendor/minifb)
add_subdirectory(vendor/portaudio)

file(GLOB COMMON_SOURCES RELATIVE "${CMAKE_SOURCE_DIR}" "src/*.c")
file(GLOB M3_SOURCES RELATIVE "${CMAKE_SOURCE_DIR}" "vendor/wasm3/source/*.c")

#
# Desktop (minifb + portaudio) backend
#

set(MINIFB_SOURCES
    src/backend/main.c
    src/backend/wasm_wasm3.c
    src/backend/window_minifb.c
)
add_executable(wasm4 ${COMMON_SOURCES} ${MINIFB_SOURCES} ${M3_SOURCES})
target_include_directories(wasm4 PRIVATE "${CMAKE_SOURCE_DIR}/vendor/wasm3/source")
target_link_libraries(wasm4 minifb PortAudio)
set_target_properties(wasm4 PROPERTIES C_STANDARD 99)
install(TARGETS wasm4)

# #
# # Desktop (glfw + portaudio) backend
# #
# 
# set(GLFW_SOURCES
#     src/backend/main.c
#     src/backend/wasm_wasm3.c
#     src/backend/window_glfw.c
#     vendor/glad/src/glad.c
# )
# add_executable(wasm4_glfw)
# target_sources(wasm4_glfw PRIVATE ${COMMON_SOURCES} ${GLFW_SOURCES} ${M3_SOURCES})
# target_include_directories(wasm4_glfw PRIVATE "${CMAKE_SOURCE_DIR}/vendor/wasm3/source")
# target_include_directories(wasm4_glfw PRIVATE "${CMAKE_SOURCE_DIR}/vendor/glad/include")
# target_link_libraries(wasm4_glfw glfw PortAudio)
# set_target_properties(wasm4_glfw PROPERTIES C_STANDARD 99)
# install(TARGETS wasm4_glfw)

#
# Libretro backend
#
set(LIBRETRO_SOURCES
    src/backend/main_libretro.c
    src/backend/wasm_wasm3.c
)
add_library(wasm4_libretro SHARED ${COMMON_SOURCES} ${LIBRETRO_SOURCES} ${M3_SOURCES})
target_include_directories(wasm4_libretro PRIVATE "${CMAKE_SOURCE_DIR}/vendor/wasm3/source")
target_include_directories(wasm4_libretro PRIVATE "${CMAKE_SOURCE_DIR}/vendor/libretro/include")
set_target_properties(wasm4_libretro PROPERTIES C_STANDARD 99)
install(TARGETS wasm4_libretro)

# Hide all symbols by default
set_target_properties(wasm4_libretro PROPERTIES
    C_VISIBILITY_PRESET hidden
    VISIBILITY_INLINES_HIDDEN 1)

# Don't prefix with lib*
set_target_properties(wasm4_libretro PROPERTIES PREFIX "")
