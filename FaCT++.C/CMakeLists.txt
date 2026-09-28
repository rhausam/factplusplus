cmake_minimum_required(VERSION 3.10...3.27)

set(SOURCE_FILES
	fact.cpp
	fact.h
)

add_library(fact SHARED ${SOURCE_FILES})
# fact.h marks the API __declspec(dllexport) only when this is defined (Windows)
target_compile_definitions(fact PRIVATE FPP_BUILD_SHARED)
target_link_libraries(fact LINK_PUBLIC Kernel)

add_executable(fact_test test.c)
target_link_libraries(fact_test LINK_PUBLIC Kernel fact)

# allow library users to find headers
target_include_directories(fact PUBLIC ${CMAKE_CURRENT_SOURCE_DIR})
