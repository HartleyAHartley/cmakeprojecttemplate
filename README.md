# A Cmake project template for C++ work
I keep finding myself reusing the same cmake setup for projects. So, now a template repo.

Note: this template is for a C++ library not executable, and the tools folder sets up a executable project.

# Development

## Getting Started

Install the following dependencies:

- `git`

- `cmake`

- `clang` (or [clang-11](https://packages.ubuntu.com/bionic/clang-11))

Clone the repository:

```
git clone git@github.com:realliance/cmakeprojecttemplate.git
```

Run the following commands within the project directory (note that these commands are for ):
```
# Bootstrap CMake Environment
cmake -S . -B build -G Ninja

# Build Project
cmake --build build
```

## Testing and Documentation

The test suite is located in `tests/` and can be run with `make -C build check`
