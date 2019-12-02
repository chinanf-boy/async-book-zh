# `async` in Traits

目前，`async fn`不能用于 trait。造成这种情况的原因有些复杂，但是将来有计划取消此限制。

但是，与此同时，可以使用[`async_trait` 箱子，来自 crates.io](https://github.com/dtolnay/async-trait) 合作下。

请注意，使用这些 trait 方法将导致 per-function-call(每个函数调用)，都搞个分配堆。对于绝大多数应用程序而言，这并不是很大的成本，但是考虑一下，低层函数的公共 API，预计每秒调用数百万次，是否使用此功能。
