---
title: "Large scale builds using cmake"
description: "meta description"
date: 2024-08-06T05:00:00Z
image: "/images/posts/01.jpg"
categories: ["kubernetes", "service mesh"]
authors: ["Balaji Balasundaram"]
tags: ["kubernetes", "k8s", "linkerd", "service mesh"]
draft: false
---

When dealing with a large number of libraries and sources, it’s essential to optimize the build process to save time and resources. Here are a few strategies you can employ:

1. **Pre-Build Libraries**

If some of your libraries are relatively stable and don't change often, you can pre-build these libraries and link against the pre-built binaries. This way, you only need to build these libraries once and not every time you build your project.

2. Use External Projects

Instead of downloading and building libraries as part of your main project, you can use ExternalProject_Add to handle them separately. This allows better management and caching of the build process.

3. Parallel Builds

Ensure you use parallel builds to leverage all your CPU cores. This can be achieved by using `cmake --build . -- -jN` where N is the number of jobs (typically, the number of CPU cores).

4. Incremental Builds

CMake is designed to handle incremental builds well, but you can ensure that changes in dependencies don't trigger unnecessary rebuilds by carefully managing dependencies and targets.

5. CCache

Using ccache can significantly speed up rebuilds by caching previous compilations and reusing them if the same source code hasn't changed.

#### Example CMakeLists.txt Using ExternalProject_Add
Here’s how you can use ExternalProject_Add to manage external dependencies separately:

```cmake
cmake_minimum_required(VERSION 3.10)

# Project name
project(MyProject)

include(ExternalProject)

# External projects
ExternalProject_Add(
  cli
  GIT_REPOSITORY https://github.com/daniele77/cli.git
  GIT_TAG main
  UPDATE_COMMAND ""
  INSTALL_COMMAND ""
)

ExternalProject_Add(
  another_project
  GIT_REPOSITORY https://github.com/username/another_project.git
  GIT_TAG main
  UPDATE_COMMAND ""
  INSTALL_COMMAND ""
)

# Add other external projects similarly...

# Find libsodium
find_package(PkgConfig REQUIRED)
pkg_check_modules(SODIUM REQUIRED libsodium)

# Set the source files
set(SRCS main.cpp)

# Add the executable
add_executable(main WIN32 ${SRCS} main.exe.manifest)

# Link external projects' libraries
add_dependencies(main cli another_project)  # Add other dependencies as necessary

ExternalProject_Get_Property(cli source_dir binary_dir)
target_include_directories(main PRIVATE ${source_dir}/include)
target_link_libraries(main PRIVATE ${binary_dir}/libcli.a)  # Adjust as necessary

ExternalProject_Get_Property(another_project source_dir binary_dir)
target_include_directories(main PRIVATE ${source_dir}/include)
target_link_libraries(main PRIVATE ${binary_dir}/libanother_project.a)  # Adjust as necessary

# Link other libraries and include directories
target_link_libraries(main PRIVATE ${SODIUM_LIBRARIES})
target_include_directories(main PRIVATE ${SODIUM_INCLUDE_DIRS})

# Parallel build
set(CMAKE_BUILD_PARALLEL_LEVEL 4)  # Adjust based on your CPU cores
```
##### Additional Notes:
- *Parallel Building*: Make sure to build your project with parallel jobs using `cmake --build . -- -jN`, where N is the number of CPU cores.
- *CCache*: To use ccache, ensure it is installed and set up in your environment. You can add it to your CMake configuration like this:
```cmake
set(CMAKE_CXX_COMPILER_LAUNCHER ccache)
set(CMAKE_C_COMPILER_LAUNCHER ccache)
```
- *Pre-Building and Caching*: For libraries that are stable, consider building them once, creating packages, and using those packages in your project. This avoids recompiling the same libraries repeatedly.

These strategies can significantly reduce the build time for large projects.