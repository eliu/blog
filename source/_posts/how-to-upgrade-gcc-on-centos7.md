---
title: CentOS 7 升级 gcc 版本
date: 2025-09-30 11:51:14
categories: 
- Linux
tags: 
- centos7
- gcc
---



## 安装配置 centos-release-scl

### 安装 centos-release-scl

```shell
sudo yum install centos-release-scl
```

### 配置 repo 文件

调整 `/etc/yum.repos.d/CentOS-SCLo-scl.repo`

```toml
[centos-sclo-sclo]
name=CentOS-7 - SCLo sclo
baseurl=https://mirrors.aliyun.com/centos/7/sclo/x86_64/sclo/
# mirrorlist=http://mirrorlist.centos.org?arch=$basearch&release=7&repo=sclo-sclo
gpgcheck=1
enabled=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-SIG-SCLo
```

调整 `/etc/yum.repos.d/CentOS-SCLo-scl-rh.repo`，注意 `baseurl` 地址末尾的 `/rh/`

```toml
[centos-sclo-rh]
name=CentOS-7 - SCLo rh
baseurl=https://mirrors.aliyun.com/centos/7/sclo/x86_64/rh/
# mirrorlist=http://mirrorlist.centos.org?arch=$basearch&release=7&repo=sclo-rh
gpgcheck=1
enabled=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-SIG-SCLo
```

### 刷新缓存

```shell
yum clean all
yum makecache
```

<!-- more -->

## 安装配置 devtoolset

### 安装 devtoolset

devtoolset 的安装规律如下：

- `gcc 7.x` 对应 `devtoolset-7-gcc*`
- `gcc 8.x` 对应 `devtoolset-8-gcc*`
- `gcc 9.x` 对应 `devtoolset-9-gcc*`
- `gcc 10.x` 对应 `devtoolset-10-gcc*`

以此类推。例如安装 gcc 9.x，运行以下命令执行安装：

```
sudo yum install -y devtoolset-9-gcc*
```

###  激活 devtoolset

你可以一次性安装多个版本的 devtoolset 需要的时候执行下面的激活命令，此处以 devtoolset-9 为例：

```
scl enable devtoolset-9 bash
```

如果不想每次都要执行激活命令的话可以在 `$HOME/.bashrc` 下加入以下命令：

```
echo 'source /opt/rh/devtoolset-9/enable' >> ~/.bashrc
```

### 确认 GCC 版本

```shell
$ gcc -v
Using built-in specs.
COLLECT_GCC=gcc
COLLECT_LTO_WRAPPER=/opt/rh/devtoolset-9/root/usr/libexec/gcc/x86_64-redhat-linux/9/lto-wrapper
Target: x86_64-redhat-linux
Configured with: ../configure --enable-bootstrap --enable-languages=c,c++,fortran,lto --prefix=/opt/rh/devtoolset-9/root/usr --mandir=/opt/rh/devtoolset-9/root/usr/share/man --infodir=/opt/rh/devtoolset-9/root/usr/share/info --with-bugurl=http://bugzilla.redhat.com/bugzilla --enable-shared --enable-threads=posix --enable-checking=release --enable-multilib --with-system-zlib --enable-__cxa_atexit --disable-libunwind-exceptions --enable-gnu-unique-object --enable-linker-build-id --with-gcc-major-version-only --with-linker-hash-style=gnu --with-default-libstdcxx-abi=gcc4-compatible --enable-plugin --enable-initfini-array --with-isl=/builddir/build/BUILD/gcc-9.3.1-20200408/obj-x86_64-redhat-linux/isl-install --disable-libmpx --enable-gnu-indirect-function --with-tune=generic --with-arch_32=x86-64 --build=x86_64-redhat-linux
Thread model: posix
gcc version 9.3.1 20200408 (Red Hat 9.3.1-2) (GCC)
```



## 自动化脚本

按照以上的实践步骤形成的自动升级的附加脚本：

```bash
#!/usr/bin/env bash
source /etc/os-release

if [[ $VERSION_ID != "7" ]] && [[ $ID != 'centos' ]]; then
  echo "Error: This addon can only be applied on centos7. Exiting..."
  exit 1
fi

SCLO_BASEURL="https://mirrors.aliyun.com/centos/${VERSION_ID}/sclo/x86_64/sclo/"
RH_BASEURL="https://mirrors.aliyun.com/centos/${VERSION_ID}/sclo/x86_64/rh/"
DEVTOOLSET_VERSION=9
sudo yum install -y centos-release-scl
sudo sed -i.bak "s~#\s*baseurl=.*mirror.*~baseurl=${SCLO_BASEURL}~; s~^\(mirrorlist\)~# \1~" \
	/etc/yum.repos.d/CentOS-SCLo-scl.repo
sudo sed -i.bak "s~#\s*baseurl=.*mirror.*~baseurl=${RH_BASEURL}~; s~^\(mirrorlist\)~# \1~" \
	/etc/yum.repos.d/CentOS-SCLo-scl-rh.repo
yum clean all
yum makecache
sudo yum install -y "devtoolset-${DEVTOOLSET_VERSION}-gcc*"
echo "source /opt/rh/devtoolset-${DEVTOOLSET_VERSION}/enable" >> ~/.bashrc

```



## 参考文档

- [CentOS 7升级gcc，2025年无坑版](https://juejin.cn/post/7506436235511988235)

