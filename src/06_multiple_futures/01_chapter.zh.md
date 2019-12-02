# Executing Multiple Futures at a Time

到目前为止，我们主要通过`.await`来使用 Futures，它将阻塞当前任务，直到特定的`Future`完成。但是，真正的异步应用程序，通常需要同时执行几个不同的操作。

# Executing Multiple Futures at a Time

在本章中，我们将介绍几种，同时执行多个异步操作的方法：

- `join!`：等待全部 Futures 完成
- `select!`：等待几种 Futures 之一，完成
- Spawning：创建一个顶级任务，周围运行一个 Future 完成
- `FuturesUnordered`：一组 Future ，yield 回每个子 Future 的结果
