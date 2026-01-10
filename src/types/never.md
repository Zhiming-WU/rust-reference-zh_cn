r[type.never]
# Never类型

r[type.never.syntax]
```grammar,types
NeverType -> `!`
```

r[type.never.intro]
never type `!` 是一种没有值的类型，表示永远不会完成的计算结果。

r[type.never.coercion]
`!` 类型的表达式可以 隐式类型转换 为任何其他类型。

r[type.never.constraint]
`!` 类型目前 **只能** 出现在函数返回类型中，表示这是一个永远不会返回的发散函数。

```rust
fn foo() -> ! {
    panic!("This call never returns.");
}
```

```rust
unsafe extern "C" {
    pub safe fn no_return_extern_func() -> !;
}
```
