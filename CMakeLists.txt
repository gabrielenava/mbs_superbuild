# MBS superbuild - cmake file for building the project
#
# Author: Gabriele Nava
#
# First release: 2020
#
# CopyPolicy: Released under the terms of the LGPLv2.1 or later, see LGPL.TXT
#
cmake_minimum_required(VERSION 3.0)
project(mbs_superbuild NONE)

# We have to enable C because it is currently used by CMake to describe packages (in Config*.cmake files)
enable_language(C)

# Disable "in source" build
if("${PROJECT_SOURCE_DIR}" STREQUAL "${PROJECT_BINARY_DIR}")
   message(FATAL_ERROR "In-source builds of mbs_superbuild are not allowed. "
                       "Please remove CMakeCache.txt and the CMakeFiles/ directory, then build out-of-source. "
                       "If you really want to do an in-source build, feel free to comment out this error.")
endif("${PROJECT_SOURCE_DIR}" STREQUAL "${PROJECT_BINARY_DIR}")

list(APPEND CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}/cmake")

include(CTest)
include(FeatureSummary)

set(YCM_USE_CMAKE_PROPOSED TRUE CACHE BOOL "Use files including unmerged cmake patches")

# Compilation options

# Simulator options
option(USES_EXTERNAL "Enable the download of external (optional) Matlab projects" FALSE)
option(USES_MODELS "Enable the download of the available urdf models" TRUE)
option(USES_APP "Enable the download of existing simulations" TRUE)

# Set default build type to "Release" in single-config generators
if(NOT CMAKE_CONFIGURATION_TYPES)
    if(NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE "Release" CACHE STRING
        "Choose the type of build, recommanded options are: Debug or Release" FORCE)
    endif()
    set(MBS_BUILD_TYPES "Debug" "Release" "MinSizeRel" "RelWithDebInfo")
    set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS ${MBS_BUILD_TYPES})
endif()

# Optionally load a file named ProjectsTags for overwriting the desired branch/tag/commit
# of the included subprojects of the superbuild. Useful for having superbuild "branches"
include(ProjectsTags OPTIONAL)

# Bootstrap YCM
set(YCM_FOLDER external)
set(YCM_COMPONENT core)
set(YCM_TAG ad8bb257df45ffe70f949c74815bf803d7445b24)
set(YCM_MINIMUM_VERSION 0.11.0)

include(YCMBootstrap)
include(FindOrBuildPackage)
include(YCMEPHelper)

# Required repositories
find_or_build_package(mbs_core)

# Optional repositories
if (${USES_EXTERNAL})
  find_or_build_package(FEX-minimize)
  find_or_build_package(FEX-function_handle)
  find_or_build_package(qpOASES)
endif()
if (${USES_MODELS})
  find_or_build_package(mbs_models)
endif()
if (${USES_APP})
  find_or_build_package(mbs_app)
endif()

# The following line is to properly configure the matlab and simulink software installed by the superbuild
configure_file(${CMAKE_CURRENT_SOURCE_DIR}/cmake/templates/startup_mbs.m.in ${CMAKE_BINARY_DIR}/startup_mbs.m)

# Other utilities
set_package_properties(Git PROPERTIES TYPE RUNTIME)
set_package_properties(Subversion PROPERTIES TYPE RUNTIME)
set_package_properties(PkgConfig PROPERTIES TYPE RUNTIME)
set_package_properties(Doxygen PROPERTIES TYPE RUNTIME)

feature_summary(WHAT ALL INCLUDE_QUIET_PACKAGES FATAL_ON_MISSING_REQUIRED_PACKAGES)

