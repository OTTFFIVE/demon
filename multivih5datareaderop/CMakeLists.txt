find_package( HDF5 REQUIRED )
find_package( OpenCV QUIET COMPONENTS core highgui imgproc)
find_package( OpenMP REQUIRED )
set(Python_ADDITIONAL_VERSIONS 3)
find_package( PythonInterp REQUIRED )

include(ExternalProject)

#
# Download some not so common dependencies
#

# webp
ExternalProject_Add( 
        webp
        PREFIX "${CMAKE_BINARY_DIR}/webp"
        URL "http://downloads.webmproject.org/releases/webp/libwebp-0.5.1.tar.gz"
        # do not update
        UPDATE_COMMAND ""
        CONFIGURE_COMMAND CFLAGS=-fPIC ../webp/configure --disable-gl --disable-png --disable-jpeg  --disable-tiff --disable-gif --disable-wic
        BUILD_COMMAND "make"
        # do not install
        INSTALL_COMMAND ""
)
ExternalProject_Get_Property( webp SOURCE_DIR BINARY_DIR )
set( webp_INCLUDE_DIR "${SOURCE_DIR}/src" )
set( webp_LIBRARY "${BINARY_DIR}/src/.libs/libwebp.a" )


# lz4
ExternalProject_Add( 
        lz4
        PREFIX "${CMAKE_BINARY_DIR}/lz4"
	URL "https://github.com/lz4/lz4/archive/v1.7.5.tar.gz"
        # do not update
        UPDATE_COMMAND ""
        SOURCE_SUBDIR "contrib/cmake_unofficial"
        CMAKE_CACHE_ARGS "-DBUILD_STATIC_LIBS:string=on"
        # do not install
        INSTALL_COMMAND ""
)
ExternalProject_Get_Property( lz4 SOURCE_DIR BINARY_DIR )
set( lz4_INCLUDE_DIR "${SOURCE_DIR}/lib" )
set( lz4_STATIC_LIB "${BINARY_DIR}/liblz4.a" )


# json for modern c++
ExternalProject_Add( 
        json_for_modern_cpp
        PREFIX "${CMAKE_BINARY_DIR}/json"
	URL "https://github.com/nlohmann/json/releases/download/v2.1.1/json.hpp"
	DOWNLOAD_NO_EXTRACT 1
        # do not update, configure, build
        UPDATE_COMMAND ""
	CONFIGURE_COMMAND ""
	BUILD_COMMAND ""
        # do not install
        INSTALL_COMMAND ""
)
ExternalProject_Get_Property( json_for_modern_cpp SOURCE_DIR )
set( json_for_modern_cpp_INCLUDE_DIR "${SOURCE_DIR}/.." )


# half
ExternalProject_Add( 
        half
        PREFIX "${CMAKE_BINARY_DIR}/half"
	URL "https://sourceforge.net/projects/half/files/latest/download?source=files"
        # do not update, configure, build
        UPDATE_COMMAND ""
	CONFIGURE_COMMAND ""
	BUILD_COMMAND ""
        # do not install
        INSTALL_COMMAND ""
)
ExternalProject_Get_Property( half SOURCE_DIR )
set( half_INCLUDE_DIR "${SOURCE_DIR}/include" )


# retrieve tensorflow include dir
execute_process( COMMAND ${PYTHON_EXECUTABLE} "-c" "from __future__ import print_function; import tensorflow as tf; print(tf.sysconfig.get_include(), end='')" 
        OUTPUT_VARIABLE TENSORFLOW_INCLUDE_DIR )
message( STATUS "tensorflow include dir: ${TENSORFLOW_INCLUDE_DIR}" )

file( GLOB multivih5datareaderop_SOURCES *.cpp *.cc )


add_library( multivih5datareaderop SHARED ${multivih5datareaderop_SOURCES} )
# do not add 'lib' prefix
set_target_properties( multivih5datareaderop PROPERTIES PREFIX "" )
set_target_properties( multivih5datareaderop PROPERTIES DEBUG_POSTFIX "_debug" )
set_target_properties( multivih5datareaderop PROPERTIES COMPILE_FLAGS "-std=c++11 ${OpenMP_CXX_FLAGS} -Wabi-tag" )
set_target_properties( multivih5datareaderop PROPERTIES LINK_FLAGS "${OpenMP_CXX_FLAGS}" )
# use old ABI with gcc 5.x
set_target_properties( multivih5datareaderop PROPERTIES COMPILE_DEFINITIONS "_GLIBCXX_USE_CXX11_ABI=0" )
add_dependencies( multivih5datareaderop lz4 webp json_for_modern_cpp half)

target_include_directories( multivih5datareaderop PRIVATE
        ${TENSORFLOW_INCLUDE_DIR}
        ${lz4_INCLUDE_DIR}
        ${webp_INCLUDE_DIR}
        ${HDF5_INCLUDE_DIRS}        
        ${OpenCV_INCLUDE_DIRS}
	${json_for_modern_cpp_INCLUDE_DIR}
	${half_INCLUDE_DIR}
)

target_link_libraries( multivih5datareaderop
        ${lz4_STATIC_LIB}
        ${webp_LIBRARY}
        ${HDF5_LIBRARIES}        
        ${OpenCV_LIBS}
)
