include_directories(${PROJECT_SOURCE_DIR})
include_directories("${PROJECT_SOURCE_DIR}/include")

add_library(secp256k1 secp256k1.c)

if (TESTS)
  add_executable(tests tests.c)
  if (NOT COVERAGE)
    target_compile_definitions(tests PRIVATE VERIFY)
  endif()
endif()

if (EXHAUSTIVE_TESTS)
  add_executable(exhaustive_tests tests_exhaustive.c)
  if (NOT COVERAGE)
    target_compile_definitions(exhaustive_tests PRIVATE VERIFY)
  endif()
endif()

if (BENCHMARK)
  add_executable(bench_verify bench_verify.c)
  set_target_properties(bench_verify PROPERTIES LINK_FLAGS libsecp256k1.a)
  add_dependencies(bench_verify secp256k1)
  add_executable(bench_sign bench_sign.c)
  set_target_properties(bench_sign PROPERTIES LINK_FLAGS libsecp256k1.a)
  add_dependencies(bench_sign secp256k1)
  add_executable(bench_internal bench_internal.c)
  add_executable(bench_ecmult bench_ecmult.c)
endif()

if (ECMULT_STATIC_PRECOMPUTATION)
  add_executable(precomputation gen_context.c)
  # alternatives considered:
  # add_file_dependencies, add_dependencies, add_custom_command
  add_custom_target(precomputed_table
    COMMAND "${PROJECT_SOURCE_DIR}/precomputation"
    BYPRODUCTS "${PROJECT_SOURCE_DIR}/src/ecmult_static_context.h"
    DEPENDS precomputation
    WORKING_DIRECTORY ${PROJECT_SOURCE_DIR}
  )
  add_dependencies(secp256k1 precomputed_table)
  if (TESTS)
    add_dependencies(tests precomputed_table)
  endif()
  if (EXHAUSTIVE_TESTS)
    add_dependencies(exhaustive_tests precomputed_table)
  endif()
  if (BENCHMARK)
    add_dependencies(bench_internal precomputed_table)
    add_dependencies(bench_ecmult precomputed_table)
  endif()
endif()
