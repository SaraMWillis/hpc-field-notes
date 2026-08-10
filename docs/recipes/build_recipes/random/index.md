# Random Software

## ESMFold

[https://anaconda.org/channels/nvidia/packages/cuda-toolkit/overview](https://anaconda.org/channels/nvidia/packages/cuda-toolkit/overview)

Git repo: [https://github.com/facebookresearch/ESM](https://github.com/facebookresearch/ESM#esmfold)
Pytorch: [https://pytorch.org/get-started/locally/](https://pytorch.org/get-started/locally/)

```
[sarawillis@r7u25n1 ~]$ module load python/3.11 cuda12 cuda12-dnn
(puma) [sarawillis@r7u25n1 myungjinlee]$ python3 -m venv --system-site-packages esmfold-env
(puma) [sarawillis@r7u25n1 myungjinlee]$ source esmfold-env/bin/activate
(puma) (esmfold-env) [sarawillis@r7u25n1 myungjinlee]$ pip install torch==2.6.0 torchvision==0.21.0 torchaudio==2.6.0 --index-url https://download.pytorch.org/whl/cu124
(puma) (esmfold-env) [sarawillis@r7u25n1 myungjinlee]$ pip install fair-esm 
(puma) (esmfold-env) [sarawillis@r7u25n1 myungjinlee]$ pip install "fair-esm[esmfold]"
(puma) (esmfold-env) [sarawillis@r7u25n1 myungjinlee]$ pip install 'dllogger @ git+https://github.com/NVIDIA/dllogger.git'



# OpenFold and its remaining dependency
pip install 'dllogger @ git+https://github.com/NVIDIA/dllogger.git'
pip install 'openfold @ git+https://github.com/aqlaboratory/openfold.git@4b41059694619831a7db195b7e0988fc4ff3a307'



(puma) [sarawillis@r7u25n1 myungjinlee]$ micromamba create --prefix=/xdisk/sarawillis/TICKETS/myungjinlee/esmfold-env-micromamba python=3.11
(puma) (esmfold-env-micromamba) [sarawillis@r7u25n1 myungjinlee]$ micromamba activate /xdisk/sarawillis/TICKETS/myungjinlee/esmfold-env-micromamba
(puma) (esmfold-env-micromamba) [sarawillis@r7u25n1 myungjinlee]$ micromamba install nvidia::cuda-toolkit==13.0.0
(puma) (esmfold-env-micromamba) [sarawillis@r7u25n1 myungjinlee]$ pip3 install torch torchvision



```inst