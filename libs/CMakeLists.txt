FetchContent_Declare(
  googletest
  GIT_REPOSITORY https://github.com/google/googletest.git
  GIT_TAG        aa9b44a18678dfdf57089a5ac22c1edb69f35da5 
)
# using a gtest commit hash instead of tag to avoid the cmake
# deprecation warning from release 1.10 will replace when
# a new release tag is created

FetchContent_GetProperties(googletest)
if(NOT googletest_POPULATED)
  FetchContent_Populate(googletest)
  add_subdirectory(${googletest_SOURCE_DIR} ${googletest_BINARY_DIR})
endif()
