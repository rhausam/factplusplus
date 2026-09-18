cmake_minimum_required(VERSION 3.10...3.27)

# We are using JNI and need to #include <jni.h>
find_package(JNI REQUIRED)

set(SOURCE_FILES
	Axioms.cpp
	Expressions.cpp
	FaCTPlusPlus.cpp
	JNIActor.h
	JNIMonitor.h
	JNISupport.h
	KnowledgeExploration.cpp
	MemoryStat.cpp
	MemoryStat.h
	Modularity.cpp
	Queries.cpp
	SaveLoad.cpp
	tJNICache.h
	uk_ac_manchester_cs_factplusplus_FaCTPlusPlus.h
)

# JNI Library should be declared as MODULE
add_library(FaCTPlusPlusJNI MODULE ${SOURCE_FILES})
target_link_libraries(FaCTPlusPlusJNI LINK_PUBLIC Kernel)

# Include search path for jni.h and jni_md.h
target_include_directories(FaCTPlusPlusJNI PRIVATE
	${JAVA_INCLUDE_PATH}
	${JAVA_INCLUDE_PATH2}
)

# The Protege plug-in manifest (Bundle-NativeCode) expects the classic
# libFaCTPlusPlusJNI.jnilib name on macOS, not the CMake default .so for MODULE.
if (APPLE)
	set_target_properties(FaCTPlusPlusJNI PROPERTIES SUFFIX ".jnilib")
endif()

# allow library users to find headers
target_include_directories(FaCTPlusPlusJNI PUBLIC ${CMAKE_CURRENT_SOURCE_DIR})
