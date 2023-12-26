---
title: 为域名快速生成自签名证书
date: 2020-05-19 03:53:18
categories: 
- DevOps
tags:
- self-signed certificate
- 自签名证书
---

本文在文章 [Configure HTTPS Access to Harbor](https://goharbor.io/docs/2.0.0/install-config/configure-https/) 的基础上使用 Bash 进行了简单的封装，可以为指定的域名一键生成自签名证书。例如域名 example.com 生成的自签名证书将匹配以下地址：

- example.com
- *.example.com



<!-- more -->

新建文件 `gencert.sh` ，编辑并加入以下内容：

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

DOMAIN="$1"
WORK_DIR="$(mktemp -d)"

if [ -z "$DOMAIN" ]; then
  echo "Domain name needed."
  exit 1
fi

echo "Temporary working dir is $WORK_DIR "
echo "Gernerating cert for $DOMAIN ..."

#
# Fix the following error:
# --------------------------
# Cannot write random bytes:
# 139695180550592:error:24070079:random number generator:RAND_write_file:Cannot open file:../crypto/rand/randfile.c:213:Filename=/home/eliu/.rnd
#
[ -f $HOME/.rnd ] || dd if=/dev/urandom of=$HOME/.rnd bs=256 count=1

openssl genrsa -out $WORK_DIR/ca.key 4096

openssl req -x509 -new -nodes -sha512 -days 3650 \
  -subj "/C=CN/ST=Beijing/L=Beijing/O=example/OU=Personal/CN=$DOMAIN" \
  -key $WORK_DIR/ca.key \
  -out $WORK_DIR/ca.crt

openssl genrsa -out $WORK_DIR/server.key 4096

openssl req -sha512 -new \
  -subj "/C=CN/ST=Beijing/L=Beijing/O=example/OU=Personal/CN=$DOMAIN" \
  -key $WORK_DIR/server.key \
  -out $WORK_DIR/server.csr

cat > $WORK_DIR/v3.ext <<-EOF
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
extendedKeyUsage = serverAuth
subjectAltName = @alt_names

[alt_names]
DNS.1=$DOMAIN
DNS.2=*.$DOMAIN
EOF

openssl x509 -req -sha512 -days 3650 \
  -extfile $WORK_DIR/v3.ext \
  -CA $WORK_DIR/ca.crt -CAkey $WORK_DIR/ca.key -CAcreateserial \
  -in $WORK_DIR/server.csr \
  -out $WORK_DIR/server.crt

openssl x509 -inform PEM -in $WORK_DIR/server.crt -out $WORK_DIR/$DOMAIN.cert

mkdir -p ./$DOMAIN
cp $WORK_DIR/server.key $WORK_DIR/server.crt ./$DOMAIN

```

假设我们要为 example.com 生成证书，执行如下命令：

```shell
./gencert.sh example.com
```

生成的后的目录结构如下：

```
.
├── example.com
│   ├── server.crt
│   └── server.key
└── gencert.sh
```

