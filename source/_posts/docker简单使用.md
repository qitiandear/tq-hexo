---
title: docker简单使用
date: 2023-01-31 15:54:41
tags:
  - docker
categories:
  - 基础
---

Docker 中有两个重要概念。一个是容器（Container）：容器特别像一个虚拟机，容器中运行着一个完整的操作系统。另一个是镜像（Image）：镜像是一个文件，它是用来创建容器的。

## 安装 Docker

在win10中安装Docker，需要win10支持虚拟化，打开虚拟化后直接运行下载的docker安装包即可安装

## 运行 Docker

接下来我们搭建一个能够托管静态文件的 Nginx 服务器

容器运行程序，而容器哪来的呢？容器是镜像创建出来的。那镜像又是哪来的呢？

镜像是通过一个 Dockerfile 打包来的，它非常像我们前端的package.json文件

所以创建关系为：

``` yaml
Dockerfile: 类似于“package.json”
 |
 V
Image: 类似于“Win7纯净版.rar”
 |
 V
Container: 一个完整操作系统
```

### 创建文件

我们创建一个目录`hello-docker`，在目录中创建一个`index.html`文件，内容为：

``` html
<h1>Hello docker</h1>
```

然后再在目录中创建一个`Dockerfile`文件，内容为：

``` bash
FROM nginx

COPY ./index.html /usr/share/nginx/html/index.html

EXPOSE 80
```

此时，你的文件结构应该是：

``` markdown
hello-docker
  |____index.html
  |____Dockerfile
```

### 打包镜像

文件创建好了，现在我们就可以根据`Dockerfile`创建镜像了！

在命令行中（Windows优先使用PowerShell）键入：

``` bash
cd hello-docker/ # 进入刚刚的目录
docker image build ./ -t hello-docker:1.0.0 # 打包镜像
```

> 注意！Docker 中的选项（Options）放的位置非常有讲究，`docker —help image`和`docker image —help`是完全不同的命令

`docker image build ./ -t hello-docker:1.0.0`的意思是：基于路径`./`（当前路径）打包一个镜像，镜像的名字是`hello-docker`，版本号是`1.0.0`。该命令会自动寻找`Dockerfile`来打包出一个镜像

> Tips: 你可以使用`docker images`来查看本机已有的镜像

不出意外，你应该能得到如下输出：

``` bash
Sending build context to Docker daemon  3.072kB
Step 1/3 : FROM nginx
 ---> 5a3221f0137b
Step 2/3 : COPY ./index.html /usr/share/nginx/html/index.html
 ---> 1c433edd5891
Step 3/3 : EXPOSE 80
 ---> Running in c2ff9ec2e945
Removing intermediate container c2ff9ec2e945
 ---> f6a472c1b0a0
Successfully built f6a472c1b0a0
Successfully tagged hello-docker:1.0.0
```

可以看到其运行了 Dockerfile 中的内容，现在我们简单拆解下：

- `FROM nginx`：基于哪个镜像
- `COPY ./index.html /usr/share/nginx/html/index.html`：将宿主机中的`./index.html`文件复制进容器里的`/usr/share/nginx/html/index.html`
- `EXPOSE 80`：容器对外暴露80端口

### 运行容器

我们刚刚使用 Dockerfile 创建了一个镜像。现在有镜像了，接下来要根据镜像创建容器：

``` bash
docker container create -p 2333:80 hello-docker:1.0.0
docker container start xxx # xxx 为上一条命令运行得到的结果
```

然后在浏览器打开`127.0.0.1:2333`，你应该能看到刚刚自己写的`index.html`内容

在上边第一个命令中，我们使用`docker container create`来创建基于`hello-docker:1.0.0`镜像的一个容器，使用`-p`来指定端口绑定——将容器中的`80`端口绑定在宿主机的`2333`端口。执行完该命令，会返回一个容器ID

而第二个命令，则是启动这个容器

启动后，就能通过访问本机的`2333`端口来达到访问容器内`80`端口的效果了

> 你可以使用`docker container ls`来查看当前运行的容器

当容器运行后，可以通过如下命令进入容器内部：

``` bash
docker container exec -it xxx /bin/bash # xxx 为容器ID
```

原理实际上是启动了容器内的`/bin/bash`，此时你就可以通过`bash shell`与容器内交互了。就像远程连接了SSH一样。

若你编译出的静态站点也是一个 `SPA` 单页应用，需要增加额外的 `Nginx` 配置来保证请求都能打到`index.html`。下边是我写的`vhost.nginx.conf` Nginx 配置文件，将不访问文件的请求全部重定向到`/index.html`：

``` ini
server {
    listen 80;
    server_name localhost;
    location / {
        root /usr/share/nginx/html;
        index index.html index.htm;
        proxy_set_header Host $host;

        if (!-f $request_filename) {
          rewrite ^.*$ /index.html break;
        }

    }

    error_page 500 502 503 504 /50x.html;
    location = /50x.html {
        root /usr/share/nginx/html;
    }
}
```

然后在 Dockerfile 中新加一行，将本机的`vhost.nginx.conf`文件复制到容器的`/etc/nginx/conf.d/pea3nut-info.conf`，让 Nginx 能够读取该配置文件：

``` diff
  FROM nginx

  COPY ./dist/ /usr/share/nginx/html/
+ COPY ./vhost.nginx.conf /etc/nginx/conf.d/pea3nut-info.conf

  EXPOSE 80
```