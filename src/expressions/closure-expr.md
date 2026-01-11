<!-- https://github.com/rust-lang/reference/blob/master/src/expressions/closure-expr.md -->
<!-- commit 68bdfd15fa9feebfbf94a06cf574de598e239198 -->

r[expr.closure]
# 闭包表达式

r[expr.closure.syntax]
```grammar,表达式
ClosureExpression ->
    `async`?[^cl-async-edition]
    `move`?
    ( `||` | `|` ClosureParameters? `|` )
    (Expression | `->` TypeNoBounds BlockExpression)

ClosureParameters -> ClosureParam (`,` ClosureParam)* `,`?

ClosureParam -> OuterAttribute* PatternNoTopAlt ( `:` Type )?
```

[^cl-async-edition]: `async` 限定符在 2015 版次中不允许使用。

r[expr.closure.intro]
一个 *闭包表达式* ，也称为 lambda 表达式或 lambda，定义了一个 [闭包类型][closure type] 并求值为该类型的一个值。
闭包表达式的 语法格式 是一个可选的 `async` 关键字，一个可选的 `move` 关键字，然后是由管道符号（`|`）分隔的逗号分隔的 [模式][patterns] 列表（称为 *闭包参数* ），每个参数后面可以可选地跟着 `:` 和一个类型，然后是一个可选的 `->` 和一个类型（称为 *返回类型* ），最后是一个表达式（称为 *闭包体操作数* ）。

r[expr.closure.param-type]
每个模式后的可选类型是该模式的类型标注。

r[expr.closure.explicit-type-body]
如果有返回类型，闭包体必须是一个 [块][block]。

r[expr.closure.parameter-restriction]
闭包表达式表示一个函数，它将参数列表映射到参数之后的表达式上。
就像 [`let` 绑定][`let` binding] 一样，闭包参数是不可反驳的 [模式][patterns]，其类型标注是可选的，如果未给出，将从上下文中推导。

r[expr.closure.unique-type]
每个闭包表达式都有一个唯一的、匿名的类型。

r[expr.closure.captures]
重要的是，闭包表达式会 _捕获其环境_ ，而普通的 [函数定义][function definitions] 则不会。

r[expr.closure.capture-inference]
如果没有 `move` 关键字，闭包表达式会 [推导它如何从其环境中捕获每个变量](../types/closure.md#捕获模式)，优先通过共享引用进行捕获，从而实际上借用了闭包体内提到的所有外部变量。

r[expr.closure.capture-mut-ref]
如果需要，编译器将推导应该采用可变引用，或者应该从环境中移动或复制值（取决于它们的类型）。

r[expr.closure.capture-move]
通过在闭包前加上 `move` 关键字，可以强制闭包通过复制或移动值来捕获其环境。
这通常用于确保闭包的生命周期是 `'static`。

r[expr.closure.trait-impl]
## 闭包特型实现

闭包类型实现哪些特型取决于变量如何被捕获、被捕获变量的类型以及 `async` 的存在。
有关闭包如何以及何时实现 `Fn`、`FnMut` 和 `FnOnce`，请参见 [调用特型与强转][call traits and coercions] 章节。
如果每个被捕获变量的类型也都实现了该特型，则闭包类型就会实现 [`Send`] 和 [`Sync`][`Sync`]。

r[expr.closure.async]
## Async闭包

r[expr.closure.async.intro]
带有 `async` 关键字标记的闭包表示它们是异步的，其方式类似于 [异步函数][items.fn.async]。

r[expr.closure.async.future]
调用 Async 闭包不会执行任何工作，而是求值为一个实现了 [`Future`] 的值，该值对应于闭包体的计算。

```rust
async fn takes_async_callback(f: impl AsyncFn(u64)) {
    f(0).await;
    f(1).await;
}

async fn example() {
    takes_async_callback(async |i| {
        core::future::ready(i).await;
        println!("done with {i}.");
    }).await;
}
```

r[expr.closure.async.edition2018]
> [!EDITION-2018]
> Async 闭包仅从 Rust 2018 版次开始可用。

## 示例

在此示例中，我们定义了一个函数 `ten_times`，它接受一个高阶函数参数，然后我们使用闭包表达式作为参数来调用它，随后是一个从其环境移动值的闭包表达式。

```rust
fn ten_times<F>(f: F) where F: Fn(i32) {
    for index in 0..10 {
        f(index);
    }
}

ten_times(|j| println!("hello, {}", j));
// 带有类型标注
ten_times(|j: i32| -> () { println!("hello, {}", j) });

let word = "konnichiwa".to_owned();
ten_times(move |j| println!("{}, {}", word, j));
```

## 闭包参数上的属性

r[expr.closure.param-attributes]
闭包参数上的属性遵循与 [普通函数参数][regular function parameters] 相同的规则和限制。

[`let` binding]: ../statements.md#let语句
[`Send`]: ../special-types-and-traits.md#send
[`Sync`]: ../special-types-and-traits.md#sync
[block]: block-expr.md
[call traits and coercions]: ../types/closure.md#调用特型与隐式类型转换
[closure type]: ../types/closure.md
[function definitions]: ../items/functions.md
[patterns]: ../patterns.md
[regular function parameters]: ../items/functions.md#函数参数上的属性
