<!-- https://github.com/rust-lang/reference/blob/master/src/expressions/return-expr.md -->
<!-- commit 68bdfd15fa9feebfbf94a06cf574de598e239198 -->

r[expr.return]
# `return`表达式

r[expr.return.syntax]
```grammar,表达式
ReturnExpression -> `return` Expression?
```

r[expr.return.intro]
`return` 表达式由关键字 `return` 表示。

r[expr.return.behavior]
求值 `return` 表达式会将其参数移动到当前函数调用的指定输出位置，销毁当前函数的活动帧，并将控制权转移给调用者帧。

一个 `return` 表达式的示例：

```rust
fn max(a: i32, b: i32) -> i32 {
    if a > b {
        return a;
    }
    return b;
}
```
