cmake_minimum_required(VERSION 3.5)
project(libtelnet VERSION 0.30.0 LANGUAGES C)

option(LIBTELNET_STRICT "Treat warnings as errors" OFF)
include(GNUInstallDirs)

# Compiler-Flags sauber mit Zeilenumbrüchen
if (MSVC)
  add_definitions(-D_SCL_SECURE_NO_WARNINGS -D_CRT_SECURE_NO_WARNINGS)
  if (LIBTELNET_STRICT)
    add_compile_options(/WX)
  endif()
else()
  if (LIBTELNET_STRICT)
    add_compile_options(-Wall -Wextra -pedantic -Werror)
  endif()
endif()

# EIN Target; Typ via -DBUILD_SHARED_LIBS=ON|OFF
add_library(telnet libtelnet.c)

# .a (bzw. statische Lib) im Projekt-Stammverzeichnis ablegen
set_target_properties(telnet PROPERTIES
  ARCHIVE_OUTPUT_DIRECTORY "${CMAKE_SOURCE_DIR}"
)

target_include_directories(telnet
  PUBLIC
    $<INSTALL_INTERFACE:include>
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
)

# macOS-Properties nur für SHARED
if (APPLE)
  get_target_property(_t telnet TYPE)
  if (_t STREQUAL "SHARED_LIBRARY")
    set_target_properties(telnet PROPERTIES
      MACOSX_RPATH ON
      INSTALL_NAME_DIR "@rpath"
      SOVERSION 0
      VERSION   ${PROJECT_VERSION}
    )
  endif()
endif()

# Installation
install(TARGETS telnet
  EXPORT libtelnet-export
  LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
  ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
)
install(FILES libtelnet.h DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})
install(EXPORT libtelnet-export
  FILE libtelnet.cmake
  NAMESPACE libtelnet::
  DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/libtelnet
)

# pkg-config
configure_file(${CMAKE_CURRENT_SOURCE_DIR}/libtelnet.pc.in
               ${CMAKE_CURRENT_BINARY_DIR}/libtelnet.pc @ONLY)
install(FILES ${CMAKE_CURRENT_BINARY_DIR}/libtelnet.pc
        DESTINATION ${CMAKE_INSTALL_LIBDIR}/pkgconfig)
