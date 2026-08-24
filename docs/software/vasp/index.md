# VASP

## Installation

### 6.3.2

The makefile.include for VASP 6.3.2 compiled with Intel on Rocky Linux 9 

```makefile title="makefile.include"
# Default precompiler options
CPP_OPTIONS = -DHOST=\"LinuxIFC\" \
              -DMPI -DMPI_BLOCK=8000 -Duse_collective \
              -DscaLAPACK \
              -DCACHE_SIZE=4000 \
              -Davoidalloc \
              -Dvasp6 \
              -Duse_bse_te \
              -Dtbdyn \
              -Dfock_dblbuf \

CPP         = fpp -f_com=no -free -w0  $*$(FUFFIX) $*$(SUFFIX) $(CPP_OPTIONS)

FC          = mpiifort
FCL         = mpiifort

FREE        = -free -names lowercase

FFLAGS      = -assume byterecl -w

OFLAG       = -O2
OFLAG_IN    = $(OFLAG)
DEBUG       = -O0

# For what used to be vasp.5.lib
CPP_LIB     = $(CPP)
FC_LIB      = $(FC)
CC_LIB      = icc
CFLAGS_LIB  = -O
FFLAGS_LIB  = -O1
FREE_LIB    = $(FREE)

OBJECTS_LIB = linpack_double.o

# For the parser library
CXX_PARS    = icpx
LLIBS       = -lstdc++

##
## Customize as of this point! Of course you may change the preceding
## part of this file as well if you like, but it should rarely be
## necessary ...
##

# When compiling on the target machine itself, change this to the
# relevant target when cross-compiling for another architecture
VASP_TARGET_CPU ?= -assume byterecl -w -avx2
FFLAGS     += $(VASP_TARGET_CPU)
 
# Intel MKL for FFTW, BLAS, LAPACK, and scaLAPACK
# (Note: for Intel Parallel Studio's MKL use -mkl instead of -qmkl)
FCL        += -qmkl
MKLROOT    ?= /opt/ohpc/pub/compiler/intel/oneapi/mkl/2024.0
INCS        =-I$(MKLROOT)/include/fftw

# Use a separate scaLAPACK installation (optional but recommended in combination with OpenMPI)
# Comment out the two lines below if you want to use scaLAPACK from MKL instead
SCALAPACK_ROOT ?= /opt/ohpc/pub/compiler/intel/oneapi/2024.0/
LLIBS      += -L${SCALAPACK_ROOT}/lib -lmkl_scalapack_lp64 -lmkl_blacs_intelmpi_lp64 -lmkl_intel_lp64 -lmkl_intel_thread

# HDF5-support (optional but strongly recommended, and mandatory for some features)
CPP_OPTIONS+= -DVASP_HDF5
HDF5_ROOT  ?= /opt/ohpc/pub/libs/intel/hdf5/1.14.6/
LLIBS      += -L$(HDF5_ROOT)/lib -lhdf5_fortran
INCS       += -I$(HDF5_ROOT)/include

# For the VASP-2-Wannier90 interface (optional)
#CPP_OPTIONS    += -DVASP2WANNIER90
#WANNIER90_ROOT ?= /path/to/your/wannier90/installation
#LLIBS          += -L$(WANNIER90_ROOT)/lib -lwannier

# For the fftlib library (hardly any benefit in combination with MKL's FFTs)
#CPP_OPTIONS+= -Dsysv
#FCL         = mpif90 fftlib.o -qmkl
#CXX_FFTLIB  = icpc -qopenmp -std=c++11 -DFFTLIB_USE_MKL -DFFTLIB_THREADSAFE
#INCS_FFTLIB = -I./include -I$(MKLROOT)/include/fftw
#LIBS       += fftlib

# For machine learning library vaspml (experimental)
#CPP_OPTIONS += -Dlibvaspml
#CPP_OPTIONS += -DVASPML_USE_CBLAS
#CPP_OPTIONS += -DVASPML_USE_MKL
#CPP_OPTIONS += -DVASPML_DEBUG_LEVEL=3
#CXX_ML      = mpic++ -qopenmp
#CXXFLAGS_ML = -O3 -std=c++17 -Wall
#INCLUDE_ML  =
```
