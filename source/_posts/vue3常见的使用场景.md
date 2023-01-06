---
title: vue3常见的使用场景
date: 2023-01-06 13:59:33
tags:
  - vue
  - vue3
categories:
  - 使用场景
---

Vue是一套用于构建用户界面的渐进式框架。这篇文章主要学习vue3的新特性和新增的API，主要是有 Vue 2 经验的用户快速熟悉vue3.

## 父子组件数据传递

### 父组件数据传递到子组件

Vue3 中父组件是通过属性传递数据，在 `<script setup>` 中，`props` 需要使用 `defineProps()` 这个宏函数来进行声明。

``` js
<!-- 父组件 -->
<script setup>
import HelloWorld from './HelloWorld.vue'
</script>

<template>
  <HelloWorld address="parent address" />
</template>
```

``` js
<!-- 子组件 -->
<script setup>
const props = defineProps({
  address: {
    type: String,
    required: true
  }
})
console.log(props.address) // parent address
</script>

<template>
  <!-- 使用 address 或 props.address -->
  <div>{{ address }}</div>
  <div>{{ props.address }}</div>
</template>
```

注意：`defineProps` 、`defineEmits` 、 `defineExpose` 和 `withDefaults` 这四个宏函数只能在 `<script setup>` 中使用。他们不需要导入，会随着 `<script setup>` 的处理过程中一起被编译。eslint检测报警告在package.json中添加

``` json
"eslintConfig" {
    "env": {
        "vue/setup-compiler-macros": true
    },
}
```

### 子组件数据传递到父组件

在 `<script setup>` 中使用 `defineEmits()`:

``` js
<!-- 子组件 -->
<script setup>
const emit = defineEmits(['getMsg'])
function onClick() {
  emit('getMsg', 'child message')
}
</script>

<template>
  <button @click="onClick">点击</button>
</template>
```

``` js
<!-- 父组件 -->
<script setup>
import HelloWorld from './HelloWorld.vue'

function getMsg(value) {
  console.log(value) // child message
}
</script>

<template>
  <HelloWorld @get-msg="getMsg" />
</template>
```

### 父组件使用子组件数据

在 `<script setup>` 中，组件的属性和方法默认都是私有的。父组件无法访问到子组件中的任何东西，除非子组件通过 `defineExpose` 显式的暴露出去：

``` js
<!-- 子组件 -->
<script setup>
import { ref } from 'vue'

const msg = ref('hello vue3!')
function change() {
  msg.value = 'hi vue3!'
  console.log(msg.value)
}
// 属性或方法必须暴露出去，父组件才能使用
defineExpose({ msg, change })
</script>
```

``` js
<!-- 父组件 -->
<script setup>
import HelloWorld from './HelloWorld.vue'
import { ref, onMounted } from 'vue'

const child = ref(null)
onMounted(() => {
  console.log(child.value.msg) // hello vue3!
  child.value.change() // hi vue3!
})
</script>

<template>
  <HelloWorld ref="child" />
</template>
```

## 获取上下文对象

Vue3 的 `setup` 中通过 `getCurrentInstance` 方法获取上下文对象:

``` js
<script setup>
import { getCurrentInstance } from 'vue'

// 以下两种方法都可以获取到上下文对象
const { ctx } = getCurrentInstance()
const { proxy }  = getCurrentInstance()
</script>
```
这样我们就可以使用 `$parent` 、`$refs` 等，干自己想干的事情了，下面是我打印出来的 `ctx` 的完整属性。

![](https://kun.nwyp123.com/20230106144156.png)

注意：`ctx` 只能在开发环境使用，生成环境为 `undefined` 。 推荐使用 `proxy` ，在开发环境和生产环境都可以使用。