project(Turi)

message("Building openmp using native build system.")

set(_openmp_dir openmp-src)

set(ON_X86 ON CACHE BOOL "build on x86 architecture")

if(${TC_BUILD_IOS}) 
  set(ON_X86 OFF)
  foreach(variable ${CMAKE_OSX_ARCHITECTURES})
    if(variable STREQUAL "arm64")
      set(LIBOMP_ARCH aarch64)
      break()
    endif() # let OMP decide it self
  endforeach(variable)
  check_cxx_compiler_flag(-no-integrated-as COMPILER_SUPPORTS_NO_ASM)
endif()


if (COMPILER_SUPPORTS_NO_ASM AND ${TC_BUILD_IOS})
  set(CMAKE_C_FLAGS "-integrated-as ${CMAKE_C_FLAGS}")
  set(CMAKE_CXX_FLAGS "-integrated-as ${CMAKE_CXX_FLAGS}")
endif()

# custom settings
set(OPENMP_STANDALONE_BUILD TRUE CACHE BOOL "")
set(LIBOMP_ENABLE_SHARED OFF CACHE BOOL "")
set(LIBOMP_USE_ADAPTIVE_LOCKS ${ON_X86} CACHE BOOL "")
set(OPENMP_ENABLE_LIBOMPTARGET OFF CACHE BOOL "")
set(LIBOMP_COPY_EXPORTS FALSE CACHE STRING "")


# Set some flags based on build_type
set(RELEASE_BUILD FALSE)
set(DEBUG_BUILD FALSE)
set(RELWITHDEBINFO_BUILD FALSE)
set(MINSIZEREL_BUILD FALSE)
string(TOLOWER "${CMAKE_BUILD_TYPE}" libomp_build_type_lowercase)
if("${libomp_build_type_lowercase}" STREQUAL "release")
  set(RELEASE_BUILD TRUE)
elseif("${libomp_build_type_lowercase}" STREQUAL "debug")
  set(DEBUG_BUILD TRUE)
elseif("${libomp_build_type_lowercase}" STREQUAL "relwithdebinfo")
  set(RELWITHDEBINFO_BUILD TRUE)
elseif("${libomp_build_type_lowercase}" STREQUAL "minsizerel")
  set(MINSIZEREL_BUILD TRUE)
endif()


# from libompExports.cmake
set(my_libomp_suffix)
macro(libomp_append flags flag)
  if(${ARGV2})
    list(APPEND ${flags} "${flag}")
  endif()
endmacro()
libomp_append(my_libomp_suffix .deb DEBUG_BUILD)
libomp_append(my_libomp_suffix .dia RELWITHDEBINFO_BUILD)
libomp_append(my_libomp_suffix .min MINSIZEREL_BUILD)
libomp_append(my_libomp_suffix .s1 LIBOMP_STATS)

set(LIBOMP_EXPORTS_DIR "${_openmp_dir}/runtime/exports")
set(LIBOMP_EXPORTS_CMN_DIR "${LIBOMP_EXPORTS_DIR}/common${libomp_suffix}/include")

# for <crt_extern.h>
include_directories(${CMAKE_CURRENT_SOURCE_DIR})

# build openmp
add_subdirectory(${_openmp_dir})

target_include_directories(omp
  INTERFACE ${CMAKE_CURRENT_BINARY_DIR}/${LIBOMP_EXPORTS_CMN_DIR})

