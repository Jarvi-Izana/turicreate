cmake_minimum_required(VERSION 2.8 FATAL_ERROR)

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
endif()


add_definitions(-DCFLAGS="-fPIC ${C_REAL_COMPILER_FLAGS}")
add_definitions(-DCPPFLAGS="-fPIC ${CPP_REAL_COMPILER_FLAGS}")

# custom settings
set(OPENMP_STANDALONE_BUILD TRUE CACHE BOOL "")
set(LIBOMP_ENABLE_SHARED OFF CACHE BOOL "")
set(LIBOMP_USE_ADAPTIVE_LOCKS ${ON_X86} CACHE BOOL "")
set(OPENMP_ENABLE_LIBOMPTARGET OFF CACHE BOOL "")

include_directories(${CMAKE_CURRENT_SOURCE_DIR})

add_subdirectory(${_openmp_dir})

target_include_directories(omp INTERFACE ${CMAKE_CURRENT_SOURCE_DIR})
