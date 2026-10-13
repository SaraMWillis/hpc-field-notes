# Ansys

## Apptainer Recipe Files

### Ansys 19.0 

Subtitle: EOL software dependent on an EOL OS built on an unsupported OS

!!! note
    Replace `<PORT>` and `<LICENSE_SERVER>` with the license server information appropriate for your environment. Replace `</PATH/TO/Ansys19.0>` with the path to your local Ansys 19.0 installation files. 

```
Bootstrap: docker
From: ubuntu:18.04

%files
  </PATH/TO/Ansys19.0> /Ansys19.0

%post

  mount -t proc proc /proc || echo "WARNING: /proc mount failed, continuing anyway"
  echo "=== /proc check ==="
  if [ -d /proc/self ]; then
    echo "/proc/self exists - mount likely succeeded"
    cat /proc/self/status | head -5
  else
    echo "/proc/self MISSING - mount did not succeed"
  fi
  echo "===================="

  export DEBIAN_FRONTEND=noninteractive

  apt update -y
  apt install python3 libglib2.0-0 libgl1-mesa-glx libldap-2.4-2 openssh-client rpm locales-all libxm4 wget -y
  apt-get update -y 
  apt-get install libsm6 libxrender1 libfontconfig1 libxmu6 libgomp1 -y  
  apt-get install -y \
  libgl1-mesa-glx \
  libgl1-mesa-dri \
  libglu1-mesa \
  libxrender1 \
  libxext6 \
  libxi6 \
  libxtst6 \
  libx11-6
  apt-get install -y mesa-utils -y
  wget -c  http://archive.ubuntu.com/ubuntu/pool/main/g/glibc/multiarch-support_2.27-3ubuntu1.6_amd64.deb
  wget -c  http://archive.debian.org/debian/pool/main/libx/libxp/libxp6_1.0.2-2_amd64.deb
  apt-get install ./multiarch-support_2.27-3ubuntu1.6_amd64.deb ./libxp6_1.0.2-2_amd64.deb
  export LANG=en_US.UTF-8
  export LC_ALL=en_US.UTF-8

  cd /Ansys19.0
  mkdir /INSTALL
  ./INSTALL -silent -install_dir /INSTALL -licserverinfo <PORT>:<LICENSE_SERVER>

  echo ANSYSProductImprovementProgram=off > /INSTALL/v190/commonfiles/globalsettings/ANSYSProductImprovementProgram.txt

  echo "=== Checking install log for MainWin/regsvr32/locale errors ==="
  grep -iE "regsvr32.*failed|locale|proc.*not.*mounted|rpm: command not found" \
    $(find /INSTALL -iname "*install*.log" 2>/dev/null) 2>/dev/null || echo "No matching log lines found (or log not found at expected path - adjust find path if needed)"
  echo "================================================================"

  unlink /bin/sh
  ln -s /bin/bash /bin/sh

  rm -rf /Ansys19.0
  apt-get clean
  rm -rf /var/lib/apt/lists/*
  mkdir -p /tmp /var/tmp
  mv /INSTALL/v190/aisol/lib/linx64/libz.so.1 /INSTALL/v190/aisol/lib/linx64/libz.so.1.old
  ln -s /lib/x86_64-linux-gnu/libz.so.1 /INSTALL/v190/aisol/lib/linx64/libz.so.1

%environment
  export LANG=en_US.UTF-8
  export LC_ALL=en_US.UTF-8
  export ANSYS_BASE=/INSTALL
  export PATH=${ANSYS_BASE}/v190/ansys/bin:${ANSYS_BASE}/v190/aisol/bin:${ANSYS_BASE}/v190/Framework/bin/Linux64:${ANSYS_BASE}/v190/ACP:${ANSYS_BASE}/v190/autodyn/bin:${ANSYS_BASE}/v190/CFD-Post/bin:${ANSYS_BASE}/v190/CFX/bin:${ANSYS_BASE}/v190/CEI/bin:${ANSYS_BASE}/v190/EKM/bin:${ANSYS_BASE}/v190/RSM/bin:${ANSYS_BASE}/v190/SystemCoupling/bin:${ANSYS_BASE}/v190/fensapice/bin:${ANSYS_BASE}/v190/fluent/bin:${ANSYS_BASE}/v190/icemcfd/linux64_amd/bin:${ANSYS_BASE}/v190/Icepak/bin:${ANSYS_BASE}/v190/polyflow/bin:${ANSYS_BASE}/v190/TurboGrid/bin:${PATH}
  export PKG_CONFIG_PATH=/INSTALL/v190/CFX/tools/xerces-3.2.2/linx64/lib/pkgconfig:/INSTALL/v190/tp/zlib/1_2_11/linx64/lib/pkgconfig:/INSTALL/v190/tp/zlib/1_2_11/linx64_ZPREFIX/lib/pkgconfig:/INSTALL/v190/tp/qt/5.9.6/linx64/lib/pkgconfig:/INSTALL/v190/CEI/apex195/machines/linux_2.6_64/Mesa-13.0.5/lib/pkgconfig:/INSTALL/v190/CEI/apex195/machines/linux_2.6_64/Python-2.7.15/lib/pkgconfig:/INSTALL/v190/CEI/apex195/machines/linux_2.6_64/qt-5.10.1/lib/pkgconfig:/INSTALL/v190/CFD-Post/tools/xerces-3.2.2/linx64/lib/pkgconfig:/INSTALL/v190/Tools/mono/Linux64/lib/pkgconfig
  export ANSYSLMD_LICENSE_FILE=<PORT>@<LICENSE_SERVER>
  export LIBGL_ALWAYS_SOFTWARE=1
  export AWP_ROOT190=/INSTALL/v190
  export ANSYS_ROOT=$AWP_ROOT190
```

## Runtime Issues

### Fluent

!!! tip "Ansys MPI"
    Note: Ansys comes with it's own prepackaged MPI. As part of this process, I unloaded our OpenMPI module (which is loaded by default) to ensure there were no library mismatches.

Below is an example of an MPI issue a user ran into when running ANSYS Fluent 2025 R1 on our servers:

```console
[user@<node> ANSYS]$ fluent 2ddp -g -t1 -i test.jou
</path/to/fluent>/bin/fluent -r25.1.0 2ddp -g -t1 -i test.jou
</path/to/fluent>/cortex/lnamd64/cortex.25.1.0 -f fluent -g -i test.jou (fluent "2ddp -pdefault  -host -r25.1.0 -t1 -mpi=intel -path</path/to/ansys>/25.1/v251/fluent -ssh")

Opening input/output transcript to file "</path/to>/ANSYS/<output>.trn".
Auto-Transcript Start Time:  10:47:11, 13 Oct 2026 
</path/to/fluent>/bin/fluent -r25.1.0 2ddp -pdefault -host -t1 -mpi=intel -path</path/to/ansys>/25.1/v251/fluent -ssh -cx <node>:<port>:<port>
Starting </path/to/fluent>/lnamd64/2ddp_host/fluent.25.1.0 host -cx node:<port>:<port> "(list (rpsetvar (QUOTE parallel/function) "fluent 2ddp -flux -node -r25.1.0 -t1 -pdefault -mpi=intel -ssh") (rpsetvar (QUOTE parallel/rhost) "") (rpsetvar (QUOTE parallel/ruser) "") (rpsetvar (QUOTE parallel/nprocs_string) "1") (rpsetvar (QUOTE parallel/auto-spawn?) #t) (rpsetvar (QUOTE parallel/trace-level) 0) (rpsetvar (QUOTE parallel/remote-shell) 1) (rpsetvar (QUOTE parallel/path) "</path/to/ansys/25.1/v251/fluent") (rpsetvar (QUOTE parallel/hostsfile) "") (rpsetvar (QUOTE gpuapp/devices) ""))"

              Welcome to ANSYS Fluent 2025 R1

              Copyright 1987-2025 ANSYS, Inc. All Rights Reserved.
              Unauthorized use, distribution or duplication is prohibited.
              This product is subject to U.S. laws governing export and re-export.
              For full Legal Notice, see documentation.

Build Time: Nov 26 2024 15:35:47 EST  Build Id: 10210  
 
Connected License Server List:	<port>@<lic.server>

*********************************************
Info: Your license enables 4-way parallel execution.
For faster simulations, please start the application with the appropriate parallel options.
*********************************************


     --------------------------------------------------------------
     This is an academic version of ANSYS FLUENT. Usage of this product
     license is limited to the terms and conditions specified in your ANSYS
     license form, additional terms section.
     --------------------------------------------------------------
Host spawning Node 0 on machine "<node>" (unix).
</path/to/fluent>/bin/fluent -r25.1.0 2ddp -flux -node -t1 -pdefault -mpi=intel -ssh -mport <internal_ip>:<internal_ip>:<port>:0
Starting </path/to/fluent>/multiport/mpi/lnamd64/intel2021/bin/mpirun -f /tmp/fluent-appfile.<user>.<pid> -genv I_MPI_FABRICS shm -genv FLUENT_ARCH lnamd64 -genv I_MPI_DEBUG 0 -genv I_MPI_ADJUST_GATHERV 3 -genv I_MPI_ADJUST_ALLREDUCE 2 -genv I_MPI_PLATFORM auto -genv PYTHONHOME </path/to/fluent>/../../commonfiles/CPython/3_10/linx64/Release/python -genv FLUENT_PROD_DIR </path/to/fluent>/fluent25.1.0 -genv FLUENT_AFFINITY 0 -genv I_MPI_PIN enable -genv KMP_AFFINITY disabled -n 1 </path/to/fluent>/lnamd64/2ddp_node/fluent_mpi.25.1.0 node -mpiw intel -pic shmem -mport <internal_ip>:<internal_ip>:<port>:0
[mpiexec@<node>] Error: Unable to run bstrap_proxy on <node> (pid <pid>, exit code 256)
[mpiexec@<node>] poll_for_event (../../../../../src/pm/i_hydra/libhydra/demux/hydra_demux_poll.c:157): check exit codes error
[mpiexec@<node>] HYD_dmx_poll_wait_for_proxy_event (../../../../../src/pm/i_hydra/libhydra/demux/hydra_demux_poll.c:206): poll for event error
[mpiexec@<node>] HYD_bstrap_setup (../../../../../src/pm/i_hydra/libhydra/bstrap/src/intel/i_hydra_bstrap.c:1063): error waiting for event
[mpiexec@<node>] Error setting up the bootstrap proxies
[mpiexec@<node>] Possible reasons:
[mpiexec@<node>] 1. Host is unavailable. Please check that all hosts are available.
[mpiexec@<node>] 2. Cannot launch hydra_bstrap_proxy or it crashed on one of the hosts.
[mpiexec@<node>]    Make sure hydra_bstrap_proxy is available on all hosts and it has right permissions.
[mpiexec@<node>] 3. Firewall refused connection.
[mpiexec@<node>]    Check that enough ports are allowed in the firewall and specify them with the I_MPI_PORT_RANGE variable.
[mpiexec@<node>] 4. slurm bootstrap cannot launch processes on remote host.
[mpiexec@<node>]    You may try using -bootstrap option to select alternative launcher.
srun: error: Unable to create step for job <jobid>: Requested node configuration is not available

Application is timed out, aborting ...

* * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * 
   Timed out in Fluent startup [600 seconds], aborting ... 

   Check networks and computers to make sure they are available and healthy! 
   If you believe the systems are good, you can wait longer by setting 
     the environment variable in seconds as needed: 

       FLUENT_START_COMPUTE_NODE_TIME_OUT 
* * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * 



</path/to/fluent>/bin/fluent: line 3111: <pid> Aborted                 (core dumped) $NO_RUN $CORTEX_PRE $EXE -f $FLUENT_PROD $CX_FLAGS "$CX_FUNCTION"
```

The issue turned out to be the `-ssh` bootstrap method. Fluent was launching processes with SSH, but was getting hung up on the interactive steps of password entry and:

```console
The authenticity of host '<node> (<ip>)' can't be established.
ED25519 key fingerprint is <fingerprint>.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
```

I was able to resolve the issue by setting up ssh keys:

```console
[<user>@<node>]$ ssh-keygen -t rsa
[<user>@<node>]$ cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
```
and then setting [strict host key checking](https://man.openbsd.org/ssh_config.5) to off in `~/.ssh/config`

```
Host *.<host>
    StrictHostKeyChecking off
```