cmake_minimum_required(VERSION 3.10...3.27)
project(factplusplus)

# global definition
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

option(FPP_USE_LOGGING "Switch on FaCT++ internal logging" OFF)

# On macOS default to the host architecture unless the caller asks for
# something else (e.g. -DCMAKE_OSX_ARCHITECTURES="arm64;x86_64" for a fat build).
if (APPLE AND NOT CMAKE_OSX_ARCHITECTURES)
	set(CMAKE_OSX_ARCHITECTURES "${CMAKE_HOST_SYSTEM_PROCESSOR}" CACHE STRING "macOS build architectures" FORCE)
endif()

# define warnings/optimisaiton flags for GCC/CLANG compilers
if (NOT MSVC)
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wextra -ffast-math -fPIC -fomit-frame-pointer")
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wpedantic")
	# -Wsuggest-override is GCC-only; clang spells the equivalent -Winconsistent-missing-override
	if (CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
		set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wsuggest-override")
	endif()
else()
	# Set exception handling in VC++
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /EHsc")
	# Disable warning about treating int as bool in VC++
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /wd4800")
endif()

if (FPP_USE_LOGGING)
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -DUSE_LOGGING=1")
endif()

add_subdirectory(Kernel)
add_subdirectory(FaCT++)
add_subdirectory(FaCT++.C)
add_subdirectory(FaCT++.JNI)
