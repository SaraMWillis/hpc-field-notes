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