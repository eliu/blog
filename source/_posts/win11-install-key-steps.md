---
title: 安装Win11的详细步骤
date: 2025-05-17 20:08:14
categories:
tags:
---

本文介绍如何在非Windows主机环境下虚拟化安装和配置 Windows 11 操作系统。 

## 关键步骤

1. 如何在无激活码的情况下完成系统安装

2. 如何绕过微软账号登陆验证

## 实验环境

- 主机操作系统：macOS Sequoia (15.4.1)

- 虚拟机软件：VMware Fusion 13.6.3

- Windows 安装镜像：Win11_24H2_Chinese_Simplified_x64.iso

## 详细安装步骤

<!-- more -->

### 一、获取 Windows 11 磁盘映像（ISO)

访问以下地址，根据指引从微软官方下载win11镜像文件。

https://www.microsoft.com/zh-cn/software-download/windows11

![](win11-install-key-steps/2025-05-17-20-23-44-image.png)

我们选择直接下载 iso 映像文件，所以直接跳转到`下载适用于 x64 设备的 Windows 11 磁盘映像 (ISO)`章节，在下拉框中选择`Windows 11（适用于 x64 设备的多版本 ISO）`。

![](win11-install-key-steps/2025-05-17-20-59-07-image.png)

点击`立即下载`按钮之后，页面会刷新出`选择产品语言`选项，选择`简体中文`，点击`确定`按钮完成下载。

![](win11-install-key-steps/2025-05-17-21-03-19-image.png)

最终下载的文件名类似于 `Win11_24H2_Chinese_Simplified_x64.iso`，有了系统映像文件，接下来我们开始执行安装过程。

### 二、创建虚拟机

启动 `VMware Fusion`，如图所示，点选`新建`：

![](win11-install-key-steps/2025-05-17-21-08-24-image.png)

之后选择`从光盘或映像中安装`，点击`继续`按钮。

![](win11-install-key-steps/2025-05-17-21-10-39-image.png)

选择我们刚刚下载的映像文件之后，点击`继续`按钮

![](win11-install-key-steps/2025-05-17-21-11-37-image.png)

固件类型保持默认选项，即 UEFI，点击`继续`按钮

![](win11-install-key-steps/2025-05-17-21-12-35-image.png)

此处选择`Partial Encryption`，点击`Auto Generate Password`按钮将会自动生成加密密码，密码我们不需要刻意去记录，直接勾选`Remember Password and store it in Mac's Keychain` 选项即可，最后点击`继续`按钮。

![](win11-install-key-steps/2025-05-17-21-15-22-image.png)

下图是我根据向导默认生成的虚拟机配置汇总，如果想要调整，可以继续点击`自定设置`按钮进行配置，最后点击`完成`按钮。

![](win11-install-key-steps/2025-05-17-21-17-04-image.png)

### 三、安装Windows 11

双击刚刚我们创建的虚拟机之后，会显示如下信息，此时要快速点击回车按钮进入光驱引导，否则会显示timeout超时。

![](win11-install-key-steps/2025-05-17-21-19-21-image.png)

语言设置保持默认，点击下一步：

![](win11-install-key-steps/2025-05-17-21-22-18-image.png)

键盘设置默认，点击下一步：

![](win11-install-key-steps/2025-05-17-21-22-57-image.png)

选择`安装 Windows11`，勾选 I agree....，点击下一步：

![](win11-install-key-steps/2025-05-17-21-23-55-image.png)

#### 产品秘钥步骤

这个步骤比较重要，如果手上暂时没有可用的产品秘钥，就在左下角勾选`我没有产品秘钥`。

![](win11-install-key-steps/2025-05-17-21-25-38-image.png)

接下来选择安装的 Windows 版本，此处选择`Windows 11 专业版`，点击下一步。

![](win11-install-key-steps/2025-05-17-21-27-15-image.png)

接受条款，点击下一步。

![](win11-install-key-steps/2025-05-17-21-28-06-image.png)

在Windows的安装位置步骤中，全部保持默认即可，稍后会自动格式化磁盘，点击下一步。

![](win11-install-key-steps/2025-05-17-21-30-08-image.png)

此处已准备就绪，我们点击`安装`按钮开始执行安装。

![](win11-install-key-steps/2025-05-17-21-31-57-image.png)

执行安装过程略。

![](win11-install-key-steps/2025-05-17-21-33-07-image.png)

### 四、配置Windows 11

#### 绕过微软账号登陆验证

在安装完成之后就到了配置Windows 11的配置向导，首先弹出的国家（地区）的设置向导

![](win11-install-key-steps/2025-05-17-21-51-10-image.png)

从现在开始，我们就可以实施`绕过微软账号登陆`的步骤了，具体如下：

1. 断开网络连接（可直接禁掉主机的wifi）

2. 按 `CTRL` + `F10` 调出终端窗口

3. 输入命令：`oobe\bypassnro.cmd`

4. 重启虚拟机（第3步执行完应该会自动重启）

之后正常配置到网络连接步骤后，在此处要点击`我没有 Internet 连接`，之后就可以设置本地账号了！

![](win11-install-key-steps/2025-05-17-21-59-59-image.png)

此时输入你的本地账号名称：

![](win11-install-key-steps/2025-05-17-22-01-32-image.png)

再输入本地账户的密码：

![](win11-install-key-steps/2025-05-17-22-02-06-image.png)

三个安全问题：

![](win11-install-key-steps/2025-05-17-22-02-52-image.png)

隐私设置，全部不勾选，之后点击接受：

![](win11-install-key-steps/2025-05-17-22-04-18-image.png)

#### 同意个人数据跨境传输

这个选项一般点击`下一步`按钮即可，但是如果你不想同意这么做，可以参考下面的指引进行操作：

[跳过电脑个人数据跨境传输提示的方法](https://baijiahao.baidu.com/s?id=1828346126065690979&wfr=spider&for=pc)

![](win11-install-key-steps/2025-05-17-22-09-46-image.png)

好了，一个完整的 Windows 11操作系统就已经安装完成了！至于产品激活码之类的事情，就不过多讲了，大家自行发挥。

![](win11-install-key-steps/2025-05-17-22-11-25-image.png)
