# Lab 5 

## Step 1 and 2

tutorial.cxx:
```C++
// A simple program that computes the square root of a number
#include <cmath>
#include <cstdlib>
#include <iostream>
#include <string>

int main(int argc, char* argv[])
{
  if (argc < 2) {
    std::cout << "Usage: " << argv[0] << " number" << std::endl;
    return 1;
  }

  double inputValue = std::stod(argv[1]);

  double outputValue = sqrt(inputValue);
#ifdef USE_MYMATH
    double outputValue = mysqrt(inputValue);
  #else
    double outputValue = sqrt(inputValue);
  #endif
  std::cout << "The square root of " << inputValue << " is " << outputValue
            << std::endl;
  return 0;
}
```
CMakelists.txt
```
cmake_minimum_required(VERSION 3.3)
project(Tutorial)

set(CMAKE_CXX_STANDARD 14)
# the version number.
set(Tutorial_VERSION_MAJOR 1)
set(Tutorial_VERSION_MINOR 0)

# configure a header file to pass some of the CMake settings
# to the source code
configure_file(
  "${PROJECT_SOURCE_DIR}/TutorialConfig.h.in"
  "${PROJECT_BINARY_DIR}/TutorialConfig.h"
  )

# add the executable
add_executable(Tutorial tutorial.cxx)

# add the binary tree to the search path for include files
# so that we will find TutorialConfig.h
target_include_directories(Tutorial PUBLIC
                           "${PROJECT_BINARY_DIR}"
                           ) 
```

## Step 3

CMakeLists.txt
```
cmake_minimum_required(VERSION 3.3)
project(Tutorial)

set(CMAKE_CXX_STANDARD 14)

# should we use our own math functions
option(USE_MYMATH "Use tutorial provided math implementation" ON)

# the version number.
set(Tutorial_VERSION_MAJOR 1)
set(Tutorial_VERSION_MINOR 0)

# configure a header file to pass some of the CMake settings
# to the source code
configure_file(
  "${PROJECT_SOURCE_DIR}/TutorialConfig.h.in"
  "${PROJECT_BINARY_DIR}/TutorialConfig.h"
  )

# add the MathFunctions library?
if(USE_MYMATH)
  add_subdirectory(MathFunctions)
  list(APPEND EXTRA_LIBS MathFunctions)
  #list(APPEND EXTRA_INCLUDES "${PROJECT_SOURCE_DIR}/MathFunctions")
endif(USE_MYMATH)

# add the executable
add_executable(Tutorial tutorial.cxx)

target_link_libraries(Tutorial ${EXTRA_LIBS})

# add the binary tree to the search path for include files
# so that we will find TutorialConfig.h
target_include_directories(Tutorial PUBLIC
                           "${PROJECT_BINARY_DIR}" 
			   )
```
MathFunctions/CMakelists.txt
```
add_library(MathFunctions mysqrt.cxx)

# state that anybody linking to us needs to include the current source dir
# to find MathFunctions.h, while we don't.
target_include_directories(MathFunctions
          INTERFACE ${CMAKE_CURRENT_SOURCE_DIR}
          )
```

## Step 4 

CMakeLists.txt

```
cmake_minimum_required(VERSION 3.3)
project(Tutorial)

set(CMAKE_CXX_STANDARD 14)

# should we use our own math functions
option(USE_MYMATH "Use tutorial provided math implementation" ON)

# the version number.
set(Tutorial_VERSION_MAJOR 1)
set(Tutorial_VERSION_MINOR 0)


# configure a header file to pass some of the CMake settings
# to the source code
configure_file(
  "${PROJECT_SOURCE_DIR}/TutorialConfig.h.in"
  "${PROJECT_BINARY_DIR}/TutorialConfig.h"
  )

# add the MathFunctions library?
if(USE_MYMATH)
  add_subdirectory(MathFunctions)
  list(APPEND EXTRA_LIBS MathFunctions)
endif(USE_MYMATH)

# add the executable
add_executable(Tutorial tutorial.cxx)

target_link_libraries(Tutorial PUBLIC ${EXTRA_LIBS})

# add the binary tree to the search path for include files
# so that we will find TutorialConfig.h
target_include_directories(Tutorial PUBLIC
                           "${PROJECT_BINARY_DIR}"
                           )

install(TARGETS Tutorial DESTINATION bin)
install(FILES "${PROJECT_BINARY_DIR}/TutorialConfig.h"
        DESTINATION include
        )

# enable testing
  enable_testing()

  # does the application run
  add_test(NAME Runs COMMAND Tutorial 25)

  # does the usage message work?
  add_test(NAME Usage COMMAND Tutorial)
  set_tests_properties(Usage
    PROPERTIES PASS_REGULAR_EXPRESSION "Usage:.*number"
    )

  # define a function to simplify adding tests
  function(do_test target arg result)
    add_test(NAME Comp${arg} COMMAND ${target} ${arg})
    set_tests_properties(Comp${arg}
      PROPERTIES PASS_REGULAR_EXPRESSION ${result}
      )
  endfunction(do_test)

  # do a bunch of result based tests
  do_test(Tutorial 25 "25 is 5")
  do_test(Tutorial -25 "-25 is [-nan|nan|0]")
  do_test(Tutorial 0.0001 "0.0001 is 0.01")

```
MathFunctions/CMakeLists.txt

```
add_library(MathFunctions mysqrt.cxx)



# state that anybody linking to us needs to include the current source dir
# to find MathFunctions.h, while we don't.
target_include_directories(MathFunctions
          INTERFACE ${CMAKE_CURRENT_SOURCE_DIR}
          )

install (TARGETS MathFunctions DESTINATION bin)
install (FILES MathFunctions.h DESTINATION include)
```

## Step 5

CMakeLists.txt

```
cmake_minimum_required(VERSION 3.3)
project(Tutorial)

set(CMAKE_CXX_STANDARD 14)

# should we use our own math functions
option(USE_MYMATH "Use tutorial provided math implementation" ON)

# the version number.
set(Tutorial_VERSION_MAJOR 1)
set(Tutorial_VERSION_MINOR 0)

# configure a header file to pass some of the CMake settings
# to the source code
configure_file(
  "${PROJECT_SOURCE_DIR}/TutorialConfig.h.in"
  "${PROJECT_BINARY_DIR}/TutorialConfig.h"
  )

# add the MathFunctions library?
if(USE_MYMATH)
  add_subdirectory(MathFunctions)
  list(APPEND EXTRA_LIBS MathFunctions)
endif()

# add the executable
add_executable(Tutorial tutorial.cxx)
target_link_libraries(Tutorial PUBLIC ${EXTRA_LIBS})

# add the binary tree to the search path for include files
# so that we will find TutorialConfig.h
target_include_directories(Tutorial PUBLIC
                           "${PROJECT_BINARY_DIR}"
                           )

# add the install targets
install(TARGETS Tutorial DESTINATION bin)
install(FILES "${PROJECT_BINARY_DIR}/TutorialConfig.h"
  DESTINATION include
  )

# enable testing
enable_testing()

# does the application run
add_test(NAME Runs COMMAND Tutorial 25)

# does the usage message work?
add_test(NAME Usage COMMAND Tutorial)
set_tests_properties(Usage
  PROPERTIES PASS_REGULAR_EXPRESSION "Usage:.*number"
  )

# define a function to simplify adding tests
function(do_test target arg result)
  add_test(NAME Comp${arg} COMMAND ${target} ${arg})
  set_tests_properties(Comp${arg}
    PROPERTIES PASS_REGULAR_EXPRESSION ${result}
    )
endfunction(do_test)

# do a bunch of result based tests
do_test(Tutorial 4 "4 is 2")
do_test(Tutorial 9 "9 is 3")
do_test(Tutorial 5 "5 is 2.236")
do_test(Tutorial 7 "7 is 2.645")
do_test(Tutorial 25 "25 is 5")
do_test(Tutorial -25 "-25 is [-nan|nan|0]")
do_test(Tutorial 0.0001 "0.0001 is 0.01")


# does this system provide the log and exp functions?
  include(CheckSymbolExists)
  set(CMAKE_REQUIRED_LIBRARIES "m")
  check_symbol_exists(log "math.h" HAVE_LOG)
  check_symbol_exists(exp "math.h" HAVE_EXP)
```
MathFunctions/CMakeLists.txt
```
add_library(MathFunctions mysqrt.cxx)

target_include_directories(MathFunctions
    INTERFACE ${CMAKE_CURRENT_SOURCE_DIR}
    PRIVATE ${Tutorial_BINARY_DIR}
    )

install(TARGETS MathFunctions DESTINATION lib)
install(FILES MathFunctions.h DESTINATION include)
```

## Lab-Example

### CMakeLists:
cmake_minimum_required(VERSION 3.3)
project(block)


set(CMAKE_CXX_STANDARD 14)


add_executable(dynamic_block block.c)
add_executable(static_block block.c)

install(TARGETS dynamic_block DESTINATION build)
install(TARGETS static_block DESTINATION build)

### Makefile:

all: dynamic_block static_block
dynamic_block: dynamic_block.o lib.so
	cc dynamic_block.o lib.so -o dynamic_block -W1, -rpath='$$ORIGIN'	
static_block: static_block.o lib.so
	cc static_block.o lib.so -o static_block -W1, -rpath='$$ORIGIN'

dynamic_block.o: ../source/block.c
	cc -c ../source/block.c -o dynamic_block.o

static_block.o: ../source/block.c
	cc -c ../source/block.c -o static_block.o
static_block.o: ../headers/block.h
dynamic_block.o: ../headers/block.h

### Cmake Makefile:

`
# CMAKE generated file: DO NOT EDIT!
# Generated by "Unix Makefiles" Generator, CMake Version 3.10

# Default target executed when no arguments are given to make.
default_target: all

.PHONY : default_target

# The main recursive all target
all:

.PHONY : all

# The main recursive preinstall target
preinstall:

.PHONY : preinstall

#=============================================================================
# Special targets provided by cmake.

# Disable implicit rules so canonical targets will work.
.SUFFIXES:


# Remove some rules from gmake that .SUFFIXES does not remove.
SUFFIXES =

.SUFFIXES: .hpux_make_needs_suffix_list


# Suppress display of executed commands.
$(VERBOSE).SILENT:


# A target that is always out of date.
cmake_force:

.PHONY : cmake_force

#=============================================================================
# Set environment variables for the build.

# The shell in which to execute make rules.
SHELL = /bin/sh

# The CMake executable.
CMAKE_COMMAND = /usr/bin/cmake

# The command to remove a file.
RM = /usr/bin/cmake -E remove -f

# Escaping for special characters.
EQUALS = =

# The top-level source directory on which CMake was run.
CMAKE_SOURCE_DIR = /home/oaklea/Documents/DevStuff/gitRepos/CSCI-49XX-OpenSource/Modules/BuildSystems/Lab-Example/source

# The top-level build directory on which CMake was run.
CMAKE_BINARY_DIR = /home/oaklea/Documents/DevStuff/gitRepos/CSCI-49XX-OpenSource/Modules/BuildSystems/Lab-Example/build

#=============================================================================
# Target rules for target CMakeFiles/dynamic_block.dir

# All Build rule for target.
CMakeFiles/dynamic_block.dir/all:
	$(MAKE) -f CMakeFiles/dynamic_block.dir/build.make CMakeFiles/dynamic_block.dir/depend
	$(MAKE) -f CMakeFiles/dynamic_block.dir/build.make CMakeFiles/dynamic_block.dir/build
	@$(CMAKE_COMMAND) -E cmake_echo_color --switch=$(COLOR) --progress-dir=/home/oaklea/Documents/DevStuff/gitRepos/CSCI-49XX-OpenSource/Modules/BuildSystems/Lab-Example/build/CMakeFiles --progress-num=1,2 "Built target dynamic_block"
.PHONY : CMakeFiles/dynamic_block.dir/all

# Include target in all.
all: CMakeFiles/dynamic_block.dir/all

.PHONY : all

# Build rule for subdir invocation for target.
CMakeFiles/dynamic_block.dir/rule: cmake_check_build_system
	$(CMAKE_COMMAND) -E cmake_progress_start /home/oaklea/Documents/DevStuff/gitRepos/CSCI-49XX-OpenSource/Modules/BuildSystems/Lab-Example/build/CMakeFiles 2
	$(MAKE) -f CMakeFiles/Makefile2 CMakeFiles/dynamic_block.dir/all
	$(CMAKE_COMMAND) -E cmake_progress_start /home/oaklea/Documents/DevStuff/gitRepos/CSCI-49XX-OpenSource/Modules/BuildSystems/Lab-Example/build/CMakeFiles 0
.PHONY : CMakeFiles/dynamic_block.dir/rule

# Convenience name for target.
dynamic_block: CMakeFiles/dynamic_block.dir/rule

.PHONY : dynamic_block

# clean rule for target.
CMakeFiles/dynamic_block.dir/clean:
	$(MAKE) -f CMakeFiles/dynamic_block.dir/build.make CMakeFiles/dynamic_block.dir/clean
.PHONY : CMakeFiles/dynamic_block.dir/clean

# clean rule for target.
clean: CMakeFiles/dynamic_block.dir/clean

.PHONY : clean

#=============================================================================
# Target rules for target CMakeFiles/static_block.dir

# All Build rule for target.
CMakeFiles/static_block.dir/all:
	$(MAKE) -f CMakeFiles/static_block.dir/build.make CMakeFiles/static_block.dir/depend
	$(MAKE) -f CMakeFiles/static_block.dir/build.make CMakeFiles/static_block.dir/build
	@$(CMAKE_COMMAND) -E cmake_echo_color --switch=$(COLOR) --progress-dir=/home/oaklea/Documents/DevStuff/gitRepos/CSCI-49XX-OpenSource/Modules/BuildSystems/Lab-Example/build/CMakeFiles --progress-num=3,4 "Built target static_block"
.PHONY : CMakeFiles/static_block.dir/all

# Include target in all.
all: CMakeFiles/static_block.dir/all

.PHONY : all

# Build rule for subdir invocation for target.
CMakeFiles/static_block.dir/rule: cmake_check_build_system
	$(CMAKE_COMMAND) -E cmake_progress_start /home/oaklea/Documents/DevStuff/gitRepos/CSCI-49XX-OpenSource/Modules/BuildSystems/Lab-Example/build/CMakeFiles 2
	$(MAKE) -f CMakeFiles/Makefile2 CMakeFiles/static_block.dir/all
	$(CMAKE_COMMAND) -E cmake_progress_start /home/oaklea/Documents/DevStuff/gitRepos/CSCI-49XX-OpenSource/Modules/BuildSystems/Lab-Example/build/CMakeFiles 0
.PHONY : CMakeFiles/static_block.dir/rule

# Convenience name for target.
static_block: CMakeFiles/static_block.dir/rule

.PHONY : static_block

# clean rule for target.
CMakeFiles/static_block.dir/clean:
	$(MAKE) -f CMakeFiles/static_block.dir/build.make CMakeFiles/static_block.dir/clean
.PHONY : CMakeFiles/static_block.dir/clean

# clean rule for target.
clean: CMakeFiles/static_block.dir/clean

.PHONY : clean

#=============================================================================
# Special targets to cleanup operation of make.

# Special rule to run CMake to check the build system integrity.
# No rule that depends on this can have commands that come from listfiles
# because they might be regenerated.
cmake_check_build_system:
	$(CMAKE_COMMAND) -H$(CMAKE_SOURCE_DIR) -B$(CMAKE_BINARY_DIR) --check-build-system CMakeFiles/Makefile.cmake 0
.PHONY : cmake_check_build_system

`

Own makefile is much smaller than CMake file




	



