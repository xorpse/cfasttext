cmake_minimum_required(VERSION 2.8 FATAL_ERROR)
project(cfasttext)

set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -std=c99")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11 -pthread -funroll-loops")

if(NOT CMAKE_BUILD_TYPE STREQUAL "Debug")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -O3")
endif()

if(APPLE)
  # on macOS "uname -m" returns the architecture (x86_64 or arm64)
  execute_process(
      COMMAND uname -m
      RESULT_VARIABLE result
      OUTPUT_VARIABLE FASTTEXT_OSX_NATIVE_ARCHITECTURE
      OUTPUT_STRIP_TRAILING_WHITESPACE
  )

  if(FASTTEXT_OSX_NATIVE_ARCHITECTURE STREQUAL "arm64")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -mcpu=apple-m1")
  #else()
  #  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -march=native")
  endif()
#else()
  #set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -march=native")
endif()

set(CMAKE_MACOSX_RPATH 1)
set(CMAKE_POSITION_INDEPENDENT_CODE ON)

include_directories(
    ${PROJECT_SOURCE_DIR}/fasttext/src
    ${PROJECT_SOURCE_DIR}/include)

set(SOURCES
    ${PROJECT_SOURCE_DIR}/lib/cfasttext.cc)

set(HEADER_FILES
    ${PROJECT_SOURCE_DIR}/fasttext/src/args.h
    ${PROJECT_SOURCE_DIR}/fasttext/src/autotune.h
    ${PROJECT_SOURCE_DIR}/fasttext/src/densematrix.h
    ${PROJECT_SOURCE_DIR}/fasttext/src/dictionary.h
    ${PROJECT_SOURCE_DIR}/fasttext/src/fasttext.h
    ${PROJECT_SOURCE_DIR}/fasttext/src/loss.h
    ${PROJECT_SOURCE_DIR}/fasttext/src/matrix.h
    ${PROJECT_SOURCE_DIR}/fasttext/src/meter.h
    ${PROJECT_SOURCE_DIR}/fasttext/src/model.h
    ${PROJECT_SOURCE_DIR}/fasttext/src/productquantizer.h
    ${PROJECT_SOURCE_DIR}/fasttext/src/quantmatrix.h
    ${PROJECT_SOURCE_DIR}/fasttext/src/real.h
    ${PROJECT_SOURCE_DIR}/fasttext/src/utils.h
    ${PROJECT_SOURCE_DIR}/fasttext/src/vector.h)

set(SOURCE_FILES
    ${PROJECT_SOURCE_DIR}/fasttext/src/args.cc
    ${PROJECT_SOURCE_DIR}/fasttext/src/autotune.cc
    ${PROJECT_SOURCE_DIR}/fasttext/src/densematrix.cc
    ${PROJECT_SOURCE_DIR}/fasttext/src/dictionary.cc
    ${PROJECT_SOURCE_DIR}/fasttext/src/fasttext.cc
    ${PROJECT_SOURCE_DIR}/fasttext/src/loss.cc
    ${PROJECT_SOURCE_DIR}/fasttext/src/main.cc
    ${PROJECT_SOURCE_DIR}/fasttext/src/matrix.cc
    ${PROJECT_SOURCE_DIR}/fasttext/src/meter.cc
    ${PROJECT_SOURCE_DIR}/fasttext/src/model.cc
    ${PROJECT_SOURCE_DIR}/fasttext/src/productquantizer.cc
    ${PROJECT_SOURCE_DIR}/fasttext/src/quantmatrix.cc
    ${PROJECT_SOURCE_DIR}/fasttext/src/utils.cc
    ${PROJECT_SOURCE_DIR}/fasttext/src/vector.cc)

add_library(objlib OBJECT ${SOURCES})

add_library(cfasttext SHARED $<TARGET_OBJECTS:objlib>)
add_library(cfasttext_static STATIC $<TARGET_OBJECTS:objlib>)

set_target_properties(cfasttext PROPERTIES PUBLIC_HEADER ${PROJECT_SOURCE_DIR}/include/cfasttext.h)

add_library(fasttext-shared SHARED ${SOURCE_FILES} ${HEADER_FILES})
add_library(fasttext-static STATIC ${SOURCE_FILES} ${HEADER_FILES})
add_library(fasttext-static_pic STATIC ${SOURCE_FILES} ${HEADER_FILES})
set_target_properties(fasttext-shared PROPERTIES OUTPUT_NAME fasttext
  SOVERSION "${fasttext_VERSION_MAJOR}")
set_target_properties(fasttext-static PROPERTIES OUTPUT_NAME fasttext)
set_target_properties(fasttext-static_pic PROPERTIES OUTPUT_NAME fasttext_pic
  POSITION_INDEPENDENT_CODE True)

target_link_libraries(cfasttext fasttext-static_pic)
target_link_libraries(cfasttext_static fasttext-static)
