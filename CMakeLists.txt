cmake_minimum_required(VERSION 3.16)

project(apriltags_halide
        DESCRIPTION "AprilTag timing harness with optional Halide acceleration"
        LANGUAGES C CXX)

add_subdirectory(apriltag)
# include(FetchContent)
# FetchContent_Declare(
#     apriltag
#     GIT_REPOSITORY https://github.com/AprilRobotics/apriltag
#     GIT_TAG 22b44c13c6f0b68d7eaa1504cec0b8a20850122e # latest as of 12/1/25
# )
# FetchContent_MakeAvailable(apriltag)

set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

set(CMAKE_BUILD_TYPE RelWithDebInfo CACHE STRING "Build type" FORCE)

option(USE_HALIDE "Build the Halide-backed threshold pipeline" ON)
set(HALIDE_ROOT "" CACHE PATH "Root path to a Halide installation (used when auto-detection fails)")

add_executable(apriltag_timing
    apriltag_timing.cpp
)

set_property(TARGET apriltag_timing PROPERTY CMAKE_C_STANDARD 99)
set_property(TARGET apriltag_timing PROPERTY CMAKE_C_STANDARD_REQUIRED ON)
set_property(TARGET apriltag_timing PROPERTY CMAKE_C_EXTENSIONS OFF)

set_property(TARGET apriltag_timing PROPERTY CMAKE_CXX_STANDARD 17)
set_property(TARGET apriltag_timing PROPERTY CMAKE_CXX_STANDARD_REQUIRED ON)
set_property(TARGET apriltag_timing PROPERTY CMAKE_CXX_EXTENSIONS OFF)

target_include_directories(apriltag_timing
    PRIVATE
        ${CMAKE_CURRENT_SOURCE_DIR}
)

include(FetchContent)

FetchContent_Declare(
  fmt
  GIT_REPOSITORY https://github.com/fmtlib/fmt
  GIT_TAG        e69e5f977d458f2650bb346dadf2ad30c5320281) # 10.2.1
FetchContent_MakeAvailable(fmt)

find_package(Threads REQUIRED)
find_package(ZLIB REQUIRED)

target_link_libraries(apriltag_timing
    PRIVATE
        apriltag
        Threads::Threads
        ZLIB::ZLIB
        fmt::fmt
)

find_library(MATH_LIBRARY m)
if (MATH_LIBRARY)
    target_link_libraries(apriltag_timing PRIVATE ${MATH_LIBRARY})
endif()
