# Recursion

在内部，`async fn`创建一个状态机类型，它包含每个子-`Future`，且都正处于`.await`ed。这使得递归`async fn`有点棘手，因状态机类型的结果必须包含自身：

```rust
// 这个函数:
async fn foo() {
    step_one().await;
    step_two().await;
}
// 生成了一个类型，如下:
enum Foo {
    First(StepOne),
    Second(StepTwo),
}

// 所以，这个函数:
async fn recursive() {
    recursive().await;
    recursive().await;
}

// 就生成了一个类型，如下:
enum Recursive {
    First(Recursive),
    Second(Recursive),
}
```

这行不通——我们创建了一个无限大的类型！编译器会抱怨：

```
error[E0733]: recursion in an `async fn` requires boxing
 --> src/lib.rs:1:22
  |
1 | async fn recursive() {
  |                      ^ an `async fn` cannot invoke itself directly
  |
  = note: a recursive `async fn` must be rewritten to return a boxed future.
```

为了搞定这一点，我们必须用`Box`剑走偏锋。但不幸的是，编译器的局限性意味着，仅将对`recursive()`的 call 包裹进`Box::pin`，是还不够的，我们必须将`recursive`变成非`async`函数，且它返回一个`.boxed()`
`async`代码块：

```rust
{{#include ../../examples/07_05_recursion/src/lib.rs:example}}
```
