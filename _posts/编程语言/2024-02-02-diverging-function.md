---
title: rust 中的发散函数
category: [编程语言]
tags: [diverging function, 发散函数, rust]
# image:
#     path: /assets/img/headers/diverging.webp
---

> 在 rust 中，发散函数（diverging functions）是指那些永远不会返回的函数。它们使用 ! 作为返回类型，来表明这个函数不会返回到调用它的代码。这通常用于表示函数会引发 panic 或者进入无限循环，或者有可能调用操作系统的退出函数。
{: .prompt-info }

## 发散函数的返回类型

发散函数的返回类型是 ! , 表示永远不会返回，无论你的函数返回值是啥，使用 ! 作为返回类型都是合法的，因为实际上永远不会返回，看下面的例子:

```rust
fn example(flag: bool) -> i32 {
    if flag {
        42
    } else {
        panic!("haha")
    }
}

fn main() {
    example(false);
}
```

在上面的例子中，panic! 是一个发散宏函数，其返回值为 ! ，表示永远不会返回所以即便 example 要求返回值是 i32，它也是合法的。

## 常见的发散函数场景
+ panic! 宏:
这个宏会使程序 panic, 终止后续的执行。任何调用 panic! 的函数都是发散函数。

```rust
fn diverges() -> ! {
    panic!("This function never returns!");
}
```

+ 无限循环:
一个无限的循环也会使函数永远不会返回, 所以这样的函数同样发散。

```rust
fn diverges() -> ! {
    loop {
        println!("This function never returns!");
    }
}
```

+ 调用 std::process::exit():
这个函数会终止当前进程, 所以任何调用它的函数也会发散。

```rust
fn diverges() -> ! {
    std::process::exit(1);
}
```

+ 调用 std::hint::unreachable_unchecked():
这个函数表示代码不应该执行到这里, 如果执行到了会终止程序。

```rust
fn diverges() -> ! {
    std::hint::unreachable_unchecked();
}
```


