# `async`/`.await`

在[第一章节][the first chapter]，我们简要介绍了`async`/`.await`，并用它来构建一个简单的服务器。本章将更为详细讨论`async`/`.await`的它如何工作以及如何`async`代码与传统的 Rust 程序不同。

`async`/`.await`是 Rust 语法的特殊部分，它使得可以 yield 对当前线程的控制而不是阻塞，从而允许在等待操作完成时，其他代码可以运行。

`async`有两种主要的使用方式：`async fn`和`async`代码块。每个返回一个实现`Future`trait：

```rust
{{#include ../../examples/03_01_async_await/src/lib.rs:async_fn_and_block_examples}}
```

正如我们在第一章所看到的，`async`主体和其他 Future 是懒惰的：它们在运行之前什么也不做。最常见的，运行一个`Future`的方式是`.await`它。当在`Future`上调用`.await`的时候，它将尝试运行它，以完成操作。如果`Future`阻塞，它将 yield（归还）当前线程的控制。当 Future 可以更进一步时，`Future`将由 executor 接管并恢复运行，允许`.await`搞定这个 future。

## `async` Lifetimes

与传统函数不同，`async fn`会接受一个引用，或其他非`'static`的参数，并返回一个`Future`，这受到这些个参数生命周期的限制：

```rust
{{#include ../../examples/03_01_async_await/src/lib.rs:lifetimes_expanded}}
```

这意味着，来自一个`async fn`的 Future 必须被`.await`ed，期间它的非`'static`参数仍然有效。在通常情况下，在调用`.await`之后，会立即执行 （如`foo(&x).await`），而这并不是问题。但是，如果存储这个 Future，或将其发送到另一个任务或线程，则可能会出现问题。

一种常见的变通方法是，将引用作为参数的`async fn`函数转换成一个`'static` Future，具体是在一个`async`代码块里面，将这个参数与这`async fn`的调用捆绑在一起：

```rust
{{#include ../../examples/03_01_async_await/src/lib.rs:static_future_with_borrow}}
```

通过将参数移到`async`代码块，我们延长了其生命周期，以匹配，来自`good`的调用返回的`Future`。

## `async move`

`async`代码块和闭包允许`move`关键字，很像普通的闭包。一个`async move`代码块将拥有，对其引用的变量的所有权，从而使生命周期超过当前范围，但放弃了与其他代码共享这些变量的能力：

```rust
{{#include ../../examples/03_01_async_await/src/lib.rs:async_move_examples}}
```

## `.await`ing on a Multithreaded Executor

请注意，使用一个多线程的`Future` executor，一个`Future`可能会在线程之间移动，因此在`async`主体内的任何使用变量，必须能够在线程之间移动，正如任一`.await`都具有在一个 switch，就去到新线程的潜在结果。

这意味着，使用`Rc`，`&RefCell`或任何其他未实现`Send` trait，包括那些未实现`Sync`trait 的类型的引用都是不安全。

（注意：在调用`.await`期间，只要它们不在范围内，就有可能使用这些类型）

同样，想在一个`.await`上，搞个传统的 non-futures-aware 锁，也不是一个好主意，因为它可能导致线程池锁定：一项任务可以拿一个锁，之后`.await`并 yield 到 executor，而再允许另一个任务尝试获取该锁，也就会导致死锁。为避免这种情况，请在`futures::lock`使用`Mutex`，而不是`std::sync`里的那个。

[the first chapter]: ../01_getting_started/04_async_await_primer.zh.md
