# `join!`

`futures::join`宏的魔力在于，同时执行 Futures 时，等待多个不同的 Futures 完成。

# `join!`

当执行多个异步操作时，一串`.await`，就搞定他们：

```rust
{{#include ../../examples/06_02_join/src/lib.rs:naiive}}
```

但是，这还是比所要的速度慢，因为它不会在`get_book`已经完成之后，开始尝试`get_music`。在其他一些语言中， Future 是环境运行到完成，因此可以，先调用每个`async fn`，来开始 futures，这样两个操作就是同时运行的，然后就是等待两个：

```rust
{{#include ../../examples/06_02_join/src/lib.rs:other_langs}}
```

但是，Rust Futures 在处于`.await`ed 之前不会做任何工作。这意味着，上面的两个代码片段，都将连续运行`book_future`和`music_future`，而不是同时运行它们。要同时正确运行两个 Future ，请使用`futures::join!`：

```rust
{{#include ../../examples/06_02_join/src/lib.rs:join}}
```

`join!`传回的值，是一个元组，包含每个传递进去的`Future`的输出。

## `try_join!`

要想 Futures 返回的是 `Result`，请考虑使用`try_join!`而不是`join!`。只因`join!`仅在所有子 Future 都完成后，才完成，即便是它的其中一个 subfutures 是返回了一个`Err`。

不像`join!`，在`try_join!`中，如果其中一个 subfutures 返回一个错误，将立即完成。

```rust
{{#include ../../examples/06_02_join/src/lib.rs:try_join}}
```

请注意， 传递给`try_join!`的 Futures 必须都具有相同的错误类型。考虑使用`futures::future::TryFutureExt`中的`.map_err(|e| ...)`和`.err_into()`函数，来合并错误类型：

```rust
{{#include ../../examples/06_02_join/src/lib.rs:try_join_map_err}}
```
