# `Send` Approximation

一些`async fn`状态机可以安全地越过线程发送，而其他则不能。判断一个`async fn` `Future`是不是`Send`，由非`Send`类型是否越过一个`.await`据点决定的。当可以越过了`.await`据点，编译器会尽力去估计这个是/否。但是今天的许多地方，这种分析都太保守了。

例如，考虑一个简单的非`Send`类型，也许包含一个`Rc`：

```rust
use std::rc::Rc;

#[derive(Default)]
struct NotSend(Rc<()>);
```

类型`NotSend`的变量可以短暂的，像暂时变量一样出现在`async fn`s，即使说`async fn`返回的`Future`类型结果，一定要是`Send`：

```rust
# use std::rc::Rc;
# #[derive(Default)]
# struct NotSend(Rc<()>);
async fn bar() {}
async fn foo() {
    NotSend::default();
    bar().await;
}

fn require_send(_: impl Send) {}

fn main() {
    require_send(foo());
}
```

但是，如果我们对`foo`修改一下，将`NotSend`存储在一个变量中，那么此示例不再编译：

```rust
# use std::rc::Rc;
# #[derive(Default)]
# struct NotSend(Rc<()>);
async fn foo() {
    let x = NotSend::default();
    bar().await;
}
```

```
error[E0277]: `std::rc::Rc<()>` cannot be sent between threads safely
  --> src/main.rs:15:5
   |
15 |     require_send(foo());
   |     ^^^^^^^^^^^^ `std::rc::Rc<()>` cannot be sent between threads safely
   |
   = help: within `impl std::future::Future`, the trait `std::marker::Send` is not implemented for `std::rc::Rc<()>`
   = note: required because it appears within the type `NotSend`
   = note: required because it appears within the type `{NotSend, impl std::future::Future, ()}`
   = note: required because it appears within the type `[static generator@src/main.rs:7:16: 10:2 {NotSend, impl std::future::Future, ()}]`
   = note: required because it appears within the type `std::future::GenFuture<[static generator@src/main.rs:7:16: 10:2 {NotSend, impl std::future::Future, ()}]>`
   = note: required because it appears within the type `impl std::future::Future`
   = note: required because it appears within the type `impl std::future::Future`
note: required by `require_send`
  --> src/main.rs:12:1
   |
12 | fn require_send(_: impl Send) {}
   | ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

error: aborting due to previous error

For more information about this error, try `rustc --explain E0277`.
```

此错误是正确的。如果我们将`x`存储到一个变量中，在`.await`搞完之前，这个变量都不会 drop，而此时，这个`async fn`有可能在其他线程上运行。因`Rc`不是`Send`，让它越过线程传播是不合理的。一个简单的解决方案是在`.await`之前，就对`Rc`进行`drop`，但是很遗憾，今天这个还不能用。

为了成功解决此问题，您可能必须引入一个封装了所有非`Send`的变量。这使编译器更容易知道这些变量，不越过`.await`据点。

```rust
# use std::rc::Rc;
# #[derive(Default)]
# struct NotSend(Rc<()>);
async fn foo() {
    {
        let x = NotSend::default();
    }
    bar().await;
}
```
