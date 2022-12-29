---
title: Cargo介绍
date: 2022-12-29 14:33:06
tags:
  - rust
  - cargo
categories:
  - 介绍
---

cargo是一个构建系统和包管理工具,用来管理rust工程和处理许多任务（构建代码、下载包）,cargo在安装rust的环境时，会自动安装，所以这里的话，直接调用

``` bash
# 可以直接看到对应的cargo版本
cargo --version
```

## cargo工程的创建

1. 创建

使用`cargo new [projectName]`来创建一个新的工程

``` bash
cargo new blog_cargo
```

![](https://kun.nwyp123.com/20221229144156.png)

2. 目录解析

创建的目录包括：

- cargo.toml：用于定义包和信息`[package]`和依赖`[dependencies]`
- 源文件目前是放在src的目录下的

3. toml文件中需要注意的点

[package]下面的一些配置:

- build：用于指定build来指定这个包用源码的方式编译

> 比如在rust中引用了比如C的库，这些东西编译就放在一个build.rs中，会通过源码的方式进行编译

- include和exclude：用来指定那些包需要被打包或发布，可以通过`carge package --list`来查看
- publish: 用来指定包是否发布到包仓库，`保护包的私有性`

## 使用cargo构建和跑rust工程

1. 构建rust工程

``` bash
cargo build
```

2. 运行cargo工程

构建成功之后的文件都在`target/debug/hello_cargo`下，直接运行目录下的`exe`文件也可以运行

``` bash
.\target\debug\hello_cargo.exe
```

也可以通过cargo来运行对应的exe打包文件

``` bash
cargo run
```

在执行cargo run的时候有两个比较有意思的点：

- 源文件没有变化，直接执行cargo build出来的文件
- 如果源文件产生了变化，会先编译，然后再执行
- 这里可以通过`cargo check`来检验当前版本的编译后的代码，是否还有更新为编译的版本

3. 构建发布版本

通过`--release`来指定构建的版本为发布版本，通过`release`来发布，会做很多优化，使rust代码运行更快，但也会让`构建时间更长`，另外构建发布的时候，这个地方生成的文件是在target下的。