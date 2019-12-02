# The `Future` Trait

`Future` trait 是 Rust 异步编程的中心。一个`Future`就是可以产生值的异步计算（尽管该值可能为空，例如`()`）。一种*简化*版本的 Future trait 可能如下所示：

```rust
{{#include ../../examples/02_02_future_trait/src/lib.rs:simple_future}}
```

Future 可以通过调用`poll`函数，推动 Future，让其尽可能接近完成。如果 Future 完成，它将返回`Poll::Ready(result)`。如果 Future 还不能完成，它将返回`Poll::Pending`，并安排`wake()`在`Future`准备进一步的时候，调用。当`wake()`调用，executor 驱使 `Future`，再次调用`poll`，这样`Future`就离完成再进一步了。

若是没有`wake()`，executor 将无法知道某个特定的 Future 何时可以前进，并且将不得不，不断地 poll 每个 Future 。而有了`wake()`，executor 就能确切知道哪些 Future，已准备`poll`ed。

例如，考虑以下情况：我们想从一个 socket(套接字)中，读取数据，而该 socket 的数据不知道有没有。如果**有**数据，我们可以读并返回`Poll::Ready(data)`，但如果没有任何数据 ready，那么我们的 Future 将阻塞，无法再前进。处在没有可用数据时期，当 socket 上的数据 ready 时，我们必须挂上要调用的`wake`，这将告诉 executor，我们的 Future 已准备好前进。

一个简单的`SocketRead` Future 可能看起来像这样：

```rust
{{#include ../../examples/02_02_future_trait/src/lib.rs:socket_read}}
```

这个`Future`s 模型可以将多个异步操作组合在一起，而无需中间分配。一次运行多个 Future 或将 Future 链接在一起，可以通过无分配状态机来实现，如下所示：

```rust
{{#include ../../examples/02_02_future_trait/src/lib.rs:join}}
```

这显示了，如何在无需单独分配的情况下，同时运行多个 Future ，从而可以实现更高效的异步程序。同样，可以依次运行多个有序 Future ，如下所示：

```rust
{{#include ../../examples/02_02_future_trait/src/lib.rs:and_then}}
```

这些示例说明了`Future`trait 可用于表示异步控制流，而无需多个分配的对象，和深层嵌套的回调。随着基本控制流程的发展，让我们谈谈真正的`Future` trait 及其不同之处。

```rust
{{#include ../../examples/02_02_future_trait/src/lib.rs:real_future}}
```

您会注意到的第一个变化是`self`类型，不再`&mut self`，而更改为`Pin<&mut Self>`。我们将详细讨论 pinning 在[稍后章节][pinning]，但现在知道它使我们能够创建 Immovable(无法移动) 的 Future 。无法移动的对象可以在其字段之间存储指针，例如`struct MyFut { a: i32, ptr_to_a: *const i32 }`。Pinning 是启用 async/await 所必需的。

其次，`wake: fn()`已更改为`&mut Context<'_>`。在`SimpleFuture`，我们使用了对函数指针（`fn()`）的一个 call，去告诉 Future 的 executor，应该对有问题的 Future 进行 poll。但是，由于`fn()`大小为零(zero-sized)，无法存储有关*哪一个* `Future`调用了`wake`。

在现实世界中，像 Web 服务器这样的复杂应用程序，可能具有成千上万个不同的连接，其唤醒都应单独进行管理。`Context` type 通过提供对一个`Waker`类型值的访问，来解决此问题，可用于唤醒特定任务。

[pinning]: ../04_pinning/01_chapter.zh.md
