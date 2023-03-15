# This repository is deprecated in favor of [KWasm.sh](https://kwasm.sh)


# kind-crun-wasm
This repository shows how to run WASM on [KinD](https://github.com/kubernetes-sigs/kind/). It is build on the awesome work of https://github.com/second-state/wasmedge-containers-examples.

## Quickstart
To get started just run:
```bash
# Create a "WASM in KinD" Cluster
kind create cluster --image ghcr.io/liquid-reply/kind-crun-wasm:v1.23.4
```
To test the WASM capabilities of your cluster you can start the example:
```bash
# Run the example
kubectl run -it --rm --restart=Never wasi-demo --image=hydai/wasm-wasi-example:with-wasm-annotation --annotations="module.wasm.image/variant=compat" /wasi_example_main.wasm 50000000
```
To create your own containers you can start here: https://wasmedge.org/book/en/kubernetes/demo.html

## Approach
Upstream Kind uses Containerd and runc to run Containers. However crun has built in support to run wasm workloads. Therefore this Repository patches the original Kind node image and replaces [runc](https://github.com/opencontainers/runc) with [crun](https://github.com/containers/crun).
```
KinD -> Containerd -> crun -> WasmEdge
```

## How to run
To create a KinD cluster with [WASMEdge](https://github.com/WasmEdge/WasmEdge) support you can use the following cluster description:
```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  image: ghcr.io/liquid-reply/kind-crun-wasm:v1.23.4
- role: worker
  image: ghcr.io/liquid-reply/kind-crun-wasm:v1.23.4
```
Start the cluster:
```bash
kind create cluster --config=cluster.yaml

    Creating cluster "kind" ...
    ✓ Ensuring node image (ghcr.io/liquid-reply/kind-crun-wasm:v1.23.4) 🖼 
    ✓ Preparing nodes 📦 📦  
    ✓ Writing configuration 📜 
    ✓ Starting control-plane 🕹️ 
    ✓ Installing CNI 🔌 
    ✓ Installing StorageClass 💾 
    ✓ Joining worker nodes 🚜 
    Set kubectl context to "kind-kind"
```
Run the demo app:
```
kubectl run -it --rm --restart=Never wasi-demo --image=hydai/wasm-wasi-example:with-wasm-annotation --annotations="module.wasm.image/variant=compat" /wasi_example_main.wasm 50000000

    Random number: 1272007318
    Random bytes: [203, 141, 55, 34, 190, 15, 21, 226, 138, 11, 221, 207, 99, 95, 191, 61, 94, 51, 78, 227, 204, 87, 13, 159, 165, 105, 237, 171, 218, 51, 87, 123, 251, 211, 41, 252, 232, 239, 208, 229, 73, 237, 65, 217, 90, 8, 86, 80, 131, 134, 200, 89, 192, 60, 231, 166, 180, 1, 59, 11, 160, 5, 43, 199, 5, 165, 201, 6, 210, 167, 147, 135, 174, 136, 176, 211, 193, 186, 227, 208, 144, 141, 243, 5, 87, 77, 246, 227, 246, 60, 183, 100, 23, 194, 85, 139, 164, 153, 164, 151, 73, 53, 130, 82, 255, 109, 222, 182, 82, 177, 238, 13, 241, 9, 146, 184, 196, 57, 164, 121, 31, 162, 255, 211, 181, 61, 135, 154]
    Printed from wasi: This is from a main function
    This is from a main function
    The env vars are as follows.
    The args are as follows.
    /wasi_example_main.wasm
    50000000
    File content is This is in a file
    pod "wasi-demo" deleted
```
