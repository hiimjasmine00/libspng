cmake_minimum_required(VERSION 3.12)

project(libspng C)

set(CMAKE_C_STANDARD 99)

set(SPNG_MAJOR 0)
set(SPNG_MINOR 7)
set(SPNG_REVISION 4)
set(SPNG_VERSION ${SPNG_MAJOR}.${SPNG_MINOR}.${SPNG_REVISION})

option(ENABLE_OPT "Enable architecture-specific optimizations" ON)
option(SPNG_SHARED "Build shared lib" OFF)
option(SPNG_STATIC "Build static lib" ON)
option(BUILD_EXAMPLES "Build examples" OFF)

include(GNUInstallDirs)
include(CMakePackageConfigHelpers)

if(NOT CMAKE_HOST_WIN32)
    set(MATH_LIBRARY "m")
else()
    set(MATH_LIBRARY "")
endif()

if(NOT ENABLE_OPT)
    add_definitions( -DSPNG_DISABLE_OPT=1 )
endif()

set(spng_TARGETS "")

set(spng_SOURCES spng/spng.c)

if(SPNG_SHARED)
    add_library(spng SHARED ${spng_SOURCES})
    list(APPEND spng_TARGETS spng)

    if(BUILD_EXAMPLES)
        add_executable(example examples/example.c)
        target_link_libraries(example PRIVATE spng)
    endif()
endif()

if(SPNG_STATIC)
    add_library(spng_static STATIC ${spng_SOURCES})
    target_compile_definitions(spng_static PUBLIC SPNG_STATIC)
    list(APPEND spng_TARGETS spng_static)
endif()

foreach(spng_TARGET ${spng_TARGETS})
    target_include_directories(${spng_TARGET} PUBLIC
      $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/spng>
      $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
    )
    target_link_libraries(${spng_TARGET} PRIVATE ${MATH_LIBRARY})
endforeach()
