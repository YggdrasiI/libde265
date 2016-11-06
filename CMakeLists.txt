project (libde265)
cmake_minimum_required (VERSION 2.8.8)

# The version number.
set (NUMERIC_VERSION 0x01000000)
set (PACKAGE_VERSION 1.0.0)

include (${CMAKE_ROOT}/Modules/CheckCCompilerFlag.cmake)
include (${CMAKE_ROOT}/Modules/CheckIncludeFile.cmake)
include (${CMAKE_ROOT}/Modules/CheckIncludeFileCXX.cmake)
include (${CMAKE_ROOT}/Modules/FindSDL.cmake)
include (${CMAKE_ROOT}/Modules/FindThreads.cmake)

set(headers
	malloc.h stdint.h stdbool.h
	stdlib.h string.h
	signal.h setjmp.h
	stddef.h sys/time.h
	)
set(headers_CXX	cstdint)

configure_file (
  "${PROJECT_SOURCE_DIR}/libde265/de265-version.h.in"
  "${PROJECT_BINARY_DIR}/libde265/de265-version.h"
)

if(CMAKE_COMPILER_IS_GNUCXX)
  set(GCC 1)
  add_definitions(-Wall)
  set(CMAKE_CXX_FLAGS "-std=gnu++0x ${CMAKE_CXX_FLAGS}")
elseif("${CMAKE_CXX_COMPILER_ID}" MATCHES "Clang")
  add_definitions(-Wall)
  set(CMAKE_CXX_FLAGS "-std=c++11 ${CMAKE_CXX_FLAGS}")
endif()

function(AS_TR_SH header_name header_var)
	string(REGEX REPLACE "^(.*[\\/])*([^\\/]+)$" "\\2"
		tmp "${header_name}")
	string(TOUPPER "${tmp}" tmp)
	string(REPLACE "." "_" tmp ${tmp})
	set(${header_var} ${tmp} PARENT_SCOPE)
endfunction()

# C header search
foreach(header ${headers})
	AS_TR_SH("${header}" "CONF_HEADER")
	CHECK_INCLUDE_FILE(${header} "HAVE_${CONF_HEADER}")
	if("HAVE_${CONF_HEADER}")
		add_definitions(-DHAVE_${CONF_HEADER})
	endif()
endforeach()

# C++ header search (depends on flags, see CMAKE_COMPILER_IS_GNUCXX)
foreach(header ${headers_CXX})
	AS_TR_SH("${header}" "CONF_HEADER")
	CHECK_INCLUDE_FILE_CXX(${header} "HAVE_${CONF_HEADER}")
	if("HAVE_${CONF_HEADER}")
		add_definitions(-DHAVE_${CONF_HEADER})
	endif()
endforeach()

option(DISABLE_SSE "Disable SSE optimizations")
if(NOT ${DISABLE_SSE} EQUAL OFF)
  if(MSVC)
    set(SUPPORTS_SSE4_1 1)
  else()
    CHECK_C_COMPILER_FLAG(-msse4.1 SUPPORTS_SSE4_1)
  endif()
endif()

include_directories ("${PROJECT_SOURCE_DIR}")
include_directories ("${PROJECT_BINARY_DIR}")
include_directories ("${PROJECT_SOURCE_DIR}/libde265")
if(MSVC)
  include_directories ("${PROJECT_SOURCE_DIR}/extra")
  add_definitions(-DHAVE_STDINT_H)
  add_definitions(-DHAVE_STDBOOL_H)
  add_definitions(-DNOMINMAX)
endif()

if(UNIX)
  set(LIBDE265_LIBRARY_NAME de265)
else()
  set(LIBDE265_LIBRARY_NAME libde265)
endif()

option(DISABLE_ARM "Disable ARM optimizations")
option(DISABLE_NEON "Disable ARM NEON optimizations")
option(DISABLE_THUMB "Disable ARM Thumb")

if(NOT ${DISABLE_ARM} EQUAL OFF)
	set(SUPPORTS_ARM 1)

	if(NOT ${DISABLE_ARM} EQUAL OFF)
		if(MSVC)
			set(SUPPORTS_NEON 1)
		else()
			CHECK_C_COMPILER_FLAG(-mfpu=neon SUPPORTS_NEON)
		endif()
	endif()
	if("${SUPPORTS_NEON}" AND
			(NOT "${HAVE_SIGNAL_H}" OR NOT "${HAVE_SETJMP_H}" ))
		message("NEON supported but not all headers found to enable it at runtime.")
	endif()

	if(NOT ${DISABLE_THUMB} EQUAL OFF)
		set(ENABLE_THUMB 1)
	endif()
endif()

add_subdirectory (libde265)
add_subdirectory (dec265)
add_subdirectory (enc265)
