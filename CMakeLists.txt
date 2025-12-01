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

find_package(Threads REQUIRED)
find_package(ZLIB REQUIRED)

target_link_libraries(apriltag_timing
    PRIVATE
        apriltag
        Threads::Threads
        ZLIB::ZLIB
)

find_library(MATH_LIBRARY m)
if (MATH_LIBRARY)
    target_link_libraries(apriltag_timing PRIVATE ${MATH_LIBRARY})
endif()

# if (USE_HALIDE)
#     set(_halide_enabled FALSE)

#     if (CMAKE_VERSION VERSION_GREATER_EQUAL 3.28)
#         find_package(Halide CONFIG QUIET)
#         if (Halide_FOUND)
#             target_sources(apriltag_timing PRIVATE halide_threshold.cpp halide_metal_pipeline.cpp)
#             target_link_libraries(apriltag_timing PRIVATE Halide::Halide)
#             target_compile_definitions(apriltag_timing PRIVATE APRILTAG_HAVE_HALIDE=1)
#             set(_halide_enabled TRUE)
#             message(STATUS "Halide found via CMake package - hybrid detector enabled")
#         endif()
#     endif()

#     if (NOT _halide_enabled)
#         # Manual discovery fallback (works with common package layouts, e.g. Homebrew, source builds)
#         set(_halide_search_paths
#             ${HALIDE_ROOT}
#             $ENV{HALIDE_DIR}
#             /opt/homebrew/opt/halide
#             /usr/local/opt/halide
#             /usr/local
#             /opt/halide
#         )

#         find_path(HALIDE_INCLUDE_DIR
#             NAMES Halide.h
#             HINTS ${_halide_search_paths}
#             PATH_SUFFIXES include
#         )

#         find_library(HALIDE_LIBRARY
#             NAMES Halide
#             HINTS ${_halide_search_paths}
#             PATH_SUFFIXES lib lib64
#         )

#         if (HALIDE_INCLUDE_DIR AND HALIDE_LIBRARY)
#             target_sources(apriltag_timing PRIVATE halide_threshold.cpp halide_metal_pipeline.cpp)
#             target_include_directories(apriltag_timing PRIVATE ${HALIDE_INCLUDE_DIR})
#             target_link_libraries(apriltag_timing PRIVATE ${HALIDE_LIBRARY})
#             target_compile_definitions(apriltag_timing PRIVATE APRILTAG_HAVE_HALIDE=1)
#             set(_halide_enabled TRUE)
#             message(STATUS "Halide located at: include=${HALIDE_INCLUDE_DIR}, lib=${HALIDE_LIBRARY}")
#         endif()
#     endif()

#     if (NOT _halide_enabled)
#         message(WARNING "Halide not found. Re-run CMake with -DUSE_HALIDE=OFF or set HALIDE_ROOT to your Halide install.")
#     endif()
# endif()
