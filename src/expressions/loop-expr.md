r[expr.loop]
# 循环及其他可中断表达式

r[expr.loop.syntax]
```grammar,expressions
LoopExpression ->
    LoopLabel? (
        InfiniteLoopExpression
      | PredicateLoopExpression
      | IteratorLoopExpression
      | LabelBlockExpression
    )
```

r[expr.loop.intro]
Rust 支持四种循环表达式：

*   [`loop` 表达式](#infinite-loops) 表示一个无限循环。
*   [`while` 表达式](#predicate-loops) 循环直到谓词为假。
*   [`for` 表达式](#iterator-loops) 从迭代器中提取值，循环直到迭代器为空。
*   [标签块表达式][expr.loop.block-labels] 正好运行一次，但允许使用 `break` 提前退出。

r[expr.loop.break-label]
所有四种类型的循环都支持 [`break` 表达式](#break-expressions) 和 [标签](#loop-labels)。

r[expr.loop.continue-label]
除标签块表达式外，所有表达式都支持 [`continue` 表达式](#continue-expressions)。

r[expr.loop.explicit-result]
只有 `loop` 和标签块表达式支持 [求值为非平凡值](#break-and-loop-values)。

r[expr.loop.infinite]
## 无限循环

r[expr.loop.infinite.syntax]
```grammar,expressions
InfiniteLoopExpression -> `loop` BlockExpression
```

r[expr.loop.infinite.intro]
`loop` 表达式连续重复执行其主体：
`loop { println!("I live."); }`。

r[expr.loop.infinite.diverging]
没有关联 `break` 表达式的 `loop` 表达式是发散的，其类型为 [`!`](../types/never.md)。

r[expr.loop.infinite.break]
包含关联 [`break` 表达式](#break-expressions) 的 `loop` 表达式可能会终止，并且其类型必须与 `break` 表达式的值兼容。

r[expr.loop.while]
## 谓词循环

r[expr.loop.while.grammar]
```grammar,expressions
PredicateLoopExpression -> `while` Conditions BlockExpression
```

r[expr.loop.while.intro]
`while` 循环表达式允许在条件集保持为真的情况下重复求值一个块。

r[expr.loop.while.syntax]
`while` 表达式的 语法格式 是一个或多个由 `&&` 分隔的条件操作数序列，后跟一个 [块表达式][BlockExpression]。

r[expr.loop.while.condition]
条件操作数必须是具有 [布尔类型][boolean type] 的 [表达式][Expression] 或条件 `let` 匹配。
如果所有条件操作数求值结果均为 `true`，且所有 `let` 模式都成功匹配其 [待匹配值][scrutinee]，则执行循环体块。

r[expr.loop.while.repeat]
在循环体成功执行后，会重新求值条件操作数，以确定是否应再次执行主体。

r[expr.loop.while.exit]
如果任何条件操作数求值为 `false` 或任何 `let` 模式未匹配其待匹配值，则不执行主体，并在 `while` 表达式之后继续执行。

r[expr.loop.while.eval]
一个 `while` 表达式求值为 `()`。

示例：

```rust
let mut i = 0;

while i < 10 {
    println!("hello");
    i = i + 1;
}
```

r[expr.loop.while.let]
### `while let`模式

r[expr.loop.while.let.intro]
`while` 条件中的 `let` 模式允许在模式匹配成功时将新变量绑定到作用域中。
以下示例演示了使用 `let` 模式进行的绑定：

```rust
let mut x = vec![1, 2, 3];

while let Some(y) = x.pop() {
    println!("y = {}", y);
}

while let _ = 5 {
    // 不可反驳模式总是为真
    println!("Irrefutable patterns are always true");
    break;
}
```

r[expr.loop.while.let.desugar]
`while let` 循环等效于包含 [`match` 表达式][`match` expression] 的 `loop` 表达式，如下所示。

<!-- ignore: expansion example -->
```rust,ignore
'label: while let PATS = EXPR {
    /* 循环体 */
}
```

等效于

<!-- ignore: expansion example -->
```rust,ignore
'label: loop {
    match EXPR {
        PATS => { /* 循环体 */ },
        _ => break,
    }
}
```

r[expr.loop.while.let.or-pattern]
可以使用 `|` 运算符指定多个模式。这与 `match` 表达式中的 `|` 具有相同的语义：

```rust
let mut vals = vec![2, 3, 1, 2, 2];
while let Some(v @ 1) | Some(v @ 2) = vals.pop() {
    // 打印 2, 2, 然后是 1
    println!("{}", v);
}
```

r[expr.loop.while.chains]
### `while`条件链

r[expr.loop.while.chains.intro]
多个条件操作数可以用 `&&` 分隔。这些与 [`if` 条件链][`if` condition chains] 具有相同的语义和限制。

以下是一个链接多个表达式的示例，混合了 `let` 绑定和布尔表达式，并且表达式能够引用之前表达式中的模式绑定：

```rust
fn main() {
    let outer_opt = Some(Some(1i32));

    while let Some(inner_opt) = outer_opt
        && let Some(number) = inner_opt
        && number == 1
    {
        println!("Peek a boo");
        break;
    }
}
```

r[expr.loop.for]
## 迭代器循环

r[expr.loop.for.syntax]
```grammar,expressions
IteratorLoopExpression ->
    `for` Pattern `in` Expression _except [StructExpression]_ BlockExpression
```
<!-- TODO: The exception above isn't accurate, see https://github.com/rust-lang/reference/issues/569 -->

r[expr.loop.for.intro]
一个 `for` 表达式是一个用于循环遍历由 `std::iter::IntoIterator` 实现提供的元素的语法结构。

r[expr.loop.for.condition]
如果迭代器产生一个值，该值将与不可反驳模式匹配，执行循环体，然后控制流返回到 `for` 循环头部。
如果迭代器为空，则 `for` 表达式完成。

数组内容的 `for` 循环示例：

```rust
let v = &["apples", "cake", "coffee"];

for text in v {
    println!("I like {}.", text);
}
```

一系列整数上的 `for` 循环示例：

```rust
let mut sum = 0;
for n in 1..11 {
    sum += n;
}
assert_eq!(sum, 55);
```

r[expr.loop.for.desugar]
`for` 循环等效于包含 [`match` 表达式][`match` expression] 的 `loop` 表达式，如下所示：

<!-- ignore: expansion example -->
```rust,ignore
'label: for PATTERN in iter_expr {
    /* 循环体 */
}
```

等效于

<!-- ignore: expansion example -->
```rust,ignore
{
    let result = match IntoIterator::into_iter(iter_expr) {
        mut iter => 'label: loop {
            let mut next;
            match Iterator::next(&mut iter) {
                Option::Some(val) => next = val,
                Option::None => break,
            };
            let PATTERN = next;
            let () = { /* 循环体 */ };
        },
    };
    result
}
```

r[expr.loop.for.lang-items]
这里的 `IntoIterator`、`Iterator` 和 `Option` 始终是标准库 项 ，而不是当前作用域中解析为这些名称的任何内容。

变量名 `next`、`iter` 和 `val` 仅用于说明，它们实际上没有用户可以键入的名称。

> [!NOTE]
> 外部 `match` 用于确保 `iter_expr` 中的任何 [临时值][temporary values] 在循环结束前不会被丢弃。`next` 在被赋值前声明，是因为这通常能让类型推导更准确。

r[expr.loop.label]
## 循环标签

r[expr.loop.label.syntax]
```grammar,expressions
LoopLabel -> LIFETIME_OR_LABEL `:`
```

r[expr.loop.label.intro]
循环表达式可以可选地带有一个 *标签*。标签写在循环表达式之前的生命周期，例如 `'foo: loop { break 'foo; }`、`'bar: while false {}`、`'humbug: for _ in 0..0 {}`。

r[expr.loop.label.control-flow]
如果存在标签，则嵌套在该循环内的带标签 `break` 和 `continue` 表达式可以退出该循环或返回到其头部。
参见 [break 表达式](#break-expressions) 和 [continue 表达式](#continue-expressions)。

r[expr.loop.label.ref]
标签遵循局部变量的 卫生性 和 遮蔽 规则。例如，这段代码将打印 "outer loop"：

```rust
'a: loop {
    'a: loop {
        break 'a;
    }
    print!("outer loop");
    break 'a;
}
```

`'_` 不是有效的循环标签。

r[expr.loop.break]
## `break`表达式

r[expr.loop.break.syntax]
```grammar,expressions
BreakExpression -> `break` LIFETIME_OR_LABEL? Expression?
```

r[expr.loop.break.intro]
当遇到 `break` 时，关联的循环体执行将立即终止，例如：

```rust
let mut last = 0;
for x in 1..100 {
    if x > 12 {
        break;
    }
    last = x;
}
assert_eq!(last, 12);
```

r[expr.loop.break.label]
`break` 表达式通常与包围该 `break` 表达式的最内层 `loop`、`for` 或 `while` 循环相关联，
但可以使用 [标签](#loop-labels) 来指定受影响的是哪个外层循环。
示例：

```rust
'outer: loop {
    while true {
        break 'outer;
    }
}
```

r[expr.loop.break.value]
`break` 表达式仅允许在循环体中使用，并具有 `break`、`break 'label` 或（[见下文](#break-and-loop-values)）`break EXPR` 或 `break 'label EXPR` 形式之一。

r[expr.loop.block-labels]
## 标签块表达式

r[expr.loop.block-labels.syntax]
```grammar,expressions
LabelBlockExpression -> BlockExpression
```

r[expr.loop.block-labels.intro]
标签块表达式与块表达式完全一样，除了它们允许在块内使用 `break` 表达式。

r[expr.loop.block-labels.break]
与循环不同，标签块表达式中的 `break` 表达式 *必须* 带有标签（即标签不是可选的）。

r[expr.loop.block-labels.label-required]
类似地，标签块表达式 *必须* 以标签开头。

```rust
# fn do_thing() {}
# fn condition_not_met() -> bool { true }
# fn do_next_thing() {}
# fn do_last_thing() {}
let result = 'block: {
    do_thing();
    if condition_not_met() {
        break 'block 1;
    }
    do_next_thing();
    if condition_not_met() {
        break 'block 2;
    }
    do_last_thing();
    3
};
```

r[expr.loop.continue]
## `continue`表达式

r[expr.loop.continue.syntax]
```grammar,expressions
ContinueExpression -> `continue` LIFETIME_OR_LABEL?
```

r[expr.loop.continue.intro]
当遇到 `continue` 时，关联循环体的当前迭代将立即终止，将控制权返回给循环 *头部*。

r[expr.loop.continue.while]
在 `while` 循环的情况下，头部是控制循环的条件操作数。

r[expr.loop.continue.for]
在 `for` 循环的情况下，头部是控制循环的调用表达式。

r[expr.loop.continue.label]
与 `break` 一样，`continue` 通常与最内层包围的循环相关联，但 `continue 'label` 可用于指定受影响的循环。

r[expr.loop.continue.in-loop-only]
`continue` 表达式仅允许在循环体中使用。

r[expr.loop.break-value]
## `break`和循环值

r[expr.loop.break-value.intro]
当与 `loop` 关联时，break 表达式可用于通过 `break EXPR` 或 `break 'label EXPR` 形式从该循环返回一个值，其中 `EXPR` 是一个结果将从 `loop` 返回的表达式。
例如：

```rust
let (mut a, mut b) = (1, 1);
let result = loop {
    if b > 10 {
        break b;
    }
    let c = a + b;
    a = b;
    b = c;
};
// 斐波那契数列中第一个大于 10 的数字：
assert_eq!(result, 13);
```

r[expr.loop.break-value.loop]
在 `loop` 有关联 `break` 的情况下，它不被认为是发散的，并且 `loop` 必须具有与每个 `break` 表达式兼容的类型。
没有表达式的 `break` 被认为等同于带有表达式 `()` 的 `break`。

[`if` condition chains]: if-expr.md#chains-of-conditions
[`if` expressions]: if-expr.md
[`match` expression]: match-expr.md
[boolean type]: ../types/boolean.md
[scrutinee]: ../glossary.md#scrutinee
[temporary values]: ../expressions.md#temporaries
