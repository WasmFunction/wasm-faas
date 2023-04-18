# Tests with crictl

crictl 是 K8s 提供的 cri 开发和测试辅助工具，提供所有 cri 接口的调用命令。如果 crictl 和 critest 能够通过的话，kubelet 基本也是没问题的。kubelet 内部有多处 cri 相关的调用，测试较为复杂，问题不好定位，因此可以先通过 crictl 进行测试。

## Prerequisites

**wasmedge shim**

- [containerd/runwasi: Facilitates running Wasm / WASI workloads managed by containerd (github.com)](https://github.com/containerd/runwasi)

可以参照 [containerd-wasm-shim-test](./containerd-wasm-shim-test.md) 中的安装方法。

**containerd**

- [leviyanx/containerd: An open and reliable container runtime (github.com)](https://github.com/leviyanx/containerd)

```bash
git clone https://github.com/leviyanx/containerd.git
cd containerd
git checkout wasm_container
make bin/containerd
# run
bin/containerd
```

**crictl**

- [kubernetes-sigs/cri-tools: CLI and validation tools for Kubelet Container Runtime Interface (CRI) . (github.com)](https://github.com/kubernetes-sigs/cri-tools)

参照官网中的安装方法

***issue***

使用 crictl 可能遇到的问题

```
FATA[0000] run pod sandbox: rpc error: code = Unknown desc = failed to create containerd task: failed to create shim task: OCI runtime create failed: runc create failed: expected cgroupsPath to be of format "slice:prefix:name" for systemd cgroups, got "/k8s.io/9a586f6684ee8ad93c29343da5813fa996109ee936732e1927bacd970b65ece6" instead: unknown
```

**解决方案**

根据报错信息可以看出 crictl 默认不是使用 systemd 作为 cgroup driver。将 containerd 的 cgroup driver 改回 cgroupfs。

```toml
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
	...
	SystemdCgroup = false
	# SystemdCgroup = true
	...
```

## Tests

测试用例在 `containerd/wasmtests/crictl` 下。

```bash
# pwd containerd
cd wasmtests/crictl/
```

### Image

```bash
# pull
crictl pull \
	--annotation wasm.module.url="https://raw.githubusercontent.com/Youtirsin/wasi-demo-apps/main/wasi-demo-app.wasm" \
	--annotation wasm.module.filename="wasi-demo-app.wasm" \
	wasi-demo-app

# list
crictl image

# delete
crictl rmi wasi_example_main
```

### Instance

需要预先跑一个 pod (sandbox)

```yaml
# wasm-sandbox.yaml
metadata:
  name: wasm-sandbox
  namespace: default
  attempt: 1
  uid: "hdishd83djaidwnduwk28bcsb"
log_directory: "/temp"
linux:
```

```bash
# run pod
crictl runp --runtime=wasm wasm-sandbox.yaml

# list pod
crictl pods

# stop pod
crictl stopp [POD-ID]

# delete pod
crictl rmp [POD-ID]
```

运行 wasm。*确保运行前 Pod 状态是 Ready。*

```yaml
metadata:
  name: wasi-demo-app
image:
  image: wasi-demo-app
  annotations:
    wasm.module.url: "https://raw.githubusercontent.com/Youtirsin/wasi-demo-apps/main/wasi-demo-app.wasm"
    wasm.module.filename: "wasi-demo-app.wasm"
command: ["wasi-demo-app.wasm", "daemon"]
log_path: "wasi-demo-app.0.log"
linux:
```

```bash
# create
crictl create [POD-ID] wasm-demo-app.yaml wasm-sandbox.yaml
# list
crictl ps -a
# start
crictl start [CONTAINER-ID]
# stop
crictl stop [CONTAINER-ID]
# delete
crictl rm [CONTAINER-ID]
```

