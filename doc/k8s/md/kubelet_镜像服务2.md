# 镜像

- [镜像 | Kubernetes](https://kubernetes.io/zh-cn/docs/concepts/containers/images/)

## 镜像名称

镜像名称字段`spec / containers / image`，如：

```go
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.14.2
    ports:
    - containerPort: 80
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:   
  labels:   
    app.kubernetes.io/name: load-balancer-example   
  name: hello-world   
spec:   
  replicas: 5   
  selector:   
    matchLabels:   
      app.kubernetes.io/name: load-balancer-example   
  template:   
    metadata:   
      labels:   
        app.kubernetes.io/name: load-balancer-example   
    spec:   
      containers:   
      - image: gcr.io/google-samples/node-hello:1.0   
        name: hello-world   
        ports:   
        - containerPort: 8080
```

**镜像名称**

容器镜像通常会被赋予 `pause`、`example/mycontainer` 或者 `kube-apiserver` 这类的名称。 镜像名称也可以包含所在仓库的主机名。例如：`fictional.registry.example/imagename`。 还可以包含仓库的端口号，例如：`fictional.registry.example:10443/imagename`。

如果你不指定仓库的主机名，Kubernetes 认为你在使用 Docker 公共仓库。

在镜像名称之后，你可以添加一个**标签（Tag）**（与使用 `docker` 或 `podman` 等命令时的方式相同）。 使用标签能让你辨识同一镜像序列中的不同版本。

镜像标签可以包含小写字母、大写字母、数字、下划线（`_`）、句点（`.`）和连字符（`-`）。 关于在镜像标签中何处可以使用分隔字符（`_`、`-` 和 `.`）还有一些额外的规则。 如果你不指定标签，Kubernetes 认为你想使用标签 `latest`。

## 镜像拉取

**ImagePullPolicy**

Kubernetes 支持用户自定义镜像文件的获取方式策略，例如在网络资源紧张的时候可以禁止从仓库中获取文件镜像等，容器的 ImagePullPolicy 字段用于为其指定镜像获取策略，可用值包括:

`IfNotPresent`: 本地有镜像则使用本地镜像，本地不存在则拉取镜像
`Always`: 每次都尝试拉取镜像，忽略容器运行时维护的所有本地缓存
`Never`: 永不拉取，禁止从仓库下载镜像，如果本地镜像已经存在，kubelet 会尝试启动容器，否则，启动失败

当你最初创建一个 Deployment、 StatefulSet、Pod 或者其他包含 Pod 模板的对象时，如果没有显式设定的话， Pod 中所有容器的默认镜像拉取策略是 `IfNotPresent`。这一策略会使得 kubelet 在镜像已经存在的情况下直接略过拉取镜像的操作。

当你（或控制器）向 API 服务器提交一个新的 Pod 时，你的集群会在满足特定条件时设置 imagePullPolicy 字段：

- 如果你省略了 imagePullPolicy 字段，并且容器镜像的标签是 :latest， imagePullPolicy 会自动设置为 Always。
- 如果你省略了 imagePullPolicy 字段，并且没有指定容器镜像的标签， imagePullPolicy 会自动设置为 Always。
- 如果你省略了 imagePullPolicy 字段，并且为容器镜像指定了非 :latest 的标签， imagePullPolicy 就会自动设置为 IfNotPresent。

**ImagePullBackOff**

当 kubelet 使用容器运行时创建 Pod 时，容器可能因为 ImagePullBackOff 导致状态为 Waiting。

ImagePullBackOff 状态意味着容器无法启动， 因为 Kubernetes 无法拉取容器镜像（原因包括无效的镜像名称，或从私有仓库拉取而没有 ImagePullSecret）。 BackOff 部分表示 Kubernetes 将继续尝试拉取镜像，并增加回退延迟。

Kubernetes 会增加每次尝试之间的延迟，直到达到编译限制，即 300 秒（5 分钟）。

**在 Pod 上指定 `ImagePullSecrets`**

```bash
kubectl create secret docker-registry <name> \
  --docker-server=DOCKER_REGISTRY_SERVER \
  --docker-username=DOCKER_USER \
  --docker-password=DOCKER_PASSWORD \
  --docker-email=DOCKER_EMAIL
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: foo
  namespace: awesomeapps
spec:
  containers:
    - name: foo
      image: janedoe/awesomeapp:v1
  imagePullSecrets:
    - name: myregistrykey
```

**代码**

ImageManager 的唯一方法 `EnsureImageExists` 确保容器运行时其镜像是存在的。工作流程：

1. 如果`Container`使用的`Image`没有指定`Tag`，那么使用`Latest`作为`Tag`

2. 根据`Pod`的`Annotation`调用`CRI`的`GetImageRef`接口获取镜像

3. 根据镜像的拉取策略判断是否需要拉取镜像，如果不需要，则直接返回，如果需要，那么继续往下

4. 拉取镜像

```go
func (m *imageManager) EnsureImageExists(ctx context.Context, pod *v1.Pod, container *v1.Container, pullSecrets []v1.Secret, podSandboxConfig *runtimeapi.PodSandboxConfig) (string, string, error)
```

`EnsureImageExists` 只调用于 `startContainer` 中，工作流程：

1. pull the image
2. create the container
3. start the container
4. run the post start lifecycle hooks (if applicable)

```go
func (m *kubeGenericRuntimeManager) startContainer(ctx context.Context, podSandboxID string, podSandboxConfig *runtimeapi.PodSandboxConfig, spec *startSpec, pod *v1.Pod, podStatus *kubecontainer.PodStatus, pullSecrets []v1.Secret, podIP string, podIPs []string) (string, error)
```

`startContainer `只调用于 `SyncPod` ，工作流程：

1. 计算sandbox和容器的变化。
2. 必要情况下停止sandbox。
3. 停止不需要运行的容器。
4. 创建sandbox。
5. 创建ephemeral容器。
6. 创建init容器。
7. 创建普通容器。

```go
func (m *kubeGenericRuntimeManager) SyncPod(ctx context.Context, pod *v1.Pod, podStatus *kubecontainer.PodStatus, pullSecrets []v1.Secret, backOff *flowcontrol.Backoff) (result kubecontainer.PodSyncResult)
```

**镜像拉取流程**

1. 用户发送请求创建Pod, PodSpec中包含各镜像的的名称和拉取策略
2. 调度器将Pod调度到合适的节点上
3. 节点的kubelet中的syncLoop通过ListWatch机制检测到Pod创建事件并获取Pod Updates，为pod开启专有协程pod worker
4. pod worker调用runtimeManager提供的syncPod方法
5. syncPod判断需要创建容器，调用startContainer方法
6. startContainer方法创建容器前调用imageManager的EnsureImageExists方法确保镜像的存在，根据镜像拉取策略判断是否拉取镜像
7. 需要拉取镜像，从PodSpec中获取镜像信息，转换成ImageSpec，并调用runtimeManager的PullImage方法拉取镜像
8. PullImage方法预处理镜像信息，然后调用ImageService拉取镜像
9. ImageService发起grpc请求调用服务

***提前拉取镜像***

官方仅简要描述，没有具体做法

```txt
By default, the kubelet tries to pull each image from the specified registry. However, if the imagePullPolicy property of the container is set to IfNotPresent or Never, then a local image is used (preferentially or exclusively, respectively).

If you want to rely on pre-pulled images as a substitute for registry authentication, you must ensure all nodes in the cluster have the same pre-pulled images.

This can be used to preload certain images for speed or as an alternative to authenticating to a private registry.

All pods will have read access to any pre-pulled images.
```

- DaemonSet

- [Can Kubernetes pre-pull and cache images? : kubernetes (reddit.com)](https://www.reddit.com/r/kubernetes/comments/oeruh9/can_kubernetes_prepull_and_cache_images/)

## 镜像回收

Kubernetes 通过 imageController 和 kubelet 中集成的 cAdvisor 共同管理镜像的生命周期，根据node的磁盘使用触发镜像的GC

Kubernetes 垃圾回收（Garbage Collection）机制由kubelet完成，kubelet定期清理不再使用的容器和镜像，每分钟进行一次容器的GC，每五分钟进行一次镜像的GC

**代码**

1. 通过 cadvisor 获取磁盘使用情况
2. 通过 RuntimeManager 获取镜像状态
3. 找到没有正在使用的镜像，按照上次使用时间排序，删除最久没有使用过的镜像。上次使用时间差必须大于MinAge，最后通过CRI删除镜像。如果释放资源不满足要求，会返回错误。


```go
func (im *realImageGCManager) GarbageCollect(ctx context.Context) error
```
