r[expr.await]
# `await`表达式

r[expr.await.syntax]
```grammar,表达式
AwaitExpression -> Expression `.` `await`
```

r[expr.await.intro]
`await` 表达式是一种语法结构，用于挂起由 `std::future::IntoFuture` 的实现提供的计算，直到给定的 future 准备好产生一个值。

r[expr.await.construct]
`await` 表达式的 语法格式 是一个具有实现了 [`IntoFuture`] 特型的类型的表达式（称为 *future 操作数*），后跟 词法单元 `.`，然后是 `await` 关键字。

r[expr.await.allowed-positions]
`await` 表达式仅在 [异步上下文][async context]（如 [`async fn`]、[`async` 闭包][`async` closure] 或 [`async` 块][`async` block]）中合法。

r[expr.await.effects]
更具体地说，一个 `await` 表达式具有以下效果。

1. 通过在 future 操作数上调用 [`IntoFuture::into_future`] 来创建一个 future。
2. 将该 future 求值为一个 [future] `tmp`；
3. 使用 [`Pin::new_unchecked`] 固定 `tmp`；
4. 然后通过调用 [`Future::poll`] 方法并向其传递当前 [任务上下文](#task-context) 来轮询此固定的 future；
5. 如果对 `poll` 的调用返回 [`Poll::Pending`]，则 future 返回 `Poll::Pending`，并挂起其状态，以便当周围的异步上下文被重新轮询时，执行返回到步骤 3；
6. 否则对 `poll` 的调用必须返回 [`Poll::Ready`]，在这种情况下，[`Poll::Ready`] 变体中包含的值将用作 `await` 表达式本身的结果。

r[expr.await.edition2018]
> [!EDITION-2018]
> `await` 表达式仅从 Rust 2018 版次开始提供。

r[expr.await.task]
## 任务上下文

任务上下文是指当异步上下文本身被轮询时提供给当前 [异步上下文][async context] 的 [`Context`]。由于 `await` 表达式仅在异步上下文中合法，因此必须存在某种可用的任务上下文。

r[expr.await.desugar]
## 近似脱糖

实际上，`await` 表达式大致等效于以下非规范性的脱糖：

<!-- ignore: example expansion -->
```rust,ignore
match operand.into_future() {
    mut pinned => loop {
        let mut pin = unsafe { Pin::new_unchecked(&mut pinned) };
        match Pin::future::poll(Pin::borrow(&mut pin), &mut current_context) {
            Poll::Ready(r) => break r,
            Poll::Pending => yield Poll::Pending,
        }
    }
}
```

其中 `yield` 伪代码返回 `Poll::Pending`，并且在重新调用时从该点恢复执行。变量 `current_context` 是指从异步环境中获取的上下文。

[`async fn`]: ../items/functions.md#async-functions
[`async` closure]: closure-expr.md#async-closures
[`async` block]: block-expr.md#async-blocks
[`Context`]: std::task::Context
[`future::poll`]: std::future::Future::poll
[`pin::new_unchecked`]: std::pin::Pin::new_unchecked
[`poll::Pending`]: std::task::Poll::Pending
[`poll::Ready`]: std::task::Poll::Ready
[async context]: ../expressions/block-expr.md#async-context
[future]: std::future::Future
[`IntoFuture`]: std::future::IntoFuture
[`IntoFuture::into_future`]: std::future::IntoFuture::into_future
