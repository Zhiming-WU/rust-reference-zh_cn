r[expr.placeholder]
# `_`表达式

r[expr.placeholder.syntax]
```grammar,表达式
UnderscoreExpression -> `_`
```

r[expr.placeholder.intro]
下划线表达式由符号 `_` 表示，用于在解构赋值中表示占位符。

r[expr.placeholder.lhs-assignment-only]
它们只能出现在赋值语句的左侧。

r[expr.placeholder.pattern]
请注意，这与 [通配符模式](../patterns.md#wildcard-pattern) 不同。

`_` 表达式的示例：

```rust
let p = (1, 2);
let mut a = 0;
(_, a) = p;

struct Position {
    x: u32,
    y: u32,
}

Position { x: a, y: _ } = Position{ x: 2, y: 3 };

// 未使用的结果，赋值给 `_` 用于声明意图并消除警告
_ = 2 + 2;
// 触发 unused_must_use 警告
// 2 + 2;

// 在 let 绑定中使用通配符模式的等效技术
let _ = 2 + 2;
```
