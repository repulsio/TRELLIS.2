# Run `TRELLIS.2` in Cloud

This is working on the latest `TRELLIS.2` commit ([`1762f493fe7731a3b7cc6b79ad5da7b015b516c1`](https://github.com/microsoft/TRELLIS.2/commit/1762f493fe7731a3b7cc6b79ad5da7b015b516c1)).

<br/>

## HyperStack

You can run a `RTX-A6000` VM on **HyperStack** for **$0.50/hour** with the following specs:

- 1 GPU
- 28 CPUs
- 58 GB RAM
- 100 GB Disk

<br/>

1. Choose the `Ubuntu Server 22.04 LTS R550 CUDA 12.4` OS Image.
2. Enable **SSH Access** to your VM.
3. Assign a **Public IP Address** to your VM.

<br/>

> [!NOTE]
> Make sure you choose `Ubuntu Server 22.04 LTS R550 CUDA 12.4` instead of the default `Ubuntu Server 22.04 LTS R535 CUDA 12.2` OS Image!
> 
> This is because `TRELLIS.2`'s README says:
> > Recommended version is 12.4.

<br/>

> [!CAUTION]
> On Hyperstack, incoming traffic to VMs are blocked by default, so you need to create a Firewall with a rule that allows incoming TCP traffic to port `7860` (or all ports) and apply it to the VM.

<br/>

## SSH into VM

```shell
ssh ubuntu@<PUBLIC_IP_ADDRESS_OF_VM>
```

<br/>

## Install Conda and Python

**References**:

- [Installing Conda on Ubuntu](https://medium.com/@mustafa_kamal/a-step-by-step-guide-to-installing-conda-in-ubuntu-and-creating-an-environment-d4e49a73fc46)
- [Anaconda versions](https://repo.anaconda.com/archive/)

<br/>

```shell
curl -O https://repo.anaconda.com/archive/Anaconda3-2024.10-1-Linux-x86_64.sh
bash Anaconda3-2024.10-1-Linux-x86_64.sh -b -p $HOME/anaconda3
source $HOME/anaconda3/bin/activate
```

> [!NOTE]
> Installing `Anaconda3-2024.10-1` also installs `Python 3.12.7`. I checked that this is the last Anaconda version that comes with `Python 3.12`.

## `TRELLIS.2`

The commands below are directly from `TRELLIS.2`'s [README](https://github.com/microsoft/TRELLIS.2/blob/main/README.md):

```shell
git clone -b main https://github.com/microsoft/TRELLIS.2.git --recursive
cd TRELLIS.2

. ./setup.sh --new-env --basic --flash-attn --nvdiffrast --nvdiffrec --cumesh --o-voxel --flexgemm
```

> [!NOTE]
> The above command creates a Conda virtual environment named `trellis2` and activates it.
> 
> All of the below commands expect that the virtual environment `trellis2` is activated.
> 
> If you need to reactivate it, run `conda activate trellis2`.

> [!WARNING]
> Because of the 2 following issues, we need to run the 2 following commands:
> - [Unable to install `flash-attn` even if I first install `torch` alone #1421](https://github.com/Dao-AILab/flash-attention/issues/1421)
> - [undefined symbol: `_ZN3c105ErrorC2ENS_14SourceLocationENSt7__cxx1112basic_stringIcSt11char_traitsIcESaIcEEE` #1832](https://github.com/Dao-AILab/flash-attention/issues/1832)

```shell
pip install psutil
pip install flash_attn==2.7.3 --no-build-isolation
```

> [!WARNING]
> Unfortunately, `TRELLIS.2` has 2 gated models as dependencies:
> 1. [`facebook/dinov3-vitl16-pretrain-lvd1689m`](https://huggingface.co/facebook/dinov3-vitl16-pretrain-lvd1689m)
> 2. [`briaai/RMBG-2.0`](https://huggingface.co/briaai/RMBG-2.0)
> 
> Because of this, you need to create a HuggingFace account and request access to both of these models. Then, you need to create a HuggingFace **WRITE Access Token** for your account.

```shell
hf auth login

# Enter your WRITE Access Token
# Type and enter `Y`
```

<br/>

## Run Web Demo

```shell
GRADIO_SERVER_NAME=0.0.0.0 python app.py
```

Open `http://<PUBLIC_IP_ADDRESS_OF_VM>:7860` in your web browser.
