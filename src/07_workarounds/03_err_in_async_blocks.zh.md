# `?` in `async` Blocks

就像在`async fn`，`?`在`async`代码块内的使用很常见。但是，`async`代码块的返回类型是没有明确说明的。这可能会导致编译器无法推断`async`代码块的 error 类型。

例如，此代码：

```rust
let fut = async {
    foo().await?;
    bar().await?;
    Ok(())
};
```

将触发此错误：

```
error[E0282]: type annotations needed
 --> src/main.rs:5:9
  |
4 |     let fut = async {
  |         --- consider giving `fut` a type
5 |         foo().await?;
  |         ^^^^^^^^^^^^ cannot infer type
```

不幸的是，目前没有办法“giving `fut` a type”(给`fut`一个类型)，解决的办法也不是明确指定`async`代码块的返回类型。

要解决此问题，请使用“turbofish”操作符，为`async`代码块提供成功和错误类型。：

```rust
let fut = async {
    foo().await?;
    bar().await?;
    Ok::<(), MyError>(()) // <- 注意这里的明确类型声明
};
```
