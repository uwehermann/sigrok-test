##
## This file is part of the sigrok-firmware project.
##
## Copyright (C) 2020 Uwe Hermann <uwe@hermann-uwe.de>
##
## This program is free software: you can redistribute it and/or modify
## it under the terms of the GNU General Public License as published by
## the Free Software Foundation, either version 2 of the License, or
## (at your option) any later version.
##
## This program is distributed in the hope that it will be useful,
## but WITHOUT ANY WARRANTY; without even the implied warranty of
## MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
## GNU General Public License for more details.
##
## You should have received a copy of the GNU General Public License
## along with this program.  If not, see <http://www.gnu.org/licenses/>.
##

cmake_minimum_required(VERSION 3.12)

project(sigrok-test VERSION 0.1.0 LANGUAGES C)

option(BUILD_SHARED_LIBS "Shared (non-static) build" TRUE)

if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
	set_property(CACHE CMAKE_BUILD_TYPE PROPERTY VALUE RelWithDebInfo)
endif()

set_property(GLOBAL PROPERTY USE_FOLDERS TRUE)

find_package(PkgConfig REQUIRED)
pkg_check_modules(LIBGLIB REQUIRED IMPORTED_TARGET "glib-2.0>=2.24.0")
pkg_search_module(LIBPYTHON REQUIRED IMPORTED_TARGET "python3-embed>=3.2;python3>=3.2")
message(STATUS "  Found Python, version ${LIBPYTHON_VERSION}")

find_package(sigrok 6.0.0 REQUIRED)
find_package(sigrokdecode 6.0.0 REQUIRED)
message(STATUS "libsigrok: ${sigrok_VERSION}")
message(STATUS "libsigrokdecode: ${sigrokdecode_VERSION}")

add_executable(runtc "")

target_sources(runtc PRIVATE decoder/runtc.c)

set_target_properties(runtc PROPERTIES RUNTIME_OUTPUT_DIRECTORY decoder)

target_compile_features(runtc PRIVATE c_std_99)

target_compile_options(runtc PRIVATE
	$<IF:$<C_COMPILER_ID:MSVC>,/W4,-Wall -Wextra>)

target_compile_definitions(runtc PRIVATE
	-DDECODERS_DIR="${sigrokdecode_INCLUDE_DIR}/../share/libsigrokdecode/decoders")

target_link_libraries(runtc PRIVATE sigrok::sigrok sigrok::sigrokdecode)

function(pkgconfig_lib p)
	if(${p}_FOUND AND BUILD_SHARED_LIBS)
		target_link_libraries(runtc PRIVATE PkgConfig::${p})
	elseif(${p}_FOUND)
		target_include_directories(runtc PRIVATE ${${p}_STATIC_INCLUDE_DIRS})
		target_link_libraries(runtc PRIVATE ${${p}_STATIC_LDFLAGS})
	endif()
endfunction()

pkgconfig_lib(LIBGLIB)
pkgconfig_lib(LIBPYTHON)

message(STATUS "CMake: ${CMAKE_COMMAND} ${CMAKE_VERSION}")
message(STATUS "Building ${PROJECT_NAME} ${PROJECT_VERSION} for ${CMAKE_SYSTEM} ${CMAKE_SYSTEM_PROCESSOR}")
