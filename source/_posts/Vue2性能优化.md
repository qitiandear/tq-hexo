---
title: Vue2性能优化
date: 2022-12-13 14:54:08
tags:
  - vue
categories:
  - 性能优化
---
性能优化分为四个模块：代码层面的优化，打包层面的优化，服务层面的优化，图片资源的优化。个数分别为8，3，5，5（记住个数后不容易漏忘）

## 一、代码层面的优化

共8个，记住开头字，这样方便记忆：路函缓脚监，循活容

### 1. 路由懒加载

``` js
const Home = () => import(/* webpackChunkName: "Home" */'../components/layout/home.vue')
```

### 2. 函数式组件

函数型组件也被称为无状态组件，无生命周期（意味着钩子函数不能使用），无状态响应（methods中的方法不能响应）。

在template模板中增加functional即可

``` xml
<template functional>    <el-table      :data="props.items"  //2.3.0之前的Vue版本需要通过props接收传的值，之后的版本可省略      height="400"      border      @selection-change="props.handleSelectChange"    >      <slot></slot>    </el-table></template><script>export default {    props:["items","handleSelectChange"]}</script>
```

优点：无生命周期，渲染快。

### 3. 缓存不活动的组件实例（ keep-alive）

### 4. 脚本延迟加载

``` xml
<script src="" defer></script>
<script src="" async></script>
```

HTML4的defer是“渲染完再执行”，HTML5的async是“下载完就执行”。多个defer脚本，会按照它们在页面出现的顺序加载，而多个async脚本不能保证加载顺序（如果脚本之间有继承关系，则不能使用async，比如vue与vuex）

### 5. 监听事件销毁

Vue组件销毁时（切换路由时），会自动清理它与其他实例的连接，解绑它的全部指令及事件监听。如果使用原生的方式，比如addEventListener，事件总线等方式是不会自动销毁的，我们需要在组件销毁时（beforeDestroy或destroyed生命周期）手动移除这些事件的监听，以免造成内存泄漏。

### 6. 循环添加key

给每个vnode增加一个唯一id，高效的更新VNode。

### 7. 活用v-show，减少v-if

v-if会改变DOM数，v-show通过display:none的方式来控制显示内容，不会改变DOM数

### 8. 容易触发重排的元素与“静态”元素分层（使用z-index）

一个页面是由许多层级组成。在一个页面构建完渲染树后，是经历以下流程才最终呈现在我们面前：

（1）浏览器会先获取DOM树并依据样式将其分割成多个独立的渲染层

（2）CPU将每个层绘制进绘图中

（3）将位图作为纹理上传至GPU（显卡）绘制

（4）GPU将所有的渲染层缓存（如果下次上传的渲染层没有发生变化，GPU就不需要对其进行重绘）并符合多个渲染层最终形成我们的图像

由上可知：CPU负责布局，GPU负责绘制。

进行分层让GPU分担更多的渲染工作，我们通常把这样的措施称为硬件加速

## 二、打包层面的优化

### 1.按需引入，减少打包体积

### 2.不生成.map文件

vue.config.js配置

``` js
productionSourceMap: process.env.NODE_ENV === 'production' ? false : true,
```

### 3.打包移除console.log

原因：毕竟是一次函数调用，并且被console.log调用的函数，不会被垃圾回收机制回收，可能会导致内存泄漏。

使用babel-plugin-transform-remove-console

## 三、服务层面的优化

### 1.减少HTTP请求数

例如：使用雪碧图

``` css
background: url('./images/css_sprites.png') -116px -10px; //通过调整position来展示图片
```

雪碧图自动生成网站：[https://www.toptal.com/developers/css/sprite-generator](https://www.toptal.com/developers/css/sprite-generator)

### 2.开启gzip传输压缩

### 3.DNS预解析

X-DNS-Prefetch-Control头控制着浏览器的DNS预读取功能。DNS预读取是一项使浏览器主动去执行域名解析的功能，其范围包括文档的所有链接，无论是图片的，CSS的，还是JavaScript等其他用户能够点击的URL

### 4.使用CDN加速

使用BootCDN免费的加速服务，网址：[www.bootcdn.cn/](https://www.bootcdn.cn/)

实例：引入echarts

在index.html中使用cdn引入

``` js
<script src="https://cdn.bootcss.com/echarts/3.7.2/echarts.min.js"></script>
```

在vue.config.js配置

``` js
configureWebpack: {
    //externals中的key是用于import，value表示在全局中访问到该对象，
    //就是window.echarts，window可省略，直接通过echart访问，echart.init()
    externals: {
      'echarts': 'echarts'
    }
}
```

### 5. 使用SSR渲染

## 四、图片资源优化

### 1. 压缩图片

在线压缩图片网站：[tinypng.com](https://tinypng.com/)

### 2. 不在HTML里缩放图片

定义的图片多大，就拿多大的图片，不要在`200*200`的区域，放`400*400`的图片

### 3. img标签增加alt属性

在图片加载失败时，同alt属性显示文字，加快页面的反应速度

### 4. 图片懒加载

方法一：使用element-UI的图片懒加载

方法二：当元素滚动到可视区域处，再给图片的src属性赋值，去加载图片

### 5. 使用字体图标代替图片