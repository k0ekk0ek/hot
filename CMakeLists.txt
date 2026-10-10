cmake_minimum_required(VERSION 3.10)
project(hot LANGUAGES CXX)

set(CXX_STANDARD C++14)

option(USEAVX2 "USEAVX2" ON)
option(USEAVX512 "USEAVX512" OFF)
option(USE_COUNTERS "USE_COUNTERS" OFF)
option(COVERAGE "COVERAGE" OFF)

if(NOT CMAKE_BUILD_TYPE)
	set(CMAKE_BUILD_TYPE Release)
endif()

set(CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/cmake/Modules)

add_compile_options(
  -march=native
  -Wno-deprecated-declarations
  -Wall
  -fno-builtin-malloc
  -fno-builtin-calloc
  -fno-builtin-realloc
  -fno-builtin-free
  -faligned-new)

if(USEAVX2)
  add_compile_options(-mavx -mavx2 -mbmi2 -mlzcnt)
elseif(USEAVX512)
	add_compile_options(-mavx512f -mavx512vl -mavx512bw -mavx512dq -mavx512cd)
	add_definitions(-DUSE_AVX512)
endif()

if(USE_COUNTERS)
	SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -DUSE_COUNTERS=1")
endif(USE_COUNTERS)

set(TC_MALLOC_NAMES tcmalloc tcmalloc_minimal libtcmalloc_minimal.so.4)

find_library(TC_MALLOC_LIBRARY
	NAMES ${TC_MALLOC_NAMES}
	PATHS /usr/lib/x86_64-linux-gnu /usr/lib64 /lib /usr/lib /usr/local/lib /opt/local/lib
	REQUIRED
)

find_package(TBB REQUIRED)
find_package(Boost CONFIG REQUIRED COMPONENTS unit_test_framework)

IF(COVERAGE)
	#call with -DCOVERAGE=1
	set(CMAKE_EXE_LINKER_FLAGS "-fprofile-arcs -ftest-coverage")
	SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -g -O0 -Wall -W -fprofile-arcs -ftest-coverage")
	SET(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -g -O0 -Wall -W -fprofile-arcs -ftest-coverage")

	include(CodeCoverage)
	message("Code Coverage Activated")
ENDIF(COVERAGE)

ENABLE_TESTING()

ADD_CUSTOM_TARGET(check COMMAND ${CMAKE_CTEST_COMMAND})

ADD_SUBDIRECTORY(apps)
ADD_SUBDIRECTORY(libs)
ADD_SUBDIRECTORY(tests)
