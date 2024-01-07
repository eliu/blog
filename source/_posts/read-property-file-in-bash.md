---
title: 在 Bash 中如何读取配置文件
date: 2023-12-20 23:41:00
categories:
- Linux
tags:
- Bash
- Shell
---

## Properties 文件

一种简单的键值对配置文件形式是 properties 文件，可以被 Java 语言原生支持读取。我们可以使用它为项目进行简单的配置管理，典型的 Properties 文件如下所示：

```properties
# config.properties
# Valid values are info, verbose, debug
logging.level=info

setup.hosts.enabled=false

installer.maven.enabled=true
installer.git.enabled=true
```

那么像这种文件，我们如何在 Bash 环境下进行读取和使用呢？本文接下来将逐步分析实现的过程。

<!-- more -->

## 思路分析

我们以刚刚所展示的 Properties 文件实例来进行分析，不难发现这种文件一般包含以下几个特点：

1. 允许有注释行，该行以 `#` 开头且不包含实际的配置项
2. 允许有空行
3. 配置项以等号 `=` 作为分割符，左侧为属性名称，右侧为值

好了，知道所有的特征之后，接下来我们着手进行代码的实现。

## 代码实现

### 过滤注释行和空行

这两种情况都属于无效行，应该被过滤剔除。由于我们是在 GNU/Linux 和 Bash 环境下进行实现，那么可以想到的方法是先把文件读到内存或者管道中，然后逐层进行过滤。读取文件我们可以使用 `cat` 命令，而对于过滤无效行这种任务的话，GNU/Linux 下有很多工具可以用，如 `grep` 和  `sed` 等等。我们此处选择的是 `sed` 因为它允许通过 `-e` 参数来指定多条正则表达式条件进行操作。接下来我们对正则表达式进行逐个击破。

判断注释行：以 `#` 开头的全部删除，正则表达式为 `/^#/`

<div style="background-color:#EEE;text-align:center"><img src="properties_pattern_01.png" /></div>

判断空行：仅包含0个或多个空格符的行，正则表达式为 `/^\s*$/`

<div style="background-color:#EEE;text-align:center"><img src="properties_pattern_02.png" /></div>

最后，在 sed 中剔除命中的行时使用操作符 `d` ，例如删除注释行，我们可以写成 `/^#/d`，以下就是我们的最终的命令：

```bash
cat config.properties | sed -e '/^#/d' -e '/^\s*$/d'
```

验证结论：

```bash
$ cat config.properties | sed -e '/^#/d' -e '/^\s*$/d'
logging.level=info
setup.hosts.enabled=false
installer.maven.enabled=true
installer.git.enabled=true
```

### 解析配置项

配置项是 `key=value` 形式，我们可以循环管道中过滤后的结果中的每一行，然后按等号 `=` 进行分割。我们可以利用 `while` 和 `read` 结合起来进行读取，read 命令可以以环境变量 `IFS` 的值作为分隔符进行拆解和读取到指定的变量中。例如下面的代码：

```bash
$ IFS='=' read -r prop value
# 输入 a=b 然后回车
$ echo "$prop -> $value"
a -> b
```

结合前面的成果和 while 循环，我们目前的解析过程可以写成：

```bash
$ cat config.properties \
| sed -e '/^#/d' -e '/^\s*$/d' \
| while IFS='=' read -r prop value; do echo "$prop -> $value"; done
```

输出结果为：

```
logging.level -> info
setup.hosts.enabled -> false
installer.maven.enabled -> true
installer.git.enabled -> true
```

看起来符合我们的预期，因为我们使用 `echo` 已经输出了我们想要的读取的值，完美！然而，真的是这样么？到目前为止，我们只是解析出来了配置项的值，但是还没有进行存储以备其他过程进一步使用。

### 值不见了？

接下来我们来声明一个关联数组，我们准备用它来存储所有的配置项到内存中，以便随时使用。以下是我们的代码：

```bash
#!/bin/bash
declare -A config

cat config.properties \
| sed -e '/^#/d' -e '/^\s*$/d' \
| while IFS='=' read -r prop value; do
	config[$prop]=$value
do

echo ${config[@]}

```

读者可以实际执行一下，结果竟然是**没有任何信息输出**！这里给出根因：

> 根因：`...| while ... do; ... done` 管道命令 `|` 会调用内核的 fork 产生一个子进程，在子进程中所进行的while 循环中的任何的变量赋值操作的作用域仅在子进程内部，语句执行完之后，子进程也随之被销毁。所以尝试在此处进行的赋值操作没有任何效果。

好了，根因我们知道了，那么我们要如何解决呢？对，我们不用管道，我们要使用 bash 中的“输入转向”功能来实现我们的目的，结构如下所示：

```bash
while ...; do
  ...
done < /path/to/file
```

但是这里 while 循环读取的是输入设备是文件，而我们在读取文件之后是要做系列过滤加工的，上面的这种形式显然不能满足要求。别着急，Bash 还为我们提供了另外一种产生“输入”的方式：`<(command)`，那么改造后的形式就是：

```bash
while ...; do
  ...
done < <(command)
```

套用到我们的需求之后，最终形成的代码如下：

```bash
#!/bin/bash
declare -A config

while IFS='=' read -r prop value; do
  config[$prop]=$value
done < <(cat config.properties | sed -e '/^\s*$/d' -e '/^#/d')

echo ${!config[@]}
# 输出结果：installer.git.enabled logging.level setup.hosts.enabled installer.maven.enabled
echo ${config[@]}
# 输出结果：true info false true
```

至此，所有问题都已解决。在我的项目 [devbox](https://github/eliu/devbox) 中，配置管理模块 `config.sh` 就是使用上面的解决方案读取的配置文件。

[devbox/lib/modules/config.sh at master · eliu/devbox (github.com)](https://github.com/eliu/devbox/blob/master/lib/modules/config.sh)

## 鸣谢

正则表达式可视化图片由 [Regulex - JavaScript Regular Expression Visualizer](https://jex.im/regulex/#) 生成，特此感谢！

End~











