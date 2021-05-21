cmake_minimum_required(VERSION 3.12...3.20)
set(CMAKE_CXX_STANDARD 20)
include(FetchContent)
set(PROJECT_NAME "testname")
project(${PROJECT_NAME})
include(ProcessorCount)

set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
option(PACKAGE_TESTS "Build the tests" OFF)
option(PACKAGE_TOOLS "Build the tools" ON)
option(USE_CLANG_TIDY "Use Clang Tidy" OFF)
option(CI_CLANG_FORMAT_FLAGS "Use CI Clang Format Flags" OFF)

add_subdirectory(${${PROJECT_NAME}_SOURCE_DIR}/libs)

if(PACKAGE_TESTS)
  enable_testing()
  include(GoogleTest)
  add_subdirectory(${${PROJECT_NAME}_SOURCE_DIR}/tests)
endif()

if(PACKAGE_TOOLS)
  enable_testing()
  add_subdirectory(${${PROJECT_NAME}_SOURCE_DIR}/tools)
endif()

add_library(${PROJECT_NAME} SHARED)

set_target_properties(${PROJECT_NAME} PROPERTIES PREFIX "")

ProcessorCount(N)
if(NOT N EQUAL 0)
  set(CTEST_BUILD_FLAGS -j${N})
  set(ctest_test_args ${ctest_test_args} PARALLEL_LEVEL ${N})
endif()

find_program(clang_format_path NAMES clang-format-10 clang-format)
if (USE_CLANG_TIDY)
  find_program(clang_tidy_path NAMES clang-tidy-10 clang-tidy clang-tidy-11 REQUIRED)
  set(CMAKE_CXX_CLANG_TIDY
    clang-tidy;
  )
else()
  find_program(clang_tidy_path NAMES clang-tidy-10 clang-tidy clang-tidy-11)
endif()

if(CMAKE_CXX_COMPILER_ID STREQUAL "Clang")
  target_compile_options(${PROJECT_NAME} PRIVATE -pedantic -Wall -Wmost -Wextra)
elseif(CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
  target_compile_options(${PROJECT_NAME} PRIVATE -pedantic -Wall -Wextra)
endif()

add_subdirectory(${${PROJECT_NAME}_SOURCE_DIR}/include)
add_subdirectory(${${PROJECT_NAME}_SOURCE_DIR}/src)

get_target_property(${PROJECT_NAME}_sources ${PROJECT_NAME} SOURCES)

target_link_libraries(${PROJECT_NAME} PRIVATE ${CMAKE_DL_LIBS})

target_include_directories(${PROJECT_NAME} PUBLIC "${${PROJECT_NAME}_SOURCE_DIR}/include")

add_custom_target(
  check
  COMMAND ${CMAKE_CTEST_COMMAND} --output-on-failure
  VERBATIM
)

find_program(iwyu_path NAMES include-what-you-use iwyu)
if(iwyu_path)
  set(iwyu_path_and_options
    ${iwyu_path}
    -Xiwyu
    --no_comments 
    -Xiwyu
    --cxx17ns
  )
  set_property(TARGET ${PROJECT_NAME} PROPERTY CXX_INCLUDE_WHAT_YOU_USE ${iwyu_path_and_options})
endif()

if (clang_tidy_path)
  add_custom_target(
    tidy
    COMMAND clang-tidy
      --quiet --warnings-as-errors=* -p ${${PROJECT_NAME}_BINARY_DIR} ${${PROJECT_NAME}_sources}
    WORKING_DIRECTORY ${${PROJECT_NAME}_SOURCE_DIR}
    VERBATIM
  )
endif()

if (clang_format_path)
  if(CI_CLANG_FORMAT_FLAGS)
    set(CLANG_FORMAT_FLAGS 
      --verbose;
      --Werror;
      -i;
      -n;
    )
  else()
    set(CLANG_FORMAT_FLAGS
      -i;
    )
  endif()

  add_custom_target(
    format
    ALL
    COMMAND ${clang_format_path} ${CLANG_FORMAT_FLAGS}
    ${${PROJECT_NAME}_sources}
    WORKING_DIRECTORY ${${PROJECT_NAME}_SOURCE_DIR}
    VERBATIM
  )

endif()
