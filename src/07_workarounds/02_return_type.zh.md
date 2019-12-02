# Return Type Errors

在典型的 Rust 函数中，返回的值若是有个错误的类型，将导致出现如下所示的错误：

```
error[E0308]: mismatched types
 --> src/main.rs:2:12
  |
1 | fn foo() {
  |           - expected `()` because of default return type
2 |     return "foo"
  |            ^^^^^ expected (), found reference
  |
  = note: expected type `()`
             found type `&'static str`
```

但是，目前`async fn`的支持，还不知道“信任”函数签名中编写的返回类型，从而导致不匹配甚至反标准错误。例如，函数`async fn foo() { "foo" }`导致此错误：

```
error[E0271]: type mismatch resolving `<impl std::future::Future as std::future::Future>::Output == ()`
 --> src/lib.rs:1:16
  |
1 | async fn foo() {
  |                ^ expected &str, found ()
  |
  = note: expected type `&str`
             found type `()`
  = note: the return type of a function must have a statically known size
```

这个错误说得是：它 _expected_ `&str`，但发现了`()`，实际上，这就与您想要的完全相反。这是因为编译器错误地信任，函数主体会返回正确的类型。

此问题的变通办法是识别，指向带有"expected `SomeType`, found `OtherType`"信息的函数签名的错误，通常表示一个或多个返回站点不正确。

Fix in [this bug](https://github.com/rust-lang/rust/issues/54326)，可以跟踪浏览下。

## `Box<dyn Trait>`

同样，由于函数签名的返回类型，没有正确传播，因此来自`async fn`的值，没有正确地强制使用其预期的类型。

实践中，这意味着，`async fn`返回的`Box<dyn Trait>`对象需要手动`as`，将`Box<MyType>`转为`Box<dyn Trait>`。

此代码将导致错误：

```
async fn x() -> Box<dyn std::fmt::Display> {
    Box::new("foo")
}
```

可以通过使用`as`，这个错误就消除了：

```
async fn x() -> Box<dyn std::fmt::Display> {
    Box::new("foo") as Box<dyn std::fmt::Display>
}
```

Fix in [this bug](https://github.com/rust-lang/rust/issues/60424)，可以跟踪浏览。
