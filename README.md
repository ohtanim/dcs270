# dcs270

## 1. Setting up

```bash
# login to data center

# login to frontend of DCS Cluster
ssh dcsfen01

# start an interactive session
salloc -t 120 --reservation=root_48 --gres=gpu:1 -p dcs-2024

# login to dcs270 node
ssh dcs270

# recommend to install Rust under Barn directory because they consumes large size of filesystem.
export CARGO_HOME=$HOME/barn/rust/.cargo
export RUSTUP_HOME=$HOME/barn/rust/.rustup

# Install Rust toolsets
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

Refer [this document](https://docs.cci.rpi.edu/software/Conda/#installing-conda-on-ppc64le) to setup conda for ppc64le.

```bash
conda create -y --name qrmi_ppc
conda activate qrmi_ppc
conda install python
conda install clang
conda install libclang
conda install openblas-devel
conda install jpeg
conda deactivate
```

> [!NOTE]
> `openblas-devel` is required by `scipy`, as dependencies of Pasqal's `pulser` modules.

> [!NOTE]
> `jpeg` is required by `matplotlib`, as dependencies of Pasqal's `pulser` modules.

```bash
# set proxy to allow internet connection from node.
export http_proxy=http://proxy:8888
export https_proxy=$http_proxy

# create clones of QRMI and its spank-plugin
cd $HOME/barn
git clone https://github.com/qiskit-community/spank-plugins.git
git clone https://github.com/qiskit-community/qrmi.git
```

## 2. Building and Installing SPANK plugin & QRMI

### Using GCC 15 (System default is 8.4)

```bash
module load gcc/15.1.0
```

Following 2 environment variables are required for Rust cross compilation.

```bash
export CC=/gpfs/u/software/dcs-rhel8/gcc/15.1.0/bin/gcc
export CXX=/gpfs/u/software/dcs-rhel8/gcc/15.1.0/bin/g++
```

### Setting environment variables for Rust tools

```bash
# set environment variables for Rust toolset
source $HOME/barn/rust/.cargo/env
```

### Activating conda

```bash
conda activate qrmi_ppc
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

> [!NOTE]
> The following environment variables will statically link the OpenSSL 3 library in Conda (= no need to install OpenSSL 3 as a runtime dependency). If you want dynamic linking, specify only `export OPENSSL_DIR=$CONDA_PREFIX`. 

```bash
export OPENSSL_DIR=$HOME/barn/miniconda3
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

### Building and installing QRMI python binding library

```bash
cd $HOME/barn/qrmi
pip install -r requirements-dev.txt
maturin build --release
pip install --force-reinstall $HOME/barn/qrmi/target/wheels/qrmi-0.7.1-cp312-abi3-manylinux_2_28_ppc64le.whl
```

### Deactivating conda

```bash
conda deactivate
```

### Verify installed python modules
```bash
pip list
Package                Version
---------------------- -----------
   :
   :
pulser                    1.5.5
pulser-core               1.5.5
pulser-pasqal             0.20.1
pulser-simulation         1.5.5
   :
   :
qiskit                    2.1.2
qiskit-ibm-runtime        0.41.1
qiskit_pasqal_provider    0.1.1
qiskit-qasm3-import       0.6.0
qrmi                      0.7.1
   :
   :
```

### Deactivating conda

```bash
conda deactivate
```

## 3. Unit Testing

### QRMI Primitive (Python)

> [!NOTE]
> For running python programs, you need to `conda activate qrmi_ppc` because some modules are installed to conda environment.

#### Prerequisites
- IQP API Key and Service CRN, which can be obtained in https://quantum.cloud.ibm.com/signin.
- For Direct Access, you need to get the following values for testing.
  - Direct Access API endpoint URL
  - S3 Bucket endpoint
  - S3 AWS ACCESS KEY ID
  - S3 AWS SECRET ACCESS KEY
  - S3 Bucket name
  - S3 Region name

#### Activating conda

```bash
conda activate qrmi_ppc
```

#### Change directory to example
```bash
cd $HOME/barn/qrmi/examples/qiskit_primitives/ibm
```

#### Creating .env file
Create .env file with parameter values:

1) Qiskit Runtime Service
```bash
SLURM_JOB_QPU_RESOURCES=ibm_kingston
SLURM_JOB_QPU_TYPES=qiskit-runtime-service
ibm_kingston_QRMI_IBM_QRS_ENDPOINT=https://quantum.cloud.ibm.com/api/v1
ibm_kingston_QRMI_IBM_QRS_IAM_ENDPOINT=https://iam.cloud.ibm.com
ibm_kingston_QRMI_IBM_QRS_IAM_APIKEY=<your API key>
ibm_kingston_QRMI_IBM_QRS_SERVICE_CRN=<your Service CRN>
```

2) Direct Access
```bash
SLURM_JOB_QPU_RESOURCES=ibm_rensselaer
SLURM_JOB_QPU_TYPES=direct-access
ibm_rensselaer_QRMI_IBM_DA_ENDPOINT=<your Direct Access endpoint URL for ibm_rensselaer>
ibm_rensselaer_QRMI_IBM_QRS_IAM_ENDPOINT=https://iam.cloud.ibm.com
ibm_rensselaer_QRMI_IBM_QRS_IAM_APIKEY=<your API key>
ibm_rensselaer_QRMI_IBM_QRS_SERVICE_CRN=<your Service CRN>
ibm_rensselaer_QRMI_IBM_DA_AWS_ACCESS_KEY_ID=<your AWS access key ID>
ibm_rensselaer_QRMI_IBM_DA_AWS_SECRET_ACCESS_KEY=<your AWS secret access key>
ibm_rensselaer_QRMI_IBM_DA_S3_ENDPOINT=<your S3 endpoint URL>
ibm_rensselaer_QRMI_IBM_DA_S3_BUCKET=<your S3 bucket name>
ibm_rensselaer_QRMI_IBM_DA_S3_REGION=<your S3 region>
ibm_rensselaer_QRMI_JOB_TIMEOUT_SECONDS=86400
```

#### Run example
```bash
python estimator.py
```

You will see:
```bash
{'backend_name': 'ibm_kingston'}
>>> Observable: ['IIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIII...',
 'IIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIII...',
 'IIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIII...',
 'IIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIII...',
 'IIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIII...',
 'IIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIII...',
 'IIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIII...',
 'IIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIII...',
 'IIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIII...',
 'IIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIII...',
 'IIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIII...',
 'IIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIIII...', ...]
>>> Circuit ops (ISA): OrderedDict({'rzz': 704, 'rz': 312, 'rx': 312, 'sx': 156})
>>> Job ID: d1f6ki2q8ogc73espoeg
>>> Job Status: JobStatus.QUEUED
>>> PrimitiveResult([PubResult(data=DataBin(evs=np.ndarray(<shape=(), dtype=float64>), stds=np.ndarray(<shape=(), dtype=float64>), ensemble_standard_error=np.ndarray(<shape=(), dtype=float64>)), metadata={'shots': 4096, 'target_precision': 0.015625, 'circuit_metadata': {}, 'resilience': {}, 'num_randomizations': 32})], metadata={'dynamical_decoupling': {'enable': False, 'sequence_type': 'XX', 'extra_slack_distribution': 'middle', 'scheduling_method': 'alap'}, 'twirling': {'enable_gates': False, 'enable_measure': True, 'num_randomizations': 'auto', 'shots_per_randomization': 'auto', 'interleave_randomizations': True, 'strategy': 'active-accum'}, 'resilience': {'measure_mitigation': True, 'zne_mitigation': False, 'pec_mitigation': False}, 'version': 2})
  > Expectation value: 33.649813025539515
  > Metadata: {'shots': 4096, 'target_precision': 0.015625, 'circuit_metadata': {}, 'resilience': {}, 'num_randomizations': 32}
```

#### Deactivating conda

```bash
conda deactivate
```

### QRMI Task Runner

#### Creating input data
```bash
conda activate qrmi_ppc
cd $HOME/barn/qrmi/examples/task_runner/qiskit
python gen_estimator_inputs.py <backend_name> https://quantum.cloud.ibm.com/api <your apikey> <your Service CRN>
python gen_sampler_inputs.py <backend_name> https://quantum.cloud.ibm.com/api <your apikey> <your Service CRN>
conda deactivate
```

#### Changing directory to your workspace
```bash
cd $HOME/barn/
```

#### Creating .env file

Refer [Creating .env file](#creating-env-file).

#### Run
```bash
task_runner <backend_name> $HOME/barn/qrmi/examples/task_runner/qiskit/estimator_input_<backend_name>.json
```

You are expected to see like:
```bash
Task ID: d1f6tddqbivc73ebs4i0
{"results": [{"data": {"evs": 33.144319662700426, "stds": 0.2581068379167223, "ensemble_standard_error": 0.24209042405484843}, "metadata": {"shots": 5024, "target_precision": 0.01414213562373095, "circuit_metadata": {}, "resilience": {}, "num_randomizations": 32}}], "metadata": {"dynamical_decoupling": {"enable": false, "sequence_type": "XX", "extra_slack_distribution": "middle", "scheduling_method": "alap"}, "twirling": {"enable_gates": false, "enable_measure": true, "num_randomizations": "auto", "shots_per_randomization": "auto", "interleave_randomizations": true, "strategy": "active-accum"}, "resilience": {"measure_mitigation": true, "zne_mitigation": false, "pec_mitigation": false}, "version": 2}}
```

### Slurm plugins

Refer [this](https://github.com/qiskit-community/spank-plugins/tree/main/plugins/tests/metadata) to build test.

1) Verify `spank_qrmi.so` can be dynamically loaded as Slurm plugin
```bash
LD_LIBRARY_PATH=/lib64/slurm:$CONDA_PREFIX/lib:$LD_LIBRARY_PATH $HOME/barn/spank-plugins/plugins/tests/metadata/build/test $HOME/barn/spank-plugins/plugins/spank_qrmi/build/spank_qrmi.so
```

Expected:
```bash
Valid Slurm plugin library. name=spank_qrmi, type=spank, version=23.11.10
```

## END OF DOCUMENT

