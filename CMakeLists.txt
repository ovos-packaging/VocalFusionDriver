# Set minimum CMake version (required for CMake 3.0 or later)
cmake_minimum_required(VERSION 3.0.0 FATAL_ERROR)

PROJECT(vocal-fusion-driver C)

IF(POLICY CMP0063)
    CMAKE_POLICY(SET CMP0063 NEW)
ENDIF(POLICY CMP0063)

set(CMAKE_C_STANDARD 11)
set(CMAKE_C_STANDARD_REQUIRED ON)

include_directories(${CMAKE_CURRENT_BINARY_DIR})

# Find kernel headers
list(APPEND CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}/cmake")
list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_LIST_DIR}/cmake")

set(KERNELHEADERS_DIR /usr/src/linux-headers-5.4.0-1008-raspi)

set(KERNELHEADERS_INCLUDE_DIRS
  ${KERNELHEADERS_DIR}/include
  CACHE PATH "Kernel headers include dirs"
)

message( "Found Kernel DIR" )
message( ${KERNELHEADERS_DIR} )

message( "Found Kernel Include DIR" )
message( ${KERNELHEADERS_DIR}/include )

include_directories(${KERNELHEADERS_INCLUDE_DIRS})

set(DRIVER_FILE vocalfusion-soundcard.ko)
set(KBUILD_CMD $(MAKE) -C ${KERNELHEADERS_DIR} modules M=${CMAKE_CURRENT_BINARY_DIR} src=${CMAKE_CURRENT_SOURCE_DIR})

FILE(WRITE ${CMAKE_CURRENT_SOURCE_DIR}/Kbuild "obj-m := vocalfusion-soundcard.o")

# Generate the Kbuild file through cmake.

add_custom_command(OUTPUT ${DRIVER_FILE}
        COMMAND ${KBUILD_CMD}
        WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
        DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/vocalfusion-soundcard.c VERBATIM)

add_custom_target(vocal-fusion-driver ALL DEPENDS ${DRIVER_FILE})

install(FILES vocalfusion-soundcard.ko DESTINATION /lib/modules)
install(FILES xvf3510.dtbo DESTINATION /boot/overlays/)
install(FILES xvf3510.dts DESTINATION /boot/overlays/)
