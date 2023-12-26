---
title: macOS 下 node-gyp rebuild failed 的解决方法
date: 2020-02-27 19:36:29
tags: 
- 'macOS Catalina'
- 'node-gyp rebuild'
- 'No Xcode or CLT version detected'
---



## 0x00 环境信息

在 macOS Catalina 操作系统下（多数是从 High Sierra 升级过来的）

```bash
$ sw_vers
ProductName:	Mac OS X
ProductVersion:	10.15.3
BuildVersion:	19D76
$ node -v
v13.8.0
$ npm -v
6.13.7
```

<!-- more -->

## 0x01 现象和错误信息

执行 `npm install -g xxxx` 或者 `yarn` 命令的时候出现如下面类似的报错信息：

```
> fsevents@1.2.11 install /usr/local/lib/node_modules/hzero-cli/node_modules/fork-ts-checker-webpack-plugin-alt/node_modules/fsevents
> node-gyp rebuild

No receipt for 'com.apple.pkg.CLTools_Executables' found at '/'.

No receipt for 'com.apple.pkg.DeveloperToolsCLILeo' found at '/'.

No receipt for 'com.apple.pkg.DeveloperToolsCLI' found at '/'.

gyp: No Xcode or CLT version detected!
gyp ERR! configure error
gyp ERR! stack Error: `gyp` failed with exit code: 1
gyp ERR! stack     at ChildProcess.onCpExit (/usr/local/lib/node_modules/npm/node_modules/node-gyp/lib/configure.js:351:16)
gyp ERR! stack     at ChildProcess.emit (events.js:321:20)
gyp ERR! stack     at Process.ChildProcess._handle.onexit (internal/child_process.js:275:12)
gyp ERR! System Darwin 19.3.0
gyp ERR! command "/usr/local/Cellar/node/13.8.0/bin/node" "/usr/local/lib/node_modules/npm/node_modules/node-gyp/bin/node-gyp.js" "rebuild"
gyp ERR! cwd /usr/local/lib/node_modules/hzero-cli/node_modules/fork-ts-checker-webpack-plugin-alt/node_modules/fsevents
gyp ERR! node -v v13.8.0
gyp ERR! node-gyp -v v5.0.7
gyp ERR! not ok
```



## 0x02 分析和解决办法

以上错误大概率是 `Xcode Command Line Tools` 没有安装或者升级到 `macOS Catalina` 之后 `Xcode Command Line Tools` 安装异常所致。 所以接下来我们来检查一下这个命令行工具的安装情况。

### 检查 Xcode 命令行工具

首先，运行以下命令检查命令行工具是否已安装：

```bash
xcode-select --install
```

若返回以下信息，说明已经安装，否则会提示你继续完成这个命令行工具的安装过程。

```
xcode-select: error: command line tools are already installed, use "Software Update" to install updates
```

接下来，运行以下命令来确认已安装的命令行工具是否正常：

```bash
/usr/sbin/pkgutil --packages | grep CL
```

若返回以下信息，则说明以满足 `node-gyp` 的要求，如果`没有返回任何信息`，则可判断 Xcode 命令行工具出现了异常。

```
com.apple.pkg.CLTools_Executables
com.apple.pkg.CLTools_SDK_macOS1015
com.apple.pkg.CLTools_SDK_macOS1014
com.apple.pkg.CLTools_macOS_SDK
```

### 修复 Xcode 命令行工具

若以上检查步骤出现异常情况，则尝试以下步骤重新安装 Xcode 命令行工具，之后重复执行检查的步骤确认有信息返回。

```bash
sudo rm -rf $(xcode-select -print-path)
xcode-select --install
```

最后重新执行你的 npm 或者 yarn 命令，理论上就可以恢复正常了，如果以上步骤仍无法解决你的问题，请直接打开 https://github.com/nodejs/node-gyp/blob/master/macOS_Catalina.md 文档详细阅读 `node-gyp` 的作者所介绍的诊断方法。



## 0x03 参考链接

- https://github.com/nodejs/node-gyp#on-macos
- https://github.com/nodejs/node-gyp/blob/master/macOS_Catalina.md