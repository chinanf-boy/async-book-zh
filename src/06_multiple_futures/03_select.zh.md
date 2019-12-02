# `select!`

`futures::select`宏能同时运行多个 Future ，从而使用户，可以在任何 Future 完成后，立即做出响应。

```rust
{{#include ../../examples/06_03_select/src/lib.rs:example}}
```

上面的函数将同时运行`t1`和`t2`同时。当`t1`或是`t2`完成后，相应的处理程序将调用`println!`，而该函数将在不完成剩余任务的情况下，结束。

`select`的基本语法是`<pattern> = <expression> => <code>,`，重复您想要在`select`上使用的，任意数量的 Future 。

## `default => ...` and `complete => ...`

`select`也支持`default`和`complete`分支。

如果`select`了的 Futures 没有一个是完成的，`default`分支将运行。`select`带上一个`default`分支的组合，始终会立即返回，因为`default`在其他 Future 均未准备好，就运行了。

`complete`分支可以用来处理，`select`ed Future 全部完成，并且将不再前进。对一个`select!`循环访问时通常很方便。

```rust
{{#include ../../examples/06_03_select/src/lib.rs:default_and_complete}}
```

## Interaction with `Unpin` and `FusedFuture`

您可能在上面的第一个示例中，注意到的一件事是，我们必须在两个`async fn`返回的 Future 上调用`.fuse()`，以及将用`pin_mut`固定。这两个调用都是必需的，因为在`select`中使用的 futures ，必须同时实现了`Unpin` trait 与`FusedFuture` trait。

`Unpin`是必要的，因为`select`不是取值的，而是可变的引用。由于不拥有 Future 的所有权，因此可以在对`select`的调用后，未完成的 futures 还可以再次使用。

同样，`FusedFuture` trait 是必需的，因为`select`在一个 future 完成后，必不得对它再轮询。`FusedFuture`是 一个 Future trait，作用是追踪 Future 本身是否完成的。这样使得，`select`能在一个循环中使用，只轮询仍未完成的 Futures。可以在上面的示例中看到，其中`a_fut`或是`b_fut`是在循环第二次时完成。因为 `future::ready`返回的 Future 实现了`FusedFuture`，它可以告诉`select`不要再次轮询。

请注意，streams 具有对应的`FusedStream` trait。实现此 trait 或由`.fuse()`封装的 Streams，会从他们的 Future `.next()`/`.try_next()`组合器中， yield 出`FusedFuture` futures。

```rust
{{#include ../../examples/06_03_select/src/lib.rs:fused_stream}}
```

## Concurrent tasks in a `select` loop with `Fuse` and `FuturesUnordered`

一个有点难以发现但方便的函数是`Fuse::terminated()`，它允许构造一个已经终止的，空的 Future，之后可以用需要运行的 Future 填充它。

有个方便的情况就是，有一个任务需要在一个`select`循环内运行，但这个循环又是在这个`select`循环本身里面创建的。

注意使用`.select_next_some()`函数。可以与`select`合作，只运行那些由 stream 返回的`Some(_)`值，而忽略`None`s。

```rust
{{#include ../../examples/06_03_select/src/lib.rs:fuse_terminated}}
```

如果需要同时运行多个相同 Future 的副本，请使用`FuturesUnordered`类型。以下示例与上面的示例相似，但是将运行`run_on_new_num_fut`的每个副本，直到完成，而不是在创建新的时，终止它们。还会打印出一个由`run_on_new_num_fut`返回的值。

```rust
{{#include ../../examples/06_03_select/src/lib.rs:futures_unordered}}
```
