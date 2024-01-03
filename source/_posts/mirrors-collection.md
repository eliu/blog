---
title: 国内软件和镜像源集合
date: 2023-11-21 21:48:27
categories:
- DevOps
tags:
- 国内源
- 镜像源
---

本文将持续更新目前已知的可以设置的国内软件源以及镜像源的设置方法，方便作者自己备查和减轻开发人员搭建环境的阻碍。

## GNU/Linux 发行版软件源

### CentOS 7

```shell
# run as root or sudo
rm -fr /etc/yum.repos.d/*.repo
curl -sSL https://mirrors.aliyun.com/repo/Centos-7.repo -o /etc/yum.repos.d/CentOS-Base.repo
sed -i -e '/mirrors.cloud.aliyuncs.com/d' \
       -e '/mirrors.aliyuncs.com/d' \
       /etc/yum.repos.d/CentOS-Base.repo
curl -sSL https://mirrors.aliyun.com/repo/epel-7.repo -o /etc/yum.repos.d/epel.repo
curl -sSL https://mirrors.aliyun.com/ius/ius-7.repo -o /etc/yum.repos.d/ius.repo
sed -i 's repo.ius.io mirrors.aliyun.com/ius/ g' /etc/yum.repos.d/ius.repo
yum clean all
yum makecache fast
```

<!-- more -->

### RockyLinux

适用 RockyLinux 所有主流版本

配置指引：https://developer.aliyun.com/mirror/rockylinux

```shell
# run as root or sudo
sed -i.bak \
    -e 's|^mirrorlist=|#mirrorlist=|g' \
    -e 's|^#baseurl=http://dl.rockylinux.org/$contentdir|baseurl=https://mirrors.aliyun.com/rockylinux|g' \
    /etc/yum.repos.d/rocky*.repo
```

之后运行 `sudo dnf makecache` 更新缓存。

### AlmaLinux

```shell
# https://developer.aliyun.com/mirror/almalinux
sed -i.bak \
	-e 's|^mirrorlist=|#mirrorlist=|g' \
	-e 's|^#\s*baseurl=https\?://repo.almalinux.org|baseurl=https://mirrors.aliyun.com|g' \
	/etc/yum.repos.d/almalinux*.repo
```

之后运行 `sudo dnf makecache` 更新缓存。

## 设置 DNS

刚刚安装完某个 GNU/Linux 发行版之后，发现软件包安装时跑不动或者经常超时，这时我们需要设置一下上网的网卡的DNS。

```shell
# run as root or sudo
network_uuid=

# 找到自动上网的那个网卡
for uuid in $(nmcli -get-values UUID conn show --active); do
  if [ "auto" = "$(nmcli -terse conn show uuid $uuid | grep ipv4.method | awk -F '[:/]' '{print $2}')" ]; then
    network_uuid=$uuid
  fi
done

# 增加 DNS
nmcli con mod $network_uuid +ipv4.dns 114.114.114.114
nmcli con mod $network_uuid +ipv4.dns 8.8.8.8
# 重启网络
systemctl restart NetworkManager
```

## npm 软件源

```shell
$ npm config set registry https://registry.npmmirror.com
```

## Python PIP 软件源

安装的软件包的时候直接用 `-i` 制定源即可：

```shell
$ pip3 install some-package -i https://mirrors.aliyun.com/pypi/simple
```

## Maven 仓库

阿里开源镜像站：[apache-maven安装包下载_开源镜像站-阿里云 (aliyun.com)](https://mirrors.aliyun.com/apache/maven/)

## 容器相关

### Podman

```shell
# run as root or sudo
mv /etc/containers/registries.conf /etc/containers/registries.conf.bak
cat > /etc/containers/registries.conf <<< EOF
unqualified-search-registries = ["docker.io"]

[[registry]]
prefix = "docker.io"
insecure = false
blocked = false
location = "docker.io"
[[registry.mirror]]
location = "hub-mirror.c.163.com"
[[registry.mirror]]
location = "registry.docker-cn.com"
EOF
```

### Docker

```shell
# run as root or sudo
# 使用阿里云镜像源安装 docker
# https://yq.aliyun.com/articles/110806?spm=a2c4e.11153940.0.0.108e435aDMp0n2&p=4#comments
{
	export VERSION="17.09" # docker ce version
	curl -sSL https://get.docker.com | bash -s docker --mirror Aliyun	
}

# 设置国内 docker 镜像源
mv /etc/docker/daemon.json /etc/docker/daemon.json.bak
cat > /etc/docker/daemon.json <<< EOF
{
    "registry-mirrors": [
        "https://8km017g6.mirror.aliyuncs.com",
        "https://hub-mirror.c.163.com",
        "https://registry.docker-cn.com"
    ]
}
EOF
```

### Minikube

[AliyunContainerService/minikube: 普大喜奔，官方Minikube提供了完整对国内用户支持，完美支持Addon组件。 建议参考 https://yq.aliyun.com/articles/221687 或 https://github.com/AliyunContainerService/minikube/wiki 最新支持minikube v1.24.0](https://github.com/AliyunContainerService/minikube)

Wiki: [Home · AliyunContainerService/minikube Wiki (github.com)](https://github.com/AliyunContainerService/minikube/wiki)

引用 Wiki 的一段话：

> 为了方便大家开发和体验Kubernetes，社区提供了可以在本地部署的开发环境 [Minikube](https://github.com/kubernetes/minikube)。由于网络访问原因，很多朋友无法直接使用minikube进行实验。在v1.24.0的官方 Minikube 中，已经合并了由阿里云团队支持的方案，可以帮助大家利用阿里云的服务来获取所需Docker镜像，二进制文件和配置，也可以完美支持 Minikube 丰富的 addon 组件！

### Kubernetes

适用国内网络环境下安装 K8S 集群的工具：[TimeBye/kubeadm-ha](https://github.com/TimeBye/kubeadm-ha)



















