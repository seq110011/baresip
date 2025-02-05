#
# CMakeLists.txt
#
# Copyright (C) 2010 - 2022 Alfred E. Heggestad
# Copyright (C) 2022 Sebastian Reimers
#

##############################################################################
#
# Versioning
#

cmake_minimum_required(VERSION 3.7)

project(baresip)

list(APPEND CMAKE_MODULE_PATH ${CMAKE_CURRENT_LIST_DIR}/cmake)

##############################################################################
#
# Module/Package Includes
#

find_package(OpenSSL)
find_package(OPUS)


##############################################################################
#
# Compile options
#

option(USE_OPENSSL "Enable OpenSSL" ${OPENSSL_FOUND})
option(STATIC "Build static" OFF)

include_directories(
  include
  src
  ../re/include
  ../rem/include
  /usr/local/include
  /usr/local/include/re
  /usr/local/include/rem
)

link_directories(/usr/local/lib)

add_definitions(
  -DHAVE_GETOPT
  -DHAVE_INET6
  -DHAVE_PTHREAD
  -DHAVE_SIGNAL
  -DHAVE_UNISTD_H
  -DUSE_ZLIB
  -DSHARE_PATH="/usr/share/baresip"
  )

if(USE_OPENSSL)
  add_definitions(
    -DUSE_DTLS
    -DUSE_OPENSSL
    -DUSE_OPENSSL_AES
    -DUSE_OPENSSL_DTLS
    -DUSE_OPENSSL_HMAC
    -DUSE_OPENSSL_SRTP
    -DUSE_TLS
  )
endif()

if(STATIC)
  add_definitions(-DSTATIC)
endif()

##############################################################################
#
# Source section
#

set(SRCS
  src/account.c
  src/aucodec.c
  src/audio.c
  src/aufilt.c
  src/auplay.c
  src/ausrc.c
  src/baresip.c
  src/bundle.c
  src/call.c
  src/cmd.c
  src/conf.c
  src/config.c
  src/contact.c
  src/custom_hdrs.c
  src/descr.c
  src/dial_number.c
  src/event.c
  src/log.c
  src/mediadev.c
  src/menc.c
  src/message.c
  src/metric.c
  src/mnat.c
  src/module.c
  src/net.c
  src/play.c
  src/reg.c
  src/rtpext.c
  src/rtpstat.c
  src/sdp.c
  src/sipreq.c
  src/stream.c
  src/stunuri.c
  src/timestamp.c
  src/ua.c
  src/uag.c
  src/ui.c
  src/vidcodec.c
  src/video.c
  src/vidfilt.c
  src/vidisp.c
  src/vidsrc.c
  src/vidutil.c
)


##############################################################################
#
# Modules
#

set(MODULES "")

add_subdirectory(modules/echo)
add_subdirectory(modules/menu)
add_subdirectory(modules/stdio)

if(OPUS_FOUND)
add_subdirectory(modules/opus)
endif()


if(STATIC)
  foreach(mod IN LISTS MODULES)
    set(MOD_EXPORTS
      "${MOD_EXPORTS}extern const struct mod_export exports_${mod};\n")
    set(MOD_EXPORT_TABLE
      "${MOD_EXPORT_TABLE}  &exports_${mod},\n")
  endforeach()

  configure_file(src/static.c.in src/static.c)
  list(APPEND SRCS ${CMAKE_CURRENT_BINARY_DIR}/src/static.c)
endif()


##############################################################################
#
# Main target library
#

if(STATIC)
  add_library(baresip STATIC ${SRCS})
  target_link_libraries(baresip PUBLIC ${MODULES} rem re -lm)
else()
  add_library(baresip SHARED ${SRCS})
  target_link_libraries(baresip PRIVATE rem re -lm)
endif()

set_target_properties(baresip PROPERTIES POSITION_INDEPENDENT_CODE ON)


##############################################################################
#
# Target baresip executable
#

add_executable(baresip_exe src/main.c)

set_target_properties(baresip_exe PROPERTIES OUTPUT_NAME baresip)

target_link_libraries(baresip_exe baresip rem re -lz -lm -lpthread)
