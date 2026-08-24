# Installations

## cmake crud


Notes on debugging an installation of [AxiSEM3D](https://github.com/AxiSEMunity/AxiSEM3D)

```console
(axisem3d) [lookitsme@r7u02n1 AxiSEM3D]$ rm -rf build && cmake -B build
-- The C compiler identification is GNU 15.3.0
-- The CXX compiler identification is GNU 15.3.0
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Check for working C compiler: /groups/lookitsme/SOFTWARE/micromamba/envs/axisem3d/bin/cc - skipped
-- Detecting C compile features
-- Detecting C compile features - done
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Check for working CXX compiler: /groups/lookitsme/SOFTWARE/micromamba/envs/axisem3d/bin/c++ - skipped
-- Detecting CXX compile features
-- Detecting CXX compile features - done
-- Could NOT find MPI_CXX (missing: MPI_CXX_WORKS) 
CMake Error at /groups/lookitsme/SOFTWARE/micromamba/envs/axisem3d/share/cmake-4.2/Modules/FindPackageHandleStandardArgs.cmake:290 (message):
  Could NOT find MPI (missing: MPI_CXX_FOUND CXX)
Call Stack (most recent call first):
  /groups/lookitsme/SOFTWARE/micromamba/envs/axisem3d/share/cmake-4.2/Modules/FindPackageHandleStandardArgs.cmake:654 (_FPHSA_FAILURE_MESSAGE)
  /groups/lookitsme/SOFTWARE/micromamba/envs/axisem3d/share/cmake-4.2/Modules/FindMPI.cmake:2006 (find_package_handle_standard_args)
  CMakeLists.txt:115 (find_package)
  (axisem3d) [lookitsme@r7u02n1 AxiSEM3D]$ which mpicxx
/groups/lookitsme/SOFTWARE/micromamba/envs/axisem3d/bin/mpicxx
```

The issue is due to library mismatches between the system-installed MPI and the MPI installed in the environment. Figured this out by:

  ```console
  (axisem3d) [lookitsme@r7u02n1 build]$ cmake --debug-trycompile ..
```

Found a bunch of build attempts here:

`/xdisk/lookitsme/TICKETS/user/AxiSEM3D/build/CMakeFiles/CMakeScratch/`

```
(axisem3d) [lookitsme@r7u02n1 build]$ cd /xdisk/lookitsme/TICKETS/user/AxiSEM3D/build/CMakeFiles/CMakeScratch/TryCompile-5WCSgN 
(axisem3d) [lookitsme@r7u02n1 TryCompile-5WCSgN]$ make VERBOSE=1
```

which produced

```
/groups/lookitsme/SOFTWARE/micromamba/envs/axisem3d/bin/x86_64-conda-linux-gnu-ld: warning: libxml2.so.2, needed by /opt/ohpc/pub/libs/hwloc/lib/libhwloc.so.15, not found (try using -rpath or -rpath-link)
```

Doing a `module purge` to get all module MPI and UCX executables and libraries out of the environment fixed the build. The software eventually ran with:

```
UCX_TLS=tcp,self,sm mpiexec -n 2 ./axisem3d --help
```