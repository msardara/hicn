cmake_minimum_required(VERSION 3.5 FATAL_ERROR)

project(hicn-light)

if (NOT CMAKE_BUILD_TYPE)
    message(STATUS "No build type selected, default to Release")
    set(CMAKE_BUILD_TYPE "Release")
endif()

set(CMAKE_MODULE_PATH
  ${CMAKE_MODULE_PATH}
  "${CMAKE_CURRENT_SOURCE_DIR}/../cmake/Modules"
  "${CMAKE_CURRENT_SOURCE_DIR}/cmake/Modules"
)

set(CMAKE_C_STANDARD 11)
set(CMAKE_C_STANDARD_REQUIRED ON)

include( CTest )
include( detectCacheSize )

if(NOT WIN32)
  set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wall")
else ()
  set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} /wd4996")
endif ()
if(ANDROID_API)
  message("############ Detected cross compile for $ENV{CMAKE_SYSTEM_NAME}")
  set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${ANDROID_C_FLAGS}")
  endif()

set(CMAKE_C_FLAGS_RELEASE "${CMAKE_C_FLAGS_RELEASE} -DLIBRTA_DISABLE_VALIDATION -DPARCLibrary_DISABLE_VALIDATION")

include(IosMacros)
include(WindowsMacros)

find_package_wrapper(Libparc REQUIRED)

if(CMAKE_SOURCE_DIR STREQUAL PROJECT_SOURCE_DIR)
  find_package_wrapper(Libhicn REQUIRED)
  find_package_wrapper(Libparc REQUIRED)
  set(LIBHICN_LIGHT hicn-light)
  set(HICN_LIGHT_CONTROL hicnLightControl)
  set(HICN_LIGHT_DAEMON hicnLightDaemon)
else()
  set(HICN_LIBRARIES ${LIBHICN_SHARED})
  set(DEPENDENCIES
    ${LIBHICN}
    ${LIBHICN_SHARED}
  )
endif()

include( Packaging )

find_package(Threads REQUIRED)

set(HICN_LIGHT_LINK_LIBRARIES
  hicn-light
  ${HICN_LIBRARIES}
  ${LIBPARC_LIBRARIES}
  ${CMAKE_THREAD_LIBS_INIT}
  ${WINDOWS_LIBRARIES}
)

# Include dirs -- Order does matter!
list(APPEND HICN_LIGHT_INCLUDE_DIRS
  ${HICN_INCLUDE_DIRS}
  ${LIBPARC_INCLUDE_DIRS}
  ${PTHREAD_INCLUDE_DIRS}
)

if (UNIX)
  list(APPEND HICN_LIGHT_LINK_LIBRARIES
    m
  )
endif()

set(CMAKE_INSTALL_RPATH "${CMAKE_INSTALL_PREFIX}/lib")

if("${CMAKE_CXX_COMPILER_ID}" STREQUAL "AppleClang")
  set(CMAKE_SHARED_LIBRARY_CREATE_C_FLAGS "${CMAKE_SHARED_LIBRARY_CREATE_C_FLAGS} -undefined dynamic_lookup")
	message(STATUS "Set \"-undefined dynamic_lookup\" for shared libraries")
endif()

add_subdirectory(src)