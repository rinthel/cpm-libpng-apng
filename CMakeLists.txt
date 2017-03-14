cmake_minimum_required(VERSION 2.8 FATAL_ERROR)

#-----------------------------------------------------------------------
# CPM configuration
#-----------------------------------------------------------------------
set(CPM_MODULE_NAME libpng)
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

# # All externals *must* define this.
# CPM_ForceOnlyOneModuleVersion()

if (NOT EMSCRIPTEN)
  find_package(ZLIB)
  if (NOT ZLIB_FOUND)
    message(STATUS "ZLib not found.")
    CPM_AddModule("zlib"
        GIT_REPOSITORY "https://github.com/rinthel/cpm-zlib"
        EXPORT_MODULE TRUE)
  endif()
endif()

CPM_InitModule(${CPM_MODULE_NAME})

#------------------------------------------------------------------------------
# LIBPNG + APNG Support
#------------------------------------------------------------------------------
set(LIBPNG_DIR ${CMAKE_CURRENT_SOURCE_DIR}/3rdParty/libpng)

CPM_EnsureRepoIsCurrent(
  TARGET_DIR ${LIBPNG_DIR}
  GIT_REPOSITORY "https://github.com/rinthel/libpng-apng.git"
  USE_CACHING TRUE
  )

project(png)

file(COPY ${LIBPNG_DIR}/scripts/pnglibconf.h.prebuilt DESTINATION ${LIBPNG_DIR})
file(RENAME ${LIBPNG_DIR}/pnglibconf.h.prebuilt ${LIBPNG_DIR}/pnglibconf.h)

set(SOURCE_FILES
  ${LIBPNG_DIR}/png.c
  ${LIBPNG_DIR}/png.h
  ${LIBPNG_DIR}/pngconf.h
  ${LIBPNG_DIR}/pngdebug.h
  ${LIBPNG_DIR}/pngerror.c
  ${LIBPNG_DIR}/pngget.c
  ${LIBPNG_DIR}/pnginfo.h
  ${LIBPNG_DIR}/pngmem.c
  ${LIBPNG_DIR}/pngpread.c
  ${LIBPNG_DIR}/pngpriv.h
  ${LIBPNG_DIR}/pngread.c
  ${LIBPNG_DIR}/pngrio.c
  ${LIBPNG_DIR}/pngrtran.c
  ${LIBPNG_DIR}/pngrutil.c
  ${LIBPNG_DIR}/pngset.c
  ${LIBPNG_DIR}/pngstruct.h
  ${LIBPNG_DIR}/pngtrans.c
  ${LIBPNG_DIR}/pngwio.c
  ${LIBPNG_DIR}/pngwrite.c
  ${LIBPNG_DIR}/pngwtran.c
  ${LIBPNG_DIR}/pngwutil.c
  ${LIBPNG_DIR}/arm/filter_neon_intrinsics.c
  ${LIBPNG_DIR}/arm/arm_init.c
  )

add_library(png ${SOURCE_FILES})
if (NOT EMSCRIPTEN AND CPM_LIBRARIES)
  target_link_libraries(png ${CPM_LIBRARIES})
endif()
if (EMSCRIPTEN)
  set_source_files_properties(${SOURCE_FILES} PROPERTIES COMPILE_FLAGS "-s USE_ZLIB=1")
endif()

CPM_ExportAdditionalIncludeDir("${LIBPNG_DIR}")
CPM_ExportAdditionalLibraryTarget(png)
