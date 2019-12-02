# Under the Hood: Executing `Future`s and Tasks

在本节中，我们会介绍`Future`和异步是如何调度的。如果您只想学习如何编写使用`Future`类型的高阶代码，并对`Future`类型的工作细节不感兴趣，您可以直接跳到`async`/`await`章节。但是，本章讨论的几个主题如：了解`async`/`await`的工作; `async`/`await`代码的 runtime 和性能属性，构建新的异步原语等，这些对运用异步代码都会有所帮助。如果您决定现在跳过此部分，则可能需要将其添加为书签，以便将来再次访问。

现在，让我们来谈谈`Future` trait。
