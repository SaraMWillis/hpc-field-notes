# Miscellaneous Software

## ESMFold

* Cuda Toolkit Info: [https://anaconda.org/channels/nvidia/packages/cuda-toolkit/overview](https://anaconda.org/channels/nvidia/packages/cuda-toolkit/overview)
* Git repo: [https://github.com/facebookresearch/ESM](https://github.com/facebookresearch/ESM#esmfold)
* Pytorch: [https://pytorch.org/get-started/locally/](https://pytorch.org/get-started/locally/)

This one was a PITA. I was facing similar issues to [this GitHub issues report](https://github.com/facebookresearch/esm/issues/677) and wound up having to do some archeological digging into the specific version of openfold to hammer down the dependencies. The yaml file (`environment.yaml`) I found with:

```console
(esmfold-env) [lookitsme@r7u02n1 openfold]$ history | egrep "git clone|git checkout"
 1098  11/08/26 12:14:36 git clone https://github.com/aqlaboratory/openfold.git
 1100  11/08/26 12:14:41 git checkout 4b41059694619831a7db195b7e0988fc4ff3a307
```

That gave me enough to work with when setting up the environment. However, the MKL version wasn't pinned down, so it wound up giving me a 2026 version that resulted in an openfold installation error `undefined symbol: iJIT_NotifyEvent`. I found the specific MKL version required by this version of PyTorch using:

```console
(esmfold-env) [lookitsme@r7u02n1 openfold]$ micromamba repoquery depends pytorch=1.12 --channel pytorch
Getting repodata from channels...

pytorch/linux-64                                            Using cache
pytorch/noarch                                              Using cache
 Name                                      Version Build          Channel Subdir 
──────────────────────────────────────────────────────────────────────────────────
 blas * mkl >>> NOT FOUND <<<                                                    
 mkl >=2018 >>> NOT FOUND <<<                                                    
 python >=3.10,<3.11.0a0 >>> NOT FOUND <<<                                       
 pytorch                                   1.12.1  py3.10_cpu_0   pytorch pytorch
 pytorch-mutex                             1.0     cpu            pytorch pytorch
 typing_extensions >>> NOT FOUND <<<  
```

Which more or less got me the rest of the way there.

From OpenFold 1.0's environment yaml file:

```yaml
name: openfold_venv
channels:
  - conda-forge
  - bioconda
  - pytorch
dependencies:
  - conda-forge::python=3.7
  - conda-forge::setuptools=59.5.0
  - conda-forge::pip
  - conda-forge::openmm=7.5.1
  - conda-forge::pdbfixer
  - conda-forge::cudatoolkit==11.3.*
  - bioconda::hmmer==3.3.2
  - bioconda::hhsuite==3.3.0
  - bioconda::kalign2==2.04
  - pytorch::pytorch=1.12.*
  - pip:
      - biopython==1.79
      - deepspeed==0.5.10
      - dm-tree==0.1.6
      - ml-collections==0.1.0
      - numpy==1.21.2
      - PyYAML==5.4.1
      - requests==2.26.0
      - scipy==1.7.1
      - tqdm==4.62.2
      - typing-extensions==3.10.0.2
      - pytorch_lightning==1.5.10
      - wandb==0.12.21
      - git+https://github.com/NVIDIA/dllogger.git
```

I used that to build the relevant environment and forced MKL to downgrade to 2022. I also had to install `gcc` and `g++` in my micromamba environment since there were a few other issues that cropped up when trying to use the system gcc. The full installation wound up being:

```console
[lookitsme@r7u02n1 openfold]$ micromamba create -n esmfold-env python=3.7
[lookitsme@r7u02n1 openfold]$ micromamba activate esmfold-env
(esmfold-env) [lookitsme@r7u02n1 openfold]$ micromamba install -c "nvidia/label/cuda-11.3.1" cuda-toolkit
(esmfold-env) [lookitsme@r7u02n1 openfold]$ micromamba install \
    conda-forge::setuptools=59.5.0 \
    conda-forge::openmm=7.5.1 \
    conda-forge::pdbfixer \
    bioconda::hmmer=3.3.2 \
    bioconda::hhsuite=3.3.0 \
    bioconda::kalign2=2.04 \
    pytorch::pytorch=1.12.*
(esmfold-env) [lookitsme@r7u02n1 openfold]$ pip install \
    biopython==1.79 \
    deepspeed==0.5.10 \
    dm-tree==0.1.6 \
    ml-collections==0.1.0 \
    numpy==1.21.2 \
    PyYAML==5.4.1 \
    requests==2.26.0 \
    scipy==1.7.1 \
    tqdm==4.62.2 \
    typing-extensions==3.10.0.2 \
    pytorch_lightning==1.5.10 \
    wandb==0.12.21
(esmfold-env) [lookitsme@r7u02n1 openfold]$ pip install fair-esm 
(esmfold-env) [lookitsme@r7u02n1 openfold]$ pip install "fair-esm[esmfold]"
(esmfold-env) [lookitsme@r7u02n1 openfold]$ pip install 'dllogger @ git+https://github.com/NVIDIA/dllogger.git'
(esmfold-env) [lookitsme@r7u02n1 openfold]$ micromamba install "mkl=2022.0"
(esmfold-env) [lookitsme@r7u02n1 openfold]$ micromamba install conda-forge::gcc==11.1.0
(esmfold-env) [lookitsme@r7u02n1 openfold]$ micromamba install conda-forge::gxx==11.1.0
(esmfold-env) [lookitsme@r7u02n1 openfold]$ pip install 'openfold @ git+https://github.com/aqlaboratory/openfold.git@4b41059694619831a7db195b7e0988fc4ff3a307'
(esmfold-env) [lookitsme@r7u02n1 openfold]$ micromamba install matplotlib
(esmfold-env) [lookitsme@r7u02n1 openfold]$ micromamba install --channel=conda-forge libxcrypt
(esmfold-env) [lookitsme@r7u02n1 openfold]$ export CPATH=/groups/sarawillis/SOFTWARE/micromamba/envs/esmfold-env/include/
(esmfold-env) [lookitsme@r7u02n1 openfold]$ pip install biotite
```

With the `biotite`, `libcryptx`, and `matplotlib` installations done to allow me to run the provided examples. The `CPATH` had to be used, otherwise the `biotite` package wasn't able to find the `crypt.h` header file.

Once I did all that, I ran some test Python scripts they gave (modifying it to save the figure rather than display it):

```python
import torch
import esm

# Load ESM-2 model
model, alphabet = esm.pretrained.esm2_t33_650M_UR50D()
batch_converter = alphabet.get_batch_converter()
model.eval()  # disables dropout for deterministic results

# Prepare data (first 2 sequences from ESMStructuralSplitDataset superfamily / 4)
data = [
    ("protein1", "MKTVRQERLKSIVRILERSKEPVSGAQLAEELSVSRQVIVQDIAYLRSLGYNIVATPRGYVLAGG"),
    ("protein2", "KALTARQQEVFDLIRDHISQTGMPPTRAEIAQRLGFRSPNAAEEHLKALARKGVIEIVSGASRGIRLLQEE"),
    ("protein2 with mask","KALTARQQEVFDLIRD<mask>ISQTGMPPTRAEIAQRLGFRSPNAAEEHLKALARKGVIEIVSGASRGIRLLQEE"),
    ("protein3",  "K A <mask> I S Q"),
]
batch_labels, batch_strs, batch_tokens = batch_converter(data)
batch_lens = (batch_tokens != alphabet.padding_idx).sum(1)

# Extract per-residue representations (on CPU)
with torch.no_grad():
    results = model(batch_tokens, repr_layers=[33], return_contacts=True)
token_representations = results["representations"][33]

# Generate per-sequence representations via averaging
# NOTE: token 0 is always a beginning-of-sequence token, so the first residue is token 1.
sequence_representations = []
for i, tokens_len in enumerate(batch_lens):
    sequence_representations.append(token_representations[i, 1 : tokens_len - 1].mean(0))

# Look at the unsupervised self-attention map contact predictions
import matplotlib.pyplot as plt
for (_, seq), tokens_len, attention_contacts in zip(data, batch_lens, results["contacts"]):
    plt.matshow(attention_contacts[: tokens_len, : tokens_len])
    plt.title(seq)
    plt.savefig("test.png")
```

and it generated 

<img src="./images/test.png" width=350px>

and the second test, to test ESM Fold:

```python 
import torch
import esm

model = esm.pretrained.esmfold_v1()
model = model.eval().cuda()

# Optionally, uncomment to set a chunk size for axial attention. This can help reduce memory.
# Lower sizes will have lower memory requirements at the cost of increased speed.
# model.set_chunk_size(128)

sequence = "MKTVRQERLKSIVRILERSKEPVSGAQLAEELSVSRQVIVQDIAYLRSLGYNIVATPRGYVLAGG"
# Multimer prediction can be done with chains separated by ':'

with torch.no_grad():
    output = model.infer_pdb(sequence)

with open("result.pdb", "w") as f:
    f.write(output)

import biotite.structure.io as bsio
struct = bsio.load_structure("result.pdb", extra_fields=["b_factor"])
print(struct.b_factor.mean())  # this will be the pLDDT
# 88.3
```

gave

```console title="ESM Fold test on a GPU node"
(esmfold-env) [lookitsme@r5u15n1 openfold]$ python3 test.py 
88.28930830039526
```