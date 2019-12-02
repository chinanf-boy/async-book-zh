# `async`/`.await` Primer

`async`/`.await`是 Rust 的内置工具，用于编写看起来像同步代码的异步函数。`async`将一个代码区块，转换为实现称为 `Future` trait 的状态机。而在同步方法中，调用阻塞函数将阻塞整个线程，`Future`s 将 yield 对线程的控制权，允许其他`Future`运行。

要创建异步功能，您可以使用`async fn`语法：

```rust
async fn do_something() { ... }
```

`async fn`传回的值，是一个`Future`。对要发生事情的衡量，`Future`需要在一个 executor 上运行。

```rust
{{#include ../../examples/01_04_async_await_primer/src/lib.rs:hello_world}}
```

在`async fn`内部， 您可以使用`.await`，等待另一种实现`Future` trait 的类型完成它的操作，例如另一个`async fn`的输出。不像`block_on`，`.await`不会阻塞当前线程，而是异步等待 future 完成。这样的话，如果 future 当前无法取得进展，则允许其他任务运行。

例如，假设我们有三个`async fn`：`learn_song`，`sing_song`和`dance`：

```rust
async fn learn_song() -> Song { ... }
async fn sing_song(song: Song) { ... }
async fn dance() { ... }
```

选择学习(learn_song)，唱歌(sing_song)和跳舞(dance)的任一种，都会分别阻塞每个人：

```rust
{{#include ../../examples/01_04_async_await_primer/src/lib.rs:block_on_each}}
```

但是，我们并没有达到这异步形式的最佳性能 —— 我们一次只能做一件事情！显然，我们必须先学习这首歌，然后才能唱歌，但是可以在学习和唱歌的同时跳舞。为此，我们可以创建两个单独的`async fn`，而它们可以同时运行：

```rust
{{#include ../../examples/01_04_async_await_primer/src/lib.rs:block_on_main}}
```

在此示例中，学习歌曲必须在唱歌之前发生，但是学习和唱歌都可以与跳舞同时发生。如果在`learn_and_sing`，我们使用`block_on(learn_song())`，而不是`learn_song().await`，那只要`learn_song`正在运行，该线程就无法执行其他任何操作。这样就不可能同时跳舞。通过对 `learn_song` future 使用`.await`，那么如果`learn_song`被阻塞，就允许其他任务接管当前线程。这样就可以在同一线程上，同时运行多个 futures。

现在，您已经了解了`async`/`await`的基础知识，让我们尝试一个例子。
