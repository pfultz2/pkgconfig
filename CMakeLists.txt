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

macro(define_gnu_var VAR)
    string(TOLOWER ${VAR} GNU_VAR)
    set(${GNU_VAR} ${CMAKE_INSTALL_${VAR}})
endmacro()

define_gnu_var(prefix)
define_gnu_var(exec_prefix)
define_gnu_var(includedir)
define_gnu_var(libdir)
define_gnu_var(datarootdir)
define_gnu_var(datadir)

macro(define_var VAR VALUE)
    set(${VAR} "${VALUE}" CACHE STRING "${VAR}")
    string(TOLOWER ${VAR} GNU_VAR)
    set(${GNU_VAR} "${VALUE}")
    add_definitions(-DPKG_CONFIG_${VAR}="${${VAR}}")
endmacro()

define_var(PC_PATH "${CMAKE_INSTALL_LIBDIR}/pkgconfig:${CMAKE_INSTALL_DATADIR}/pkgconfig:/usr/lib/pkgconfig:/usr/share/pkgconfig")
define_var(SYSTEM_INCLUDE_PATH "/usr/include")
define_var(SYSTEM_LIBRARY_PATH "/usr/lib:/lib")

set(PACKAGE_VERSION "${pkgconfig_VERSION}")

add_definitions(-DVERSION="${pkgconfig_VERSION}")

# TODO: Read from deplibs_check_method from `libtool --config`
set(ENABLE_INDIRECT_DEPS OFF CACHE BOOL "")
set(use_indirect_deps ${ENABLE_INDIRECT_DEPS})
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

include(CTest)

configure_file(check/config.sh.in ${CMAKE_SOURCE_DIR}/check/config.sh)

add_custom_target(check COMMAND ${CMAKE_CTEST_COMMAND} --output-on-failure -C ${CMAKE_CFG_INTDIR})

find_program(TEST_SHELL NAMES bash ksh sh)

macro(check_test TEST_NAME)
    add_test(NAME check-${TEST_NAME} WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}/check COMMAND ${TEST_SHELL} check-${TEST_NAME})
    set_tests_properties(check-${TEST_NAME} PROPERTIES ENVIRONMENT "srcdir=${CMAKE_SOURCE_DIR}/check;PKG_CONFIG=${CMAKE_BINARY_DIR}/pkg-config")
endmacro()

check_test(cflags)
check_test(libs)
check_test(mixed-flags)
check_test(non-l-flags)
check_test(define-variable)
check_test(libs-private)
check_test(requires-private)
check_test(circular-requires)
check_test(includedir)
check_test(conflicts)
check_test(missing)
check_test(idirafter)
check_test(sort-order)
check_test(duplicate-flags)
check_test(whitespace)
check_test(cmd-options)
check_test(version)
check_test(requires-version)
check_test(print-options)
check_test(path)
check_test(sysroot)
check_test(uninstalled)
check_test(debug)
check_test(gtk)
check_test(tilde)
