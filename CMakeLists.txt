cmake_minimum_required(VERSION 3.00)
project(grlib_demo)

# Specify build type
if(NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE "Release")
endif()

set(CMAKE_BUILD_TYPE "Release" CACHE STRING "Select build type")
set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS "Debug" "Release" "RelWithDebInfo" "MinSizeRel")

set(TIVAWARE_PATH "/mnt/c/ti/TivaWare_C_Series-2.2.0.295/")

# Path to the TivaWare C SDK folder
# option(TIVAWARE_PATH "Path to TivaWare C Series SDK folder")
if(NOT TIVAWARE_PATH)
  message(FATAL_ERROR "You must specify TIVAWARE_PATH variable!")
endif()

include_directories(${TIVAWARE_PATH})
include_directories(${TIVAWARE_PATH}examples/boards/ek-tm4c123gxl-boostxl-kentec-s1/)

# Project source files
file(GLOB SOURCES *.c *.h)
file(GLOB DRIVERS ${TIVAWARE_PATH}examples/boards/ek-tm4c123gxl-boostxl-kentec-s1/drivers/*.c ${TIVAWARE_PATH}examples/boards/ek-tm4c123gxl-boostxl-kentec-s1/drivers/*.h)
file(GLOB UTILS ${TIVAWARE_PATH}utils/ustdlib.c ${TIVAWARE_PATH}utils/ustdlib.h)

set(LINKER_SCRIPT ${CMAKE_CURRENT_LIST_DIR}/grlib_demo.ld)


# Add the project
add_executable(${PROJECT_NAME} ${SOURCES} ${DRIVERS} ${UTILS})

# Include the TivaWare C SDK
target_include_directories(${PROJECT_NAME} PUBLIC ${TIVAWARE_PATH})

# Platform specific flags
target_compile_options(${PROJECT_NAME} PRIVATE 
 -mthumb -mcpu=cortex-m4 -mfpu=fpv4-sp-d16 -mfloat-abi=hard -ffunction-sections -fdata-sections -MD -Wall -pedantic
)

# Platform specific definitions
target_compile_definitions(${PROJECT_NAME} PRIVATE
 -DPART_TM4C123GH6PM
 -DTARGET_IS_TM4C123_RA1
 -Dgcc
)

# Find the compiler specific libraries
get_target_property(COMPILE_FLAGS ${PROJECT_NAME} COMPILE_OPTIONS)
execute_process(COMMAND ${CMAKE_C_COMPILER} ${COMPILE_FLAGS} -print-libgcc-file-name OUTPUT_VARIABLE LIBGCC_PATH)
execute_process(COMMAND ${CMAKE_C_COMPILER} ${COMPILE_FLAGS} -print-file-name=libc.a OUTPUT_VARIABLE LIBC_PATH)
execute_process(COMMAND ${CMAKE_C_COMPILER} ${COMPILE_FLAGS} -print-file-name=libm.a OUTPUT_VARIABLE LIBM_PATH)
execute_process(COMMAND ${CMAKE_C_COMPILER} ${COMPILE_FLAGS} -print-file-name=libstdc++.a OUTPUT_VARIABLE LIBSTDCPP_PATH)
string(STRIP "${LIBGCC_PATH}" LIBGCC_PATH)
string(STRIP "${LIBSTDCPP_PATH}" LIBSTDCPP_PATH)
string(STRIP "${LIBC_PATH}" LIBC_PATH)
string(STRIP "${LIBM_PATH}" LIBM_PATH)

# Platform specific link definitions
target_link_options(${PROJECT_NAME} PRIVATE
  -T ${LINKER_SCRIPT} --entry ResetISR --gc-sections
)

# Include other libraries
target_link_libraries(${PROJECT_NAME} PUBLIC
  ${TIVAWARE_PATH}grlib/gcc/libgr.a
  ${TIVAWARE_PATH}/driverlib/gcc/libdriver.a
  ${LIBM_PATH}
  ${LIBC_PATH}
  ${LIBGCC_PATH}
  ${LIBSTDCPP_PATH}
)

set(FLASH_EXECUTABLE "lm4flash")
ADD_CUSTOM_TARGET("flash"
  # DEPENDS ${CMAKE_PROJECT_NAME}.axf 
  #COMMAND ${CMAKE_OBJCOPY} -O binary ${CMAKE_PROJECT_NAME}.axf ${CMAKE_PROJECT_NAME}.bin 
  COMMAND ${FLASH_EXECUTABLE} ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}.bin
)

# Objcopy to create a binary file
add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD COMMAND 
  ${CMAKE_OBJCOPY} -O binary $<TARGET_FILE:${PROJECT_NAME}>.axf 
  ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}.bin
)

# Enable C++14 and C11
set_property(TARGET ${PROJECT_NAME} PROPERTY CXX_STANDARD 14)
set_property(TARGET ${PROJECT_NAME} PROPERTY C_STANDARD 11)

