---
title: 使用 kubeadm 快速安装 Kubernetes 集群
date: 2018-10-02 16:11:04
tags: 
- Kubernetes
- kubeadm
categories: 
- DevOps
---

## 概述

本教程将利用 Kubernetes 官方认为安装部署集群环境的最佳实践 - kubeadm 进行环境搭建。安装成功之后，我们仍然使用《Kubernetes 本地快速搭建指南》中所介绍的 Web 应用示例来部署至集群环境中，并以此确认集群是否可用。

<!-- more -->

### kubeadm 简介

在 CentOS / RHEL 环境中，安装 Kubernetes 的最简单的方式就是通过 `yum install kubernetes` 来完成，但仍然需要修改各个组件的启动参数，才能完成 Kubernetes 集群的设置，整个过程较为复杂，也容易出错。因此，从 Kubernetes v1.4 版本开始，官方正式引入了命令行工具 kubeadm ，致力于简化集群环境的安装和解决 Kubernetes 集群的高可用的问题。但直到 v1.9 版本，此工具还不能用于生产环境。按照官方的计划，此工具将会在 2018 年内正式面向生产环境推出。所以在现阶段，此工具比较适合初学者安装和学习 Kubernetes。

### kubeadm 版本

本教程将使用 kubeadm 的 `v1.9.2` 版本安装 Kubernetes 集群。在默认情况下，使用 kubeadm 安装的 Kubernetes 的主版本和 kubeadm 相一致，此处为 `v1.9.x` 。如果读者想要安装其他的版本，则使用 `--kubernetes-version` 参数来指定 Kubernetes 的版本。

```bash
kubeadm init --kubernetes-version=vx.y.z
```



## 环境准备

本教程将使用64位的 CentOS 7 操作系统作为安装环境，读者可以根据实际情况来进行准备。笔者个人推荐 VirtualBox + Vagrant 在本机快速搭建环境，具体可以参考《Kubernetes 本地快速搭建指南》，此处不再赘述。

## 安装 kubeadm

本节将详细介绍 kubeadm 在 CentOS 7 下的安装步骤。

### 设置主机名称

```bash
hostnamectl set-hostname kubeadm-master
echo "127.0.0.1 kubeadm-master" >> /etc/hosts
```

### 关闭 SELinux

```bash
setenforce 0
sed -i 's/SELINUX=enforcing/SELINUX=permissive/g' /etc/selinux/config
```

### 禁用 SWAP

```bash
swapoff -a
```

### 禁用防火墙

```bash
systemctl disable firewalld
systemctl stop firewalld
```

### 微调内核设置

对于 RHEL/CentOS 系统，由于 iptables 的设置，网络流量可能被错误的路由，因此要对内核进行如下调整：

```bash
cat <<EOF >  /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl --system
```

### 安装依赖的程序包

```bash
yum -y update && yum -y upgrade
yum -y install docker socat
```

### 离线安装 kubeadm

官方推荐设置 yum 源进行 kubeadm 的安装，不过在国内的网络环境下，谷歌的服务无法被正确的访问。此处采用离线安装方式安装 rpm 包。读者可以前往 https://pan.baidu.com/s/1kWfpPdl 进行下载。

```bash
rpm -ivh /path/to/kubectl-1.9.2-0.x86_64.rpm \
         /path/to/kubernetes-cni-0.6.0-0.x86_64.rpm \
         /path/to/kubelet-1.9.2-0.x86_64.rpm \
         /path/to/kubeadm-1.9.2-0.x86_64.rpm
```

## 启动 kubeadm

本节将对 kubeadm 启动前进行参数配置，同时将 Kubernetes 的附加功 Dashboard 也同时进行部署。

### 设置 Docker 镜像加速器

```bash
cat <<EOF > /etc/docker/daemon.json
{
  "registry-mirrors" : [
    "http://4a0fee72.m.daocloud.io"
  ]
}
EOF
```

### 启动 docker 和 kubelet 服务

```bash
systemctl enable docker && systemctl start docker
systemctl enable kubelet && systemctl start kubelet
```

### 导入 Docker 镜像

由于谷歌服务无法访问，因此相关的 Docker 镜像需要通过代理提前下载好，并进行打包。读者可以从 https://pan.baidu.com/s/1kWfpPdl 进行下载。导入镜像命令如下：

```bash
$ docker load -i /path/to/kubeadm/v1.9.2/docker-images.tar
Loaded image: gcr.io/google_containers/k8s-dns-sidecar-amd64:1.14.7
Loaded image: gcr.io/google_containers/k8s-dns-kube-dns-amd64:1.14.7
Loaded image: gcr.io/google_containers/k8s-dns-dnsmasq-nanny-amd64:1.14.7
Loaded image: gcr.io/google_containers/etcd-amd64:3.1.11
Loaded image: gcr.io/google_containers/kube-controller-manager-amd64:v1.9.2
Loaded image: gcr.io/google_containers/kube-scheduler-amd64:v1.9.2
Loaded image: k8s.gcr.io/kubernetes-dashboard-amd64:v1.8.2
Loaded image: gcr.io/google_containers/kube-proxy-amd64:v1.9.2
Loaded image: gcr.io/google_containers/kube-apiserver-amd64:v1.9.2
Loaded image: gcr.io/google_containers/pause-amd64:3.0
```

### 启动 kubeadm

使用 `kubeadm init` 初始化并启动 Kubernetes 集群。其中的几个重要参数选项列示如下：

| 参数选项                        | 参考值           | 含义                                          |
| ------------------------------- | ---------------- | --------------------------------------------- |
| `--kubernetes-version`          | v1.9.2           | Kubernetes 的版本                             |
| `--pod-network-cidr`            | 10.244.0.0/16    | Pod 网络寻址范围，此处是网络插件 Flannel 用到 |
| `--apiserver-advertise-address` | CentOS 主机的 IP | Kubernetes 集群的 apiserver (主节点) 的地址   |

启动 kubeadm 的命令如下：

```bash
$ kubeadm init \
  --kubernetes-version=v1.9.2 \
  --pod-network-cidr=10.244.0.0/16 \
  --apiserver-advertise-address=192.168.119.101
# 以下是 kubeadm 初始化时的输出信息
[init] Using Kubernetes version: v1.9.2
[init] Using Authorization modes: [Node RBAC]
[preflight] Running pre-flight checks.
[WARNING FileExisting-crictl]: crictl not found in system path
[certificates] Generated ca certificate and key.
[certificates] Generated apiserver certificate and key.
[certificates] apiserver serving cert is signed for DNS names [kubeadm-master kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.96.0.1 192.168.119.101]
[certificates] Generated apiserver-kubelet-client certificate and key.
[certificates] Generated sa key and public key.
[certificates] Generated front-proxy-ca certificate and key.
[certificates] Generated front-proxy-client certificate and key.
[certificates] Valid certificates and keys now exist in "/etc/kubernetes/pki"
[kubeconfig] Wrote KubeConfig file to disk: "admin.conf"
[kubeconfig] Wrote KubeConfig file to disk: "kubelet.conf"
[kubeconfig] Wrote KubeConfig file to disk: "controller-manager.conf"
[kubeconfig] Wrote KubeConfig file to disk: "scheduler.conf"
[controlplane] Wrote Static Pod manifest for component kube-apiserver to "/etc/kubernetes/manifests/kube-apiserver.yaml"
[controlplane] Wrote Static Pod manifest for component kube-controller-manager to "/etc/kubernetes/manifests/kube-controller-manager.yaml"
[controlplane] Wrote Static Pod manifest for component kube-scheduler to "/etc/kubernetes/manifests/kube-scheduler.yaml"
[etcd] Wrote Static Pod manifest for a local etcd instance to "/etc/kubernetes/manifests/etcd.yaml"
[init] Waiting for the kubelet to boot up the control plane as Static Pods from directory "/etc/kubernetes/manifests".
[init] This might take a minute or longer if the control plane images have to be pulled.
[apiclient] All control plane components are healthy after 30.502464 seconds
[uploadconfig] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[markmaster] Will mark node kubeadm-master as master by adding a label and a taint
[markmaster] Master kubeadm-master tainted and labelled with key/value: node-role.kubernetes.io/master=""
[bootstraptoken] Using token: 3a9f97.1cd340e0f34adbbb
[bootstraptoken] Configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstraptoken] Configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstraptoken] Configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[bootstraptoken] Creating the "cluster-info" ConfigMap in the "kube-public" namespace
[addons] Applied essential addon: kube-dns
[addons] Applied essential addon: kube-proxy

Your Kubernetes master has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
https://kubernetes.io/docs/concepts/cluster-administration/addons/

You can now join any number of machines by running the following on each node
as root:

kubeadm join --token 3a9f97.1cd340e0f34adbbb 192.168.119.101:6443 --discovery-token-ca-cert-hash sha256:64111962ff9df5f566d21f5ad5e9d046487aff4553a738f5c6d1ca0fdc4c69b8
```

### 用户设置

拷贝 Kubernetes 的配置文件至当前用户的目录下，使当前用户可以正确使用 kubectl 进行操作。

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

### 配置单节点集群

默认情况下，master 主节点是不参与工作负载的，如果用户在本地配置的是一个单节点集群，那么需要将 master 主节点同时设置成工作节点，命令如下：

```bash
kubectl taint nodes --all node-role.kubernetes.io/master-
```

> 如果想配置多节点的集群，则需要在工作节点安装 kubeadm, kubelet, kubectl 和 docker，然后使用主节点运行 `kubeadm init` 命令所打印出来的 `join` 命令来加入集群：
>
> ```bash
> kubeadm join --token 3a9f97.1cd340e0f34adbbb 192.168.119.101:6443 --discovery-token-ca-cert-hash sha256:64111962ff9df5f566d21f5ad5e9d046487aff4553a738f5c6d1ca0fdc4c69b8
> ```

### 部署附加组件

安装 Flannel 网络插件和 Dashboard 插件。

```bash
# 安装 Flannel
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
# 安装 Dashboard
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard.yaml
# 创建 admin-user 管理员角色
kubectl apply -f /path/to/admin-user-role.yaml
```

admin-user-role.yaml 的定义如下：

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kube-system
```

使用如下命令获取 admin-user 管理员角色的访问口令：

```bash
kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep admin-user | awk '{print $1}')
# 输出如下
Name:         admin-user-token-j6rlh
Namespace:    kube-system
Labels:       <none>
Annotations:  kubernetes.io/service-account.name=admin-user
              kubernetes.io/service-account.uid=111eba61-0d3e-11e8-8763-525400ad3b43

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1025 bytes
namespace:  11 bytes
token:      eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJhZG1pbi11c2VyLXRva2VuLWo2cmxoIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6ImFkbWluLXVzZXIiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiIxMTFlYmE2MS0wZDNlLTExZTgtODc2My01MjU0MDBhZDNiNDMiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6a3ViZS1zeXN0ZW06YWRtaW4tdXNlciJ9.qY-6JaTSqpybnhusvOnNkcw7AE1iHGkBZObO62Vo9XZUEK95NlHaaIj7v8kyNLHujpHuVgTYQPyMBvPzU5JONzdEP5U2K5Q_Z3b6f95vWHGwhNO6qdz5EOxK6-_sma61zL2uQpHb637v0ICfwgFRDE_4as0bq2i5vT-dGCv5jvDjmM-b1mjjeXXC6Z7EYLN-DYp38kIzrxFoQkX0_eNd2fIIeKZshthvGmAw5yEr6opkOnbXyEaqgiMOM9ZhVWO6UYseOEgE0GdDnXgoBeYjFXkZ1FDlkyXihNv6tEU5TQCkPt1iUdeOu_dL6OdvaH91wQjeDxeej3qVyvSJaWAjnA
```

其中 token 值即为对应的授权口令。

### 访问 Dashboard

拷贝 `$HOME/.kube/config` 文件至主机的某一个目录下，比如 `/path/to/kube` ，然后使用 `kubectl proxy` 启动代理，具体命令如下：

```bash
$ kubectl --kubeconfig=/path/to/kube/config proxy
Starting to serve on 127.0.0.1:8001
```

这时，访问 http://localhost:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/ 进入 Dashboard 并使用 上面的 Token（令牌） 进行登录：

![](install-k8s-using-kubeadm/dashboard-login.png)

点击`登录`按钮进入 Dashboard 主页，我们就可以在这里管理 Kubernetes 集群了，Dashboard 主页截图如下所示。

![](install-k8s-using-kubeadm/dashboard-home.png)



## 验证 Kubernetes

本节将快速地向 Kubernetes 集群部署一个示例应用来验证集群是否工作正常。示例应用取自《Kubernetes 本地快速搭建指南》案例。

### 快速部署示例应用

示例应用的 yaml 定义如下，读者可将其另存为文件 demo.yaml，以供后面使用。

```yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: mysql
spec:
  replicas: 1
  selector:
    app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
        containers:
        - name: mysql
          image: mysql
          ports:
          - containerPort: 3306
          env:
          - name: MYSQL_ROOT_PASSWORD
            value: "123456"
---
apiVersion: v1
kind: Service
metadata:
  name: mysql
spec:
  ports:
  - port: 3306
  selector:
    app: mysql
---
apiVersion: v1
kind: ReplicationController
metadata:
  name: myweb
spec:
  replicas: 2
  selector: 
    app: myweb
  template:
    metadata:
      labels:
        app: myweb
    spec:
      containers:
      - name: myweb
        image: kubeguide/tomcat-app:v1
        ports:
        - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: myweb
spec:
  type: NodePort
  ports:
  - port: 8080
    nodePort: 30001
  selector:
    app: myweb
```

使用以下命令部署应用：

```bash
kubectl create -f /path/to/demo.yaml
```



### 访问应用主页

经过以上的步骤，我们的应用就成功地部署在了 Kubernetes 集群上了，我们可以访问 http://<虚拟机 IP>:30001/demo/ 来验证我们的应用了。

![](install-k8s-using-kubeadm/demo.png)

