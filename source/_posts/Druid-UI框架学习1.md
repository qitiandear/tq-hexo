---
title: Druid UI框架学习1
date: 2023-02-23 14:28:52
tags:
  - rust
  - Druid
  - gui
  - Linux
categories:
  - UI框架
---

Druid允许您构建可部署在Windows、macOS、Linux和web上的简单交互式图形应用程序。Druid是一个数据驱动的声明性框架。

![](https://cdn.jsdelivr.net/gh/qitiandear/qtpic/img/20230223143051.png)


## 一、创建rust项目。

``` bash
cargo new druid-app
```

## 二、使用vim（SpaceVim）编辑器打开项目

### 项目结构

``` ini
src
└── main.rs
Cargo.toml
Cargo.lock
```

我们需要在Cargo.toml中添加Druid的包

``` ini
[dependencies]
druid = "0.8.2"
```

## 三、构建我们的第一个窗体

### 1. 引入包

``` rust
use druid::widget::{Button, Flex, Label};
use druid::{AppLauncher, LocalizedString, PlatformError, Widget, WidgetExt, WindowDesc};
```

### 2. 编写主界面

``` rust
fn ui_builder() -> impl Widget<u32> {
    //创建一个本地处理话的数据
    let text = LocalizedString::new("hello-counter")
        .with_arg("count", |data:&u32,_evt|{(*data).into()});
    let label = Label::new(text).padding(5.0).center();
    let button = Button::new("+ 1")
        .on_click(|_ctx,data:&mut u32,_ent|{
            *data += 1;
        });
    Flex::column().with_child(label).with_child(button)
} 
```

### 3. 编写启动类

``` rust
fn main() -> Result<(), PlatformError>{
    let win = WindowDesc::new(ui_builder());
    let data = 0_u32;
    let _app = AppLauncher::with_window(win)
        .log_to_console()
        .launch(data);
    Ok(())
}
```

## 四、编译程序

``` ini
cargo build --release
```

点击target/release目录下的druid-app可运行文件

``` bash
[tianqi@localhost druid-app]$ ll target/release/
总用量 6.3M
drwxr-xr-x 66 tianqi tianqi 4.0K  2月 23 13:53 build
drwxr-xr-x  2 tianqi tianqi  32K  2月 23 13:54 deps
-rwxr-xr-x  2 tianqi tianqi 6.3M  2月 23 13:54 druid-app
-rw-r--r--  1 tianqi tianqi  100  2月 23 13:54 druid-app.d
drwxr-xr-x  2 tianqi tianqi 4.0K  2月 23 13:53 examples
drwxr-xr-x  2 tianqi tianqi 4.0K  2月 23 13:53 incremental
```

![](https://cdn.jsdelivr.net/gh/qitiandear/qtpic/img/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20230223144603.jpg)

PS:

使用Linux打包druid得先安装[gtk](https://www.gtk.org/docs/installations/linux/)，本人使用的是openeuler 20.03LTS版本安装的deepin桌面

``` ini
sudo dnf install gtk3-devel -y
```