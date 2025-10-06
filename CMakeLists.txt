cmake_minimum_required(VERSION 3.10)
project(TOTP LANGUAGES C CXX)

# Set C++ standard
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Target architecture handling
if(NOT DEFINED TARGET_ARCH OR TARGET_ARCH STREQUAL "")
    set(TARGET_ARCH "x86")
endif()

# Platform-specific compiler flags
if(UNIX)
    # Add -m32 flag only for x86 architecture on Unix platforms
    if(TARGET_ARCH STREQUAL "x86")
        set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -m32")
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -m32")
    endif()

    # Linux-specific flags
    add_compile_definitions(LINUX)
    add_compile_options(-w -fshort-wchar)
    set(CMAKE_C_FLAGS_RELEASE "-O3")
    set(CMAKE_CXX_FLAGS_RELEASE "-O3")
elseif(WIN32)
    # Windows-specific definitions
    add_compile_definitions(WIN32 _WINDOWS _USRDLL TOTP_EXPORTS)
    set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} /O2")
endif()

# Include directories
include_directories(lib/sdk/amx)

# Source files
set(SOURCES
    base32.cpp
    hmac.cpp
    main.cpp
    sha1.cpp
    lib/sdk/amxplugin.cpp
    lib/sdk/amx/getch.c
)

# Create shared library
add_library(TOTP SHARED ${SOURCES})

include(CheckIncludeFile)

check_include_file(alloca.h HAVE_ALLOCA_H)
if(HAVE_ALLOCA_H)
    add_definitions(-DHAVE_ALLOCA_H)
endif()

check_include_file(inttypes.h HAVE_INTTYPES_H)
if(HAVE_INTTYPES_H)
    add_definitions(-DHAVE_INTTYPES_H)
endif()

check_include_file(malloc.h HAVE_MALLOC_H)
if(HAVE_MALLOC_H)
    add_definitions(-DHAVE_MALLOC_H)
endif()

check_include_file(stdint.h HAVE_STDINT_H)
if(HAVE_STDINT_H)
    add_definitions(-DHAVE_STDINT_H)
endif()

add_definitions(-DPAWN_CELL_SIZE=32)

# Windows-specific configuration
if(WIN32)
    # Set module definition file for Win32 platform
    if(CMAKE_SIZEOF_VOID_P EQUAL 4)
        set_target_properties(TOTP PROPERTIES
            LINK_FLAGS "/DEF:${CMAKE_CURRENT_SOURCE_DIR}/TOTP.def"
        )
    endif()

    # Set character set to Unicode
    target_compile_definitions(TOTP PRIVATE UNICODE _UNICODE)
endif()

# Set output name
if(UNIX)
    set_target_properties(TOTP PROPERTIES
        PREFIX ""
        OUTPUT_NAME "TOTP"
        SUFFIX ".so"
    )
elseif(WIN32)
    set_target_properties(TOTP PROPERTIES
        OUTPUT_NAME "TOTP"
    )
endif()
