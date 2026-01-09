r[expr.paren]
# 分组表达式

r[expr.paren.syntax]
```grammar,expressions
GroupedExpression -> `(` Expression `)`
```

r[expr.paren.intro]
一个 *括号表达式* 包裹单个表达式，求值为该表达式。
括号表达式的 语法格式 是一个 `(`，然后是一个表达式，称为 *被包裹的操作数*，接着是一个 `)`。

r[expr.paren.evaluation]
括号表达式求值为被包裹的操作数的值。

r[expr.paren.place-or-value]
与其他表达式不同，括号表达式既是 [位置表达式和值表达式][place]。
当被包裹的操作数是一个位置表达式时，它就是一个位置表达式；当被包裹的操作数是一个值表达式时，它就是一个值表达式。

r[expr.paren.override-precedence]
括号可用于显式修改表达式中子表达式的优先级顺序。

一个括号表达式的例子：

```rust
let x: i32 = 2 + 3 * 4; // 未加括号
let y: i32 = (2 + 3) * 4; // 已加括号
assert_eq!(x, 14);
assert_eq!(y, 20);
```

一个必须使用括号的例子是调用 结构体 成员中的函数指针：

```rust
# struct A {
#    f: fn() -> &'static str
# }
# impl A {
#    fn f(&self) -> &'static str {
#        "The method f"
#    }
# }
# let a = A{f: || "The field f"};
#
assert_eq!( a.f (), "The method f");
assert_eq!((a.f)(), "The field f");
```

[place]: ../expressions.md#place-expressions-and-value-expressions
