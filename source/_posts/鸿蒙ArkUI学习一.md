---
layout: posts
title: 鸿蒙ArkUI学习一
date: 2023-12-27 15:20:33
tags:
  - ArkUI
  - 移动端
---

纯血鸿蒙即将到来，总结一下学习的笔记

## 线性布局（Row/Column）

线性布局（LinearLayout）是开发中最常用的布局，通过线性容器`Row`和`Column`构建。线性布局是其他布局的基础，其子元素在线性方向上（水平方向和垂直方向）依次排列。线性布局的排列方向由所选容器组件决定，Column容器内子元素按照垂直方向排列，Row容器内子元素按照水平方向排列。根据不同的排列方向，开发者可选择使用Row或Column容器创建线性布局。这个比较像flutter里面线性布局 学过flutter的就比较容易理解

## 横向线性布局

![](https://cdn.jsdelivr.net/gh/qitiandear/qtpic/img/arkui-Column.png)

## 纵向线性布局

![](https://cdn.jsdelivr.net/gh/qitiandear/qtpic/img/arkui-Row.png)

## 基础概念

- 布局容器：具有布局能力的容器组件，可以承载其他元素作为其子元素，布局容器会对其子元素进行尺寸计算和布局排列。
- 布局子元素：布局容器内部的元素。
- 主轴：线性布局容器在布局方向上的轴线，子元素默认沿主轴排列。Row容器主轴为横向，Column容器主轴为纵向。
- 交叉轴：垂直于主轴方向的轴线。Row容器交叉轴为纵向，Column容器交叉轴为横向。
- 间距：布局子元素的间距。

## 布局子元素在排列方向上的间距

在布局容器内，可以通过space属性设置排列方向上子元素的间距，使各子元素在排列方向上有等间距效果。

### Column容器内排列方向上的间距

![](https://cdn.jsdelivr.net/gh/qitiandear/qtpic/img/0000000000011111111.20231016160227.10967101881305176904101137591378.png)

``` js
Column({ space: 20 }) {
  Text('space: 20').fontSize(15).fontColor(Color.Gray).width('90%')
  Row().width('90%').height(50).backgroundColor(0xF5DEB3)
  Row().width('90%').height(50).backgroundColor(0xD2B48C)
  Row().width('90%').height(50).backgroundColor(0xF5DEB3)
}.width('100%')
```

### Row容器内排列方向上的间距

![](https://cdn.jsdelivr.net/gh/qitiandear/qtpic/img/0000000000011111111.20231016160227.26752570394419855033940741045151.png)

``` js
Row({ space: 35 }) {
  Text('space: 35').fontSize(15).fontColor(Color.Gray)
  Row().width('10%').height(150).backgroundColor(0xF5DEB3)
  Row().width('10%').height(150).backgroundColor(0xD2B48C)
  Row().width('10%').height(150).backgroundColor(0xF5DEB3)
}.width('90%')
```

## 布局子元素在交叉轴上的对齐方式

在布局容器内，可以通过alignItems属性设置子元素在交叉轴（排列方向的垂直方向）上的对齐方式。且在各类尺寸屏幕中，表现一致。其中，交叉轴为垂直方向时，取值为VerticalAlign类型，水平方向取值为HorizontalAlign。

alignSelf属性用于控制单个子元素在容器交叉轴上的对齐方式，其优先级高于alignItems属性，如果设置了alignSelf属性，则在单个子元素上会覆盖alignItems属性。

### Column容器内子元素在水平方向上的排列 

![](https://cdn.jsdelivr.net/gh/qitiandear/qtpic/img/0000000000011111111.20231016160227.43160864942208634561433911004496.png)

- HorizontalAlign.Start：子元素在水平方向左对齐。 
- HorizontalAlign.Center：子元素在水平方向居中对齐。 
- HorizontalAlign.End：子元素在水平方向右对齐。 

### Row容器内子元素在垂直方向上的排列

![](https://cdn.jsdelivr.net/gh/qitiandear/qtpic/img/0000000000011111111.20231016160228.14231809919984575390529027130914.png)

- VerticalAlign.Top：子元素在垂直方向顶部对齐。
- VerticalAlign.Center：子元素在垂直方向居中对齐。
- VerticalAlign.Bottom：子元素在垂直方向底部对齐。

## 布局子元素在主轴上的排列方式

在布局容器内，可以通过justifyContent属性设置子元素在容器主轴上的排列方式。可以从主轴起始位置开始排布，也可以从主轴结束位置开始排布，或者均匀分割主轴的空间。

### Column容器内子元素在主轴上的排列

![](https://cdn.jsdelivr.net/gh/qitiandear/qtpic/img/0000000000011111111.20231016160228.17226526730963429171773882514098.png)

- justifyContent(FlexAlign.Start)：元素在主轴方向首端对齐，第一个元素与行首对齐，同时后续的元素与前一个对齐。
- justifyContent(FlexAlign.Center)：元素在主轴方向中心对齐，第一个元素与行首的距离与最后一个元素与行尾距离相同。 
- justifyContent(FlexAlign.End)：元素在主轴方向尾部对齐，最后一个元素与行尾对齐，其他元素与后一个对齐。 
- justifyContent(FlexAlign.Spacebetween)：主轴方向均匀分配元素，相邻元素之间距离相同。第一个元素与行首对齐，最后一个元素与行尾对齐。 
- justifyContent(FlexAlign.SpaceAround)：主轴方向均匀分配元素，相邻元素之间距离相同。第一个元素到行首的距离和最后一个元素到行尾的距离是相邻元素之间距离的一半。 
- justifyContent(FlexAlign.SpaceEvenly)：主轴方向均匀分配元素，相邻元素之间的距离、第一个元素与行首的间距、最后一个元素到行尾的间距都完全一样。

### Row容器内子元素在主轴上的排列

![](https://cdn.jsdelivr.net/gh/qitiandear/qtpic/img/0000000000011111111.20231016160228.47276425594079697109134983274548.png)

- justifyContent(FlexAlign.Start)：元素在主轴方向首端对齐，第一个元素与行首对齐，同时后续的元素与前一个对齐。 
- justifyContent(FlexAlign.Center)：元素在主轴方向中心对齐，第一个元素与行首的距离与最后一个元素与行尾距离相同。 
- justifyContent(FlexAlign.End)：元素在主轴方向尾部对齐，最后一个元素与行尾对齐，其他元素与后一个对齐。 
- justifyContent(FlexAlign.Spacebetween)：主轴方向均匀分配元素，相邻元素之间距离相同。第一个元素与行首对齐，最后一个元素与行尾对齐。
- justifyContent(FlexAlign.SpaceAround)：主轴方向均匀分配元素，相邻元素之间距离相同。第一个元素到行首的距离和最后一个元素到行尾的距离是相邻元素之间距离的一半。 
- justifyContent(FlexAlign.SpaceEvenly)：主轴方向均匀分配元素，相邻元素之间的距离、第一个元素与行首的间距、最后一个元素到行尾的间距都完全一样。 

## 自适应拉伸

在线性布局下，常用空白填充组件Blank，在容器主轴方向自动填充空白空间，达到自适应拉伸效果。Row和Column作为容器，只需要添加宽高为百分比，当屏幕宽高发生变化时，会产生自适应效果。

``` js
@Entry
@Component
struct BlankExample {
  build() {
    Column() {
      Row() {
        Text('Bluetooth').fontSize(18)
        Blank()
        Toggle({ type: ToggleType.Switch, isOn: true })
      }.backgroundColor(0xFFFFFF).borderRadius(15).padding({ left: 12 }).width('100%')
    }.backgroundColor(0xEFEFEF).padding(20).width('100%')
  }
}
```

## 自适应缩放

自适应缩放是指子组件随容器尺寸的变化而按照预设的比例自动调整尺寸，适应各种不同大小的设备。在线性布局中，可以使用以下两种方法实现自适应缩放。

- 父容器尺寸确定时，使用layoutWeight属性设置子组件和兄弟元素在主轴上的权重，忽略元素本身尺寸设置，使它们在任意尺寸的设备下自适应占满剩余空间。

``` js
@Entry
@Component
struct layoutWeightExample {
  build() {
    Column() {
      Text('1:2:3').width('100%')
      Row() {
        Column() {
          Text('layoutWeight(1)')
            .textAlign(TextAlign.Center)
        }.layoutWeight(1).backgroundColor(0xF5DEB3).height('100%')

        Column() {
          Text('layoutWeight(2)')
            .textAlign(TextAlign.Center)
        }.layoutWeight(2).backgroundColor(0xD2B48C).height('100%')

        Column() {
          Text('layoutWeight(3)')
            .textAlign(TextAlign.Center)
        }.layoutWeight(3).backgroundColor(0xF5DEB3).height('100%')

      }.backgroundColor(0xffd306).height('30%')

      Text('2:5:3').width('100%')
      Row() {
        Column() {
          Text('layoutWeight(2)')
            .textAlign(TextAlign.Center)
        }.layoutWeight(2).backgroundColor(0xF5DEB3).height('100%')

        Column() {
          Text('layoutWeight(5)')
            .textAlign(TextAlign.Center)
        }.layoutWeight(5).backgroundColor(0xD2B48C).height('100%')

        Column() {
          Text('layoutWeight(3)')
            .textAlign(TextAlign.Center)
        }.layoutWeight(3).backgroundColor(0xF5DEB3).height('100%')
      }.backgroundColor(0xffd306).height('30%')
    }
  }
}
```

- 父容器尺寸确定时，使用百分比设置子组件和兄弟元素的宽度，使他们在任意尺寸的设备下保持固定的自适应占比。

``` js
@Entry
@Component
struct WidthExample {
  build() {
    Column() {
      Row() {
        Column() {
          Text('left width 20%')
            .textAlign(TextAlign.Center)
        }.width('20%').backgroundColor(0xF5DEB3).height('100%')

        Column() {
          Text('center width 50%')
            .textAlign(TextAlign.Center)
        }.width('50%').backgroundColor(0xD2B48C).height('100%')

        Column() {
          Text('right width 30%')
            .textAlign(TextAlign.Center)
        }.width('30%').backgroundColor(0xF5DEB3).height('100%')
      }.backgroundColor(0xffd306).height('30%')
    }
  }
}
```

## 自适应延伸

自适应延伸是指在不同尺寸设备下，当页面的内容超出屏幕大小而无法完全显示时，可以通过滚动条进行拖动展示。这种方法适用于线性布局中内容无法一屏展示的场景。通常有以下两种实现方式。

- 在List中添加滚动条：当List子项过多一屏放不下时，可以将每一项子元素放置在不同的组件中，通过滚动条进行拖动展示。可以通过scrollBar属性设置滚动条的常驻状态，edgeEffect属性设置拖动到内容最末端的回弹效果。
- 使用Scroll组件：在线性布局中，开发者可以进行竖向或者横向的布局。当一屏无法完全显示时，可以在Column或Row组件的外层包裹一个可滚动的容器组件Scroll来实现可滑动的线性布局。

竖向布局中使用Scroll组件：

``` js
@Entry
@Component
struct ScrollExample {
  scroller: Scroller = new Scroller();
  private arr: number[] = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9];

  build() {
    Scroll(this.scroller) {
      Column() {
        ForEach(this.arr, (item) => {
          Text(item.toString())
            .width('90%')
            .height(150)
            .backgroundColor(0xFFFFFF)
            .borderRadius(15)
            .fontSize(16)
            .textAlign(TextAlign.Center)
            .margin({ top: 10 })
        }, item => item)
      }.width('100%')
    }
    .backgroundColor(0xDCDCDC)
    .scrollable(ScrollDirection.Vertical) // 滚动方向纵向
    .scrollBar(BarState.On) // 滚动条常驻显示
    .scrollBarColor(Color.Gray) // 滚动条颜色
    .scrollBarWidth(10) // 滚动条宽度
    .edgeEffect(EdgeEffect.Spring) // 滚动到边沿后回弹
  }
}
```

横向布局中使用Scroll组件：

``` js
@Entry
@Component
struct ScrollExample {
  scroller: Scroller = new Scroller();
  private arr: number[] = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9];

  build() {
    Scroll(this.scroller) {
      Row() {
        ForEach(this.arr, (item) => {
          Text(item.toString())
            .height('90%')
            .width(150)
            .backgroundColor(0xFFFFFF)
            .borderRadius(15)
            .fontSize(16)
            .textAlign(TextAlign.Center)
            .margin({ left: 10 })
        })
      }.height('100%')
    }
    .backgroundColor(0xDCDCDC)
    .scrollable(ScrollDirection.Horizontal) // 滚动方向横向
    .scrollBar(BarState.On) // 滚动条常驻显示
    .scrollBarColor(Color.Gray) // 滚动条颜色
    .scrollBarWidth(10) // 滚动条宽度
    .edgeEffect(EdgeEffect.Spring) // 滚动到边沿后回弹
  }
}
```
