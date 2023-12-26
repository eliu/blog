# Hexo Source of [eliu.github.io](https://eliu.github.io)

## Instructions

### 1. Clone repository

```shell
$ git clone https://github.com/eliu/blog.git
```

### 2. Install hexo

```shell
$ npm install -g hexo-cli
```

### 3. Install hexo plugins

```shell
$ cd blog/
$ npm install --save hexo-generator-search
$ npm install --save hexo-deployer-git
$ npm install --save hexo-asset-link
```

### 4. Install NexT theme

```shell
$ npm install --save hexo-theme-next
```

### 5. Fill {passkey}

Edit file `_config.yml` and replace `{passkey}` with real passkey.

```yaml
deploy:
  type: git
  repository: https://{passkey}/eliu/eliu.github.io.git
  branch: master
```



