---
title: rust环境搭建
date: 2022-12-13 11:55:11
tags:
  - rust
  - install
categories:
  - 环境
---
Rust 语言是一种高效、可靠的通用高级语言。其高效不仅限于开发效率，它的执行效率也是令人称赞的，是一种少有的兼顾开发效率和执行效率的语言。

## Rust 环境搭建

本次搭建使用的是win10的wsl安装的openEuler系统

### 首先

``` bash
export RUSTUP_DIST_SERVER=https://mirrors.ustc.edu.cn/rust-static
export RUSTUP_UPDATE_ROOT=https://mirrors.ustc.edu.cn/rust-static/rustup
```

### 下载安装rust

``` bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

### 配置环境变量

``` bash
source "$HOME/.cargo/env"
```

### 检测环境是否配置成功

``` bash
cargo -V
```

![](https://kun.nwyp123.com/20221213131919.png)

``` bash
rustc -V
```

![](https://kun.nwyp123.com/20221213131948.png)

### 国内配置rust的下载镜像（可选）

``` bash
cd .cargo
ls
```

![](https://kun.nwyp123.com/20221213132308.png)

查看是否存在config文件，如果没有创建config文件，然后编辑输入一下内容：

``` bash
[source.crates-io]
registry = "https://github.com/rust-lang/crates.io-index"
replace-with = 'ustc'
[source.ustc]
registry = "git://mirrors.ustc.edu.cn/crates.io-index"
```

至此rust的环境就配置完成了