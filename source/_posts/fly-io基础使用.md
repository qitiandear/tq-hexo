---
title: fly.io基础使用
date: 2023-01-30 15:05:21
tags:
  - fly.io
  - docker
categories:
  - 工具
---

fly.io 是一个容器化的部署平台，只需要一个`Dockerfile`文件就能部署代码到fly.io 的服务器上，同时还自动生成域名。

- 有免费使用的额度。
- 自动生成域名。比如你创建一个名字叫`my_demo`的App，那么部署完成后，就会生成`my_demo.fly.dev`的域名，可以全球访问，不用自己单独买域名了。
- 可以 `SSH` 连接进入服务器。部署完成后，可以通过`flyctl ssh console` 命令登录部署的服务器，所以相当于你有了一台免费的VPS，可以做你想做的任何事情。
- 部署简单，采用`flyctl` 命令集合统一部署;支持各种语言的各种框架来搭建部署环境，能自动识别当前目录下代码所采用的是哪个框架，自动部署。

## flyctl工具安装

### macOS

``` bash
brew install flyctl
```

### Linux

``` bash
curl -L https://fly.io/install.sh | sh
```

### Windows

``` bash
iwr https://fly.io/install.ps1 -useb | iex
```

如果执行`flyctl version` 不报错，就说明安装成功了。

## 创建并登录账号

### 创建账号:

``` bash
flyctl auth signup
```

### 登录账号:

``` bash
fly auth login
```

## win10的wsl2环境安装docker

> 使用flyctl最好是本地安装docker，不然构建特别容易出错，本人使用的是openEuler(wsl2)

### Docker Desktop for windows方式

Docker 也专门开发了可以使用 WSL2 中的 Docker 守护进程的桌面管理程序, 打开 Docker Desktop WSL2 backend 页面，下载最新的 Docker Desktop for Windows 程序 ，建议下载stable版本。下载地址：https://www.docker.com/products/docker-desktop

启动Docker Desktop for Windows，点击“设置”按钮，启用基于WSL2的引擎复选框（Use the WSL 2 based engine）

![](https://cdn.jsdelivr.net/gh/qitiandear/qtpic/20230130153846.png)

在 Resources 的WSL Integration中设置要从哪个 WSL2 发行版中访问 Docker，如下图使用的是openEuler

![](https://cdn.jsdelivr.net/gh/qitiandear/qtpic/20230130153942.png)

重启 Docker desktop for Windows，重启完成后我们就可以在 WSL2里面使用 docker 命令了

![](https://cdn.jsdelivr.net/gh/qitiandear/qtpic/20230130154048.png)

## 部署静态网站

我作为一个前端部署一个自己的静态网站，使用官方提供的解决方案gostatic部署前端静态页面

### 创建Dockerfile文件

``` bash
FROM pierrezemb/gostatic
COPY ./public/ /srv/http/
```

### docker构建我们的镜像

docker build在Dockerfile同级目录下，-t 后是自己给镜像取的名字

``` bash
docker build -t gitbook:v1 .
```

### fly发布部署

使用`fly launch`，生成`fly.toml`

``` bash
fly launch
----------------------------------------------------
Creating app in /Users/chris/trystatic/hello-static
Scanning source code
Detected a Dockerfile app
? App Name (leave blank to use an auto-generated name): gostatic-example
Automatically selected personal organization: Chris Nicoll
? Select region: ewr (Secaucus, NJ (US))
Created app gostatic-example in organization personal
Wrote config file fly.toml
? Would you like to setup a Postgresql database now? No
? Would you like to deploy now? No
Your app is ready. Deploy with `flyctl deploy`
```

### 修改fly.toml文件

goStatic默认监听8043 ，但是生成的fly.toml默认是8080，我们需要修改一下

``` bash
[[services]]
  http_checks = []
  internal_port = 8043
  processes = ["app"]
  protocol = "tcp"
  script_checks = []
```

### 部署

`--local-only`只用本地docker构建的镜像，推送到fly.io服务

``` bash
flyctl deploy --local-only
```

部署成功我们可以看到：

``` bash
==> Monitoring deployment

 1 desired, 1 placed, 1 healthy, 0 unhealthy [health checks: 1 total, 1 passing]
--> v0 deployed successfully
```

### 查看我们的网站

``` bash
flyctl open
```

![](https://cdn.jsdelivr.net/gh/qitiandear/qtpic/20230130160331.png)