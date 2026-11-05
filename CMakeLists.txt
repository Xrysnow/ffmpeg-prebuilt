cmake_minimum_required(VERSION 3.28)

set(lib_name ffmpeg)
set(target_name ext_${lib_name})
set(PREBUILT_ROOT "")

if(WINDOWS OR ANDROID OR LINUX)
  project(${lib_name})
endif()

if(WINDOWS OR ANDROID OR LINUX OR APPLE OR CMAKE_SYSTEM_NAME STREQUAL "OHOS")
  add_library(${target_name} SHARED IMPORTED GLOBAL)
  
  set_target_properties(${target_name} PROPERTIES
    INTERFACE_INCLUDE_DIRECTORIES ${CMAKE_CURRENT_SOURCE_DIR}/include
  )
endif()

if(WINDOWS)
  string(TOLOWER "${CMAKE_C_COMPILER_ARCHITECTURE_ID}" FFMPEG_WINDOWS_ARCH)
  if(NOT FFMPEG_WINDOWS_ARCH MATCHES "^(x86|x64|arm64)$")
    message(FATAL_ERROR "No bundled FFmpeg 9 library for Windows ${CMAKE_C_COMPILER_ARCHITECTURE_ID}")
  endif()
  set(PREBUILT_ROOT "${CMAKE_CURRENT_SOURCE_DIR}/windows/${FFMPEG_WINDOWS_ARCH}")

  set_target_properties(${target_name} PROPERTIES
    IMPORTED_LOCATION "${PREBUILT_ROOT}/ffmpeg-9.dll"
    IMPORTED_IMPLIB "${PREBUILT_ROOT}/ffmpeg.lib"
  )
elseif(ANDROID)
  set(PREBUILT_ROOT "${CMAKE_CURRENT_SOURCE_DIR}/android/${ANDROID_ABI}")
  set_target_properties(${target_name} PROPERTIES
    IMPORTED_LOCATION "${PREBUILT_ROOT}/libffmpeg.so"
  )
elseif(APPLE)
  set(PREBUILT_ROOT "${CMAKE_CURRENT_SOURCE_DIR}/apple")
  if(NOT EXISTS "${PREBUILT_ROOT}/FFmpeg.xcframework/Info.plist"
      OR NOT EXISTS "${PREBUILT_ROOT}/include/libavcodec/avcodec.h")
    message(FATAL_ERROR "Incomplete FFmpeg XCFramework package: ${PREBUILT_ROOT}")
  endif()

  # avbuild ships common headers next to the XCFramework, not in each slice.
  set(FFMPEG_INCLUDE_DIR "${PREBUILT_ROOT}/include" CACHE INTERNAL "FFmpeg header location")
  set(FFMPEG_LIBRARY "${PREBUILT_ROOT}/FFmpeg.xcframework" CACHE INTERNAL "FFmpeg library")
  set_target_properties(${target_name} PROPERTIES
    IMPORTED_LOCATION "${FFMPEG_LIBRARY}"
    INTERFACE_INCLUDE_DIRECTORIES "${FFMPEG_INCLUDE_DIR}"
    FRAMEWORK TRUE
  )
elseif(CMAKE_SYSTEM_NAME STREQUAL "OHOS")
  set(PREBUILT_ROOT "${CMAKE_CURRENT_SOURCE_DIR}/ohos/${OHOS_ARCH}")
  if(NOT EXISTS "${PREBUILT_ROOT}/libffmpeg.so")
    message(FATAL_ERROR "No bundled FFmpeg 9 library for OpenHarmony ${OHOS_ARCH}")
  endif()
  set_target_properties(${target_name} PROPERTIES
    IMPORTED_LOCATION "${PREBUILT_ROOT}/libffmpeg.so"
  )
elseif(LINUX)
  if(CMAKE_SYSTEM_PROCESSOR MATCHES "^(x86_64|amd64|AMD64)$")
    set(FFMPEG_LINUX_ARCH amd64)
  elseif(CMAKE_SYSTEM_PROCESSOR MATCHES "^(aarch64|arm64)$")
    set(FFMPEG_LINUX_ARCH arm64)
  elseif(CMAKE_SYSTEM_PROCESSOR MATCHES "^arm")
    set(FFMPEG_LINUX_ARCH armhf)
  else()
    message(FATAL_ERROR "No bundled FFmpeg 9 library for Linux ${CMAKE_SYSTEM_PROCESSOR}")
  endif()
  set(PREBUILT_ROOT "${CMAKE_CURRENT_SOURCE_DIR}/linux/${FFMPEG_LINUX_ARCH}")
  set_target_properties(${target_name} PROPERTIES
    IMPORTED_LOCATION "${PREBUILT_ROOT}/libffmpeg.so.9"
  )
endif()
