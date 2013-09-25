cmake_minimum_required(VERSION 2.8)

project(libjson--)

# ======================================
# DEFAUT VALUES ARE SETUP HERE
# ======================================
#
# Minimal OS X SDK Version
set(OSX_SDK_VER "10.8")
# Default optimization level for Release builds
set(DEF_OPT_LEVEL "-O3")
#

IF(NOT CMAKE_BUILD_TYPE)
  SET(CMAKE_BUILD_TYPE Release CACHE STRING
    "Choose the type of build, options are: None Debug Release RelWithDebInfo MinSizeRel."
    FORCE)
ENDIF(NOT CMAKE_BUILD_TYPE)

message("-------- System: ${CMAKE_SYSTEM_NAME}")
message("---- ${CMAKE_BUILD_TYPE} build:")

if (${CMAKE_SYSTEM_NAME} MATCHES "Darwin" OR ${CMAKE_SYSTEM_NAME} MATCHES "Linux")
  # Add 3rdparty libraries
  include_directories(/opt/local/include)
  set(LIB_PATHS ~/lib /opt/local/lib)

  # clang detection
  execute_process(COMMAND clang --version OUTPUT_VARIABLE CLANG_VERSION RESULT_VARIABLE resv)
  if (${resv} MATCHES 0)
    string(REGEX MATCHALL "[0-9]" CLANG_VERSION_COMPONENTS ${CLANG_VERSION})
    list(GET CLANG_VERSION_COMPONENTS 0 CLANG_MAJOR)
    list(GET CLANG_VERSION_COMPONENTS 1 CLANG_MINOR)
    message("-- using clang: ${CLANG_MAJOR}.${CLANG_MINOR}")

    # clang stuff
    set(CMAKE_C_COMPILER "clang")
    set(CMAKE_CXX_COMPILER "clang++")
    add_definitions("-std=c++11 -Wc++11-extensions")
    set(DEF_OPT_LEVEL "-Ofast")

    if (${CLANG_MAJOR} GREATER "3" OR (${CLANG_MAJOR} EQUAL "3" OR ${CLANG_MINOR} GREATER "1"))
      add_definitions("-Wno-tautological-compare")
    endif()
  endif()

  set(EXTRA_FLAGS "")
  
  if (${CMAKE_SYSTEM_NAME} MATCHES "Darwin")
    # Xcode detection
    execute_process(COMMAND xcode-select -print-path OUTPUT_VARIABLE XCODE_PATH RESULT_VARIABLE resv OUTPUT_STRIP_TRAILING_WHITESPACE)
    if (${resv} MATCHES 0)
      set(XCODE_SDK "${XCODE_PATH}/Platforms/MacOSX.platform/Developer/SDKs/MacOSX${OSX_SDK_VER}.sdk")
      message("-- using xcode tools: ${XCODE_PATH}")
      message("-- using SDK: ${XCODE_SDK}")
    else()
      message(FATAL_ERROR "-- failed to locate xcode tools: ${resv}")
    endif()

    # extra stuff
    set(EXTRA_FLAGS "-isysroot ${XCODE_SDK} -mmacosx-version-min=${OSX_SDK_VER}")
  endif()

  set(CMAKE_C_FLAGS ${CMAKE_C_FLAGS} ${EXTRA_FLAGS})
  set(CMAKE_CXX_FLAGS ${CMAKE_CXX_FLAGS} ${EXTRA_FLAGS})
endif()

if (CMAKE_BUILD_TYPE MATCHES "Debug")
  add_definitions("-O0 -D_DEBUG=1")
else()
  add_definitions("${DEF_OPT_LEVEL}")
endif()

include_directories(json)

# Library
add_library(json-- json/json.cpp)

# Console tests
add_executable(sample sample.cpp)
target_link_libraries(sample json--)
