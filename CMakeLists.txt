cmake_minimum_required (VERSION 2.8)
project(pkgconfig)
set(pkgconfig_VERSION 0.28)
enable_language(C)

include(CheckIncludeFiles)
CHECK_INCLUDE_FILES("dirent.h;unistd.h;sys/wait.h;malloc.h" HEADER_PREREQS)

if(NOT ${HEADER_PREREQS})
    message(SEND_ERROR "Missing header files: dirent.h unistd.h sys/wait.h malloc.h")
endif()

include(GNUInstallDirs)

macro(define_var VAR VALUE)
    set(${VAR} "${VALUE}" CACHE STRING "${VAR}")
    add_definitions(-DPKG_CONFIG_${VAR}="${${VAR}}")
endmacro()

define_var(PC_PATH "${CMAKE_INSTALL_LIBDIR}/pkgconfig:${CMAKE_INSTALL_DATADIR}/pkgconfig:/usr/lib/pkgconfig:/usr/share/pkgconfig")
define_var(SYSTEM_INCLUDE_PATH "/usr/include")
define_var(SYSTEM_LIBRARY_PATH "/usr/lib:/lib")

add_definitions(-DVERSION="${pkgconfig_VERSION}")

# TODO: Read from deplibs_check_method from `libtool --config`
set(ENABLE_INDIRECT_DEPS OFF CACHE BOOL "")
if(ENABLE_INDIRECT_DEPS)
    add_definitions(-DENABLE_INDIRECT_DEPS=1)
else()
    add_definitions(-DENABLE_INDIRECT_DEPS=0)
endif()

add_executable(pkg-config
    glib-snippet.h
    glib-snippet.c
    pkg.h
    pkg.c
    parse.h
    parse.c
    main.c
)

install(TARGETS pkg-config DESTINATION bin)
