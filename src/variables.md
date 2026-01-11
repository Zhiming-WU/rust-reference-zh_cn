<!-- https://github.com/rust-lang/reference/blob/master/src/variables.md -->
<!-- commit 68bdfd15fa9feebfbf94a06cf574de598e239198 -->

r[variable]
# 变量

r[variable.intro]
一个 变量 是 栈帧 的一个组成部分，它可以是一个具名函数 参数，一个匿名 [临时量](expressions.md#临时变量)，或者一个具名 局部变量。

r[variable.local]
一个 局部变量 （或 栈-局部 分配）直接持有值，在 栈内存 中分配。该值是 栈帧 的一部分。

r[variable.local-mut]
局部变量 默认不可变，除非另行声明。例如：`let mut x = ...`。

r[variable.param-mut]
函数 参数 默认不可变，除非用 `mut` 声明。`mut` 关键字仅适用于其后的 参数。例如：`|mut x, y|` 和 `fn f(mut x: Box<i32>, y: Box<i32>)` 声明了一个 可变 变量 `x` 和一个 不可变 变量 `y`。

r[variable.init]
局部变量 在分配时不会被初始化。相反，整个 栈帧 上的 局部变量 会在 栈帧 进入时以未初始化状态分配。函数内的后续语句可能会或可能不会初始化 局部变量。局部变量 只能在它们通过所有可达的 控制流路径 被初始化后才能使用。

在下一个示例中，`init_after_if` 在 [`if` 表达式][`if` expression] 之后被初始化，而 `uninit_after_if` 没有，因为它在 `else` 分支中未被初始化。

```rust
# fn random_bool() -> bool { true }
fn initialization_example() {
    let init_after_if: ();
    let uninit_after_if: ();

    if random_bool() {
        init_after_if = ();
        uninit_after_if = ();
    } else {
        init_after_if = ();
    }

    init_after_if; // ok
    // uninit_after_if; // 错误：使用了可能未初始化的 `uninit_after_if`
}
```

[`if` expression]: expressions/if-expr.md#if表达式
