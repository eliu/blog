---
title: 使用 kind 在本地运行 k8s
date: 2021-03-24 21:30:26
categories:
- k8s
tags:
- kind
- docker
---

本文介绍如何使用 kind 在本地快速启动一个 k8s 集群。kind 是 Kubernetes in Docker 的简写，从名字上看很容易猜出 kind 的目标是将一个 k8s 集群以容器的方式部署在本机电脑上。这种方式对平台依赖少，安装部署比较干净利落，理论上本地只需要一个 Docker 运行环境即可。

<!-- more -->

## 安装过程介绍

### 实验环境

- 操作系统：macOS Catalina (10.15.7)
- 容器环境：Docker Desktop for Mac 3.2.2
- 包管理工具：Homebrew

### 设置 Docker 镜像加速

打开 Docker Desktop for Mac 的首选项界面，选择 Docker Engine，加入如下设置：

```json
{
  "registry-mirrors": [
    "https://8km017g6.mirror.aliyuncs.com"
  ]
}
```

之后点击 `Apply & Restart` 重启 Docker。

### 安装 kind 和 kubectl

kind 并不依赖于 kubectl，但是开发人员需要 kubectl 与 kind 所创建的 k8s 集群进行通讯：

```shell
$ brew install kubectl
$ brew install kind
```

### 创建 k8s 集群

使用 `kind create cluster` 命令新建集群，注意 `--name` 选项指定集群的名称，若未指定该参数，`kind` 将会是默认的集群名称。 

```shell
$ kind create cluster --name demo
Creating cluster "demo" ...
 ✓ Ensuring node image (kindest/node:v1.20.2) 🖼
 ✓ Preparing nodes 📦
 ✓ Writing configuration 📜
 ✓ Starting control-plane 🕹️
 ✓ Installing CNI 🔌
 ✓ Installing StorageClass 💾
Set kubectl context to "kind-demo"
You can now use your cluster with:

kubectl cluster-info --context kind-demo

Thanks for using kind! 😊
```

此时根据提示输入 `kubectl cluster-info --context kind-demo` 会显示集群当前的基本信息：

```shell
$ kubectl cluster-info --context kind-demo
Kubernetes control plane is running at https://127.0.0.1:56770
KubeDNS is running at https://127.0.0.1:56770/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

查看集群 demo 中的节点信息：

```shell
$ kubectl get node
NAME                 STATUS   ROLES                  AGE     VERSION
demo-control-plane   Ready    control-plane,master   4m46s   v1.20.2

$ kubectl get pods -n kube-system
NAME                                         READY   STATUS    RESTARTS   AGE
coredns-74ff55c5b-9gdc5                      1/1     Running   0          6m24s
coredns-74ff55c5b-tnb5w                      1/1     Running   0          6m24s
etcd-demo-control-plane                      1/1     Running   0          6m38s
kindnet-4fmq9                                1/1     Running   0          6m24s
kube-apiserver-demo-control-plane            1/1     Running   0          6m38s
kube-controller-manager-demo-control-plane   1/1     Running   0          6m38s
kube-proxy-6wrdz                             1/1     Running   0          6m24s
kube-scheduler-demo-control-plane            1/1     Running   0          6m38s
```

至此，一个最基本的 k8s 集群就已经创建好了。

### 删除集群

使用以下命令删除上面已经创建的集群 demo：

```shell
$ kind delete cluster --name demo
Deleting cluster "demo" ...
```

## 支持 Ingress 控制器的集群

带有 Ingress Controller 的集群则需要向主机暴露 80 和 443 端口以便于主机可以通过域名进行访问。kind 除了可以支持通过命令行选项的方式创建集群，也支持使用配置文件的方式对集群进行更细致的配置，创建命令如下：

```shell
cat <<EOF | kind create cluster --config=-
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  kubeadmConfigPatches:
  - |
    kind: InitConfiguration
    nodeRegistration:
      kubeletExtraArgs:
        node-labels: "ingress-ready=true"
  extraPortMappings:
  - containerPort: 80
    hostPort: 80
    protocol: TCP
  - containerPort: 443
    hostPort: 443
    protocol: TCP
EOF
```

当然你也可以把这个配置信息以文件的形式保存，比如叫 `kind.yaml` ，那么创建命令就是：

```shell
kind create cluster --config=kind.yaml
```

### 部署 NGINX Ingress 控制器

国内对于 GitHub 和 k8s.gcr.io 镜像仓库的访问速度不太理想，因此此处我们需要做一些针对于国内网络的准备工作。

#### 拉取镜像

```shell
# 从阿里云镜像仓库拉取镜像
$ docker pull registry.aliyuncs.com/kubeadm-ha/ingress-nginx_controller:v0.43.0
# 重命名为官方镜像名称
$ docker tag registry.aliyuncs.com/kubeadm-ha/ingress-nginx_controller:v0.43.0 k8s.gcr.io/ingress-nginx/controller:v0.43.0
```

#### 将镜像导入到 kind

kind 的 `load` 命令可以帮助我们将 Docker 中的镜像导入到由 kind 创建的集群中，命令如下：

```shell
$ kind load docker-image k8s.gcr.io/ingress-nginx/controller:v0.43.0
Image: "k8s.gcr.io/ingress-nginx/controller:v0.43.0" with ID "sha256:38dca1cbd23197f591e58fc6c949110b53f7a003e15f6d4974d86e7f7a00815d" not yet present on node "kind-control-plane", loading...
```

#### 部署 Ingress 控制器

我们使用 Gitee 的镜像仓库地址来加速资源的下载：

```shell
$ kubectl apply -f https://gitee.com/mirrors/ingress-nginx/raw/controller-v0.43.0/deploy/static/provider/kind/deploy.yaml
```

Ingress 控制器的部署和启动需要等待一会，可以使用以下命令来查看运行情况：

```shell
$ kubectl get pod -n ingress-nginx
NAME                                        READY   STATUS      RESTARTS   AGE
ingress-nginx-admission-create-ktp2g        0/1     Completed   0          7h4m
ingress-nginx-admission-patch-q4vmb         0/1     Completed   2          7h4m
ingress-nginx-controller-55bc59c885-mcp25   1/1     Running     0          7h4m
```

如果 ingress-nginx-controller-xxx 的状态的是 Running，说明已经成功运行。

### 部署测试应用

此处我们使用 kind 官网提供的测试应用：

```shell
kubectl apply -f https://kind.sigs.k8s.io/examples/ingress/usage.yaml
```

测试 Ingress 的连通性：

```shell
$ curl localhost/foo
foo
$ curl localhost/bar
bar
```

## 参考资料

- [kind – Quick Start (k8s.io)](https://kind.sigs.k8s.io/docs/user/quick-start/)
- [kind – Ingress (k8s.io)](https://kind.sigs.k8s.io/docs/user/ingress/)

