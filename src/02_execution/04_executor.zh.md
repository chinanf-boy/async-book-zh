# Applied: Build an Executor

Rust 的`Future`是懒惰的：除非是向着'完成'这一个目标积极前进，否则他们不会做任何事情。向 Future 完成前进的一种方法是，在`async`函数里面，对它`.await`，但这只会将问题升了个级：谁来管理，从顶层 `async`函数返回的 Futures ？答案是：我们需要一个`Future`执行者（executor）。

`Future` executor 获取一组顶层`Future`，并每当`Future`可以前进时，通过调用`poll`，让它们驶向完成。通常一旦开始，executor 会`poll`一个 Future 。当`Future`表示，因`wake()`的调用准备好前进，会将它们先放回到一个队列，才再次`poll`，重复直到`Future`已经完成。

在本节中，我们将编写自己的简单 executor，该 executor 能够让大量顶层 Future 同时驶向完成。

在此示例中，我们依赖`futures`箱子，`ArcWake` trait 会用到，它提供了一种轻松的方法来构建`Waker`。

```toml
[package]
name = "xyz"
version = "0.1.0"
authors = ["XYZ Author"]
edition = "2018"

[dependencies]
futures-preview = "=0.3.0-alpha.17"
```

接下来，我们需要在顶部，添加以下内容`src/main.rs`：

```rust
{{#include ../../examples/02_04_executor/src/lib.rs:imports}}
```

我们的 executor 的工作是，将通过发送任务，在通道上运行。executor 将事件从通道中拉出，并运行它们。当一个任务准备做更多的工作（被唤醒）时，它可以安排自己重新回到通道上，以计划再次进行轮询。

在这种设计中，executor 本身仅需要任务通道的接收端。用户将获得发送端，以便他们可以生成新的 Future 。任务本身就是 Future，是可以重新计划自己的。因此，我们会将它们与一个 sender 每每存储在一起，这样，任务就可以用来让自己重新排队。

```rust
{{#include ../../examples/02_04_executor/src/lib.rs:executor_decl}}
```

我们还向 spawner 添加一种方法，以使其易于生成新的 Future 。此方法将拿到一个 Future 类型，将其装箱，并放入 FutureObj 中，然后创建一个新类型`Arc<Task>`，它的内部可以在 executor 上，排队。

```rust
{{#include ../../examples/02_04_executor/src/lib.rs:spawn_fn}}
```

要轮询 Future ，我们需要创建一个`Waker`。正如在[唤醒章节][task wakeups section]，`Waker`负责安排，一旦`wake`调用了，就再次轮询的任务。记住，`Waker`s 是会告诉 executor，确切的那些任务已经准备就绪，只轮询准备好前进的 Future。创建一个新的`Waker`最简单的方法是，通过实现`ArcWake` trait ，然后使用`waker_ref`要么`.into_waker()`函数，将`Arc<impl ArcWake>`转变成一个`Waker`。让我们，为我们的任务实现`ArcWake`，这样就可以转变为`Waker`，和被唤醒啦：

```rust
{{#include ../../examples/02_04_executor/src/lib.rs:arcwake_for_task}}
```

当一个新建的`Waker`，从`Arc<Task>`而来，那么我们在它上面调用`wake()`，将导致`Arc`的一个 copy 发送到任务通道。然后，我们的 executor 需要选择任务，并进行轮询。让我们实现一下：

```rust
{{#include ../../examples/02_04_executor/src/lib.rs:executor_run}}
```

恭喜你！我们现在有一个能工作的 Future executor。我们甚至可以使用它，来运行`async/.await`代码和自定义 Future ，例如，我们之前写过的`TimerFuture`：

```rust
{{#include ../../examples/02_04_executor/src/lib.rs:main}}
```

[task wakeups section]: ./03_wakeups.zh.md
