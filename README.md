# wasm-faas
基于 WebAssembly的FaaS Serverless解决方案

### Todo List

* 将WebAssembly作为一种运行环境集成到Fission框架中，产出结果：Demo、源代码和重现技术文档，负责人：严炼
  * Demo、制作镜像的dockerfile及相关源码、说明文档都在这个仓库中：https://gitee.com/mengning997/environments/tree/wasm/wasm
* Kubernetes技术研究报告，包括Kubernetes向下对接Docker容器和WebAssembly runtime，以及Kubernetes向上云原生应用开发（比如Fission）。产出结果：研究报告文档，负责人：张士炳、胡烨
* WebAssembly和Docker容器性能评测对比，产出结果：测评代码和测评报告文档
* CRI + WASM相关技术调研，产出结果：研究报告文档，负责人：严炼
* Fission性能评测结果的复现，产出结果：Demo和重现技术文档

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

### Links

* https://github.com/kubernetes/kubernetes
* https://github.com/fission
* https://github.com/WasmEdge/WasmEdge
* https://github.com/bytecodealliance/


