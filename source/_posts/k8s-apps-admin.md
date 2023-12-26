---
title: K8S 应用启停通用脚本
date: 2020-05-19 04:54:19
categories:
- DevOps
tags:
- k8s
- kubernetes
- deployment
- admin
---

Kubernetes 集群中运行的应用中的每一个服务组件通常是以 Deployment 的形式存在的，本文中提供的管理脚本假设读者部署在 Kubernetes 中的应用服务的 Deployment 对象均已特定的前缀命名，比如 demo，那么集群中可能存在一下的 Deployment 对象：

- demo-register
- demo-gateway
- demo-oauth
- demo-config
- demo-swagger
- ...

在这个前提下，我这里提供了一个脚本可以对这些 deployment 对象进行一键启停操作。举例说明，加绒我的脚本名称为 `k8s-apps.sh` 那么可以执行如下命令：

 ```shell
# 启动所有应用，default 为命名空间
./k8s-apps.sh start default

# 停止所有应用
./k8s-apps.sh stop default
 ```

启停脚本的内容如下：

<!-- more -->

```shell
#!/usr/bin/env bash
#
# Copyright 2020 Liu Hongyu (eliuhy@163.com)
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#
set -e

prefix="demo"        # pattern that names of deployment start with
GRN="\e[32m"        # green color
YLW="\e[33m"        # yellow color
RED="\e[91m"        # red color
RST="\e[39m"        # reset color

# --- common functions definition ---
info() { echo -e "$GRN[INFO]$RST" $@
}
warn() { echo -e "$YLW[WARN]$RST" $@
}
fata() { echo -e "$RED[FATA]$RST" $@; exit 1
}
has_command() { command -v $@ >/dev/null
}

# --- print help ---
show_help() {
echo -e "
USAGE:
  ${GRN}${PROG}${RST} ${YLW}start${RST}|${YLW}stop${RST} -n|--namespace <${YLW}namespace${RST}>
"
exit 0
}

# --- Check dependencies and arguments ---
pre_check() {
  has_command kubectl || fata "This script needs 'kubectl' installed on your computer."
  [ -n "$action"    ] || fata "Need provide at least one action: start|stop"
  [[ $action =~ ^(start|stop)$ ]] || fata "Invalid action '$action'. Use one of the following actions: start|stop"
  [ -n "$namespace" ] || fata "Need provide namespace!"
  kubectl get ns | grep $namespace >/dev/null || fata "'$namespace' not found in current cluster."
  deployments=$(kubectl get deploy -n $namespace | grep ^$prefix | awk '{print $1}')
}

scale() {
  kubectl -n $namespace scale --current-replicas=$2 --replicas=$3 deployment/$1
}

scale_apps() {
  for app in $deployments; do
    scale $app $1 $2
  done
}

{
  PROG="$(basename $0)"
  PARAMS=""
  while [ $# -gt 0 ]; do
    case "$1" in
      -n|--namespace)
        namespace=$2
        shift 2
        ;;
      -h|--help)
        show_help
        ;;
      -*|--*)
        fata "Invalid option '$1'"
        ;;
      *)
        PARAMS="$1 $PARAMS"
        shift
        ;;
    esac
  done
  eval set -- "$PARAMS"
  action=$1

  pre_check

  [[ $action = "start" ]] && scale_apps 0 1 # scale up replicas from 0 to 1
  [[ $action = "stop"  ]] && scale_apps 1 0 # scale down replicas from 1 to 0
  exit 0
}
```

