# Pinning

要轮询 Future ，必须使用一种称为`Pin<T>`的特殊类型，来固定 Future。如果您阅读了在上一节["Executing `Future`s and Tasks"]中，[the `Future` trait]的解释，您会发现`Pin`来自`self: Pin<&mut Self>`，它是`Future:poll`方法的定义。但这究竟是什么意思，为什么我们需要它？

## Why Pinning

固定（Pinning）能得到一个保证，就是确保永远不会移动对象。要了解这样是必要的，我们需要记起`async`/`.await`的工作原理。考虑以下代码：

```rust
let fut_one = ...;
let fut_two = ...;
async move {
    fut_one.await;
    fut_two.await;
}
```

在幕后，新建一个实现`Future`的匿名类型，它提供一个`poll`(轮询)方法，看起来像这样的：

```rust
// 这个 `Future` 类型，由我们的 `async { ... }` 代码块生成而来
struct AsyncFuture {
    fut_one: FutOne,
    fut_two: FutTwo,
    state: State,
}

// 是我们 `async` 代码块可处于的，状态列表
enum State {
    AwaitingFutOne,
    AwaitingFutTwo,
    Done,
}

impl Future for AsyncFuture {
    type Output = ();

    fn poll(mut self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<()> {
        loop {
            match self.state {
                State::AwaitingFutOne => match self.fut_one.poll(..) {
                    Poll::Ready(()) => self.state = State::AwaitingFutTwo,
                    Poll::Pending => return Poll::Pending,
                }
                State::AwaitingFutTwo => match self.fut_two.poll(..) {
                    Poll::Ready(()) => self.state = State::Done,
                    Poll::Pending => return Poll::Pending,
                }
                State::Done => return Poll::Ready(()),
            }
        }
    }
}
```

当`poll`先被调用，它将轮询`fut_one`。如果`fut_one`还未完成，`AsyncFuture::poll`将返回。 Future 对`poll`进行调用，将在上一个停止的地方继续。这个过程一直持续到 Future 能成功完成。

但是，如果我们有一个，使用引用的`async`代码块？例如：

```rust
async {
    let mut x = [0; 128];
    let read_into_buf_fut = read_into_buf(&mut x); // &mut x
    read_into_buf_fut.await;
    println!("{:?}", x);
}
```

这会编译成什么结构？

```rust
struct ReadIntoBuf<'a> {
    buf: &'a mut [u8], // 指向下面的 `x`
}

struct AsyncFuture {
    x: [u8; 128],
    read_into_buf_fut: ReadIntoBuf<'what_lifetime?>,
}
```

在这里，`ReadIntoBuf` Future 拿着一个引用，指向我们结构的其他字段，即`x`。但是，如果`AsyncFuture`被移动(move)，`x`的位置也会移动，使存储在`read_into_buf_fut.buf`中的指针无效。

将 Future 固定到内存中的特定位置，可以避免此问题，从而可以安全地创建，对`async`代码块内部值的引用。

## How to Use Pinning

`Pin`类型会包裹着指针类型，保证指针后面的值不会移动。例如，`Pin<&mut T>`，`Pin<&T>`，`Pin<Box<T>>`，所有的这些，都保证`T`不会移动。

大多数类型在移动时，都没有问题。这些类型实现了一种称为`Unpin`的 trait。`Unpin`类型指针可以与`Pin`自由放入或取出。例如，`u8`是`Unpin`，所以`Pin<&mut u8>`表现就像正常`&mut u8`。

某些函数需要，要求与之配合使用的 Future 是`Unpin`。要使用不是`Unpin`的`Future`要么`Stream`，配合那些那需要`Unpin`类型的函数，那您首先必须 pin the value，方法有两种：`Box::pin`（创建一个`Pin<Box<T>>`） 或者 `pin_utils::pin_mut!`宏（创建一个`Pin<&mut T>`）。`Pin<Box<Fut>>`和`Pin<&mut Fut>`既可以用作 Future ，也实现了`Unpin`。

例如：

```rust
use pin_utils::pin_mut; // `pin_utils` is a handy crate available on crates.io

// A function which takes a `Future` that implements `Unpin`.
fn execute_unpin_future(x: impl Future<Output = ()> + Unpin) { ... }

let fut = async { ... };
execute_unpin_future(fut); // Error: `fut` does not implement `Unpin` trait

// Pinning with `Box`:
let fut = async { ... };
let fut = Box::pin(fut);
execute_unpin_future(fut); // OK

// Pinning with `pin_mut!`:
let fut = async { ... };
pin_mut!(fut);
execute_unpin_future(fut); // OK
```

["executing `future`s and tasks"]: ../02_execution/01_chapter.zh.md
[the `future` trait]: ../02_execution/02_future.zh.md
