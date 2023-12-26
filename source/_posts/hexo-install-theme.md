---
title: Hexo Next 主题安装配置
date: 2023-12-26 20:01:43
categories:
tags:
---

本文记录 Hexo 的 NexT 主题的安装和配置过程，使用的是官方仓库 README 的指引进行。

[theme-next/hexo-theme-next: Elegant and powerful theme for Hexo. (github.com)](https://github.com/theme-next/hexo-theme-next)

## 先决条件

- `hexo` ：安装部署可参考文章 [Hexo 快速入门](https://eliu.github.io/2021/03/11/hexo-quickstart/)

## 安装配置

### 安装 NexT 主题

Hexo NexT 主题可通过两种方式安装，一种是直接克隆官方仓库到 {hexo-site}/themes/next 下；一种是通过 npm 安装到 Hexo 主目录。本文使用第二种方式进行安装。

> 提示：`{hexo-site}` 指的是 Hexo 生成的博客项目主目录，例如 `/path/to/my-hexo-blog`

使用 npm 安装 Hexo NexT 主题的命令如下：

```shell
$ cd {hexo-site}
$ npm install hexo-theme-next
```

> npm 国内加速镜像配置：`npm config set registry http://registry.npmmirror.com`

<!-- more -->

### 启用 NexT 主题

编辑 Hexo 项目主目录下的 `_config.yml` 配置文件，并修改以下属性：

```yam
theme: next
```

### 定制 NexT 主题

NexT 主题在默认情况下不需要进行任何的改动，主题就可以直接应用到 Hexo 项目中。但如果你想做一些定制化也是完全可以的。接下来我们拷贝主题的配置文件到 hexo 主目录，文件格式为 `_config.{theme_name}.yml`。对于 NexT 主题，该配置文件的名称应为 `_config.next.yml`。我们准备定制以下内容：

1. 主题风格改为 Gemini
2. 支持暗黑模式
3. 显示首页、标签和分类
4. 个性化我的头像
5. 启用社交网络连接
6. 代码块显示风格
7. GitHub 入口（Follow me on GitHub)
7. 增加本地搜索功能，可参考 [Hexo增加搜索功能 - 简书 (jianshu.com)](https://www.jianshu.com/p/d388119a90ec)

我们打开配置文件 `_config.next.yml` 并需改配置如下所示：

```yaml
scheme: Gemini
darkmode: true
menu:
  home: / || fa fa-home
  #about: /about/ || fa fa-user
  tags: /tags/ || fa fa-tags
  categories: /categories/ || fa fa-th
avatar:
  # Replace the default image and set the url here.
  url: /uploads/avatar.png
  # If true, the avatar will be displayed in circle.
  rounded: true
  # If true, the avatar will be rotated with the cursor.
  rotated: false
social:
  GitHub: https://github.com/eliu || fab fa-github
  E-Mail: mailto:eliuhy@163.com || fa fa-envelope
codeblock:
  # Code Highlight theme
  # All available themes: https://theme-next.js.org/highlight/
  theme:
    light: a11y-light
    dark: a11y-dark
# `Follow me on GitHub` banner in the top-right corner.
github_banner:
  enable: true
  permalink: https://github.com/eliu
```

这里特别注意的是头像的定制化，你需要将你的头像文件，例如 `avatar.png` 上传到 `{hexo-site}/source/uploads/` 目录下，目录结构如下所示：

```
{hexo-site}
└── source
    └── uploads
        └── avatar.png
```

另外，不要启用 motion 动画，有 bug...

```yaml
motion:
  enable: false
```







