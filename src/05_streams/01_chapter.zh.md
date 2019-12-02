# The `Stream` Trait

`Stream` trait 类似于`Future`，但可以在完成之前，yield 多个值，类似于标准库的 `Iterator` trait：

```rust
{{#include ../../examples/05_01_streams/src/lib.rs:stream_trait}}
```

一个`Stream`的常见例子是，这个`Receiver`用于`futures`箱子的 channel 类型。它会在每次`Sender`端发送一个值，都会 yield `Some(val)`，并且一旦`Sender`被 dropped 和接收到了所有 pending 消息，就会 yield `None`：

```rust
{{#include ../../examples/05_01_streams/src/lib.rs:channels}}
```
