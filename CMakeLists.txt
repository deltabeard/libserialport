cmake_minimum_required(VERSION 3.20)
project(libserialport
        VERSION 0.1.2
        LANGUAGES C)

set(CMAKE_C_STANDARD 99)
include(CheckSymbolExists)

add_library(libserialport src/serialport.c src/timing.c)
target_include_directories(libserialport
        PUBLIC
        $<INSTALL_INTERFACE:inc>
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/inc>
        PRIVATE
        ${CMAKE_CURRENT_SOURCE_DIR}/src
        )
target_compile_features(libserialport PRIVATE c_std_99)

if (WIN32)
    target_sources(libserialport PRIVATE src/windows.c)
    target_link_libraries(libserialport PRIVATE setupapi)
    add_compile_definitions(LIBSERIALPORT_MSBUILD)
elseif (APPLE)
    target_sources(libserialport PRIVATE src/macosx.c)
elseif (UNIX)
    target_sources(libserialport PRIVATE src/linux.c src/linux_termios.c)
    #check_symbol_exists(serial_struct "linux/serial" HAVE_SERIAL_STRUCT)
elseif (CMAKE_SYSTEM_NAME MATCHES "FreeBSD")
    target_sources(libserialport PRIVATE src/freebsd.c)
endif ()

add_compile_definitions(SP_PRIV=)

set_target_properties(libserialport PROPERTIES PUBLIC_HEADER inc/libserialport.h)

include(GNUInstallDirs)
install(TARGETS libserialport
        LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
        PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})

configure_file(libserialport.pc.in libserialport.pc @ONLY)
install(FILES ${CMAKE_BINARY_DIR}/libserialport.pc DESTINATION ${CMAKE_INSTALL_DATAROOTDIR}/pkgconfig)

include(CPack)
