cmake_minimum_required(VERSION 3.31)
set(CMAKE_PROJECT_TOP_LEVEL_INCLUDES .devcontainer/cmake/xproinc.cmake)
project(argon2)
set(targetsFile ${PROJECT_NAME}-targets)
set(lib_name ${PROJECT_NAME})
if(DEFINED XP_NAMESPACE)
  xpExternPackage(NAMESPACE ${XP_NAMESPACE}
    TARGETS_FILE ${targetsFile} LIBRARIES ${lib_name}
    BASE 20210625 XPDIFF "intro" FIND_THREADS
    WEB "https://www.password-hashing.net" UPSTREAM "github.com/P-H-C/phc-winner-argon2"
    DESC "The reference C implementation of the Argon2, the password-hashing function that won the Password Hashing Competition (PHC)"
    LICENSE "[CC0-1.0 or Apache-2.0](https://github.com/P-H-C/phc-winner-argon2/blob/master/LICENSE 'dual licensed under Creative Commons Zero v1.0 Universal and Apache License, Version 2.0')"
    )
  set(nameSpace NAMESPACE ${XP_NAMESPACE}::)
elseif(NOT DEFINED CMAKE_INSTALL_CMAKEDIR)
  set(CMAKE_INSTALL_CMAKEDIR ${CMAKE_INSTALL_DATADIR}/cmake)
endif()
########################################
set(include_srcs
  include/argon2.h
  )
source_group(include FILES ${include_srcs})
list(APPEND libsrcs ${include_srcs})
####################
set(src_srcs
  src/argon2.c
  src/core.c
  src/core.h
  src/encoding.c
  src/encoding.h
  src/ref.c
  src/thread.c
  src/thread.h
  )
if(CMAKE_SYSTEM_PROCESSOR MATCHES "^(x86_64|amd64|i[3-6]86|x86)$")
  list(APPEND src_srcs
    src/opt.c
    )
endif()
source_group(src FILES ${src_srcs})
list(APPEND libsrcs ${src_srcs})
####################
set(src_blake2_srcs
  src/blake2/blake2-impl.h
  src/blake2/blake2.h
  src/blake2/blake2b.c
  src/blake2/blamka-round-opt.h
  src/blake2/blamka-round-ref.h
  )
source_group(src\\blake2 FILES ${src_blake2_srcs})
list(APPEND libsrcs ${src_blake2_srcs})
####################
set(src_bench
  src/bench.c
  )
list(APPEND exes bench)
####################
set(src_genkat
  src/genkat.c
  src/genkat.h
  )
list(APPEND exes genkat)
####################
set(src_run
  src/run.c
  )
list(APPEND exes run)
####################
set(src_testx
  src/test.c
  )
list(APPEND exes testx)
########################################
add_library(${lib_name} STATIC ${libsrcs})
target_include_directories(${lib_name} PUBLIC $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
  )
find_package(Threads REQUIRED)
target_link_libraries(${lib_name} PRIVATE Threads::Threads)
########################################
if(CMAKE_SYSTEM_PROCESSOR MATCHES "^(aarch64|arm64|ARM64)$")
  list(REMOVE_ITEM exes bench)
endif()
foreach(exe ${exes})
  add_executable(${exe} ${src_${exe}})
  target_link_libraries(${exe} PRIVATE ${lib_name})
endforeach()
include(CTest)
add_test(NAME testx COMMAND testx)
########################################
install(TARGETS ${lib_name} EXPORT ${targetsFile}
  RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
  LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
  ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
  )
install(FILES ${include_srcs} DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/${PROJECT_NAME})
install(EXPORT ${targetsFile} DESTINATION ${CMAKE_INSTALL_CMAKEDIR} ${nameSpace})
install(FILES CHANGELOG.md LICENSE README.md DESTINATION ${CMAKE_INSTALL_DATADIR})
