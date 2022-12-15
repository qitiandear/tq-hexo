---
title: rust数据结构
date: 2022-12-15 10:20:50
tags:
  - rust
categories:
  - 数据结构
---
**Rust** 的 **Vec** 其实是动态数组, 很多语言内置动态数组, 譬如 **JavaScript** **Python**这类, 像 **Rust** 这种具有内存控制能力的语言, 就选择了标准库内置动态数组.

## 基本用法

``` rust
let mut v1 : Vec<i32> = vec![];
dbg!(v1.len(), v1.capacity());
for i in 1..10 {
  v1.push(i);
}
dbg!(&v1);
```

## 布局

现在直接写一下 **Rust** 动态数组直观实现

``` rust
pub struct MyVec<T> {
  ptr: *mut T,
  cap: usize,
  len: usize,
}
```

这样直接编译能通过, 但是 `ptr` 这个裸指针不能让 `Drop check` 正常工作, 因为直接在这里使用裸指针 `Drop` 检查器会认为你没有持有任何值, 因此我们可以用 `Unique<T>` 处理祼指针 `*mut T`, 通过它内部的 `PhantomData` 来帮助 `Drop check` 工作.

> `Unique<T>` 可以封装祼指针
>
> - `T` 是可变的
> - 可以进行 `drop` 检查
> - `T` 实现了 `Send/Sync`, 该指针也具备 `Send/Sync` 特性, 等于讲线程安全
> - 指针具备非空性

既然如此自己实现一个吧, 解析源码自己写不会浪费什么时间, 还能体会别人的设计, 顺道实现一下 `Send/Sync`

``` rust
use std::marker::PhantomData;

struct MyUnique<T: ?Sized> {
  ptr: *const T,
  _marker: PhantomData<T>,
}

unsafe impl<T: Send + ?Sized> Send for MyUnique<T> {}

unsafe impl<T: Sync + ?Sized> Sync for MyUnique<T> {}

impl<T: ?Sized> MyUnique<T> {
  #[inline]
  pub fn new(ptr: *mut T) -> Option<Self> {
    if !ptr.is_null() {
      Some(unsafe {
        MyUnique {
          ptr: ptr as _,
          _marker: PhantomData,
        }
      })
    } else {
      None
    }
  }

  #[inline]
  pub const fn as_ptr(&self) -> *mut T {
    self.ptr as *mut T
  }
}
```

我们看到有个 `?Sized` 的 `trait bound` 这玩意其实是指定一下泛型的特性, 意思是指编译时确定大小, 默认情况下直接写 `T` 就是 `Sized` 了, 加了个问号就会放宽约束的范围, 编译时不确定大小的也被接受. 加上 `#[inline]` 这个属性就表示内联函数, 因为这几个函数可能会经常使用到, 内联会带点加速效果.

## 内存分配

之后我们要考虑初始化容器了, 如果容器放了东西肯定会开辟内存空间, 但是初始化的情况容器应该是空的, 既然是空的肯定不会分配内存, 那就用 `MyUnique` 建个空的东西

``` rust
imp<T: ?Sized> MyUnique<T> {
  #[inline]
  pub const unsafe fn new_unchecked(ptr: *mut T) -> Self {
    MyUnique {
      ptr: ptr as _,
      _marker: PhantomData,
    }
  }
}

impl<T> MyUnique<T> {
  pub const fn empty() -> Self {
    unsafe { MyUnique::new_unchecked(mem::align_of::<T>() as *mut T) }
  }
}

pub struct MyVec<T> {
  ptr: MyUnique<T>,
  cap: usize,
  len: usize,
}

impl<T> MyVec<T> {
  fn new() -> Self {
    assert_ne!(mem::size_of::<T>(), 0, "We're not ready to handle ZSTs");
    MyVec {
      ptr: MyUnique::empty(),
      len: 0,
      cap: 0,
    }
  }
}
```

接下来写一下内存分配相关的代码, 既然我们需要分配内存, 这块其实没什么东西, 就是读一下内存分配的文档然后使用起来, 当然也要考虑对齐. 其实这是个扩容处理

``` rust
use std::alloc::{handle_alloc_error, realloc, Layout};
use std::mem;

impl<T> MyVec<T> {
  fn grow(&self) -> Self {
    unsafe {
      let (new_cap, ptr) = if self.cap == 0 {
        let ptr = alloc(Layout::array::<T>(1).unwrap());
        (1, ptr)
      } else {
        let new_cap = self.cap * 2;
        let layout = Layout::array::<T>(self.cap).unwrap();
        let ptr = realloc(self.ptr.as_ptr() as *mut _, layout, layout.size());
        if ptr.is_null() {
          handle_alloc_error(Layout::from_size_align_unchecked(
            new_cap * elem_size,
            mem::align_of::<T>(),
          ));
        }
        (new_cap, ptr)
      };

      Self {
        ptr: MyUnique::new_unchecked(ptr as *mut _),
        cap: new_cap,
        len: self.len,
      }
    }
  }
}
```

## push & pop

上面已经做到能分配内存了, 接下来自然是实现基本的功能.

先分析一波 **push** 的行为, **push** 就是往动态数组添加元素, 如果满了就需要重新分配内存, 其实就是调用 `grow`, 每次添加元素后长度也要相应 `+1`, 还需要对相应地址写元素. 写入行为用 `std::ptr` 的 `write` 函数来处理, 相应的 `pop` 也很好理解, 只要把最后一个读取出来同时把长度 `-1`

``` rust
use std::{mem, ptr};

impl<T> MyVec<T> {
  pub fn ptr(&self) -> *mut T {
    self.ptr.as_ptr()
  }

  pub fn push(&mut self, element: T) {
    if self.len == self.cap {
      self.grow();
    }
    unsafe {
      ptr::write(self.ptr().add(self.len), element);
    }
    self.len += 1;
  }

  pub fn pop(&mut self) -> Option<T> {
    if self.len == 0 {
      None
    } else {
      self.len -= 1;
      unsafe { Some(ptr::read(self.ptr().add(self.len))) }
    }
  }
}
```

## 回收资源

**Rust** 的机制让我们处理这个问题非常简单, 只需要实现一下 `trait Drop`

``` rust
impl<T> Drop for MyVec<T> {
  fn drop(&mut self) {
    let elem_size = mem::size_of::<T>();
    if elem_size != 0 {
      while let Some(_) = self.pop() {}
      unsafe {
        dealloc(self.ptr() as *mut _, Layout::array::<T>(self.cap).unwrap());
      }
    }
  }
}
```

## 解引用

至此, 已经实现了一个简单的数据结构, 但是我们还没法跟 `slice` 相通, 真实的 **Vec** 不是这样的, 所以当下我们应该实现一下自动解引用, 只要实现了下面这些东西, 我们就可以用 `slice` 的提供的接口了

``` rust
use std::ops::{Deref, DerefMut};

impl<T> Deref for MyVec<T> {
  type Target = [T];

  fn deref(&self) -> &[T] {
    unsafe { std::slice::from_raw_parts(self.ptr(), self.len) }
  }
}

impl<T> DerefMut for MyVec<T> {
  fn deref_mut(&mut self) -> &mut [T] {
    unsafe { std::slice::from_raw_parts_mut(self.ptr(), self.len) }
  }
}
```

## 插入跟删除

### 插入

插入的行为其实就是把当前要插入的位置之后的所有元素分别向右移动一位, 譬如一个数组 `[1, 2, 3]`, 我要把 `10` 插入 索引 `1` 位置(就是元素 `2`), 那么 `10` 的下标就是 `1`, 同时元素 `2` 跟 `3` 的下标就是 `2` 跟 `3`, 最后就变成了 `[1, 10, 2, 3]`.

``` rust
impl<T> MyVec<T> {
  // ...
  pub fn insert(&mut self, index: usize, element: T) {
    if self.cap == self.len {
      self.grow();
    }
    unsafe {
      if index < self.len {
        ptr::copy(self.ptr().add(index), self.ptr().add(index + 1), self.len - index);
      }
      ptr::write(self.ptr().add(index), element);
      self.len += 1;
    }
  }
}
```

### 删除

删除的行为也很好理解, 跟插入反着来就可以了, 只要把要删除的位置之后所有的下标向左移动一位, 譬如现在把之前插入后的数组 `[1, 10, 2, 3]` `10` 给删除掉, `10` 所在的下标是 `1`, 后面的元素的下标分别是 `2` 跟 `3`, 后面的 `-1` 就完成了.

``` rust
impl<T> MyVec<T> {
  // ...
  pub fn remove(&mut self, index: usize) -> T {
    assert!(index < self.len, "index out of bounds");
    unsafe {
      self.len -= 1;
      let result = ptr::read(self.ptr().add(index));
      ptr::copy(self.ptr().add(index + 1), self.ptr().add(index), self.len - index);
      result
    }
  }
}
```

## IntoIter

这趟开始处理一下 **Vec** 才有的迭代器, 其实只要实现了自动解引用的 `trait`, 就可以使用 `slice` 的 `iter` 还有 `iter_mut`, 但是 `slice` 是没有 `into_iter` 的, 所以我们得实现一下.
现在有个问题, 既然已经有了 `slice` 的迭代器功能了, 我们为什么要实现这个 `IntoIter`?
我们可以看到一个 **Vec** 可以直接用 `for` 进行循环遍历, 原因就是只要一个自定义的类型实现了 `IntoIter` 就具备能被 `for` 迭代的能力

``` rust
let v = vec![1, 2, 3];
for i in v {
  dbg!(i);
}
```

现在可以用两个指针来处理迭代器的操作, 一个在开头, 一个在结尾后面那一个, 只要开头的指针跟结尾后一个的指针地址相同, 就表明迭代结束了.

``` sql
[1, 2, 3, 4, 5, sth]
 ^              ^
start           end
```

现在来建立个迭代器结构, 大概长这样

``` rust
struct IntoIter<T> {
  start: *const T,
  end: *const T,
}
```

当然后续还要处理内存相关的, 所以我们应该把 **Vec** 分配的空间讯息保存一下, 当然还要把 `MyVec` 转化成 `IntoIter` 类型

``` rust
struct IntoIter<T> {
  start: *const T,
  end: *const T,
  buf: MyUniuqe<T>,
  cap: usize,
}

impl<T> MyVec<T> {
  fn into_iter(self) -> IntoIter<T> {
    let MyVec { ptr, cap, len } = self;
    mem::forget(self);
    unsafe {
      IntoIter {
        buf: ptr,
        cap,
        start: ptr.as_ptr(),
        end: if cap == 0 { ptr.as_ptr() } else { ptr.as_ptr().add(len) },
      }
    }
  }
}
```

还要实现一下迭代器, `size_hint` 是仿写标准库的, 主要作用是表达剩余可迭代元素数量上下界, 下面是 `next` 相关的操作

``` rust
impl<T> Iterator for IntoIter<T> {
  type Item = T;
  fn next(&mut self) -> Option<T> {
    if self.start == self.end {
      None
    } else {
      unsafe {
        let result = ptr::read(self.start);
        self.start = self.start.offset(1);
        Some(result)
      }
    }
  }

  fn size_hint(&self) -> (usize, Option<usize>) {
    let len = (self.end as usize - self.start as usize) / mem::size_of::<T>();
    (len, Some(len))
  }
}
```

还有 `next_back` 的操作

``` rust
impl<T> DoubleEndedIterator for IntoIter<T> {
  fn next_back(&mut self) -> Option<T> {
    if self.start == self.end {
      None
    } else {
      unsafe {
        self.end = self.end.offset(-1);
        Some(ptr::read(self.end))
      }
    }
  }
}
```

为了处理内存相关的, 我们要给 `IntoIter` 实现一下 `Drop trait`

``` rust
impl<T> Drop for IntoIter<T> {
  fn drop(&mut self) {
    if self.cap != 0 {
      for _ in &mut *self {}
      unsafe {
        dealloc(self.buf.as_ptr() as *mut _, Layout::array::<T>(self.cap).unwrap());
      }
    }
  }
}
```

## RawVec

现在继续重构代码, 因为我们分别给 `IntoIter` 跟 `MyVec` 实现了一遍 `Drop`, 所以重构一下代码是有必要的

``` rust
pub struct MyVec<T> {
  buf: RawVec<T>,
  len: usize,
}

impl<T> MyVec<T> {
  pub fn push(&mut self, element: T) {
    if self.len == self.cap() {
      self.buf.grow();
    }
    unsafe {
      ptr::write(self.ptr().add(self.len), element);
    }
    self.len += 1;
  }

  pub fn pop(&mut self) -> Option<T> {
    if self.len == 0 {
      None
    } else {
      self.len -= 1;
      unsafe { Some(ptr::read(self.ptr().add(self.len))) }
    }
  }

  pub fn insert(&mut self, index: usize, element: T) {
    assert!(index <= self.len, "index out of bounds");
    if self.cap() == self.len {
      self.buf.grow();
    }
    unsafe {
      if index < self.len {
        ptr::copy(self.ptr().add(index), self.ptr().add(index + 1), self.len - index);
      }
      ptr::write(self.ptr().add(index), element);
      self.len += 1;
    }
  }

  pub fn remove(&mut self, index: usize) -> T {
    assert!(index < self.len, "index out of bounds");
    unsafe {
      self.len -= 1;
      let result = ptr::read(self.ptr().add(index));
      ptr::copy(self.ptr().add(index + 1), self.ptr().add(index), self.len - index);
      result
    }
  }
}

impl<T> Drop for MyVec<T> {
  fn drop(&mut self) {
    while let Some(_) = self.pop() {}
  }
}

struct IntoIter<T> {
  start: *const T,
  end: *const T,
  _buf: RawVec<T>,
}

impl<T> Iterator for IntoIter<T> {
  type Item = T;
  fn next(&mut self) -> Option<T> {
    if self.start == self.end {
      None
    } else {
      unsafe {
        let result = ptr::read(self.start);
        self.start = self.start.offset(1);
        Some(result)
      }
    }
  }

  fn size_hint(&self) -> (usize, Option<usize>) {
    let len = (self.end as usize - self.start as usize) / mem::size_of::<T>();
    (len, Some(len))
  }
}

impl<T> DoubleEndedIterator for IntoIter<T> {
  fn next_back(&mut self) -> Option<T> {
    if self.start == self.end {
      None
    } else {
      unsafe {
        self.end = self.end.offset(-1);
        Some(ptr::read(self.end))
      }
    }
  }
}

impl<T> Drop for IntoIter<T> {
  fn drop(&mut self) {
    for _ in &mut *self {}
  }
}

struct RawVec<T> {
  ptr: MyUnique<T>,
  cap: usize,
}

impl<T> RawVec<T> {
  fn new() -> Self {
    RawVec {
      ptr: MyUnique::empty(),
      cap: 0,
    }
  }

  fn grow(&self) -> Self {
    unsafe {
      let (new_cap, ptr) = if self.cap == 0 {
        let ptr = alloc(Layout::array::<T>(1).unwrap());
        (1, ptr)
      } else {
        let new_cap = self.cap * 2;
        let layout = Layout::array::<T>(self.cap).unwrap();
        let ptr = realloc(self.ptr.as_ptr() as *mut _, layout, layout.size());
        if ptr.is_null() {
          handle_alloc_error(Layout::from_size_align_unchecked(
            new_cap * mem::size_of::<T>(),
            mem::align_of::<T>(),
          ));
        }
        (new_cap, ptr)
      };

      Self {
        ptr: MyUnique::new_unchecked(ptr as *mut _),
        cap: new_cap,
      }
    }
  }
}

impl<T> Drop for RawVec<T> {
  fn drop(&mut self) {
    if self.cap != 0 {
      unsafe {
        dealloc(self.ptr.as_ptr() as *mut _, Layout::array::<T>(self.cap).unwrap());
      }
    }
  }
}
```

## 抽取迭代操作

现在我们基本的 **Vec** 结构已经做出来了, 现在仿照之前的 `RawVec` 做一份封装.

``` rust
struct RawValIter<T> {
  start: *const T,
  end: *const T,
}

impl<T> RawValIter<T> {
  unsafe fn new(slice: &[T]) -> Self {
    RawValIter {
      start: slice.as_ptr(),
      end: if slice.len() == 0 {
        slice.as_ptr()
      } else {
        slice.as_ptr().offset(slice.len() as isize)
      },
    }
  }
}

impl<T> Iterator for RawValIter<T> {
  type Item = T;

  fn next(&mut self) -> Option<T> {
    if self.start == self.end {
      None
    } else {
      unsafe {
        let result = ptr::read(self.start);
        self.start = self.start.offset(1);
        Some(result)
      }
    }
  }

  fn size_hint(&self) -> (usize, Option<usize>) {
    let len = (self.end as usize - self.start as usize) / mem::size_of::<T>();
    (len, Some(len))
  }
}

impl<T> DoubleEndedIterator for RawValIter<T> {
  fn next_back(&mut self) -> Option<T> {
    if self.start == self.end {
      None
    } else {
      unsafe {
        self.end = self.end.offset(-1);
        Some(ptr::read(self.end))
      }
    }
  }
}
```

然后改造一下迭代器, 现在只要在各函数内部调用 `RawValIter` 的实现就行了

``` rust
struct IntoIter<T> {
  _buf: RawVec<T>,
  iter: RawValIter<T>,
}

impl<T> Drop for MyVec<T> {
  fn drop(&mut self) {
    while let Some(_) = self.pop() {}
  }
}

impl<T> Iterator for IntoIter<T> {
  type Item = T;
  fn next(&mut self) -> Option<T> {
    self.iter.next()
  }

  fn size_hint(&self) -> (usize, Option<usize>) {
    self.iter.size_hint()
  }
}

impl<T> DoubleEndedIterator for IntoIter<T> {
  fn next_back(&mut self) -> Option<T> {
    self.iter.next_back()
  }
}

impl<T> Drop for IntoIter<T> {
  fn drop(&mut self) {
    for _ in &mut self.iter {}
  }
}
```

## 处理 Zero-Sized Types

通常情况下, **Rust** 是不需要处理 **Zero-Sized Types** 的, 但是现在我们的代码中有大量关于裸指针的操作, 假如给分配器传递 zst, 会导致未定义行为, 对 `zst` 裸指针进行 `offset` 是一个 `no-op` 行为.

先把 `new` 函数的 `cap` 处理一下, 如果是 `size_of` 处理出来的 `T` 是 `0` 的情况, 就给 `0` 按位取反(`usize::MAX`), 因为 `T` 的 `size_of` 为 `0` 其实不需要开辟内存, 反正你存进来的都是 `0`, 逻辑上不会占用内存.

``` rust
impl<T> RawVec<T> {
  fn new() -> Self {
    let cap = if mem::size_of::<T>() == 0 { !0 } else { 0 };
    RawVec {
      ptr: MyUnique::empty(),
      cap,
    }
  }
}
```

然后就是 `grow` 函数也处理一下

``` rust
impl<T> RawVec<T> {
  // ...
  fn grow(&self) -> Self {
    unsafe {
      let elem_size = mem::size_of::<T>();
      assert_ne!(elem_size, 0, "capacity overflow");

      let (new_cap, ptr) = if self.cap == 0 {
        let ptr = alloc(Layout::array::<T>(1).unwrap());
        (1, ptr)
      } else {
        let new_cap = self.cap * 2;
        let layout = Layout::array::<T>(self.cap).unwrap();
        let ptr = realloc(self.ptr.as_ptr() as *mut _, layout, layout.size());
        (new_cap, ptr)
      };

      if ptr.is_null() {
        handle_alloc_error(Layout::from_size_align_unchecked(
          new_cap * elem_size,
          mem::align_of::<T>(),
        ));
      }

      Self {
        ptr: MyUnique::new_unchecked(ptr as *mut _),
        cap: new_cap,
      }
    }
  }
}
```

`RawVec` 的 `Drop` 也需要处理, 其实之前的实现也可以, 但是假装对齐一下吧!

``` rust
impl<T> Drop for RawVec<T> {
  fn drop(&mut self) {
    let elem_size = mem::size_of::<T>();
    if self.cap != 0 && elem_size != 0 {
      unsafe {
        dealloc(
          self.ptr.as_ptr() as *mut _,
          Layout::from_size_align_unchecked(self.cap * elem_size, mem::align_of::<T>()),
        );
      }
    }
  }
}
```

然后是 `RawValIter` 的 `zst` 处理

``` rust
impl<T> RawValIter<T> {
  unsafe fn new(slice: &[T]) -> Self {
    RawValIter {
      start: slice.as_ptr(),
      end: if mem::size_of::<T>() == 0 {
        ((slice.as_ptr() as usize) + slice.len()) as *const _
      } else if slice.len() == 0 {
        slice.as_ptr()
      } else {
        slice.as_ptr().offset(slice.len() as isize)
      },
    }
  }
}
```

迭代器也处理一下, `size_hint` 除数为 `0` 的情况需要处理

``` rust
impl<T> Iterator for RawValIter<T> {
  type Item = T;

  fn next(&mut self) -> Option<T> {
    if self.start == self.end {
      None
    } else {
      unsafe {
        let result = ptr::read(self.start);
        self.start = if mem::size_of::<T>() == 0 {
          (self.start as usize + 1) as *const _
        } else {
          self.start.offset(1)
        };
        Some(result)
      }
    }
  }

  fn size_hint(&self) -> (usize, Option<usize>) {
    let elem_size = mem::size_of::<T>();
    let len = (self.end as usize - self.start as usize) / (if elem_size == 0 { 1 } else { elem_size });
    (len, Some(len))
  }
}

impl<T> DoubleEndedIterator for RawValIter<T> {
  fn next_back(&mut self) -> Option<T> {
    if self.start == self.end {
      None
    } else {
      unsafe {
        self.end = if mem::size_of::<T>() == 0 {
          (self.end as usize - 1) as *const _
        } else {
          self.end.offset(-1)
        };
        Some(ptr::read(self.end))
      }
    }
  }
}
```

## with_capacity

这个实现一下差不多算结束了, 先把 `MyVec` 改一下

``` rust
impl<T> MyVec<T> {
  // ...
  pub fn with_capacity(capacity: usize) -> MyVec<T> {
    MyVec {
      buf: RawVec::with_capacity(capacity),
      len: 0
    }
  }
}
```

然后给 `RawVec` 添加个接口

``` rust
impl<T> RawVec<T> {
  fn new() -> Self {
    let cap = if mem::size_of::<T>() == 0 { !0 } else { 0 };
    RawVec {
      ptr: MyUnique::empty(),
      cap,
    }
  }

  pub fn with_capacity(cap: usize) -> Self {
    RawVec::allocate_in(cap, None)
  }

  fn allocate_in(cap: usize, p: Option<MyUnique<T>>) -> Self {
    unsafe {
      let elem_size = mem::size_of::<T>();
      assert_ne!(elem_size, 0, "capacity overflow");

      let (new_cap, ptr) = if cap == 0 {
        let ptr = alloc(Layout::array::<T>(1).unwrap());
        (1, ptr)
      } else {
        if let Some(some_p) = p {
          let new_cap = cap * 2;
          let layout = Layout::array::<T>(cap).unwrap();
          let ptr = realloc(some_p.as_ptr() as *mut _, layout, layout.size());
          (new_cap, ptr)
        } else {
          let ptr = alloc(Layout::array::<T>(cap).unwrap());
          (cap, ptr)
        }
      };

      if ptr.is_null() {
        handle_alloc_error(Layout::from_size_align_unchecked(
          new_cap * elem_size,
          mem::align_of::<T>(),
        ));
      }

      Self {
        ptr: MyUnique::new_unchecked(ptr as *mut _),
        cap: new_cap,
      }
    }
  }

  fn grow(&self) -> Self {
    RawVec::allocate_in(self.cap, Some(self.ptr))
  }
}
```

现在已经把 `grow` 抽取出来, 同时给 `with_capacity` 调用