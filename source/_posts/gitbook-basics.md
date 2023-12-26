---
title: GitBook 基本操作
date: 2020-08-07 17:41:00
categories:
- tools
tags:
- gitbook
- git
---

## 写在前面

本文主要的目的是介绍如何快速的在本地快速启动一个 GitBook 项目并使用浏览器进行阅读。内容比较简单，仅用作备忘，以便后面查阅。

<!-- more -->

## 软件安装

gitbook 命令行工具基于 Node.js 平台，并且只兼容 `10.x` 版本，所以读者需要访问 https://nodejs.org/dist/latest-v10.x/ 下载对应平台的 Node.js 安装文件进行安装。之后运行以下命令安装 `gitbook-cli`:

> 注意： Windows 用户请打开 PowerShell 执行安装命令。

```shell
# 确认 Node.js 版本
$ node -v
v10.22.0

# 安装 gitbook-cli
$ npm install -g gitbook-cli

# 确认 gitbook-cli 版本
$ gitbook ls
GitBook Versions Installed:

    * 3.2.3

Run "gitbook update" to update to the latest version.
```



## 启动 GitBook 项目

请访问 https://www.npmjs.com/package/gitbook 详细了解如何构建一个符合 GitBook 的项目。进入 gitbook 项目主目录，执行 `gitbook serve` 在本地启动服务：

 ```shell
$ gitbook serve
info: 7 plugins are installed
info: loading plugin "livereload"... OK
info: loading plugin "highlight"... OK
info: loading plugin "search"... OK
info: loading plugin "lunr"... OK
info: loading plugin "sharing"... OK
info: loading plugin "fontsettings"... OK
info: loading plugin "theme-default"... OK
info: found 82 pages
info: found 236 asset files
info: >> generation finished with success in 14.0s !

Starting server ...
Serving book on http://localhost:4000
 ```

最后根据提示访问 http://localhost:4000/ 在浏览器查看文档。