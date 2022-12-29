project(pixman)
set(PACKAGE pixman)
cmake_minimum_required(VERSION 3.12)

# detect version
file (READ configure.ac configure_ac)
string (REGEX REPLACE ".*pixman_major], ([0-9]+).*" "\\1" PIXMAN_VERSION_MAJOR ${configure_ac})
string (REGEX REPLACE ".*pixman_minor], ([0-9]+).*" "\\1" PIXMAN_VERSION_MINOR ${configure_ac})
string (REGEX REPLACE ".*pixman_micro], ([0-9]+).*" "\\1" PIXMAN_VERSION_MICRO ${configure_ac})
set(PIXMAN_VERSION "${PIXMAN_VERSION_MAJOR}.${PIXMAN_VERSION_MINOR}.${PIXMAN_VERSION_MICRO}")

option(BUILD_SHARED "build shared library" ON)
option(BUILD_STATIC "build static library" ON)
option(BUILD_TESTS "build tests" ON)

set(CMAKE_DEBUG_POSTFIX "d")

#dependencies
find_package(Threads)
if(BUILD_TESTS)
    find_package(PNG)
    find_package(OpenMP)
endif()

#arch optimizations
include(${CMAKE_SOURCE_DIR}/cmake/arch_detect.cmake)
include(${CMAKE_SOURCE_DIR}/cmake/arch_configure.cmake)

#config
include(${CMAKE_SOURCE_DIR}/cmake/config_configure.cmake)

include_directories(
    ${CMAKE_BINARY_DIR}/pixman
    ${CMAKE_SOURCE_DIR}/pixman
)

if(MSVC)
	set(CMAKE_C_FLAGS "/wd4244 /wd4146 -D_CRT_SECURE_NO_WARNINGS ${CMAKE_C_FLAGS}")
endif()

if(BUILD_SHARED OR BUILD_STATIC)
    add_subdirectory(pixman)
endif()

if(BUILD_TESTS)
    add_subdirectory(test)
endif(BUILD_TESTS)

include(${CMAKE_SOURCE_DIR}/cmake/cmake_package.cmake)
set(prefix      "\${pcfiledir}/../..")
set(exec_prefix "\${prefix}")
set(libdir      "\${exec_prefix}/lib")
set(includedir  "\${prefix}/include")
set(PACKAGE_VERSION ${PIXMAN_VERSION})
set(ENABLE_SHARED_TRUE "")
set(ENABLE_SHARED_FALSE "#")
set(FRIBIDI_CPPFLAGS "-DFRIBIDI_BUILD")
configure_file(${CMAKE_SOURCE_DIR}/pixman-1.pc.in ${CMAKE_CURRENT_BINARY_DIR}/pixman-1.pc @ONLY)
install(FILES ${CMAKE_CURRENT_BINARY_DIR}/pixman-1.pc DESTINATION lib/pkgconfig)
