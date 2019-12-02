# Iteration and Concurrency

类似于同步  方式的`Iterator`，这里有很多不同的方法可以迭代和处理一个`Stream`中的值。有组合器样式的方法，例如`map`，`filter`和`fold`和他们的有错误就早退的表弟`try_map`，`try_filter`和`try_fold`。

不幸，`for`循环不适用于`Stream`s，但对于命令式代码，`while let`和`next`/`try_next`函数可以这样用：

```rust
{{#include ../../examples/05_02_iteration_and_concurrency/src/lib.rs:nexts}}
```

但是，如果我们一次只处理一个元素，则可能会失去了并发的机会，这毕竟这是我们要编写异步代码的首要原因。要同时处理一个 stream 中的多个 items，请使用`for_each_concurrent`和`try_for_each_concurrent`方法：

```rust
{{#include ../../examples/05_02_iteration_and_concurrency/src/lib.rs:try_for_each_concurrent}}
```
