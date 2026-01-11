r[expr.if]
# `if`表达式

r[expr.if.syntax]
```grammar,表达式
IfExpression ->
    `if` Conditions BlockExpression
    (`else` ( BlockExpression | IfExpression ) )?

Conditions ->
      Expression _except [StructExpression]_
    | LetChain

LetChain -> LetChainCondition ( `&&` LetChainCondition )*

LetChainCondition ->
      Expression _except [ExcludedConditions]_
    | OuterAttribute* `let` Pattern `=` Scrutinee _except [ExcludedConditions]_

@root ExcludedConditions ->
      StructExpression
    | LazyBooleanExpression
    | RangeExpr
    | RangeFromExpr
    | RangeInclusiveExpr
    | AssignmentExpression
    | CompoundAssignmentExpression
```
<!-- TODO: The struct exception above needs clarification, see https://github.com/rust-lang/reference/issues/1808
     The chain grammar could use some work, see https://github.com/rust-lang/reference/issues/1811
-->

r[expr.if.intro]
`if` 表达式的 语法格式 是一个或多个由 `&&` 分隔的条件操作数序列，后跟一个结果块、任意数量的 `else if` 条件和块，以及一个可选的尾随 `else` 块。

r[expr.if.condition]
条件操作数必须是具有 [布尔类型][boolean type] 的 [表达式][Expression] 或条件 `let` 匹配。

r[expr.if.condition-true]
如果所有条件操作数求值结果均为 `true`，且所有 `let` 模式都成功匹配其 [待匹配值][scrutinee]，则执行结果块，并跳过任何后续的 `else if` 或 `else` 块。

r[expr.if.else-if]
如果任何条件操作数求值为 `false` 或任何 `let` 模式未匹配其待匹配值，则跳过结果块，并对任何后续的 `else if` 条件求值。

r[expr.if.else]
如果所有 `if` 和 `else if` 条件求值结果均为 `false` ，则执行任何 `else` 块。

r[expr.if.result]
一个 `if` 表达式求值为所执行块的相同值，如果没有块被执行，则求值为 `()`。

r[expr.if.type]
在一个 `if` 表达式中，所有情况下的类型必须相同。

```rust
# let x = 3;
if x == 4 {
    println!("x is four");
} else if x == 3 {
    println!("x is three");
} else {
    println!("x is something else");
}

// `if` 可以用作表达式。
let y = if 12 * 15 > 150 {
    "Bigger"
} else {
    "Smaller"
};
assert_eq!(y, "Bigger");
```

r[expr.if.let]
## `if let`模式

r[expr.if.let.intro]
`if` 条件中的 `let` 模式允许在模式匹配成功时将新变量绑定到作用域中。

以下示例演示了使用 `let` 模式进行的绑定：

```rust
let dish = ("Ham", "Eggs");

// 此主体将被跳过，因为模式被反驳了。
if let ("Bacon", b) = dish {
    println!("Bacon is served with {}", b);
} else {
    // 改为对该块求值。
    println!("No bacon will be served");
}

// 此主体将执行。
if let ("Ham", b) = dish {
    println!("Ham is served with {}", b);
}

if let _ = 5 {
    println!("Irrefutable patterns are always true");
}
```

r[expr.if.let.or-pattern]
可以使用 `|` 运算符指定多个模式。这与 [`match` 表达式][`match` expressions] 中的 `|` 具有相同的语义：

```rust
enum E {
    X(u8),
    Y(u8),
    Z(u8),
}
let v = E::Y(12);
if let E::X(n) | E::Y(n) = v {
    assert_eq!(n, 12);
}
```

r[expr.if.chains]
## 条件链

r[expr.if.chains.intro]
多个条件操作数可以用 `&&` 分隔。

r[expr.if.chains.order]
类似于 `&&` [惰性布尔表达式][LazyBooleanExpression]，每个操作数从左到右求值，直到某个操作数求值为 `false` 或 `let` 匹配失败，此时后续操作数将不再求值。

r[expr.if.chains.bindings]
每个模式的绑定都会被放入作用域中，以便后续的条件操作数和结果块使用。

以下是一个链接多个表达式的示例，混合了 `let` 绑定和布尔表达式，并且表达式能够引用之前表达式中的模式绑定：

```rust
fn single() {
    let outer_opt = Some(Some(1i32));

    if let Some(inner_opt) = outer_opt
        && let Some(number) = inner_opt
        && number == 1
    {
        println!("Peek a boo");
    }
}
```

在不使用条件链的情况下，上述代码等效于：

```rust
fn nested() {
    let outer_opt = Some(Some(1i32));

    if let Some(inner_opt) = outer_opt {
        if let Some(number) = inner_opt {
            if number == 1 {
                println!("Peek a boo");
            }
        }
    }
}
```

r[expr.if.chains.or]
如果任何条件操作数是 `let` 模式，由于与 `let` 待匹配值存在歧义和优先级关系，任何条件操作数都不能是 `||` [惰性布尔运算符表达式][expr.bool-logic]。如果需要 `||` 表达式，可以使用括号。例如：

```rust
# let foo = Some(123);
# let condition1 = true;
# let condition2 = false;
// 此处需要括号。
if let Some(x) = foo && (condition1 || condition2) { /*...*/ }
```

r[expr.if.edition2024]
> [!EDITION-2024]
> 在 2024 版次之前，不支持 let 链。也就是说，`if` 表达式中不允许使用 [LetChain] 语法。

[`match` expressions]: match-expr.md
[boolean type]: ../types/boolean.md
[scrutinee]: ../glossary.md#scrutinee
