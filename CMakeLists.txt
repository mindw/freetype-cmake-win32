# CMakeLists.txt
#
# Copyright 2013, 2014 by
# David Turner, Robert Wilhelm, and Werner Lemberg.
#
# Written by John Cary <cary@txcorp.com>
#
# This file is part of the FreeType project, and may only be used, modified,
# and distributed under the terms of the FreeType project license,
# LICENSE.TXT.  By continuing to use, modify, or distribute this file you
# indicate that you have read the license and understand and accept it
# fully.
#
#
# Say
#
#   cmake CMakeLists.txt
#
# to create a Makefile that builds a static version of the library.
#
# For a dynamic library, use
#
#   cmake CMakeLists.txt -DBUILD_SHARED_LIBS:BOOL=true
#
# For a framework on OS X, use
#
#   cmake CMakeLists.txt -DBUILD_FRAMEWORK:BOOL=true -G Xcode
#
# instead.  Please refer to the cmake manual for further options, in
# particular, how to modify compilation and linking parameters.
#
# Some notes.
#
# . `cmake' will overwrite FreeType's original (top-level) `Makefile' file.
#
# . You can use `cmake' directly on a freshly cloned FreeType git
#   repository.
#
# . `CMakeLists.txt'  is provided as-is since it is not used by the
#   developer team.


cmake_minimum_required(VERSION 2.6)

project(freetype)

set(local_lib_name freetype)

if (BUILD_FRAMEWORK)
  if (NOT "${CMAKE_GENERATOR}" STREQUAL "Xcode")
    message(FATAL_ERROR "You should use Xcode generator with BUILD_FRAMEWORK enabled")
  endif ()
  set(CMAKE_OSX_ARCHITECTURES "$(ARCHS_STANDARD_32_64_BIT)")
endif ()

set(BUILD_SHARED_LIBS ON)

find_package(ZLIB)
find_package(BZip2)
find_package(PNG)

set(VERSION_MAJOR "2")
set(VERSION_MINOR "5")
set(VERSION_PATCH "3")
set(PROJECT_VERSION ${VERSION_MAJOR}.${VERSION_MINOR}.${VERSION_PATCH})

set(SHARED_SOVERSION "6.11.3")
set(SHARED_VERSION "6")

# Compiler definitions for building the library
add_definitions(-DFT2_BUILD_LIBRARY)

if (MSVC)
    add_definitions(-D_CRT_SECURE_NO_WARNINGS)
endif()

# Specify library include directories
include_directories("${PROJECT_SOURCE_DIR}/include")

# Create the configuration file
#message(STATUS "Creating directory, ${PROJECT_BINARY_DIR}/include/freetype2.")
#file(MAKE_DIRECTORY ${PROJECT_BINARY_DIR}/include/freetype2)

# For the auto-generated ftconfig.h file
#include_directories(BEFORE "${PROJECT_BINARY_DIR}/include/freetype2")
# message(STATUS "Creating ${PROJECT_BINARY_DIR}/include/freetype2/ftconfig.h.")
# execute_process(
  # COMMAND sed -e "s/FT_CONFIG_OPTIONS_H/<ftoption.h>/" -e "s/FT_CONFIG_STANDARD_LIBRARY_H/<ftstdlib.h>/" -e "s?/undef ?#undef ?"
  # INPUT_FILE ${PROJECT_SOURCE_DIR}/builds/unix/ftconfig.in
  # OUTPUT_FILE ${PROJECT_BINARY_DIR}/include/freetype2/ftconfig.h
# )

file(GLOB PUBLIC_HEADERS "include/*.h")
file(GLOB PUBLIC_CONFIG_HEADERS "include/config/*.h")
file(GLOB PRIVATE_HEADERS "include/internal/*.h")

set(BASE_SRCS
  src/autofit/autofit.c
  src/base/ftadvanc.c
  src/base/ftbbox.c
  src/base/ftbdf.c
  src/base/ftbitmap.c
  src/base/ftcalc.c
  src/base/ftcid.c
  src/base/ftdbgmem.c
  src/base/ftdebug.c
  src/base/ftfstype.c
  src/base/ftgasp.c
  src/base/ftgloadr.c
  src/base/ftglyph.c
  src/base/ftgxval.c
  src/base/ftinit.c
  src/base/ftlcdfil.c
  src/base/ftmm.c
  src/base/ftobjs.c
  src/base/ftotval.c
  src/base/ftoutln.c
  src/base/ftpatent.c
  src/base/ftpfr.c
  src/base/ftrfork.c
  src/base/ftsnames.c
  src/base/ftstream.c
  src/base/ftstroke.c
  src/base/ftsynth.c
  src/base/ftsystem.c
  src/base/fttrigon.c
  src/base/fttype1.c
  src/base/ftutil.c
  src/base/ftwinfnt.c
  src/base/ftxf86.c
  src/bdf/bdf.c
  src/bzip2/ftbzip2.c
  src/cache/ftcache.c
  src/cff/cff.c
  src/cid/type1cid.c  
  src/gxvalid/gxvalid.c
  src/gzip/ftgzip.c
  src/lzw/ftlzw.c
  src/otvalid/otvalid.c
  src/pcf/pcf.c
  src/pfr/pfr.c
  src/psaux/psaux.c
  src/pshinter/pshinter.c
  src/psnames/psmodule.c
  src/raster/raster.c
  src/sfnt/sfnt.c
  src/smooth/smooth.c
  src/truetype/truetype.c
  src/type1/type1.c
  src/type42/type42.c
  src/winfonts/winfnt.c
)

include_directories("src/truetype")
include_directories("src/sfnt")
include_directories("src/autofit")
include_directories("src/smooth")
include_directories("src/raster")
include_directories("src/psaux")
include_directories("src/psnames")

if (BUILD_FRAMEWORK)
  set(BASE_SRCS
    ${BASE_SRCS}
    builds/mac/freetype-Info.plist
  )
endif()

if (MSVC)
    if (NOT DEFINED ARCH)
        set (ARCH ${MSVC_C_ARCHITECTURE_ID})
    endif()

    # use boost convention - you cant' get the VC compiler without VS
    if (NOT DEFINED MSVCVER)
        math(EXPR MSVCVER "(${MSVC_VERSION} / 10) - 60")
    endif()

    # add the runtime version and target architecture into the DLL name per MS
    # guidelines
    set(MSVC_SHARED_SUFFIX_BASE "-${SHARED_VERSION}-vc${MSVCVER}-mt-${ARCH}")
    set(CMAKE_SHARED_LIBRARY_SUFFIX "${MSVC_SHARED_SUFFIX_BASE}.dll")
    
    if (BUILD_SHARED_LIBS)

        #set(runtime_name freetype-vc${MSVCVER}-mt-${ARCH})
        
        set(VER_PRODUCTNAME_STR "The FreeType Project")
        set(VER_ORIGINALFILENAME_STR "${local_lib_name}${PROJECT_VERSION_MAJOR}${CMAKE_SHARED_LIBRARY_SUFFIX}")
        set(VER_INTERNALNAME_STR freetype)
        set(VER_FILEDESCRIPTION_STR "FreeType Dynamic Link Library")
        set(VER_LEGALCOPYRIGHT_STR "Copyright 1996-2002, 2006 by David Turner, Robert Wilhelm, and Werner Lemberg.")
        set(VER_STR "${PROJECT_VERSION}")
        set(VER_NUM "${VERSION_MAJOR},${VERSION_MINOR},${VERSION_PATCH},0")
        configure_file(
              ${CMAKE_CURRENT_SOURCE_DIR}/builds/windows/version.rc.in
              ${CMAKE_CURRENT_BINARY_DIR}/version.rc
              @ONLY
          )         
        set(BASE_SRCS ${BASE_SRCS} ${CMAKE_CURRENT_BINARY_DIR}/version.rc)
        
    endif()    
endif()

add_library(freetype
  ${PUBLIC_HEADERS}
  ${PUBLIC_CONFIG_HEADERS}
  ${PRIVATE_HEADERS}
  ${BASE_SRCS}
)

if (BUILD_FRAMEWORK)
  set_property(SOURCE ${PUBLIC_CONFIG_HEADERS}
    PROPERTY MACOSX_PACKAGE_LOCATION Headers/config
  )
  set_target_properties(freetype PROPERTIES
    FRAMEWORK TRUE
    MACOSX_FRAMEWORK_INFO_PLIST builds/mac/freetype-Info.plist
    PUBLIC_HEADER "${PUBLIC_HEADERS}"
    XCODE_ATTRIBUTE_INSTALL_PATH "@rpath"
  )
endif ()

if (BZIP2_FOUND)
    target_link_libraries(freetype ${BZIP2_LIBRARIES})
    target_include_directories(freetype PRIVATE ${BZIP2_INCLUDE_DIR})
endif()

if (PNG_FOUND)
    target_link_libraries(freetype ${PNG_LIBRARIES})
    target_include_directories(freetype PRIVATE ${PNG_INCLUDE_DIR})
endif()

if (ZLIB_FOUND)
    target_link_libraries(freetype ${ZLIB_LIBRARIES})
    target_include_directories(freetype PRIVATE ${ZLIB_INCLUDE_DIRS})
endif()

if (WIN32)
    set_target_properties(${local_lib_name} PROPERTIES PDB_NAME "${local_lib_name}${MSVC_SHARED_SUFFIX_BASE}")
endif ()

set_target_properties(freetype PROPERTIES 
    VERSION ${SHARED_VERSION}
    SOVERSION ${SHARED_SOVERSION}
    )

# Installations
# Note the trailing slash in the argument to the `DIRECTORY' directive
install(DIRECTORY ${PROJECT_SOURCE_DIR}/include/
  DESTINATION include/freetype2
  PATTERN "internal" EXCLUDE
)
install(TARGETS freetype
  RUNTIME DESTINATION bin
  LIBRARY DESTINATION lib
  ARCHIVE DESTINATION lib
  FRAMEWORK DESTINATION Library/Frameworks
)

# Packaging
# CPack version numbers for release tarball name.
set(CPACK_PACKAGE_VERSION_MAJOR ${VERSION_MAJOR})
set(CPACK_PACKAGE_VERSION_MINOR ${VERSION_MINOR})
set(CPACK_PACKAGE_VERSION_PATCH ${VERSION_PATCH}})
if (NOT DEFINED CPACK_PACKAGE_DESCRIPTION_SUMMARY)
  set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "${CMAKE_PROJECT_NAME}")
endif ()
if (NOT DEFINED CPACK_SOURCE_PACKAGE_FILE_NAME)
  set(CPACK_SOURCE_PACKAGE_FILE_NAME
    "${CMAKE_PROJECT_NAME}-${PROJECT_VERSION}-r${PROJECT_REV}"
    CACHE INTERNAL "tarball basename"
  )
endif ()
set(CPACK_SOURCE_GENERATOR TGZ)
set(CPACK_SOURCE_IGNORE_FILES
  "/CVS/;/.svn/;.swp$;.#;/#;/build/;/serial/;/ser/;/parallel/;/par/;~;/preconfig.out;/autom4te.cache/;/.config")
set(CPACK_GENERATOR TGZ)
include(CPack)

# add make dist target
add_custom_target(dist COMMAND ${CMAKE_MAKE_PROGRAM} package_source)

# eof
