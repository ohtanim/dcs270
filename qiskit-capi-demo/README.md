# qiskit-capi-demo on dcs270


## Logging on to dcs270 

```bash
# login to data center
ssh <your userid>@blp01.ccni.rpi.edu

# login to frontend of DCS Cluster
ssh dcsfen01

# start an interactive session
salloc -t 120 --reservation=root_48 --gres=gpu:1 -p dcs-2024

# login to dcs270 node
ssh dcs270
```

## Setting up

### Installing Rust toolset

```bash
# recommend to install Rust under Barn directory because they consumes large size of filesystem.
export CARGO_HOME=$HOME/barn/rust/.cargo
export RUSTUP_HOME=$HOME/barn/rust/.rustup

# Install Rust toolsets
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

### Installing dependencies

Refer [Installing conda on PPC64le](https://docs.cci.rpi.edu/software/Conda/#installing-conda-on-ppc64le) to setup conda for ppc64le.

And then,

```bash
conda create -y --name qiskit_capi_demo
conda activate qiskit_capi_demo
conda install python
# required to build aws-lc-sys, as dependencies of qrmi.
conda install clang
# required to build aws-lc-sys, as dependencies of qrmi.
conda install libclang
conda install openblas-devel
conda install eigen -c conda-forge
# gcc/g++ 14.3.0
conda install cxx-compiler -c conda-forge
conda install openmpi cmake
conda deactivate
```

### Setting proxy for internet connection

```bash
# set proxy to allow internet connection from node.
export http_proxy=http://proxy:8888
export https_proxy=$http_proxy
```
