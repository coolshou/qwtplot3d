# CMake Build script for qwtplot3d
#
# Frank T. Bergmann
#
cmake_minimum_required (VERSION 3.6)
project (qwtplot3d)

include(GNUInstallDirs)

include_directories("${PROJECT_SOURCE_DIR}")
include_directories("${PROJECT_SOURCE_DIR}/include")

set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "qwtplot3d library")
set(CPACK_PACKAGE_NAME "qwtplot3d")
set(CPACK_PACKAGE_VERSION_MAJOR "0")
set(CPACK_PACKAGE_VERSION_MINOR "3")
set(CPACK_PACKAGE_VERSION_PATCH "0")

set(CPACK_SOURCE_IGNORE_FILES "${CMAKE_CURRENT_BINARY_DIR};/.svn/;/.libs/;/.deps/;/.bzr/;.*.o$;.*.lo$;.*.la$;${CPACK_SOURCE_IGNORE_FILES};/.DS_Store;/.svnignore;blib;libsbml-dist")

if(UNIX)
    set(CPACK_INSTALL_PREFIX "${CMAKE_INSTALL_PREFIX}")
    set(CPACK_SET_DESTDIR "ON")
endif()

if(CMAKE_SIZEOF_VOID_P EQUAL 4)
    set(CPACK_DEBIAN_PACKAGE_ARCHITECTURE "i386")
    set(CPACK_RPM_PACKAGE_ARCHITECTURE "i386")
else()
    set(CPACK_DEBIAN_PACKAGE_ARCHITECTURE "amd64")
    set(CPACK_RPM_PACKAGE_ARCHITECTURE "x86_64")
endif()

include(CPack)

# are we using the intel compiler
set(USING_INTEL)
if (WIN32 AND CMAKE_C_COMPILER AND ${CMAKE_C_COMPILER} MATCHES ".*icl.*$")
  message(STATUS "Detected Intel Compiler")
  set(USING_INTEL TRUE)
endif ()

# compile as position independent code
if (UNIX OR CYGWIN)
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fPIC")
set(CMAKE_C_FLAGS "${CMAKE_CXX_FLAGS} -fPIC")
endif()


# if using msvc or intel windows compiler allow for the use of the static runtime
if(MSVC OR USING_INTEL)
    add_definitions(-D_CRT_SECURE_NO_WARNINGS -D_CRT_NONSTDC_NO_DEPRECATE -DQWT3D_DLL -DQWT3D_MAKEDLL)
    set(BUILD_DEFINITIONS "${BUILD_DEFINITIONS} -D_CRT_SECURE_NO_WARNINGS")
    option(WITH_STATIC_RUNTIME "Compile using the static MSVC Runtime." OFF)
    if(WITH_STATIC_RUNTIME)
        foreach(flag_var
            CMAKE_CXX_FLAGS CMAKE_CXX_FLAGS_DEBUG CMAKE_CXX_FLAGS_RELEASE
            CMAKE_CXX_FLAGS_MINSIZEREL CMAKE_CXX_FLAGS_RELWITHDEBINFO
            CMAKE_C_FLAGS CMAKE_C_FLAGS_DEBUG CMAKE_C_FLAGS_RELEASE
            CMAKE_C_FLAGS_MINSIZEREL CMAKE_C_FLAGS_RELWITHDEBINFO)

            if(${flag_var} MATCHES "/MD")
                string(REGEX REPLACE "/MD" "/MT" ${flag_var} "${${flag_var}}")
            endif(${flag_var} MATCHES "/MD")
        endforeach(flag_var)
    add_definitions( -D_MT)
    endif(WITH_STATIC_RUNTIME)
endif(MSVC OR USING_INTEL)



set(HEADERS

		include/qwt3d_color.h
		include/qwt3d_global.h
		include/qwt3d_types.h
		include/qwt3d_axis.h
		include/qwt3d_coordsys.h
		include/qwt3d_drawable.h
		include/qwt3d_helper.h
		include/qwt3d_label.h
		include/qwt3d_openglhelper.h
		include/qwt3d_colorlegend.h
		include/qwt3d_plot.h
		include/qwt3d_enrichment.h
		include/qwt3d_enrichment_std.h
		include/qwt3d_autoscaler.h
		include/qwt3d_autoptr.h
		include/qwt3d_io.h
		include/qwt3d_io_reader.h
		include/qwt3d_scale.h
		include/qwt3d_portability.h

		include/qwt3d_mapping.h
		include/qwt3d_gridmapping.h
		include/qwt3d_parametricsurface.h
		include/qwt3d_function.h

		include/qwt3d_surfaceplot.h
		include/qwt3d_volumeplot.h
		include/qwt3d_graphplot.h
		include/qwt3d_multiplot.h

		3rdparty/gl2ps/gl2ps.h
		include/qwt3d_io_gl2ps.h

)

set(MOC_HEADERS

		include/qwt3d_plot.h
		include/qwt3d_surfaceplot.h
		include/qwt3d_volumeplot.h
		include/qwt3d_graphplot.h
		include/qwt3d_multiplot.h

)

set(SOURCES

		src/qwt3d_axis.cpp
		src/qwt3d_color.cpp
		src/qwt3d_coordsys.cpp
		src/qwt3d_drawable.cpp
		src/qwt3d_mousekeyboard.cpp
		src/qwt3d_movements.cpp
		src/qwt3d_lighting.cpp
		src/qwt3d_colorlegend.cpp
		src/qwt3d_plot.cpp
		src/qwt3d_label.cpp
		src/qwt3d_types.cpp
		src/qwt3d_enrichment_std.cpp
		src/qwt3d_autoscaler.cpp
		src/qwt3d_io_reader.cpp
		src/qwt3d_io.cpp
		src/qwt3d_scale.cpp

		src/qwt3d_gridmapping.cpp
		src/qwt3d_parametricsurface.cpp
		src/qwt3d_function.cpp

		src/qwt3d_surfaceplot.cpp
		src/qwt3d_gridplot.cpp
		src/qwt3d_meshplot.cpp

		src/qwt3d_io_gl2ps.cpp
		3rdparty/gl2ps/gl2ps.c
)

option(WITH_ZLIB "Add support for compressed ps export." OFF)
if (WITH_ZLIB)
find_package(ZLIB)
if (ZLIB_FOUND)

add_definitions(-DGL2PS_HAVE_ZLIB)
include_directories( ${ZLIB_INCLUDE_DIRS})

endif(ZLIB_FOUND)
endif()

include(${CMAKE_CURRENT_SOURCE_DIR}/ChooseQtVersion.cmake)
QT_FIND_MODULES(Core Gui OpenGL Xml)

set( CMAKE_AUTOMOC ON )

add_library(qwtplot3d_static STATIC ${SOURCES} ${HEADERS})
set_target_properties(qwtplot3d_static PROPERTIES OUTPUT_NAME qwtplot3d)
QT_BIND_TO_TARGET(qwtplot3d_static)

add_library(qwtplot3d SHARED ${SOURCES} ${HEADERS})
set_property(TARGET qwtplot3d PROPERTY POSITION_INDEPENDENT_CODE 1)
QT_BIND_TO_TARGET(qwtplot3d)

find_package( OpenGL COMPONENTS OpenGL REQUIRED )
target_link_libraries( qwtplot3d_static  OpenGL::GLU )
target_link_libraries( qwtplot3d  OpenGL::GLU )
if (ZLIB_FOUND)
target_link_libraries(qwtplot3d_static ${ZLIB_LIBRARIES})
target_link_libraries(qwtplot3d ${ZLIB_LIBRARIES})
endif(ZLIB_FOUND)


install(FILES ${HEADERS} DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}")
INSTALL(TARGETS qwtplot3d_static qwtplot3d
        RUNTIME DESTINATION "${CMAKE_INSTALL_BINDIR}"
        LIBRARY DESTINATION "${CMAKE_INSTALL_LIBDIR}"
        ARCHIVE DESTINATION "${CMAKE_INSTALL_LIBDIR}")

option( BUILD_EXAMPLES "Build examples" OFF )
if ( BUILD_EXAMPLES )
    message( STATUS "Building examples" )
    add_subdirectory( examples )
endif()
