---
title: JSDoc使用
date: 2023-08-22 15:23:54
tags:
  - JavaScript
categories:
  - 使用
---

JSDoc 提供了向 JavaScript 代码库添加类型的功能，并在注释中使用适当的约定，这样不同的 IDE（如 Visual Studio Code）可以识别定义的类型，显示它们，并通过自动完成使编码更容易。定义放在/** */注释中。

## 原始值

原始值的注释很简单，我们可以从中学习到`JSDoc`的注释风格，以及`@type`标签。

类型名称在注释中均为小写

``` js
/**
 * 最小值。数字
 * @type {number}
 */
const MIN_VALUE = 1

// MIN_VALUE

/**
 * 临时名称。字符串
 * @type {string}
 */
let tempName = 'joe'

// tempName

/**
 * 是否显示。布尔
 * @type {boolean}
 */
let isShow = false

// isShow

/**
 * 空对象
 * @type {null}
 */
let hello = null

// hello

/**
 * 未定义
 * @type {undefined}
 */
const last = undefined

// last

```

## 数组

对于子项同类型的注释也很简单，与`TypeScript`类似，格式是 `type[]`。

``` js
/**
 * 字符串列表
 * @type {string[]}
 */
const wordList = ['ai', 'box', 'cell', 'delete']

// wordList

```

## 对象

对于内置对象，直接使用对象的类型，IDE会自动给予代码提示。

``` js
/**
 * 时间对象
 * @type {Date}
 */
const ins = new Date()

// ins.getDay()

/**
 * promise对象
 * @type {Promise<string>}
 */
const promise = new Promise((resolve => resolve('hello')))

// promise.then()
```

对于自定义对象类型，可以在类型中直接写出定义，注释需要写在属性上。

``` js
/**
 * 自定义对象
 * @type {{x: number, y: ((function(number): string)|*), z: {a: number}}}
 */
const position = {
  /**
   * 横轴坐标
   */
  x: 1,
  /**
   * 竖轴坐标
   * @param {number} data 一个入参
   * @return {string} 返回值
   */
  y: function (data) {},
  /**
   * 立体坐标
   * @type {{a: number}}
   */
  z: {
    /**
     * 一个属性
     */
    a: 3
  }
}

// position.y
```

上面的写法对于简单的对象还行。但是如果对象非常复杂，就会有问题，主要有两个问题：① 类型注释冗杂在一行内，② 描述注释跟类型注释是分开的。

下面我们请出`JSDoc`中拓展性最强的标签组合`@typedef`和`@property`来解决复杂东西中会出现的这两个问题。

``` js
/**
 * @typedef ComplexObj
 * @property {object} a 对象a
 * @property {object} a.b 对象a.b
 * @property {number} a.b.c 数字a.b.c
 * @property {object} y 有一个对象y
 * @property {string} y.x 字符串
 * @property {string[]} z 字符串数组
 */

/**
 * 复杂的对象
 * @type {ComplexObj}
 */
const complexObj = {
  a: {
    b: {
      c: 1
    }
  },
  y: {
    x: 'hello'
  },
  z: ['r', 'y']
}

// complexObj.y.x
```

数组对象也差不多。我们顺便学习一下类型操作符，即使用 `|`链接两个类型。以及 `[]`表示该属性是可选的。

``` js
/**
 * @typedef Animal 动物
 * @property {string} name 名称
 * @property {string} [live] 名称
 * @property {number | string} foot 脚
 */

/**
 * 动物列表
 * @type {Animal[]}
 */
const animalList = [
  {
    name: 'cat',
    live: 'land',
    foot: 4
  },
  {
    name: 'fish',
    live: 'water',
    foot: 0
  },
  {
    name: 'dragon',
    foot: 'some'
  }
]

// animalList[0].live
```

混合数组与对象。

``` js
/**
 * @typedef School 学习
 * @property {string} name 名称
 * @property {object[]} class 班级
 * @property {number} class[].build 教室
 * @property {string} class[].students 学生人数
 * @property {object[]} class[].examples 模范学生
 * @property {string} class[].examples[].name 学生名字
 * @property {number} class[].examples[].age 学生年龄
 */

/**
 * 数组对象混合
 * @type {School}
 */
const school = {
  name: 'North Big',
  class: [
    {
      build: 4,
      students: '1w+',
      examples: [
        {
          name: 'No.1',
          age: 43
        }
      ]
    }
  ]
}

// school.class[0].examples[0].age
```

## 函数

函数的注释一般包括入参（`param`）和出参（`return`）。入参可以有多个。

``` js
/**
 * 函数，待办事项
 * @param {string} thing 加如一个待办
 * @param {number} index 所以
 * @return {string[]}
 */
function toDo (thing, index) {
  const list = ['pay apple']
  list[index] = thing
  return list
}

let todo = toDo('buy dog', 3)
```

## 类

一般无需对类进行特别的注释，IDE都会自动生成对应的文档。只需要对类成员进行注释，而类成员也就是我们上面提到的常见数据类型。

## 其他

可以使用`|`操作符实现类似枚举值的效果，这对代码提示很友好

``` js
/**
 * 捕获错误
 * @param {object} status 错误代码
 * @param { 1 | 2 | 3 | 'unknown'} status.code 错误代码。1 请求错误 2. 代码错误 3. 环境错误 4. 位置错误
 */
function handleCatch (status) {
  if (status.code === 1) {
    status.code
  }
}
```

未定义的属性也可以预先写明对象的类型。 `&`符合可以用来合并对象

``` js
/**
 * 预置类型
 * @type {{x: number} & {y: string}}
 */
let someVal = {}

// someVal.y
```

也支持不定数量参数的注释。

``` js
/**
 * 不定数量参数
 * @param {...number} arg
 */
function candyCount (...arg) {
  
}
// candyCount(2, 4)
```