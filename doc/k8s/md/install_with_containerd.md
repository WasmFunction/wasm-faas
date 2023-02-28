# Install

## Containerd


### Install

- [containerd/getting-started.md at main · containerd/containerd (github.com)](https://github.com/containerd/containerd/blob/main/docs/getting-started.md)

最好跟随手动安装的，apt 等包管理工具可能不是最新版的

### Configuration

加载配置文件到`/etc/containerd/config.toml`

```bash
containerd config default > /etc/containerd/config.toml
```

主要配置以下内容

````toml
# 1. 配置cgroup driver为systemd driver
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
...
    [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
    ...
    SystemdCgroup = true # 设为true
    
# 2. 配置镜像源为国内
# 原本为
sandbox_image = "k8s.gcr.io/pause:3.5"
# 改为
sandbox_image = "registry.aliyuncs.com/k8sxio/pause:3.5"

````

修改完后重启containerd

```bash
systemctl enable containerd
systemctl restart containerd
```

## Kubernetes

### 安装Kubeadm, Kubectl, Kubelet

- [kubernetes镜像_kubernetes下载地址_kubernetes安装教程-阿里巴巴开源镜像站 (aliyun.com)](https://developer.aliyun.com/mirror/kubernetes?spm=a2c6h.13651102.0.0.3e221b11J1ao39)

```shell
apt-get update && apt-get install -y apt-transport-https
curl https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | apt-key add - 
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main
EOF
apt-get update
apt-get install -y kubelet kubeadm kubectl
# 指定版本
apt-get install -y kubelet=1.19.0-00 kubeadm=1.19.0-00 kubectl=1.19.0-00
apt-get install -y kubelet=1.25.0-00 kubeadm=1.25.0-00 kubectl=1.25.0-00
```

**注意**：如果安装版本大于等于1.24，并使用Docker作为runtime，则需要安装cri-dockerd。

### 关闭Swap

```shell
swapoff -a
```

### 主节点

```shell
# 主节点初始化
kubeadm init --pod-network-cidr=192.168.0.0/16 --image-repository registry.aliyuncs.com/google_containers

# 根据安装成功的提示，赋予普通用户权限
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

# 安装Clico或flannel等网络插件
# flannel比较简单
# 注意flannel：
# If you use custom podCIDR (not 10.244.0.0/16) you first need to download the above manifest and modify the network to match your one.
```

- Calico CNI网络插件
  
  [Quickstart for Calico on Kubernetes (tigera.io)](https://projectcalico.docs.tigera.io/getting-started/kubernetes/quickstart)

- 网络插件flannel

  [flannel-io/flannel: flannel is a network fabric for containers, designed for Kubernetes (github.com)](https://github.com/flannel-io/flannel)

### 工作节点

与主节点安装类似，但是不需要安装Kubectl

```shell
# 设置kubelet开机自启
systemctl start kubelet
systemctl enable kubelet

# 使用主节点生成的加入方式加入集群，如
kubeadm join 192.168.31.243:6443 --token pu0kg3.yve8khsarxoiez0r \
    --discovery-token-ca-cert-hash sha256:d01877657e44bab34118a914957b1c23ef0d2d128e5d0ea73bf33ee64211b917
```

## Issues

1.  /proc/sys/net/bridge/bridge-nf-call-iptables does not exist

   ```bash
   # 开启内核模块
   modprobe br_netfilter
   
   # /proc/sys/net/ipv4/ip_forward contents are not set to 1
   echo 1 > /proc/sys/net/ipv4/ip_forward
   ```

## Kubernetes 重置

```shell
kubeadm reset
# $HOME 为普通用户目录
rm -rf /etc/kubernetes/ /etc/cni/ $HOME/.kube
```
