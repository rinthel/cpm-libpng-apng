cmake_minimum_required(VERSION 2.8 FATAL_ERROR)

#-----------------------------------------------------------------------
# CPM configuration
#-----------------------------------------------------------------------
set(CPM_MODULE_NAME png)
set(CPM_LIB_TARGET_NAME ${CPM_MODULE_NAME})

if ((DEFINED CPM_DIR) AND (DEFINED CPM_UNIQUE_ID) AND (DEFINED CPM_TARGET_NAME))
  set(CPM_LIB_TARGET_NAME ${CPM_TARGET_NAME})
  set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${CPM_DIR})
  include(CPM)
else()
  set(CPM_DIR "${CMAKE_CURRENT_BINARY_DIR}/cpm-packages" CACHE TYPE STRING)
  find_package(Git)
  if(NOT GIT_FOUND)
    message(FATAL_ERROR "CPM requires Git.")
  endif()
  if (NOT EXISTS ${CPM_DIR}/CPM.cmake)
    execute_process(
      COMMAND "${GIT_EXECUTABLE}" clone https://github.com/iauns/cpm ${CPM_DIR}
      RESULT_VARIABLE error_code
      OUTPUT_VARIABLE head_sha)
    if(error_code)
      message(FATAL_ERROR "CPM failed to get the hash for HEAD")
    endif()
  endif()
  include(${CPM_DIR}/CPM.cmake)
endif()

# All externals *must* define this.
CPM_ForceOnlyOneModuleVersion()

CPM_InitModule(${CPM_MODULE_NAME})

#------------------------------------------------------------------------------
# LIBPNG + APNG Support
#------------------------------------------------------------------------------
set(PNG_SHARED OFF CACHE BOOL "" FORCE)
set(PNG_TESTS OFF CACHE BOOL "" FORCE)
set(SKIP_INSTALL_ALL ON CACHE BOOL "" FORCE)

set(LIBPNG_DIR ${CMAKE_CURRENT_SOURCE_DIR}/3rdParty/libpng)
set(LIBPNG_BIN_DIR ${CMAKE_CURRENT_BINARY_DIR}/3rdParty/libpng)
CPM_EnsureRepoIsCurrent(
  TARGET_DIR ${LIBPNG_DIR}
  GIT_REPOSITORY "https://github.com/rinthel/libpng-apng.git"
  USE_CACHING TRUE
  )

file(COPY ${LIBPNG_DIR}/scripts/pnglibconf.dfa DESTINATION ${LIBPNG_DIR}/pngusr.dfa)

if (POLICY CMP0040)
  cmake_policy(SET CMP0040 NEW)
endif(POLICY CMP0040)

add_subdirectory(${LIBPNG_DIR})

# add_custom_command(TARGET ${LIBPNG_DIR} POST_BUILD
#   COMMAND ${CMAKE_COMMAND} -E copy ${LIBPNG_BIN_DIR}/libpng16.a ${CMAKE_CURRENT_BINARY_DIR}/libpng.a)

CPM_ExportAdditionalIncludeDir("${LIBPNG_DIR}")
CPM_ExportAdditionalLibraryTarget(libpng z bz)

get_cmake_property(_variableNames VARIABLES)
foreach (_variableName ${_variableNames})
  message(STATUS "${_variableName}=${${_variableName}}")
endforeach()

