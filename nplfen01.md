# nlpfen01 (CentOS 7.2/x86_64)

## Logging on to nplfen01 

```bash
# login to data center
ssh <your userid>@blp01.ccni.rpi.edu

# login to frontend of DCS Cluster
ssh nplfen01
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

Refer [Installing conda on x86](https://docs.cci.rpi.edu/software/Conda/#installing-conda-on-x86) to setup conda for x86_64.

And then,

```bash
conda create -y --name qrmi_x86
conda activate qrmi_x86
conda install python
# required to build spank plugin.
conda install anaconda::cmake
# use OpenSSL v3 for security
conda install anaconda::openssl
conda deactivate
```

### Cloning Git respositories

```bash
# set proxy to allow internet connection from node.
export http_proxy=http://proxy:8888
export https_proxy=$http_proxy

# create clones of spank-plugin
cd $HOME/barn
git clone https://github.com/qiskit-community/spank-plugins.git
```

## Building and Installing SPANK plugin & QRMI

### Setting environment variables for Rust tools

```bash
# set environment variables for Rust toolset
source $HOME/barn/rust/.cargo/env
```

### Activating conda

```bash
conda activate qrmi_x86
```

### Setting other environment variables

> [!NOTE]
> Ensure to use libraries and header files from the Conda environment, as the modules on nplfen01 are outdated. For example, openssl is 1.1.1 which is incompatible to 3.0 installed during `conda install python`.

```bash
export LD_LIBRARY_PATH="$CONDA_PREFIX/lib:$LD_LIBRARY_PATH"
export LIBRARY_PATH="$CONDA_PREFIX/lib:$LIBRARY_PATH"
export CPATH="$CONDA_PREFIX/include:$CPATH"
export LDFLAGS="-L$CONDA_PREFIX/lib $LDFLAGS"
export CFLAGS="-I$CONDA_PREFIX/include $CFLAGS"
export LIBCLANG_PATH=$CONDA_PREFIX/lib
```

The following environment variables will statically link the OpenSSL 3 library in Conda (= no need to install OpenSSL 3 as a runtime dependency). If you want dynamic linking, specify only `export OPENSSL_DIR=$CONDA_PREFIX`. 

```bash
export OPENSSL_DIR=$HOME/barn/miniconda3x86
export OPENSSL_STATIC=1
```

### Building spank_qrmi plugin
```bash
cd $HOME/barn/spank-plugins/plugins/spank_qrmi
mkdir build
cd build
cmake ..
make
```

Refer [this document](https://github.com/qiskit-community/spank-plugins/blob/main/plugins/spank_qrmi/README.md#installation) to deploy `spank_qrmi.so`, `qrmi_config.json` and `plugstack.conf` to your Cluster.


## Unit Testing

### Slurm plugins

Refer [this](https://github.com/qiskit-community/spank-plugins/tree/main/plugins/tests/metadata) to build `test` executable.

Verify `spank_qrmi.so` can be dynamically loaded as Slurm plugin
```bash
LD_LIBRARY_PATH=/lib64/slurm:$CONDA_PREFIX/lib:$LD_LIBRARY_PATH $HOME/barn/spank-plugins/plugins/tests/metadata/build/test $HOME/barn/spank-plugins/plugins/spank_qrmi/build/spank_qrmi.so
```

Expected:
```bash
Valid Slurm plugin library. name=spank_qrmi, type=spank, version=23.11.10
```

## References
* [Rensselaer - Center for Computational Innovation - Documentation](https://docs.cci.rpi.edu/)
* [Spank plugin for Quantum workload](https://github.com/qiskit-community/spank-plugins)
* [Quantum Resource Management Interface(QRMI)](https://github.com/qiskit-community/qrmi)
* [Slurm SPANK Plugin doc](https://slurm.schedmd.com/spank.html)

## END OF DOCUMENT

