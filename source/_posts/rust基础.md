---
title: rust基础
date: 2022-12-30 13:12:24
tags:
  - rust
categories:
  - 变量
  - 数据类型
  - 函数
  - 注释
---
变量，基本类型，函数，注释和控制流，这些几乎是每种编程语言都具有的编程概念。

这些基础概念将存在于每个 Rust 程序中，及早学习它们将使你以最快的速度学习 Rust 的使用。

## 变量和可变性

### 1. 概念

- 变量（variables）:用来存储值的一个地方（Storing values with Variables）,默认是immutable
- 常量（constants）:仅可被赋值一次的变量
- 可变性（mutability）：通过mut来定义，代表值可变的变量（类似let，可以被重新赋值）

【错误写法】

``` rust
fn main() {
    // let mut a = 6;
    let a = 5;
    println!("the variable is {}", a);
    // 这里rust会报错
    a = 6;
    println!("the variable is {}", a);
}
```

【正确写法】当使用`mut`的时候，我们允许变量的值被更改

``` rust
fn main() {
    // 通过mut来区分类似const和let的概念
    let mut a = 5;
    println!("the variable is {}", a);
    a = 6;
    println!("the variable is {}", a);
}
```

> It’s important that we get compile-time errors when we attempt to change a value that we previously designated as immutable because this very situation can lead to bugs. If one part of our code operates on the assumption that a value will never change and another part of our code changes that value, it’s possible that the first part of the code won’t do what it was designed to do. The cause of this kind of bug can be difficult to track down after the fact, especially when the second piece of code changes the value only sometimes
> 大体的意思就是，为了避免bug和代码中非预期的成分如果我们一个变量定义的时候假设他是不可变的，那么其实如果对其二次赋值，其实是非符合预期的，所以编译阶段会报错

### 2. 变量和常量的区别

`Immutable变量`和`常量`之间的区别：

- 【区别】`mut`关键词不能用来修饰`const`的变量，`const`一直是不可变的
- 【区别】`const`关键字必须要注释类型，就是需要给出类型的说明
- 【区别】`const`关键词可以在任意的作用域进行声明，包括全局作用域
- 【区别】`const`的赋值，只能是一个常数的表达式，不能是只有在运行时才能被计算出的运行后的结果（就是不能是一个根据运行时环境参数不一样，动态计算得到的动态的结果）
- 【非区别】建议定义常量时，通过`大写字母`和`_`的方式来定义，变量定义时建议使用小写，并且在单词间通过下划线的方式进行隔离的方式，`规范有效的命名方式让我们在写代码时对变量的识别会更加准确，利于代码的维护`。

``` rust
// 全局的声明是被允许的
// 区别二：需要通过 `: `来告诉rust这个常量对应的类型
// 区别三：可以在全局定义
const C_C: &str = "_global cc";
// 区别四：只能是一个常数计算结果的表达式
const THREE_HOURS_IN_SECONDS: u32 = 60 * 60 * 3;
// 区别三：这个里使用let会报错，不能再全局定义
// expected item, found keyword `let`
let dd = 32;

fn main() {
    const B_BIT: u32 = 3;
    println!("the constant is {}, {}", C_C, B_BIT);
}
```

### 3. 变量覆盖（阴影）

> TIPS： 这里我其实更倾向于将原文中的shadowing翻译成变量的覆写和作用域，因为看上去和js的作用域真的很像，当然可能会不符合rust中的叫法

`作用域的定义和特性`：通过`{}`围起来的代码块是单独的作用域，在内部作用域定义的变量，会覆盖掉上层作用域中有的参数，和JS表现相近

`变量的覆写`：通过`let`定义同名的变量来覆写上面已经通过`let`定义的变量，如果是变量的覆写是不要求前后同名变量类型相同的，但是`mut`变量的修改，需要前后有一定的类型

`注意点`：`let`定义的变量通过`let`在覆写和是不是`mut`的变量其实没有关系，因为这里是新创建一个变量，而不是对已经不可变的变量进行赋值（编译时不会报错），所以是两个维度的事情

## 数据类型

在rust中的数据类型主要包括两种：标量（scalar）和组合类型（compound），因为rust本来是一个`静态类型语言`，所以需要在编译时告诉rust所有变量的类型，所以在类型转换的时候，也需要告诉编译器，需要转成的类型声明。

### 1. 标量类型

标量类型单表的是单一的值，不是组合的值，具体的类型如下：

- 整型（integers）
- 浮点型（float）
- 布尔类型
- 字符串类型

#### 整型类型

|长度|有符号|无符号|
|:--|:--|:--|
|8-bit	|i8	|u8|
|16-bit	|i16	|u16|
|32-bit	|i32	|u32|
|64-bit	|i64	|u64|
|128-bit	|i128	|u128|
|arch	|isize	|usize|

`注意点`

- 有符号通过`i`开头，无符号通过`u`开头
- 整型的区间为，有符号为 -2^n-1^ ~ 2^n-1^ - 1 ，无符号为0~2^n^ - 1
- `arch`代表的是当前操作系统架构最大的位数，比如x64就是`u64`, `i64`
- 表示不同进制的方法

  - 十进制：1_000,1000都是十进制
  - 十六进制：0xff
  - 八进制：0o77
  - 二进制：0b1111_0000
  - 字节：b'A'

- 对于溢出的处理，在编译时（debug模式）会报错，引起你的注意，但是如果是`--release`模式下并不会报错，他会把这个值置为你超过部分的值，这是因为他会自动在外部填充两位，来接收对应的值，所以虽然不会报错，但是也需要我们对加减和最大最小值来做一下校验，主流的库中都会有相应的方法：

  - `wrapping_*`来做数的运算
  - `checked_*`来判断是否有溢出，`None`返回值代表溢出了
  - `overflowing_*`返回值和一个布尔值来判断是否溢出了
  - `saturating_*`方法来代表改值在定义的区间内

#### 浮点类型

浮点类型的定义只有两个方法`f32`和`f64`

``` rust
fn main () {
    // 数值类型
    // 浮点除法
    // 如果不是2.0 / 3.0返回的是0，默认为整型
    // let float_num = 2 / 3;  
    
    let floorded:f64 = 2.0 / 3.0;
    // 这样是0.66666
    println!("the float is {}", floorded)
}
```

`注意点`

- 默认是`f64`为了保证浮点更高的精度
- 如果是`2 / 3`的形式返回的是整型，也就是0，如果需要得到小数，那么一定是`2.0 / 3.0`保证右边都是浮点数，最后计算结果才会是浮点数

#### 布尔类型

类型通过`bool`来进行定义即可，用于条件控制，这个在`if`的控制流中会用到，和所有语言的都类似

``` rust
fn main () {
    let f: bool = false;
}
```

#### 字符串类型

rust中的字符串类型是最原始的字母类型,包含中文、日文、韩文表情等字符（Unicode Scalar比ASCII码更丰富）

``` rust
fn main() {
    let c = 'z';
    let z = 'ℤ';
    let heart_eyed_cat = '😻';
}
```

### 2. 组合类型

#### 元组类型

元组类型是将多个其他类型放到一个组合类型中，元组的特点：

- 固定的长度
- 一次性的定义
- 长度不可收缩
- 逗号分隔的列表
- 通过`()`来定义

``` rust
fn main () {
    // 组合类型
    // 元组
    let tup: (i32, i8, f64) = (500, 22, 3.14);
    let (t1, t2, t3) = tup;
    
    println!("tup -> {}, {}, {}", tup.0, tup.1, tup.2);
    println!("tup -> {}, {}, {}", t1, t2, t3);
}
```

`注意点`

- 可以通过.语法来进行对应位置元组数据的获取
- 可以通过解构的方式来回去对应位置上的值
- 每个位置元素的类型可以`不相同`

#### 数组类型

数组是同一种类型的元素的集合，和元素的不同是`必须所有元素具有相同的类型`，其特点：

- 元素类型相同
- 通过`[]`来定义

``` rust
fn main () {
    // 数组
    let array1 = [1, 2, 3, 4, 5];
    let array2: [i32;5] = [1, 2, 3, 4, 5]; 
    // 前两种比较好理解，这种的意思是生成一个整型的数组，长度为5，默认值为3
    let array3 = [3;5];
    
    println!("str -> {}", array2[1]);
}
```

`注意点`

- 接入数组的方式通过索引来接入，例如`array2[1]`
- 如果这个index是动态生成的，那么我们一定要判断index是否小于数组的长度，不然的话编译的时候不会报错，因为你的index是动态生成的，但是在运行过程中很有可能index超过了最长的数组长度报错（`越界报错`）

## 函数

### 1. 函数定义方式

函数的定义方式：

- 通过`fn`关键字进行定义
- 在`fn`后跟上相应的函数名
- 通过`{}`来 定义函数体

``` rust
fn main () {
    println!("this is main function");
    another_function();
}

fn another_function() {
    println!("this is another function, {}", get_params(3, 4))
}

fn get_params  (a: i32, b: i32) -> i32 {
    return a + b;
}
```

`注意点`

- 函数的输入可定义类型，输入通过`:`来定义类型
- 默认的函数是不带返回值的，如果带上返回值需要通过`->`指定类型，不然会报错

### 2. 函数体包括声明和表达式

Rust 函数体由一系列可以以表达式（Expression）结尾的语句（Statement）组成。到目前为止，我们仅见到了没有以表达式结尾的函数，但已经将表达式用作语句的一部分。

`注意点`

- 声明并不会返回值，所以不支持JS中的那种连等式赋值，目前是不被允许的~

  ``` rust
  fn main() {
      // 这里会报错`let` expressions in this position are experimental
      let x = (let y = 6);
  }
  ```

- 在rust中表达式是不需要包含尾分号的，一旦你使用了尾分号就把表达式变成了声明

  ``` rust
  fn main () {
      println!("this is main function");
      another_function();
      let y = {
          let x = 3;
          // 这里没有尾分号
          // 所以是个表达式，Y -> 4
          // 如果加上尾分号，就会报错，因为他其实还是没有返回值
          x+1
      };

      println!("y => {}", y)
  }
  ```

- 可以将函数的返回值和表达式结合，这样函数会变得更加简洁

  ``` rust
  fn main () {
      println!("this is main function");
      another_function();
      let y = {
          let x = 3;
          addOne(x)
      };

      // y => 4
      println!("y => {}", y)
  }

  fn addOne(num: u32) -> u32 {
      num + 1
  }
  ```

## 注释

注释主要通过`//`来进行注释，如果是多行的话，就是每一行之气都需要通过`//`来进行注释

## 控制流

基本上所有的编程语言都逃不开相关的控制流的语法，例如`if`,循环`loop`等，这一小节我们一起来看看rust中的循环和判断语法

### 1. if表达式

和其他语言中的if表达式一样，通过`if`关键字后面的声明，指定条件匹配后的操作，同样也支持`else`和`else if`

``` rust
fn main() {
    let number = 5;
    if number < 5 {
        println!("number is less than 5")
    } else if number == 5 {
        println!("number is equal to 5")
    } else {
        println!("number is greater than 5")
    }
}
```

`注意点`

- `if`作为表达式，可以在let中使用达到三元表达式的效果

  ``` rust
  fn main() {
      let number = 5;
      let a = if number < 3 {10} else {20};

      println!("a -> {}", a)
  }
  ```

- `1`中这种表达式要可以使用，有个前提就是每个表达式返回值的类型，需要相同，不然在编译时，rust编译器会报错

### 2. 使用Loop来实现重复

rust中提供了三种循环的方式，包括`loop`, `while`和`for`，接下来看看每一种的用法和写法

#### loop循环

loop循环的特点是，不会主动结束循环，直到使用人手动停止（调用`break`指令）或者直接运行`ctrl + C`才会停止该循环，具体代码示例如下：

``` rust
fn main() {
    let mut b = 1;
    loop {
        println!("b =>{}", b);
        b += 1;

        if b > 5 {
            break;
        }
    }
}
```

`注意点`

- 和其他语言一样，rust中提供了`break`和`continue`关键字，用来停止循环和跳过后续代码这两个操作

- 如果需要退出指定的循环，可以将loop的的返回值拿到，并通过break来中中断

  ``` rust
  fn loopExit() {
      let mut  count = 0;
      'loop_rlt: loop {
          println!("count -> {}", count);
          let mut inner_count = 10;
          
          loop {
              println!("inner count ->{}", inner_count);

              if inner_count == 8 {
                  break;
              }

              if inner_count == 9 && count == 2 {
                  break 'loop_rlt;
              }
              
              inner_count -= 1;
          }

          count +=1;
      }
  }
  // 运行结果
  // count -> 0
  // inner count ->10
  // inner count ->9
  // inner count ->8
  // count -> 1
  // inner count ->10
  // inner count ->9
  // inner count ->8
  // count -> 2
  // inner count ->10
  // inner count ->9
  ```

- loop返回的值需要通过`' `加上变量名加上`:`来标注这是一个循环标记，这是一个规范，例如上述的`'loop_rlt`，这样通过`continue`和`break`来退出和继续循环

- 通过`let`和`break`可以获取break最后我们想要输出的函数值

  ``` rust
  fn loop_return_value(num: u32) -> u32 {
      let mut count = 1;

      let loop_value = loop {
          count+=1;
          
          if count == num {
              break count * 2;
          }
      };

      return loop_value
  }

  fn main () {
      let rlt: u32 = loop_return_value(30);
      // rlt -> 60
      println!("rlt -> {}", rlt);
  }
  ```

#### while循环

`while`循环和`loop`中，一样可以通过`break`来进行中断，其写法也和我们熟悉的其他语言中的`while`相似

``` rust
fn main() {
    let mut number = 3;

    while number != 0 {
        println!("{}!", number);

        number -= 1;
    }

    println!("LIFTOFF!!!");
}
```

`注意点`

- 如果是使用`while`循环遍历数组等有界的结构，可能需要对动态生成的index做出判断

#### for循环

`for...in循环`

通过while循环来便利数组可能会产生越界的问题，为了避免越界可以通过`for...in``的方式来进行`遍历，便不会有越界的问题，因为在while中我们控制index是动态的不稳定。

``` rust
fn for_loop () {
    let a = [1, 2, 3, 4, 5, 6];

    for elem in a {
        println!("loop for -> {}", elem);
    }
}
```

`利用Range和for..in的组合来进行循环`

通过`for...in`除了实现对集合类型的数据结构进行遍历，如果我们想要实现对while那样的多次循环，我们可以使用`range`加`for...in`循环的方式

``` rust
fn for_loop () {
    for elem in (1..100) {
        println!("item -> {}", elem)
    }
    // item -> 1
    // item -> 2
    // ...
    /// item -> 99
}
```

> 上述方式因为是1..100所以区间是[1, 100)，如果想要双闭的区间，写法应该是(1..=100)