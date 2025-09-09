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

source $HOME/barn/rust/.cargo/env
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
```

### Setting proxy for internet connection

```bash
# set proxy to allow internet connection from node.
export http_proxy=http://proxy:8888
export https_proxy=$http_proxy
```

### Setting other environment variables

> [!NOTE]
> Ensure to use libraries and header files from the Conda environment, as the modules on dcs270 are outdated. For example, openssl is 1.1.1 which is incompatible to 3.0 installed during `conda install python`.

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
export OPENSSL_DIR=$HOME/barn/miniconda3
export OPENSSL_STATIC=1
```

### Preparing source tree

Refer README for qiskit-capi-demo.

## Building modules

### Qiskit C API

```bash
cd $HOME/barn/qiskit-capi-demo/deps/qiskit
make c
```

* [Build log](./qiskit-c-api-build.txt)

### QRMI

```bash
cd $HOME/barn/qiskit-capi-demo/deps/qrmi
cargo build --release
```

* [Build log](./qrmi-build.txt)


### Qiskit C++ API

```bash
cd $HOME/barn/qiskit-capi-demo/deps/qiskit-cpp
mkdir build
cd build
cmake -DQISKIT_ROOT=$HOME/barn/qiskit-capi-demo/deps/qiskit -DQRMI_ROOT=$HOME/barn/qiskit-capi-demo/deps/qrmi ..
make
```

* [Build log](./qiskit-cpp-build.txt)

### C API demo

```bash
cd $HOME/barn/qiskit-capi-demo/
mkdir build
cd build
cmake ..
make
```

* [CMake output](./cmake_out.txt)
* [Build log](./make_out.txt)


## Running capi-demo

```bash
export QISKIT_IBM_TOKEN=<your API token>
export QISKIT_IBM_INSTANCE=<your Service CRN>
cd $HOME/barn/qiskit-capi-demo/build
./capi-demo --fcidump ../data/fcidump_Fe4S4_MO.txt -v --tolerance 1.0e-3 --max_time 600 --recovery 1 --number_of_samples 1000 --backend_name ibm_fez
```

* [Result](./capi-demo-result.txt)
