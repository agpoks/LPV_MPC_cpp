
LPV_MPC_cpp
===========

LPV_MPC_cpp is a C++ implementation of Linear Parameter-Varying Model Predictive Control (LPV-MPC) using CasADi for automatic differentiation and IPOPT for nonlinear optimization. For visualization, matplotlib-cpp is used.

Dependencies
------------

This project depends on:

- CasADi (compiled from source)
- IPOPT (for solving NLP problems)
- GCC/G++ 11 (required for modern C++ features)
- SWIG, LAPACK
- Python3, NumPy, matplotlib
- matplotlib-cpp for plotting

### 1. Install System Dependencies
------------------------------

```bash
sudo apt update
sudo apt install -y build-essential cmake git \
  coinor-libipopt-dev swig gfortran liblapack-dev \
  pkg-config manpages-dev software-properties-common \
  python3 python3-dev python3-numpy python3-matplotlib
```

### 2. Install GCC/G++ 11
---------------------

```bash
sudo add-apt-repository ppa:ubuntu-toolchain-r/test
sudo apt update
sudo apt install -y gcc-11 g++-11

sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-11 110
sudo update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-11 110

# Verify
gcc --version
g++ --version
```

### 3. Build CasADi with IPOPT Support
----------------------------------

```bash
cd ~
git clone https://github.com/casadi/casadi.git -b main
cd casadi
mkdir build
cd build

cmake -DWITH_PYTHON=ON \
      -DWITH_IPOPT=ON \
      -DWITH_OPENMP=ON \
      -DWITH_THREAD=ON ..

make -j$(nproc)
sudo make install
```

### 4. Example: CasADi C++ Project Setup
------------------------------------

Use the following tutorial project to verify your CasADi C++ build and environment:

```bash
cd ~
git clone https://github.com/zehuilu/Tutorial-on-CasADi-with-CPP.git
cd Tutorial-on-CasADi-with-CPP
mkdir build code_gen
cd build
cmake ..
make
./example_racecar
```

### 5. matplotlib-cpp Setup for C++ Plotting
----------------------------------------

 Install Python Dependencies

```bash
sudo apt install python3-matplotlib python3-dev python3-numpy
```

Download the Header File

```bash
mkdir -p include
curl -o include/matplotlibcpp.h https://raw.githubusercontent.com/lava/matplotlib-cpp/master/matplotlibcpp.h
```

add  '''#include"matplotlibcpp.h"''' to the project.

```bash
Change the following part in 'matplotlibcpp.h'
PyTuple_SetItem(args, 0, PyFloat_FromDouble(nrows));
PyTuple_SetItem(args, 1, PyFloat_FromDouble(ncols));
PyTuple_SetItem(args, 2, PyFloat_FromDouble(plot_number));
```
to:
```bash
PyTuple_SetItem(args, 0, PyLong_FromDouble(nrows));
PyTuple_SetItem(args, 1, PyLong_FromDouble(ncols));
PyTuple_SetItem(args, 2, PyLong_FromDouble(plot_number));
```


 Test matplotlib in Python:

```bash
python3 -c "import matplotlib.pyplot as plt; plt.plot([1, 2, 3]); plt.show()"
```

### 6. CMake Configuration (CasADi, IPOPT, Python/NumPy)
----------------------------------------------------

Below is a template CMakeLists.txt you can adapt to your project. Replace <your_project_name> and <your_executable_name> with your own.

```cmake
cmake_minimum_required(VERSION 3.14)
project(<your_project_name>)

# Set compilers (ensure GCC 11 is installed and set up)
set(CMAKE_C_COMPILER "/usr/bin/gcc")
set(CMAKE_CXX_COMPILER "/usr/bin/g++")

# Build type and flags
if(NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE Release)
endif()

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_POSITION_INDEPENDENT_CODE ON)

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wextra -pthread")
set(CMAKE_CXX_FLAGS_DEBUG "-g")
set(CMAKE_CXX_FLAGS_RELEASE "-O3")

# CasADi and IPOPT paths
set(LIBRARY_DIRS /usr/local/lib)

if(UNIX AND NOT APPLE)
    set(IPOPT_INCLUDE_DIRS /usr/include/coin)
elseif(APPLE)
    set(IPOPT_INCLUDE_DIRS /usr/local/include/coin-or)
endif()

set(CASADI_INCLUDE_DIR /usr/local/include/casadi)
find_library(CASADI_LIBRARY NAMES casadi HINTS ${CASADI_INCLUDE_DIR}/../lib $ENV{CASADI_PREFIX}/lib)
if(CASADI_LIBRARY)
    set(CASADI_LIBRARIES ${CASADI_LIBRARIES} ${CASADI_LIBRARY})
endif()

# Threads
find_package(Threads REQUIRED)

# Python and NumPy (needed for matplotlib-cpp)
find_package(Python COMPONENTS Interpreter Development NumPy REQUIRED)

# Source files (adjust as needed)
file(GLOB HEADER_FILES_HPP ${CMAKE_SOURCE_DIR}/include/*.hpp)
file(GLOB HEADER_FILES_H ${CMAKE_SOURCE_DIR}/include/*.h)
file(GLOB SOURCE_FILES ${CMAKE_SOURCE_DIR}/src/*.cpp)

# Executable (rename as needed)
add_executable(<your_executable_name>
    ${SOURCE_FILES}
    <optional_specific_file>.cpp
)

# Include directories
target_include_directories(<your_executable_name> PRIVATE
    ${CMAKE_SOURCE_DIR}/include
    ${CASADI_INCLUDE_DIR}
    ${IPOPT_INCLUDE_DIRS}
    ${Python_INCLUDE_DIRS}
    ${Python_NumPy_INCLUDE_DIRS}
)

# Link libraries
target_link_libraries(<your_executable_name> PRIVATE
    ${CASADI_LIBRARIES}
    ipopt
    Threads::Threads
    Python::Python
    Python::NumPy
)
```
'' <optional_specific_file>.cpp'' can be main.cpp and mpc.cpp

------------

## Building the Project

```bash
mkdir build
cd build
cmake ..
make
```

This will build your LPV-MPC or any other CasADi-based C++ project.
