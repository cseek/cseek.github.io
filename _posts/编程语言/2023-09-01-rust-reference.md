---
title: rust 中 ref 和 & 的区别
category: 编程语言
tags: [reference, 引用, ref, rust]
# image:
#     path: /assets/img/headers/reference.webp
---

在 rust 语言中，有时候使用 ref 来创建一个引用，有时候使用 & 来创建一个引用，二者有什么区别？
其实在 rust 2018 版本中，通常不需要显式地使用 ref 关键字，直接使用 & 来借用值，ref 主要在 rust 的早期版本中更常用，但在现代 rust 中，直接使用 & 更加普遍和推荐。不过，ref 在某些复杂的模式匹配场景中仍然可以使用，请看下面的代码。

```rust
fn main() {
    // 创建一个元组
    let tuple = (String::from("hello"), String::from("world"));
    // 创建这个元组的引用    
    let (a, b) = &tuple;
    // 等效于
    // let (ref a, ref b) = tuple;
}
```
在上面的例子中，我们使用 & 来创建了一个元组的引用，但是这将引用一个整体，假如我只想引用元组的第一个或者第二个元素，这个时候就需要使用 ref 来实现，请看下面代码

```rust
fn main() {
    // 创建一个元组
    let tuple = (String::from("hello"), String::from("world"));
    // 创建这个元组的引用, 只引用了元组的第一个元素，第二个元素的所有权发生了转移    
    let (ref a, b) = tuple;
}
```