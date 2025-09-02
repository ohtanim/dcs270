# qiskit-cpp on dcs270

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
conda create -y --name qiskit_c_cpp
conda activate qiskit_c_cpp
# `python` is required to build with Qiskit C API library. 
conda install python
conda install clang
conda install libclang
# libstdc++.so.6: version `GLIBCXX_3.4.32' not found (required by ./circuit_test)
conda install -c conda-forge libstdcxx-ng
conda deactivate
```

### Cloning Git respositories

```bash
# set proxy to allow internet connection from node.
export http_proxy=http://proxy:8888
export https_proxy=$http_proxy

# create clones of QRMI and its spank-plugin
cd $HOME/barn
git clone https://github.com/Qiskit/qiskit.git
git clone https://github.com/Qiskit/qiskit-cpp.git
git clone https://github.com/qiskit-community/qrmi.git
```

## Building and Installing Qiskit Cpp

### Using GCC 15 (System default is 8.4)

Use newer version of GCC for building qiskit-cpp. 

```bash
# Clear the environment from any previously loaded modules
module purge > /dev/null 2>&1

# Load the module environment suitable for build
module load gcc/15.1.0
```

### Setting environment variables for Rust tools

```bash
# set environment variables for Rust toolset
source $HOME/barn/rust/.cargo/env
```

### Activating conda

```bash
conda activate qiskit_c_cpp
```

### Setting other environment variables

> [!NOTE]
> Ensure to use libraries and header files from the Conda environment, as the modules on dcs270 are outdated. For example, openssl is 1.1.1 which is incompatible to 3.0 installed during `conda install python`.

```bash
export LD_LIBRARY_PATH="$CONDA_PREFIX/lib:$LD_LIBRARY_PATH"
export LIBRARY_PATH="$CONDA_PREFIX/lib:$LIBRARY_PATH"
export CPATH="$CONDA_PREFIX/include:$CPATH"
```

The following environment variables will statically link the OpenSSL 3 library in Conda (= no need to install OpenSSL 3 as a runtime dependency). If you want dynamic linking, specify only `export OPENSSL_DIR=$CONDA_PREFIX`. 

```bash
export OPENSSL_DIR=$HOME/barn/miniconda3
export OPENSSL_STATIC=1
```

### Building Qiskit C API
```bash
cd $HOME/barn/qiskit
mkdir c
```

### Building QRMI API

```bash
cd $HOME/barn/qrmi
cargo build --release
```

### Building Qiskit C++ API

```bash
cd $HOME/barn/qiskit-cpp
mkdir build
cd build
cmake -DQISKIT_ROOT=$HOME/barn/qiskit -DQRMI_ROOT=$HOME/barn/qrmi -Wno-dev ..
make
```

### Deactivating conda

```bash
conda deactivate
```

## Unit Testing

### Setup

```bash
# set proxy to allow internet connection from node.
export http_proxy=http://proxy:8888
export https_proxy=$http_proxy

conda activate qiskit_c_cpp

# Clear the environment from any previously loaded modules
module purge > /dev/null 2>&1

# Load the module environment suitable for build
module load gcc/15.1.0

export LD_LIBRARY_PATH="$CONDA_PREFIX/lib:$LD_LIBRARY_PATH"
export LIBRARY_PATH="$CONDA_PREFIX/lib:$LIBRARY_PATH"
export CPATH="$CONDA_PREFIX/include:$CPATH"

cd $HOME/barn/qiskit-cpp/build
```

### cicuit_test

#### Run
```bash
./circuit_test
```

#### Expected result
```bash
h(0) 
x(0) 
reset(0) 
reset(1) 
reset(2) 
reset(3) 
x(1) 
cx(0, 1) 
rz(0) [3.1]
ry(1) [5.6]
p(1) [0.2]
rz(1) [5]
u(0) [0.6, -0.05, 1.3]
measure(0) (0) 
sx(0) 
x(1) 
xx_plus_yy(0, 1) [0.5, 0]
cz(3, 1) 
rxx(1, 3) [-3.1]
cz(3, 1) 
barrier(3, 1) 
measure(3) (3) 
t(2) 
barrier(0) 
measure(0) (0) 
measure(1) (1) 
measure(2) (2) 
measure(3) (3) 
OPENQASM 3.0;
include "stdgates.inc";
gate sxdg _gate_q_0 {
  s _gate_q_0;
  h _gate_q_0;
  s _gate_q_0;
}
gate xx_plus_yy(p0, p1) _gate_q_0, _gate_q_1 {
  rz(p1) _gate_q_0;
  sdg _gate_q_1;
  sx _gate_q_1;
  s _gate_q_1;
  s _gate_q_0;
  cx _gate_q_1, _gate_q_0;
  ry((-0.5)*p0) _gate_q_1;
  ry((-0.5)*p0) _gate_q_0;
  cx _gate_q_1, _gate_q_0;
  sdg _gate_q_0;
  sdg _gate_q_1;
  sxdg _gate_q_1;
  s _gate_q_1;
  rz(-p1) _gate_q_0;
}
gate rxx(p0) _gate_q_0, _gate_q_1 {
  h _gate_q_0;
  h _gate_q_1;
  cx _gate_q_0, _gate_q_1;
  rz(p0) _gate_q_1;
  cx _gate_q_0, _gate_q_1;
  h _gate_q_1;
  h _gate_q_0;
}
bit[4] c;
qubit[4] q;
h q[0];
x q[0];
reset q[0];
reset q[1];
reset q[2];
reset q[3];
x q[1];
cx q[0], q[1];
rz(3.10000000000000009) q[0];
ry(5.59999999999999964) q[1];
p(0.200000000000000011) q[1];
rz(5) q[1];
U(0.600000000000000089, -0.0500000000000000028, 1.30000000000000004) q[0];
c[0] = measure q[0];
sx q[0];
x q[1];
xx_plus_yy(0.5, 0) q[0], q[1];
cz q[3], q[1];
rxx(-3.10000000000000009) q[1], q[3];
cz q[3], q[1];
barrier q[3], q[1];
c[3] = measure q[3];
t q[2];
barrier q[0];
c[0] = measure q[0];
c[1] = measure q[1];
c[2] = measure q[2];
c[3] = measure q[3];
```

### observable_test

#### Run

```bash
./observable_test
```

#### Expected result

```bash
./observable_test 
SparseObservable { num_qubits: 10, coeffs: [Complex { re: 1.0, im: 0.0 }], bit_terms: [], indices: [], boundaries: [0, 0] }
SparseObservable { num_qubits: 13, coeffs: [Complex { re: 1.0, im: 0.0 }], bit_terms: [X, Y, X, X, Z, Plus, Minus], indices: [1, 2, 3, 4, 6, 8, 12], boundaries: [0, 7] }
SparseObservable { num_qubits: 10, coeffs: [Complex { re: 1.0, im: 0.0 }, Complex { re: -1.0, im: 0.0 }], bit_terms: [Y, Z, X, X, X, Z, Plus, Plus], indices: [2, 3, 5, 6, 2, 5, 7, 8], boundaries: [0, 4, 8] }
```

### sampler_test

#### Run

sample_test expects access to "ibm_torino". If you want to other quantum backend, edit `cd $HOME/barn/qiskit-cpp/test/sampler_test.cpp`.

```bash
export QISKIT_IBM_TOKEN=<your IQP API key>
export QISKIT_IBM_INSTANCE=<your IQP instance - Service CRN>
./sampler_test
```

#### Expected result

```bash
(app-root) ./sampler_test 
QRMI connecting : ibm_torino
 QRMI Job submitted to ibm_torino, JOB ID = d2r8239olshc73bmt49g
{"results": [{"data": {"c": {"samples": ["0x3ff", "0x2", "0x3ff", "0x0", "0x0", "0x0", "0xf", "0x80", "0x3ff", "0x0", "0x3ff", "0x0", "0x0", "0x3bf", "0x80", "0x3ff", "0x0", "0x2ff", "0x3ff", "0x3ff", "0x3ff", "0x3ff", "0x3ff", "0x200", "0x0", "0x7d", "0x0", "0x0", "0x0", "0x0", "0x0", "0x3ff", "0x3ff", "0x1", "0x3ff", "0x0", "0x0", "0x3ff", "0x3c0", "0x7f", "0x3fb", "0x2", "0x3ff", "0x3fe", "0x3ff", "0x100", "0x3ff", "0x1bf", "0x0", "0x3df", "0x0", "0x3ff", "0x3ff", "0x3ff", "0x0", "0x3ff", "0x3df", "0x0", "0x0", "0x0", "0x0", "0x0", "0x0", "0x0", "0x0", "0x7", "0x0", "0x3ef", "0x3ff", "0x3ff", "0x3ff", "0x0", "0x3fd", "0x0", "0x0", "0x3ff", "0x0", "0x0", "0x3ff", "0x3df", "0x0", "0x0", "0x3ff", "0x3ff", "0x3f0", "0x3ff", "0x3f", "0x0", "0x0", "0x0", "0x3ff", "0x3bf", "0x0", "0x3ff", "0x3ff", "0x3f", "0x1", "0x0", "0x3e0", "0x3e0"], "num_bits": 20}}, "metadata": {"circuit_metadata": {}}}], "metadata": {"execution": {"execution_spans": [[{"date": "2025-09-02T05:30:55.420561"}, {"date": "2025-09-02T05:30:56.388795"}, {"0": [[100], [0, 1], [0, 100]]}]]}, "version": 2}}
 ===== results in JSON =====
{"metadata":{"execution":{"execution_spans":[[{"date":"2025-09-02T05:30:55.420561"},{"date":"2025-09-02T05:30:56.388795"},{"0":[[100],[0,1],[0,100]]}]]},"version":2},"results":[{"data":{"c":{"num_bits":20,"samples":["0x3ff","0x2","0x3ff","0x0","0x0","0x0","0xf","0x80","0x3ff","0x0","0x3ff","0x0","0x0","0x3bf","0x80","0x3ff","0x0","0x2ff","0x3ff","0x3ff","0x3ff","0x3ff","0x3ff","0x200","0x0","0x7d","0x0","0x0","0x0","0x0","0x0","0x3ff","0x3ff","0x1","0x3ff","0x0","0x0","0x3ff","0x3c0","0x7f","0x3fb","0x2","0x3ff","0x3fe","0x3ff","0x100","0x3ff","0x1bf","0x0","0x3df","0x0","0x3ff","0x3ff","0x3ff","0x0","0x3ff","0x3df","0x0","0x0","0x0","0x0","0x0","0x0","0x0","0x0","0x7","0x0","0x3ef","0x3ff","0x3ff","0x3ff","0x0","0x3fd","0x0","0x0","0x3ff","0x0","0x0","0x3ff","0x3df","0x0","0x0","0x3ff","0x3ff","0x3f0","0x3ff","0x3f","0x0","0x0","0x0","0x3ff","0x3bf","0x0","0x3ff","0x3ff","0x3f","0x1","0x0","0x3e0","0x3e0"]}},"metadata":{"circuit_metadata":{}}}]}
 ===== results[0] in JSON =====
{"num_bits":20,"samples":["0x3ff","0x2","0x3ff","0x0","0x0","0x0","0xf","0x80","0x3ff","0x0","0x3ff","0x0","0x0","0x3bf","0x80","0x3ff","0x0","0x2ff","0x3ff","0x3ff","0x3ff","0x3ff","0x3ff","0x200","0x0","0x7d","0x0","0x0","0x0","0x0","0x0","0x3ff","0x3ff","0x1","0x3ff","0x0","0x0","0x3ff","0x3c0","0x7f","0x3fb","0x2","0x3ff","0x3fe","0x3ff","0x100","0x3ff","0x1bf","0x0","0x3df","0x0","0x3ff","0x3ff","0x3ff","0x0","0x3ff","0x3df","0x0","0x0","0x0","0x0","0x0","0x0","0x0","0x0","0x7","0x0","0x3ef","0x3ff","0x3ff","0x3ff","0x0","0x3fd","0x0","0x0","0x3ff","0x0","0x0","0x3ff","0x3df","0x0","0x0","0x3ff","0x3ff","0x3f0","0x3ff","0x3f","0x0","0x0","0x0","0x3ff","0x3bf","0x0","0x3ff","0x3ff","0x3f","0x1","0x0","0x3e0","0x3e0"]}
 ===== samples for pub[0] =====
0xff3, 0x2, 0xff3, 0x0, 0x0, 0x0, 0xf, 0x08, 0xff3, 0x0, 0xff3, 0x0, 0x0, 0xfb3, 0x08, 0xff3, 0x0, 0xff2, 0xff3, 0xff3, 0xff3, 0xff3, 0xff3, 0x002, 0x0, 0xd7, 0x0, 0x0, 0x0, 0x0, 0x0, 0xff3, 0xff3, 0x1, 0xff3, 0x0, 0x0, 0xff3, 0x0c3, 0xf7, 0xbf3, 0x2, 0xff3, 0xef3, 0xff3, 0x001, 0xff3, 0xfb1, 0x0, 0xfd3, 0x0, 0xff3, 0xff3, 0xff3, 0x0, 0xff3, 0xfd3, 0x0, 0x0, 0x0, 0x0, 0x0, 0x0, 0x0, 0x0, 0x7, 0x0, 0xfe3, 0xff3, 0xff3, 0xff3, 0x0, 0xdf3, 0x0, 0x0, 0xff3, 0x0, 0x0, 0xff3, 0xfd3, 0x0, 0x0, 0xff3, 0xff3, 0x0f3, 0xff3, 0xf3, 0x0, 0x0, 0x0, 0xff3, 0xfb3, 0x0, 0xff3, 0xff3, 0xf3, 0x1, 0x0, 0x0e3, 0x0e3, 
 ===== counts for pub[0] =====
0x0f3 : 1
0x7 : 1
0xfb1 : 1
0xf3 : 2
0xfe3 : 1
0x001 : 1
0xef3 : 1
0xff3 : 32
0xdf3 : 1
0x2 : 2
0x0 : 39
0xf : 1
0x0e3 : 2
0x1 : 2
0x08 : 2
0xf7 : 1
0xfd3 : 3
0xfb3 : 2
0xd7 : 1
0xbf3 : 1
0xff2 : 1
0x002 : 1
0x0c3 : 1
```

### transpile_test

#### Run

transpile_test expects access to "ibm_torino". If you want to other quantum backend, edit `cd $HOME/barn/qiskit-cpp/test/sampler_test.cpp`.

```bash
export QISKIT_IBM_TOKEN=<your IQP API key>
export QISKIT_IBM_INSTANCE=<your IQP instance - Service CRN>
./transpile_test
```

#### Expected result

```bash
./transpile_test 
QRMI connecting : ibm_torino
input circuit
h(0) 
cx(0, 1) 
cx(1, 2) 
cx(2, 3) 
cx(3, 4) 
cx(4, 5) 
cx(5, 6) 
cx(6, 7) 
cx(7, 8) 
cx(8, 9) 
measure(0) (0) 
measure(1) (1) 
measure(2) (2) 
measure(3) (3) 
measure(4) (4) 
measure(5) (5) 
measure(6) (6) 
measure(7) (7) 
measure(8) (8) 
measure(9) (9) 
transpiled circuit
rz(29) [1.5708]
rx(29) [1.5708]
rz(36) [1.5708]
rx(36) [1.5708]
rz(46) [1.5708]
rx(46) [1.5708]
rz(47) [1.5708]
rx(47) [1.5708]
rz(48) [1.5708]
rx(48) [1.5708]
rz(55) [1.5708]
rx(55) [1.5708]
rz(65) [1.5708]
rx(65) [1.5708]
rz(66) [1.5708]
rx(66) [1.5708]
rz(67) [1.5708]
rx(67) [1.5708]
rz(68) [-1.5708]
rx(68) [-1.5708]
rz(68) [-1.5708]
cz(68, 67) 
rx(67) [-1.5708]
rz(67) [-1.5708]
cz(67, 66) 
rx(66) [-1.5708]
rz(66) [-1.5708]
cz(66, 65) 
rx(65) [-1.5708]
rz(65) [-1.5708]
cz(65, 55) 
rx(55) [-1.5708]
rz(55) [-1.5708]
cz(55, 46) 
rx(46) [-1.5708]
rz(46) [-1.5708]
cz(46, 47) 
rx(47) [-1.5708]
rz(47) [-1.5708]
cz(47, 48) 
rx(48) [-1.5708]
rz(48) [-1.5708]
cz(48, 36) 
rx(36) [-1.5708]
rz(36) [-1.5708]
cz(36, 29) 
rx(29) [-1.5708]
rz(29) [-1.5708]
measure(68) (0) 
measure(67) (1) 
measure(66) (2) 
measure(65) (3) 
measure(55) (4) 
measure(46) (5) 
measure(47) (6) 
measure(48) (7) 
measure(36) (8) 
measure(29) (9) 
```


## References
* [Qiskit C++](https://github.com/Qiskit/qiskit-cpp)
* [Qiskit C API](https://quantum.cloud.ibm.com/docs/en/api/qiskit-c)
* [Rensselaer - Center for Computational Innovation - Documentation](https://docs.cci.rpi.edu/)
* [Quantum Resource Management Interface(QRMI)](https://github.com/qiskit-community/qrmi)

## END OF DOCUMENT
