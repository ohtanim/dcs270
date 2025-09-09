
## Build and run container

```bash
./build.sh

mkdir shared
cp setup_env.sh shared

./run.sh
```

## Setup environment variables

```bash
. /shared/setup_env.sh
```

## Build

```bash
cd /shared/qiskit-capi-demo/deps/qiskit
make c

cd /shared/qiskit-capi-demo/deps/qrmi
cargo build --release

cd /shared/qiskit-capi-demo/deps/qiskit-cpp
mkdir build
cd build
cmake -DQISKIT_ROOT=/shared/qiskit-capi-demo/deps/qiskit -DQRMI_ROOT=/shared/qiskit-capi-demo/deps/qrmi ..
make

cd /shared/qiskit-capi-demo/
mkdir build
cd build
cmake ..
make
```



