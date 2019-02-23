include(CheckCXXCompilerFlag)

if (NOT IS_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/openexr/OpenEXR")
  message(FATAL_ERROR "OpenEXR not found. The project has to be cloned using --recursive (or call git submodule update --init --recursive) ")
endif()

if (CMAKE_CXX_COMPILER_ID MATCHES "Clang" OR CMAKE_CXX_COMPILER_ID MATCHES "GNU")
  CHECK_CXX_COMPILER_FLAG("-std=c++14" HAS_CPP14_FLAG)
  CHECK_CXX_COMPILER_FLAG("-std=c++11" HAS_CPP11_FLAG)

  if (HAS_CPP14_FLAG)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++14")
  elseif (HAS_CPP11_FLAG)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
  else()
    message(FATAL_ERROR "C++11 support is needed (for pybind)!")
  endif()
endif()

if (MSVC)
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /D_CRT_SECURE_NO_WARNINGS")
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /MP")
endif()

# ZLib
find_package(ZLIB)
if(NOT ZLIB_FOUND)
  set(ZLIB_BUILD_STATIC_LIBS ON CACHE BOOL " " FORCE)
  set(ZLIB_BUILD_SHARED_LIBS OFF CACHE BOOL " " FORCE)
  add_subdirectory(zlib)

  set(ZLIB_INCLUDE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/zlib")
  set(ZLIB_LIBRARY zlibstatic)
  set_property(TARGET zlibstatic PROPERTY FOLDER "dependencies")
  include_directories(${ZLIB_INCLUDE_DIR}  "${CMAKE_CURRENT_BINARY_DIR}/zlib")
endif()

# OpenEXR
SET(ILMBASE_NAMESPACE_VERSIONING OFF CACHE BOOL " " FORCE)
SET(OPENEXR_NAMESPACE_VERSIONING OFF CACHE BOOL " " FORCE)
SET(OPENEXR_BUILD_SHARED_LIBS    OFF CACHE BOOL " " FORCE)
SET(ILMBASE_BUILD_SHARED_LIBS    OFF CACHE BOOL " " FORCE)

add_subdirectory(openexr)
set_property(TARGET IexMath eLut toFloat b44ExpLogTable dwaLookups IlmThread Half Iex Imath IlmImf PROPERTY FOLDER "dependencies")
set(Openexr_ILMIMF_LIBRARY IlmImf)
set(Openexr_ILMIMF_LIBRARY ${Openexr_ILMIMF_LIBRARY} PARENT_SCOPE)
set(Ilmbase_ILMTHREAD_LIBRARY IlmThread)
set(Ilmbase_ILMTHREAD_LIBRARY ${Ilmbase_ILMTHREAD_LIBRARY} PARENT_SCOPE)
set(Ilmbase_IEX_LIBRARY Iex)
set(Ilmbase_IEX_LIBRARY ${Ilmbase_IEX_LIBRARY} PARENT_SCOPE)
set(Ilmbase_HALF_LIBRARY Half)
set(Ilmbase_HALF_LIBRARY ${Ilmbase_HALF_LIBRARY} PARENT_SCOPE)


# CopyIlmBaseLibs
# Copy the EXR head files to a common folder (matching folder structure expected by OpenVDB)
file(GLOB OPENEXR_HEADER_FILES
                  "${CMAKE_CURRENT_SOURCE_DIR}/openexr/IlmBase/Imath/*.h"
                  "${CMAKE_CURRENT_SOURCE_DIR}/openexr/IlmBase/Iex/*.h"
                  "${CMAKE_CURRENT_SOURCE_DIR}/openexr/IlmBase/Half/*.h"
                  "${CMAKE_CURRENT_SOURCE_DIR}/openexr/OpenEXR/IlmImf/*.h"
                  "${CMAKE_CURRENT_BINARY_DIR}/openexr/OpenEXR/config/*.h"
                  "${CMAKE_CURRENT_BINARY_DIR}/openexr/IlmBase/config/*.h")

file(COPY ${OPENEXR_HEADER_FILES} DESTINATION ${CMAKE_CURRENT_SOURCE_DIR}/openexrheaders/OpenEXR)

set(OPENEXR_INCLUDE_DIRS
  ${CMAKE_CURRENT_SOURCE_DIR}/openexrheaders
  ${CMAKE_CURRENT_SOURCE_DIR}/openexr/IlmBase/Imath
  ${CMAKE_CURRENT_SOURCE_DIR}/openexr/IlmBase/Iex
  ${CMAKE_CURRENT_SOURCE_DIR}/openexr/IlmBase/Half
  ${CMAKE_CURRENT_SOURCE_DIR}/openexr/OpenEXR/IlmImf
  ${CMAKE_CURRENT_BINARY_DIR}/openexr/OpenEXR/config
  ${CMAKE_CURRENT_BINARY_DIR}/openexr/IlmBase/config)
set(OPENEXR_INCLUDE_DIRS ${OPENEXR_INCLUDE_DIRS} PARENT_SCOPE)


set(ILMBASE_INCLUDE_DIRS
  ${CMAKE_CURRENT_SOURCE_DIR}/openexrheaders
  ${CMAKE_CURRENT_SOURCE_DIR}/openexr/IlmBase/Imath
  ${CMAKE_CURRENT_SOURCE_DIR}/openexr/IlmBase/Iex
  ${CMAKE_CURRENT_SOURCE_DIR}/openexr/IlmBase/Half
  ${CMAKE_CURRENT_SOURCE_DIR}/openexr/OpenEXR/IlmImf
  ${CMAKE_CURRENT_BINARY_DIR}/openexr/OpenEXR/config
  ${CMAKE_CURRENT_BINARY_DIR}/openexr/IlmBase/config)
set(ILMBASE_INCLUDE_DIRS ${ILMBASE_INCLUDE_DIRS} PARENT_SCOPE)

# Use install rpath from this point onwards
set(CMAKE_BUILD_WITH_INSTALL_RPATH TRUE)

# Build Thread Building Blocks (main shared libraries only)
set(TBB_BUILD_SHARED          ON  CACHE BOOL " " FORCE)
set(TBB_BUILD_STATIC          ON CACHE BOOL " " FORCE)
set(TBB_BUILD_TESTS           OFF CACHE BOOL " " FORCE)
set(TBB_BUILD_TBBMALLOC       OFF CACHE BOOL " " FORCE)
set(TBB_BUILD_TBBMALLOC_PROXY OFF CACHE BOOL " " FORCE)
add_subdirectory(tbb)
set_property(TARGET tbb tbb_static tbb_def_files PROPERTY FOLDER "dependencies")
set(TBB_INCLUDE_DIRS ${CMAKE_CURRENT_SOURCE_DIR}/tbb/include)
set(TBB_INCLUDE_DIRS ${TBB_INCLUDE_DIRS} PARENT_SCOPE)
set(Tbb_TBB_LIBRARY tbb_static)
set(Tbb_TBB_LIBRARY ${Tbb_TBB_LIBRARY} PARENT_SCOPE)

# Build GLFW
set(GLFW_BUILD_EXAMPLES OFF CACHE BOOL " " FORCE)
set(GLFW_BUILD_TESTS OFF CACHE BOOL " " FORCE)
set(GLFW_BUILD_DOCS OFF CACHE BOOL " " FORCE)
set(GLFW_BUILD_INSTALL OFF CACHE BOOL " " FORCE)
set(GLFW_INSTALL OFF CACHE BOOL " " FORCE)
set(GLFW_USE_CHDIR OFF CACHE BOOL " " FORCE)
set(BUILD_SHARED_LIBS ${NANOGUI_BUILD_SHARED} CACHE BOOL " " FORCE)

if (CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  # Quench annoying deprecation warnings when compiling GLFW on OSX
  set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wno-deprecated-declarations")
endif()

add_subdirectory(glfw)
set_property(TARGET glfw PROPERTY FOLDER "dependencies")

set(GLFW3_INCLUDE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/glfw/include)
set(GLFW3_INCLUDE_DIR ${GLFW3_INCLUDE_DIR} PARENT_SCOPE)

set(GLFW3_glfw_LIBRARY glfw)
set(GLFW3_glfw_LIBRARY ${GLFW3_glfw_LIBRARY} PARENT_SCOPE)


# Build Blosc
set(DDEACTIVATE_AVX2          ON  CACHE BOOL " " FORCE)
set(PREFER_EXTERNAL_COMPLIBS  OFF CACHE BOOL " " FORCE)
set(BUILD_TESTS               OFF CACHE BOOL " " FORCE)
set(BUILD_BENCHMARKS          OFF CACHE BOOL " " FORCE)

add_subdirectory(c-blosc)
set_property(TARGET blosc_static blosc_shared PROPERTY FOLDER "dependencies")
set(BLOSC_INCLUDE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/c-blosc/blosc)
set(BLOSC_INCLUDE_DIR ${BLOSC_INCLUDE_DIR} PARENT_SCOPE)
set(BLOSC_LIBRARY blosc_static PARENT_SCOPE)
set(BLOSC_blosc_LIBRARY blosc_static)
set(BLOSC_blosc_LIBRARY ${BLOSC_blosc_LIBRARY} PARENT_SCOPE)


# add_subdirectory(openvdb)
# set_property(TARGET openvdb_static openvdb_shared PROPERTY FOLDER "dependencies")
# set(OPENVDB_INCLUDE_DIRS ${CMAKE_CURRENT_SOURCE_DIR}/openvdb/include)
# set(OPENVDB_INCLUDE_DIRS ${OPENVDB_INCLUDE_DIRS} PARENT_SCOPE)
