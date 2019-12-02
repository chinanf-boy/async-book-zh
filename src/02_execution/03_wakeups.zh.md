# Task Wakeups with `Waker`

Future 一次`poll`ed 就能完成的，并不常见。而多数情况下，Future 需要确保一旦准备好前进，就再次进行轮询(poll) 。而这是通过`Waker`类型，辅助完成的。

每次 Future poll 时，都会将其作为“任务(task)”的一部分。任务是已提交给 executor 的顶级 Future 。

`Waker`提供一个`wake()`方法，它可以用来告诉 executor，应该唤醒的相关**任务**。当`wake()`被调用时， executor 知道与`Waker`相关联的**任务**是准备前进，并且，它的 Future 应再次进行 poll。

`Waker`还实现了`clone()`，这样就可以将其复制和存储。

让我们尝试使用`Waker`，实现一个简单的计时器 future。

## Applied: Build a Timer

在本示例中，我们将在创建计时器（Timer）时，启动一个新线程，休眠下所需的时间，然后在时间窗口 elapsed(逝去) 后，向计时器发出信号。

这是我们需要开始的导入：

```rust
{{#include ../../examples/02_03_timer/src/lib.rs:imports}}
```

让我们从定义 future 类型本身开始。我们的 future 需要一种方法，来让线程可以传达，timer elapsed 和 这个 future 应该完成的信息。我们将使用一个`Arc<Mutex<..>>`共享值，在线程和 Future 之间进行通信。

```rust
{{#include ../../examples/02_03_timer/src/lib.rs:timer_decl}}
```

现在，让我们实际编写`Future`实现！

```rust
{{#include ../../examples/02_03_timer/src/lib.rs:future_for_timer}}
```

很简单，对吧？如果线程设置了`shared_state.completed = true`，我们就搞定了！不然的话，我们会为当前任务，clone `Waker`，并将其传递给`shared_state.waker`，这样线程才能唤醒备份的任务。

重要的是，每次 Future 进行 poll，我们必须更新`Waker`，因为 Future 可能已经转移到，具有一个不同`Waker`的不同任务上了。这种情况在 Future poll 后，在任务之间传来传去时，会发生。

最后，我们需要实际构造计时器的 API ，并启动线程：

```rust
{{#include ../../examples/02_03_timer/src/lib.rs:timer_new}}
```

Woot！这就是我们构建一个简单的计时器 future 的全部。现在，如果我们只有一个 executor，来运行 future ...
