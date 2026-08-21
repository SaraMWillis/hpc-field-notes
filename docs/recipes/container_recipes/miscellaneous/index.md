# Miscellaneous Container Builds

## AddaxAI

```apptainer title="AddaxAI.recipe"
Bootstrap: docker
From: ubuntu:24.04

%post 
  apt update -y
  apt install build-essential zlib1g-dev libncurses5-dev libgdbm-dev libnss3-dev libssl-dev libreadline-dev libffi-dev libsqlite3-dev wget libbz2-dev git -y 
  apt-get update && \
  apt-get install -y libgl1 libglib2.0-0
  wget https://petervanlunteren.github.io/AddaxAI/install_files/linux/install.command
  sed -i 's/\bsudo\b//g' install.command
  chmod +x install.command
  mkdir ~/Desktop
  ./install.command
```