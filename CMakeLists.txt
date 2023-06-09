# Based off https://github.com/friendlyanon/cmake-init-gif-engine/blob/b1bf3b75c1e724ceadc4eaf11bd61a29ddadd5a5/fuzz/CMakeLists.txt
cmake_minimum_required(VERSION 3.20.0)

project(GHASuperBuild NONE)

set(GHASuperBuild_INSTALL_PREFIX ${CMAKE_BINARY_DIR}/install CACHE PATH "Installation prefix")

# Allow the Ninja generators to output messages as they happen by assigning
# these jobs to the 'console' job pool
set(console_access "")
if(CMAKE_GENERATOR MATCHES "^Ninja")
  set(
      console_access
      USES_TERMINAL_CONFIGURE YES
      USES_TERMINAL_BUILD YES
      USES_TERMINAL_INSTALL YES
  )
endif()

set(toolchain "")
if(DEFINED CMAKE_TOOLCHAIN_FILE)
  set(toolchain "-DCMAKE_TOOLCHAIN_FILE:STRING=${CMAKE_TOOLCHAIN_FILE}")
endif()

set(common_cache_flags
    -DCMAKE_BUILD_TYPE:STRING=${CMAKE_BUILD_TYPE}
    -DCMAKE_VERBOSE_MAKEFILE:BOOL=ON
)

set(local_ext_proj_flags
    BUILD_ALWAYS YES
)
set(ci FALSE)
if(DEFINED ENV{GITHUB_ACTIONS})
    set(ci TRUE)
endif()

include(ExternalProject)

# **** fmt ****
ExternalProject_Add(
    fmt
    SOURCE_DIR "${PROJECT_SOURCE_DIR}/fmt"
    CMAKE_CACHE_ARGS
    -DFMT_TEST:BOOL=OFF
    -DFMT_DOC:BOOL=OFF
    -DCMAKE_INSTALL_PREFIX:PATH=${GHASuperBuild_INSTALL_PREFIX}/fmt
    ${toolchain}
    ${common_cache_flags}
    ${local_ext_proj_flags}
    EXCLUDE_FROM_ALL YES
    ${console_access}
    CONFIGURE_HANDLED_BY_BUILD YES
)
ExternalProject_Get_Property(fmt install_dir)
set(fmt_install_dir "${install_dir}")

# **** spdlog ****
set(spdlog_local_ext_proj_flags
    DEPENDS fmt
)
if(ci)
    set(spdlog_local_ext_proj_flags "")
endif()

ExternalProject_Add(
    spdlog
    SOURCE_DIR "${PROJECT_SOURCE_DIR}/spdlog"
    CMAKE_CACHE_ARGS
    ${toolchain}
    -DSPDLOG_FMT_EXTERNAL:BOOL=ON
    -Dfmt_ROOT:PATH=${GHASuperBuild_INSTALL_PREFIX}/fmt
    -DSPDLOG_BUILD_EXAMPLE:BOOL=OFF
    -DCMAKE_INSTALL_PREFIX:PATH=${GHASuperBuild_INSTALL_PREFIX}/spdlog
    ${common_cache_flags}
    ${local_ext_proj_flags}
    ${spdlog_local_ext_proj_flags}
    EXCLUDE_FROM_ALL YES
    ${console_access}
    CONFIGURE_HANDLED_BY_BUILD YES
)
