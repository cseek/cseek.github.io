---
title: rust 闭包之Fn、FnMut、FnOnce 的区别
category: [编程语言]
tags: [lamda, closure, 闭包, rust]
# image:
#     path: /assets/img/headers/closure.webp
---

> 在 rust 中，Fn、FnMut 和 FnOnce 是三种闭包（closure）类型，它们定义了闭包如何捕获外部环境中的变量，通常情况下，编译器会自动推断闭包的类型，但是当使用闭包作为参数时，需要显示指定闭包的类型。
- Fn：表示闭包可以通过引用来捕获外部变量（即不可变借用），并且可以多次调用。 
- FnMut：表示闭包可以通过可变引用来捕获外部变量（即可变借用），并且可以多次调用，但每次调用可能会改变捕获的变量。 
- FnOnce：表示闭包可以获取外部变量的所有权（即值传递），闭包可以调用一次，因为它可能消耗（move）了捕获的变量，使得变量在之后不再可用。
{: .prompt-info }

## Fn 用法示例
### 1、普通用法

```rust
fn main() {
    let x = 10;
    let equal_to_x = |z| z == x;
    let y = 10;
    assert!(equal_to_x(y));
}
```
在上面的代码中，由于闭包里没有改变捕获的参数的值或者所有权，所以闭包的类型会被自动推断为 Fn。

### 2、传参用法
```rust
fn function<F>(f: F) -> i32
where
    F: Fn() -> i32,
{
    f()
}

fn main() {
    let a = 10;
    let b = 20;
    let add = || a + b;
    let result = function(add);
    println!("{result}");
}

```
在上面的代码中，使用闭包作为参数传递时，需增加闭包类型的约束，显示的指出闭包的类型。

## FnMut 用法示例
### 1、普通用法

```rust
fn main() {
    let mut x = 10;
    let mut increment_x = || {
        x += 1;
        println!("x: {}", x);
    };

    increment_x(); // 调用闭包，x 变为 11
    increment_x(); // 再次调用闭包，x 变为 12
}
```
在上面的代码中，由于闭包里改变了捕获的参数的值，所以闭包的类型会被自动推断为 FnMut。


### 2、传参用法
```rust
fn function<F>(mut f: F)
where
    F: FnMut(),
{
    f();
}
fn main() {
    let mut x = 10;
    let increment_x = || x += 1;
    function(increment_x);
    println!("{x}");
}
```
在上面的代码中，显示的指出了闭包的类型为 FnMut。


## FnOnce 用法示例
### 1、普通用法

```rust
fn main() {
    let x = vec![1, 2, 3];
    // 一个消耗捕获变量的闭包
    let consume = || {
        let a = x;
        println!("a: {:?}", a);
        // 这里闭包通过值捕获 x，闭包调用后 x 不再可用
    };
    
    consume(); // 调用闭包，打印 x
    // consume(); // 再次调用闭包会报错，因为 x 的所有权已经被移走
}
```
在上面的代码中，由于闭包里改变了捕获的参数的所有权，所以闭包的类型会被自动推断为 FnOnce, 且闭包只能调用一次。


### 2、传参用法
```rust
fn function<F>(f: F)
where
    F: FnOnce(),
{
    f();
}

fn main() {
    let x = vec![1, 2, 3];
    // 一个消耗捕获变量的闭包
    let consume = || {
        let x = x;
        println!("x: {:?}", x);
        // 这里闭包通过值捕获 x，闭包调用后 x 不再可用
    };

    function(consume); // 调用闭包，打印 x
    // function(consume); // 再次调用闭包会报错，因为 x 的所有权已经被移走
}
```
在上面的代码中，显示的指出了闭包的类型为 FnOnce。