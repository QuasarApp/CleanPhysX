cmake_minimum_required(VERSION 3.16)
project(CleanPhysX C CXX)

if(TARGET ${PROJECT_NAME})
  message("The ${PROJECT_NAME} arledy included in main Project")
  return()
endif()

# Set C++ standard strictly to C++23
set(CMAKE_CXX_STANDARD 23)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

IF(NOT CMAKE_BUILD_TYPE)
  SET(CMAKE_BUILD_TYPE Release)
ENDIF(NOT CMAKE_BUILD_TYPE)

# ---------------------------------------------------------------------------
# 1. Collect all source and header files recursively
# ---------------------------------------------------------------------------
file(GLOB_RECURSE ALL_SOURCES
    CONFIGURE_DEPENDS
    "${CMAKE_CURRENT_SOURCE_DIR}/include/*.h"
    "${CMAKE_CURRENT_SOURCE_DIR}/include/*.hpp"
    "${CMAKE_CURRENT_SOURCE_DIR}/source/*.h"
    "${CMAKE_CURRENT_SOURCE_DIR}/source/*.hpp"
    "${CMAKE_CURRENT_SOURCE_DIR}/source/*.cpp"
    "${CMAKE_CURRENT_SOURCE_DIR}/source/*.c"
)

# Extract platform-specific files
set(WIN_SOURCES ${ALL_SOURCES})
list(FILTER WIN_SOURCES INCLUDE REGEX ".*/windows/.*")

set(UNIX_SOURCES ${ALL_SOURCES})
list(FILTER UNIX_SOURCES INCLUDE REGEX ".*/unix/.*")

# Base sources (excluding platform-specific directories)
set(PHYSX_SOURCES ${ALL_SOURCES})
list(FILTER PHYSX_SOURCES EXCLUDE REGEX ".*/windows/.*")
list(FILTER PHYSX_SOURCES EXCLUDE REGEX ".*/unix/.*")

# Append platform-specific sources based on the target OS
if(WIN32)
    list(APPEND PHYSX_SOURCES ${WIN_SOURCES})
elseif(UNIX)
    list(APPEND PHYSX_SOURCES ${UNIX_SOURCES})
endif()

# Android NDK specific helper files
if(ANDROID AND DEFINED CMAKE_ANDROID_NDK)
    list(APPEND PHYSX_SOURCES
        "${CMAKE_ANDROID_NDK}/sources/android/cpufeatures/cpu-features.h"
        "${CMAKE_ANDROID_NDK}/sources/android/cpufeatures/cpu-features.c"
    )
endif()

# ---------------------------------------------------------------------------
# 2. Dynamic Include Directories Discovery
# ---------------------------------------------------------------------------
file(GLOB_RECURSE ALL_HEADERS
    CONFIGURE_DEPENDS
    "${CMAKE_CURRENT_SOURCE_DIR}/include/*.h"
    "${CMAKE_CURRENT_SOURCE_DIR}/source/*.h"
)

set(ALL_HEADER_DIRS "")
foreach(HEADER_FILE ${ALL_HEADERS})
    get_filename_component(HEADER_DIR ${HEADER_FILE} DIRECTORY)
    get_filename_component(PARENT_DIR ${HEADER_DIR} DIRECTORY)

    # Include both header directory and its parent directory to support relative paths
    list(APPEND ALL_HEADER_DIRS ${HEADER_DIR} ${PARENT_DIR})
endforeach()
list(REMOVE_DUPLICATES ALL_HEADER_DIRS)

# Separate public (include/) and private (source/) include directories
set(PHYSX_PUBLIC_INCLUDES "")
set(PHYSX_PRIVATE_INCLUDES "")

foreach(DIR ${ALL_HEADER_DIRS})
    if(DIR MATCHES ".*/include($|/.*)")
        list(APPEND PHYSX_PUBLIC_INCLUDES ${DIR})
    else()
        list(APPEND PHYSX_PRIVATE_INCLUDES ${DIR})
    endif()
endforeach()
# ---------------------------------------------------------------------------
# 3. Target Definition & Configuration
# ---------------------------------------------------------------------------
add_library(CleanPhysX STATIC ${PHYSX_SOURCES})

target_include_directories(CleanPhysX
    PUBLIC ${PHYSX_PUBLIC_INCLUDES}
    PRIVATE ${PHYSX_PRIVATE_INCLUDES}
)

target_link_libraries(CleanPhysX PRIVATE ${CMAKE_DL_LIBS})

# ---------------------------------------------------------------------------
# 4. Compile Definitions
# ---------------------------------------------------------------------------
target_compile_definitions(CleanPhysX PRIVATE
    PX_COOKING
    PX_PUBLIC_RELEASE=1
    DISABLE_CUDA_PHYSX
    PX_SUPPORT_OMNI_PVD=0
    $<$<CONFIG:Debug>:PX_CHECKED=1>
)

if(WIN32)
    target_compile_definitions(CleanPhysX PRIVATE PX_PHYSX_STATIC_LIB)
endif()

if(MSYS OR MINGW)
    target_compile_definitions(CleanPhysX PUBLIC PX_SIMD_DISABLED PX_GCC_FAMILY)
endif()

if(UNIX OR MINGW)
    target_compile_definitions(CleanPhysX PRIVATE
        $<$<CONFIG:Debug>:_DEBUG>
        $<$<NOT:$<CONFIG:Debug>>:NDEBUG>
    )
endif()

# ---------------------------------------------------------------------------
# 5. Target Properties & Compiler Flags
# ---------------------------------------------------------------------------
set_target_properties(CleanPhysX PROPERTIES
    C_VISIBILITY_PRESET hidden
    CXX_VISIBILITY_PRESET hidden
    VISIBILITY_INLINES_HIDDEN ON
)

# Silence third-party compiler warnings
if(MSVC)
    target_compile_options(CleanPhysX PRIVATE /W0)
else()
    target_compile_options(CleanPhysX PRIVATE -w)
endif()

# iOS specific bitcode flag
if(IOS)
    target_compile_options(CleanPhysX PRIVATE -fembed-bitcode)
endif()