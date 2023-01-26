cmake_minimum_required(VERSION 3.9)
project(ratracer)

set(LIBS "-Wl,--gc-sections")
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -O3 -g -std=c++17 -fopenmp -Wall -Wextra -Wfatal-errors -pipe -fno-omit-frame-pointer -fdata-sections -ffunction-sections")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -O3 -g -std=c++17 -fopenmp -Wall -Wextra -Wfatal-errors -pipe -fno-omit-frame-pointer -fdata-sections -ffunction-sections")

find_package(PkgConfig REQUIRED)

pkg_check_modules(MPFR REQUIRED mpfr)
set(LIBS "${LIBS} ${MPFR_LDFLAGS}")
#include_directories("${MPFR_INCLUDE_DIRS}")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${MPFR_CFLAGS}")
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${MPFR_CFLAGS}")

pkg_check_modules(FIREFLY REQUIRED firefly)
set(LIBS "${LIBS} ${FIREFLY_LDFLAGS}")
include_directories("${FIREFLY_INCLUDE_DIRS}")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${FIREFLY_CFLAGS}")
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${FIREFLY_CFLAGS}")

string(FIND "${LIBS}" "flint" FLINT_POS)
if ("${FLINT_POS}" STREQUAL "-1")
  message(FATAL_ERROR "FireFly was not built with Flint enabled.")
endif()

include_directories("${CMAKE_CURRENT_SOURCE_DIR}")

add_executable(ratracer "${CMAKE_CURRENT_SOURCE_DIR}/ratracer.cpp")
add_dependencies(ratracer primes)
target_link_libraries(ratracer ${LIBS})

add_custom_target(check COMMAND "${CMAKE_CURRENT_SOURCE_DIR}/check" DEPENDS ratracer  WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR})
add_custom_target(bench COMMAND "${CMAKE_CURRENT_SOURCE_DIR}/bench" DEPENDS ratracer WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR})
add_custom_command(OUTPUT ${CMAKE_CURRENT_SOURCE_DIR}/primes.h COMMAND ${CMAKE_CURRENT_SOURCE_DIR}/mkprimes &> ${CMAKE_CURRENT_SOURCE_DIR}/primes.h WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR} DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/mkprimes)
add_custom_target(primes DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/primes.h)

set(INCLUDE_INSTALL_DIR "${INSTALLATION_PREFIX}/include/ratracer")
set(BIN_INSTALL_DIR "${INSTALLATION_PREFIX}/bin")
set(LIB_INSTALL_DIR "${INSTALLATION_PREFIX}/lib")

set(HEADERS
  ${CMAKE_CURRENT_SOURCE_DIR}/primes.h
  ${CMAKE_CURRENT_SOURCE_DIR}/ratbox.h
  ${CMAKE_CURRENT_SOURCE_DIR}/ratracer.h
)

install(TARGETS ratracer DESTINATION ${BIN_INSTALL_DIR})
install(FILES ${HEADERS} DESTINATION ${INCLUDE_INSTALL_DIR})
install(PROGRAMS ${CMAKE_CURRENT_SOURCE_DIR}/tools/guessfactors DESTINATION ${BIN_INSTALL_DIR})