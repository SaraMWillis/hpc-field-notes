# Amber

## Amber 26

### Installation

[Download Source Code](https://ambermd.org/GetAmber.php){ .md-button .md-button--primary }  [User Guide](https://ambermd.org/doc12/Amber26.pdf){ .md-button .md-button--primary }

After getting the software, I unpacked it:

```bash
tar xvf ambertools26.tar.bz2 
tar vxf pmemd26.tar.bz2
```

I installed AmberTools first. For running the update, I had to set a different timeout value, otherwise I got a whole bunch of Python URL timeout errors:

```bash
cd ambertools26_src/
./update_amber --update --timeout=60
```

Creating the installation directory

```bash
mkdir ../AMBER26
export AMBERHOME=/path/to/AMBER26/
export PMEMDHOME=/path/to/AMBER26/
export InstallRoot=$AMBERHOME
```

Loading the relevant software modules:

```bash
module load cuda12/12.4.1
module load cuda12-dnn/9.3.0 
```

Setting up and executing the install:

!!! warning "Turn off updates"
    If updates are enabled, the install will fail with a timeout error

```bash
cmake .. -DCMAKE_INSTALL_PREFIX=${InstallRoot} -DCOMPILER=GNU -DMPI=TRUE -DOPENMP=TRUE -DINSTALL_TESTS=TRUE -DDOWNLOAD_MINICONDA=TRUE -DCUDA=TRUE -DCHECK_UPDATES=FALSE -DAPPLY_UPDATES=FALSE
make -j8
make install
```

For the PMEMD install I again needed to set a different timeout value to avoid `urllib.error.URLError: <urlopen error _ssl.c:1130: The handshake operation timed out>`

```bash
./update_pmemd --update --timeout=600
```

Created the installation directory:

```bash
mkdir ../AMBER26
export AMBERHOME=/path/to/AMBER26/
export PMEMDHOME=$AMBERHOME
export InstallRoot=$AMBERHOME
```

and set the relevant environment variables to point to the AmberTools install:

```bash
export PATH=${AMBERHOME}/bin:${PATH}
export LD_LIBRARY_PATH=${AMBERHOME}/lib:${LD_LIBRARY_PATH}
export PKG_CONFIG_PATH=${AMBERHOME}/lib/pkgconfig:${PKG_CONFIG_PATH}
export CPPFLAGS="-I${AMBERHOME}/include ${CPPFLAGS}"
export LDFLAGS="-L${AMBERHOME}/lib ${LDFLAGS}
```

I also needed to set `NCCL_HOME` to point to the relevant install location to allow for inter-GPU communication.

```
export NCCL_HOME=/opt/ohpc/pub/apps/nvidia-hpc-sdk/24.5/Linux_x86_64/24.5/comm_libs/12.4/nccl
```




To actually get the thing to install, I wound up having to modify `pmemd26_src/cmake/3rdPartyTools.cmake` to add `nccl` as a third party tool (change highlighted below):

```cmake hl_lines="12"
set(3RDPARTY_TOOLS
blas
lapack
netcdf
netcdf-fortran
zlib
libbz2
kmmd
libm
mkl
plumed
nccl)
```

and additionally had to add a descriptor for it, otherwise cmake would fail at the last part of its setup with an index error (change highlighted below):

```cmake hl_lines="12"
set(3RDPARTY_TOOL_USES
"for fundamental linear algebra calculations"
"for fundamental linear algebra calculations"
"for creating trajectory data files"
"for creating trajectory data files from Fortran"
"for various compression and decompression tasks"
"for various compression and decompression tasks"
"Machine-learning molecular dynamics"
"for fundamental math routines if they are not contained in the C library"
"for BLAS/LAPACK/FFTW on Intel hardware"
"tools for enchanced sampling, free energies, and trajectory analysis"
"for inter-GPU communications")
```

Once that change was made:

```bash
mkdir _build && cd _build
cmake .. \
    -DCMAKE_INSTALL_PREFIX=$InstallRoot \
    -DCOMPILER=GNU  \
    -DMPI=TRUE -DCUDA=TRUE -DNCCL=TRUE -DPGM=FALSE -DINSTALL_TESTS=TRUE \
    -DDOWNLOAD_MINICONDA=FALSE -DBUILD_PYTHON=FALSE \
    -DBUILD_PERL=FALSE -DBUILD_GUI=FALSE \
    -DPMEMD_ONLY=TRUE -DCHECK_UPDATES=FALSE -DAPPLY_UPDATES=FALSE
make -j8
make install
```