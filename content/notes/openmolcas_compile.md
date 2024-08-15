---
title: "Compiling OpenMolcas"
date: 2024-02-16
draft: false
author: "Matthew R. Hennefarth"
---

## General Remarks

For whatever reason, compiling OpenMolcas seems daunting, especially for those which are not familiar with cmake. This is especially true when once considers all of the various configuration options at your disposal (OpenMP, MPI, GCC, Intel, Math libraries, etc.). Here I hope to give some general guidance on how to compile OpenMolcas on a few systems with some different parameters/options. At the end, I will list how I build OpenMolcas on the UChicago cluster (mainly so I can refer back to it).

For building these projects, I also like to generally separate my builds from the final binaries. This will ensure that if I have calculations runnings, I don't corrupt them, and than failed builds do not in any way interfere with the already build and running binaries. As such, I always will recommend one constructs both a `bin` and `build` directory and to use the cmake `--DCMAKE_INSTALL_PREFIX=../bin` option.

### Global Requirements

- Python3
- pyparsing
- Fortran Compiler
- C Compiler
- HDF5 (optional, but should be installed)

To successfully compile OpenMolcas, you will need a suitable version of python3 as well as the `pyparsing` python module. The latter can be installed via pip as 

```sh 
pip3 install pyparsing
```

Be mindful that if you are installing OpenMolcas on a cluster, this package might already be installed. And if it is not, you will have to install it locally (using the `-user` flag). 

In addition, you will need a Fortran and C compiler. If you are compiling for Intel CPUs, I would recommend the Intel compilers. For AMD CPUs, the AMD compilers, and for MacOS/generic, the GCC compilers. Irregardless, the GCC compilers recommend a great starting point for compiling OpenMolcas (though it might not squeeze out every ounce of performance). 

### Recompiling After Merging

OpenMolcas is continuously being developed and as such it is good to update every once and awhile (though of course, maybe not being projects). As such, one can merge in the newest changes using (assuming you are not modifying your local copy). 

```sh 
git pull
```

Then, one will need to rebuild the entire project. However, often times, you can get away with just re-running the `make` command and hoping that cmake has properly cached all of your variables. However, sometimes there are slightly breaking changes and one will need to run `make clean`, and this will delete all source files and recompile everything from scratch, but retaining the cmake variables cached. Finally, there is the nuclear option where one deletes the `build` directory and restarts (hence why I like to move the binaries to the `bin` directory so the nuclear option doesn't affect them!)

## MacOS Build

There are some necessary prerequisite software to install. I will assume you have [homebrew](https://brew.sh/) installed (and if you don't, you should). You will then need the following software:

- gcc compilers
- cmake
- hdf5

```sh
brew install cmake gcc hdf5
```

### Serial Compilation

We start with a simple example of trying to build OpenMolcas on a Mac using only OpenMP (shared memory threads). This differs from MPI builds for which the processes can be distributed over various CPUs (each thread has its own memory). As always, we start with cloning the OpenMolcas repo.

```sh
git clone https://gitlab.com/Molcas/OpenMolcas/
```

Next, we make our build directory and go to that.

```sh 
cd OpenMolcas
mkdir build
cd build
```

Next, we will instruct cmake to configure the project and to use the Accelerate math library.

```sh 
cmake ../ -DCMAKE_INSTALL_PREFIX=../bin -DCMAKE_BUILD_TYPE=Release -DLINALG=Accelerate
```

Note, that if you are having issues with OpenMolcas/cmake finding the right python executable and hence pyparsing package, you can specify the python executable to use by passing the `-DPython_EXECUTABLE=<path>` flag. For example, to use the `python3.11` executable, which might be installed by Homebrew, one would use the following command instead:

```sh 
cmake ../ -DCMAKE_INSTALL_PREFIX=../bin -DCMAKE_BUILD_TYPE=Release -DLINALG=Accelerate -DPython_EXECUTABLE=/opt/homebrew/bin/python3.11
```

If the above is successful, we can then build using `make`. Optionally (since this will take quite some time), we can use multiple threads to speed up the compilation using the `-j <N>` flag where `N` is the number of threads to use. For example, to compile using 4 threads, we would now issue the following command:

```sh 
make -j 4
make install
```

The `make install` will move all of the binaries to the `OpenMolcas/bin` directory. One could of course change where this directory points to. 

### OpenMP Compilation

This compilation will enable the usage of multiple shared-memory threads in the linear algebra routines. This is actually fairly straightforward to do and can be accomplished by adding the `-DOPENMP=ON` to the cmake command.

```sh 
cmake ../ -DOPENMP=ON -DCMAKE_INSTALL_PREFIX=../bin -DCMAKE_BUILD_TYPE=Release -DLINALG=Accelerate -DPython_EXECUTABLE=/opt/homebrew/bin/python3.11
```

The number of threads at runtime is then set with the `OMP_NUM_THREADS` environment variable.

### MPI/GA Compilation

One of the main challanges of compiling OpenMolcas with MPI and Global Arrays on a Mac is two fold. Firstly, Global Arrays can only be used with the Intel or OpenBLAS math libraries. Secondly, OpenMolcas can only use the OpenBLAS libraries which are compiled with 64 bit integers. Unfortunately, the OpenBLAS in homebrew is compiled with 32 bit integers meaning the only way to do this successfully is to compile our own OpenBLAS with the proper settings. One could of course use the internal linear algebra library provided with OpenMolcas, but that is not very efficient. 

In general, I am not sure of any one is using a Mac to run production level OpenMolcas calculations. In general, I am running OpenMolcas on a Mac for development and debugging purposes. Of course, for the more advanced users, it should be possible to compile OpenBLAS and then successfully compile OpenMolcas as well!

### Compilation for Development

The following cmake command has been useful in developing in OpenMolcas. It firstly builds the project in debug mode, but secondly, it turns on `BIGOT` which adds `-Werror` and a few other flags.

```sh
cmake ../ -DCMAKE_BUILD_TYPE=Debug -DBOUNDS=On -DLINALG=Accelerate -DBIGOT=On -DPython_EXECUTABLE=/opt/homebrew/bin/python3.11 -DCMAKE_C_COMPILER=gcc-14
```
