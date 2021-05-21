find_package(PkgConfig REQUIRED)

set(DIR ${CMAKE_CURRENT_LIST_DIR})

add_executable(tools "")

if(CMAKE_CXX_COMPILER_ID STREQUAL "Clang")
target_compile_options(tools PRIVATE -pedantic -Wall -Wmost -Wextra)
elseif(CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
  target_compile_options(tools PRIVATE -pedantic -Wall -Wextra)
endif()

target_sources(tools
    PRIVATE
    "${DIR}/tool.cpp"
)
