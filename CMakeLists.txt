#
# (c) LetsMakeGames 2021
# http://www.letsmake.games
#

cmake_minimum_required(VERSION 3.15)
cmake_policy(SET CMP0091 NEW) # Allows setting MSVC_RUNTIME_LIBRARY
cmake_policy(SET CMP0042 NEW)

#
# project configuration #######################################################
#

set(PROJECT_NAME HashLink)
set(HL_VERSION_MAJOR 1)
set(HL_VERSION_MINOR 12)
set(HL_VERSION_PATCH 0)
set(HL_VERSION ${HL_VERSION_MAJOR}.${HL_VERSION_MINOR}.${HL_VERSION_PATCH})

project(${PROJECT_NAME}
    VERSION ${HL_VERSION}
        LANGUAGES C CXX
)

if(BT_TESTING)
    set(BUILD_TESTS_DEFAULT ON)
else()
    set(BUILD_TESTS_DEFAULT OFF)
endif()

option(BUILD_TESTS "Build the tests" ${BUILD_TESTS_DEFAULT})
if(BUILD_TESTS)
    enable_testing()
endif()

set(BT_CXX_VERSION 17)

#
# includes ####################################################################
#

if(NOT BT_CMAKE_DIR)
    set(BT_CMAKE_DIR ${CMAKE_CURRENT_LIST_DIR}/extern/cmake-utils CACHE STRING "The top level cmake utilities directory")
endif()

# a collection of add_library add_executable and add_test macros to make adding
# targets easier
include("${BT_CMAKE_DIR}/TargetMacros.cmake")
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY "${CMAKE_CURRENT_LIST_DIR}/bin/${BT_PLATFORM}/${BT_BUILD_TYPE}")
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY "${CMAKE_CURRENT_LIST_DIR}/bin/${BT_PLATFORM}/${BT_BUILD_TYPE}")
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${CMAKE_CURRENT_LIST_DIR}/bin/${BT_PLATFORM}/${BT_BUILD_TYPE}")

# because windows is a bum
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY_DEBUG "${CMAKE_RUNTIME_OUTPUT_DIRECTORY}")
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY_RELEASE "${CMAKE_RUNTIME_OUTPUT_DIRECTORY}")
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY_DEBUG "${CMAKE_LIBRARY_OUTPUT_DIRECTORY}")
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY_RELEASE "${CMAKE_LIBRARY_OUTPUT_DIRECTORY}")
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY_DEBUG "${CMAKE_ARCHIVE_OUTPUT_DIRECTORY}")
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY_RELEASE "${CMAKE_ARCHIVE_OUTPUT_DIRECTORY}")

#
# more includes
#

include(GNUInstallDirs)
include(FindPkgConfig)
include(CTest)

#
# settings ####################################################################
#

# force Unicode over Multi-byte
if(MSVC)
    add_definitions(-DUNICODE -D_UNICODE)
endif()

list(APPEND CMAKE_MODULE_PATH
    ${CMAKE_CURRENT_SOURCE_DIR}/other/cmake
)

if(CMAKE_VERSION VERSION_LESS "3.1")
    set(CMAKE_C_FLAGS "-std=c11 ${CMAKE_C_FLAGS}")
else()
    set(CMAKE_C_STANDARD 11)
    set(CMAKE_C_STANDARD_REQUIRED ON)
endif()

#
# add dependencies ############################################################
#

add_subdirectory(extern)
add_subdirectory(libs)

#
# libhl #######################################################################
#

file(GLOB pcre_srcs
    include/pcre/*.c
)

file(GLOB std_srcs
    src/std/array.c
    src/std/buffer.c
    src/std/bytes.c
    src/std/cast.c
    src/std/date.c
    src/std/error.c
    src/std/file.c
    src/std/fun.c
    src/std/maps.c
    src/std/math.c
    src/std/obj.c
    src/std/random.c
    src/std/regexp.c
    src/std/socket.c
    src/std/string.c
    src/std/sys.c
    src/std/track.c
    src/std/types.c
    src/std/ucs2.c
    src/std/thread.c
    src/std/process.c
)

if(CMAKE_SYSTEM_NAME MATCHES "Darwin")
    list(APPEND std_srcs
        include/mdbg/mdbg.c
        include/mdbg/mach_excServer.c
        include/mdbg/mach_excUser.c
    )
endif()

if(ANDROID)
    list(APPEND std_srcs
        src/std/sys_android.c
    )
endif()

if(IOS_PLATFORM)
    list(APPEND std_srcs
        src/std/sys_ios.m
    )
endif()

add_library(libhl SHARED
    ${pcre_srcs}
    src/gc.c
    ${std_srcs}
)

target_include_directories(libhl PUBLIC
        src
        include
        include/pcre
)

target_link_libraries(libhl
        OpenAL
)

if(WIN32)
    set_target_properties(libhl
        PROPERTIES
        OUTPUT_NAME libhl
    )
else()
    set_target_properties(libhl
        PROPERTIES
        OUTPUT_NAME hl
    )
endif()

set_target_properties(libhl
    PROPERTIES
    VERSION ${HL_VERSION}
    SOVERSION ${HL_VERSION_MAJOR}
    COMPILE_DEFINITIONS "_USRDLL;LIBHL_EXPORTS"
)

#
# executable ##################################################################
#

add_executable(hl 
    src/code.c
    src/jit.c
    src/main.c
    src/module.c
    src/debugger.c
    src/profile.c
)

target_link_libraries(hl libhl)

if(WIN32)
    target_link_libraries(libhl ws2_32 user32)
    target_link_libraries(hl user32)
else()
    target_link_libraries(libhl m dl pthread)
endif()
