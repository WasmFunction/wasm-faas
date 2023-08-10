# wasm-faas
基于 WebAssembly的FaaS Serverless解决方案。

- 实现WebAssembly容器化，实现基于WebAssembly的CRI容器运行时，使Kubernetes能够实现wasm的调度
- 基于Fission在此Kubernetes实现对wasm函数的Faas平台实现

## 目前工作

### Kubernetes

1. 配合fission实现[wasmkeeper](https://github.com/Youtirsin/wasmkeeper)的使用
2. 完善kuasar的资源接口实现
3. 优化k3s，测试更改定时器是否会带来性能问题

## Fission

1. 配合底层wasmkeeper，实现新方案长短服务的统一管理



## 代码仓库



**kuasar**

使用沙盒API实现wasm沙盒，优化containerd的CRI沙盒处理

- [kuasar-io/kuasar: A multi-sandbox container runtime that provides cloud-native, all-scenario multiple sandbox container solutions. (github.com)](https://github.com/kuasar-io/kuasar)



**rust-extensions**

kuasar的核心强依赖，containerd shim的task，shim等的rust实现。

- [kuasar-io/rust-extensions: Rust crates to extend containerd (github.com)](https://github.com/kuasar-io/rust-extensions)



**wasmkeeper**

对kuasar进行改造，满足上层对短服务重入和保活的需求

- [Youtirsin/wasmkeeper: A simple http server that runs wasm functions. (github.com)](https://github.com/Youtirsin/wasmkeeper)



**OCI镜像剥离**

使用runwasi的方案，但尝试剥离OCI镜像，减少containerd部分不必要处理环节。目前暂时停用

- https://github.com/leviyanx/containerd.git



### Docs

- [WebAssembly调研相关结果（演示文稿/md文档）](./doc/web_assembly)
- [K8s调研相关结果（演示文稿/md文档）](./doc/k8s)
- [Fission调研相关结果（演示文稿/md文档）](./doc/fission)
- [Containerd调研相关结果（演示文稿/md文档）](./doc/containerd)

Tip: 各文档目录下md文档与演示文稿的存放结构（以k8s文档目录为例）

```text
k8s
|
+- md (md文档）
|
+- original keynote（存放keynote演示文稿）
|  |
|  +- *.key（keynote演示文稿）
|
+- *.pdf（由original keynote目录中的keynote演示文稿导出，方便非mac用户查看）
|
+- *.ppt/*.pptx（powerpoint演示文稿）
```

只需查看md文档、pdf文档、ppt演示文稿。



## 相关技术



**1. runwasi**

- [containerd/runwasi: Facilitates running Wasm / WASI workloads managed by containerd (github.com)](https://github.com/containerd/runwasi)

快速构建基于wasm的containerd shim的框架。

*Docker的第一版方案使用此框架*





**2. containerd-wasm-shim (Spin, Slight)**

- [deislabs/containerd-wasm-shims: containerd shims for running WebAssembly workloads in Kubernetes (github.com)](https://github.com/deislabs/containerd-wasm-shims)

使用runwasi定制化的containerd shim，但只能用于spin框架下的微服务的开发。

- [fermyon/spin: Spin is the open source developer tool for building and running serverless applications powered by WebAssembly. (github.com)](https://github.com/fermyon/spin)

*Docker的第二版方案使用此方案*





**3. wasmedge crun**

- [WasmEdge in Kubernetes - WasmEdge Runtime](https://wasmedge.org/book/en/use_cases/kubernetes)

wasmedge官方修改的crun，能够在最底层OCI运行时阶段运行wasm。



### 相关官方链接

* https://github.com/kubernetes/kubernetes
* https://github.com/fission
* https://github.com/WasmEdge/WasmEdge
* https://github.com/bytecodealliance/

