# Applied: Simple HTTP Server

让我们用`async`/`.await`建立一个回声服务器！

开始之前，运行`rustup update stable`，以确保你有 stable Rust 1.39 或更新的版本。一旦完成，就`cargo new async-await-echo`创建新项目，并打开输出的`async-await-echo`文件夹。

让我们将一些依赖项，添加到`Cargo.toml`文件：

```toml
{{#include ../../examples/01_05_http_server/Cargo.toml:9:18}}
```

既然我们已经摆脱了依赖关系，让我们开始编写一些代码。我们有一些 imports 要添加：

```rust
{{#include ../../examples/01_05_http_server/src/lib.rs:imports}}
```

一旦搞定这些 imports，我们就可以开始整理样板文件，以便满足以下要求：

```rust
{{#include ../../examples/01_05_http_server/src/lib.rs:boilerplate}}
```

如果你现在`cargo run`，你应该看到信息“Listening on http://127.0.0.1:3000“打印在你的终端上。如果你在你选择的浏览器中，打开这个网址，你会看到“hello, world!”出现在浏览器中。祝贺你！您刚刚在 Rust 中编写了，第一个异步 web 服务器。

您还可以检查 request(请求) 本身，它包含诸如，request 的 URI、HTTP 版本、header 和其他元数据等信息。例如，我们可以打印出请求的 URI，如下所示：

```rust
println!("Got request at {:?}", req.uri());
```

您可能已经注意到，在处理请求时，我们还没有做任何异步操作，我们只是立即响应，所以我们没有利用上`async fn`给我们的灵活性。与其只返回静态消息，不如尝试使用 Hyper 的 HTTP 客户端，将用户的请求代理到另一个网站。

我们首先解析出要请求的 URL：

```rust
{{#include ../../examples/01_05_http_server/src/lib.rs:parse_url}}
```

然后，我们可以新建一个新的`hyper::Client`，并使用它，制造一个`GET`请求，将响应返回给用户：

```rust
{{#include ../../examples/01_05_http_server/src/lib.rs:get_request}}
```

`Client::get`会返回一个`hyper::client::FutureResponse`，它实现了`Future<Output = Result<Response, Error>>`（或`Future<Item = Response, Error = Error>`在 futures 0.1 版）。当我们`.await`以后，会发送一个 HTTP 请求，挂起当前任务，并在响应可用时，任务会排队等待继续。

现在，如果你现在`cargo run`，在浏览器中打开`http://127.0.0.1:3000/foo`，您将看到 Rust 主页和以下终端输出：

```
Listening on http://127.0.0.1:3000
Got request at /foo
making request to http://www.rust-lang.org/en-US/
request finished-- returning response
```

祝贺你！你只是代理了一个 HTTP 请求。
