# rust-lang/async-book [![explain]][source] [![translate-svg]][translate-list]

<!-- [![size-img]][size] -->

[explain]: http://llever.com/explain.svg
[source]: https://github.com/chinanf-boy/Source-Explain
[translate-svg]: http://llever.com/translate.svg
[translate-list]: https://github.com/chinanf-boy/chinese-translate-list
[size-img]: https://packagephobia.now.sh/badge?p=Name
[size]: https://packagephobia.now.sh/result?p=Name

「 Rust 中的异步编程 」

[中文](./readme.md) | [english](https://github.com/rust-lang/async-book)

---

## 校对 ✅

<!-- doc-templite START generated -->
<!-- repo = 'rust-lang/async-book' -->
<!-- commit = 'fa3228386e9efe4e98d222394fca6b7311e7c405' -->
<!-- time = '2019-11-08' -->

| 翻译的原文 | 与日期        | 最新更新 | 更多                       |
| ---------- | ------------- | -------- | -------------------------- |
| [commit]   | ⏰ 2019-11-08 | ![last]  | [中文翻译][translate-list] |

[last]: https://img.shields.io/github/last-commit/rust-lang/async-book.svg
[commit]: https://github.com/rust-lang/async-book/tree/fa3228386e9efe4e98d222394fca6b7311e7c405

<!-- doc-templite END generated -->

- [x] [src/SUMMARY.md](src/SUMMARY.md)
- [x] [入门](src/01_getting_started/01_chapter.zh.md)
  - [x] [为什么要 async ？](src/01_getting_started/02_why_async.zh.md)
  - [x] [async Rust 状态](src/01_getting_started/03_state_of_async_rust.zh.md)
  - [x] [`async`/`.await` Primer](src/01_getting_started/04_async_await_primer.zh.md)
  - [x] [应用：HTTP 服务器](src/01_getting_started/05_http_server_example.zh.md)
- [x] [幕后：执行`Future`和任务](src/02_execution/01_chapter.zh.md)
  - [x] [`Future` Trait](src/02_execution/02_future.zh.md)
  - [x] [任务唤醒`Waker`](src/02_execution/03_wakeups.zh.md)
  - [x] [应用：生成一个执行器](src/02_execution/04_executor.zh.md)
  - [x] [执行器和系统 IO](src/02_execution/05_io.zh.md)
- [x] [`async`/`await`](src/03_async_await/01_chapter.zh.md)
- [x] [Pinning](src/04_pinning/01_chapter.zh.md)
- [x] [Streams](src/05_streams/01_chapter.zh.md)
  - [x] [迭代与并发](src/05_streams/02_iteration_and_concurrency.zh.md)
- [x] [一次执行多个 Futures](src/06_multiple_futures/01_chapter.zh.md)
  - [x] [`join!`](src/06_multiple_futures/02_join.zh.md)
  - [x] [`select!`](src/06_multiple_futures/03_select.zh.md)
  - [ ] [TODO: Spawning](src/404.zh.md)
  - [ ] [TODO：取消和超时](src/404.zh.md)
  - [ ] [TODO：`FuturesUnordered`](src/404.zh.md)
- [x] [走走看看，想想](src/07_workarounds/01_chapter.zh.md)
  - [x] [返回类型的错误](src/07_workarounds/02_return_type.zh.md)
  - [x] [`?`在`async`代码块](src/07_workarounds/03_err_in_async_blocks.zh.md)
  - [x] [`Send`估计](src/07_workarounds/04_send_approximation.zh.md)
  - [x] [递归](src/07_workarounds/05_recursion.zh.md)
  - [x] [`async`在 Traits 上](src/07_workarounds/06_async_in_traits.zh.md)
- [ ] [TODO：I/O](src/404.zh.md)
  - [ ] [TODO：`AsyncRead`以及`AsyncWrite`](src/404.zh.md)
- [ ] [TODO:async 设计模式：解决方案和建议](src/404.zh.md)
  - [ ] [TODO:建模服务器和请求/响应模式](src/404.zh.md)
  - [ ] [TODO:管理共享状态](src/404.zh.md)
- [ ] [TODO: 生态系统：Tokio 等](src/404.zh.md)
  - [ ] [TODO: 多多，多得多的东西？...](src/404.zh.md)

### 贡献

欢迎 👏 勘误/校对/更新贡献 😊 [具体贡献请看](https://github.com/chinanf-boy/chinese-translate-list#贡献)

## 生活

[If help, **buy** me coffee —— 营养跟不上了，给我来瓶营养快线吧! 💰](https://github.com/chinanf-boy/live-need-money)

---

# async-book

Rust 中的异步编程

## 要求

async-book 的构建需要[`mdbook`]，您可以使用 Cargo 进行安装。

```
cargo install mdbook
```

[`mdbook`]: https://github.com/rust-lang/mdBook

## 建造

要创建完成的书，请运行`mdbook build`在下生成它`book/`目录。

```
mdbook build
```

## 发展历程

在编写过程中，查看更改非常方便，`mdbook serve`将启动本地网络服务器来提供图书。

```
mdbook serve
```
