cmake_minimum_required(VERSION 3.10)

project(maj2random)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_POSITION_INDEPENDENT_CODE ON)

TRY_RUN(
  cpuident_result
  cpuident_compile_result
  ${CMAKE_CURRENT_BINARY_DIR}/
  ${CMAKE_CURRENT_SOURCE_DIR}/src/cpuident.c
  COMPILE_OUTPUT_VARIABLE test_compile_output
  RUN_OUTPUT_VARIABLE cpuident_output
)
foreach(opt_name_value IN ITEMS ${cpuident_output})
  string(REGEX MATCH "^[^=]+" opt_name ${opt_name_value})
  string(REPLACE "${opt_name}=" "" opt_value ${opt_name_value})
  cmake_language(EVAL CODE "SET(${opt_name} ${opt_value})")
endforeach()

message("cpuident: x86_sse4_2:${x86_sse4_2} x86_avx:${x86_avx} x86_avx2:${x86_avx2} x86_avx512f:${x86_avx512f}")

include(CheckCXXCompilerFlag)
check_cxx_compiler_flag("-march=skylake-avx512" has_skylake_avx512 "int main() { return 0; }")

include(CheckLibraryExists)
check_library_exists(m sqrtf "" HAVE_LIB_M)
if (HAVE_LIB_M)
    set(EXTRA_LIBS ${EXTRA_LIBS} m)
endif (HAVE_LIB_M)

add_executable(bench_maj2 src/bench_maj2.cc)
add_executable(test_maj2 src/test_maj2.c)
target_link_libraries(test_maj2 ${EXTRA_LIBS})

if(x86_avx512f AND has_skylake_avx512)
  set_source_files_properties(src/bench_maj2.cc PROPERTIES COMPILE_FLAGS "-O3 -march=skylake-avx512")
  set_source_files_properties(src/test_maj2.c PROPERTIES COMPILE_FLAGS "-O3 -march=skylake-avx512")
endif()
