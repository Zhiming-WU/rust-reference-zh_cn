# 限制

以下 [属性][attributes] 影响编译时限制。

## `recursion_limit`属性

*`recursion_limit` 属性* 可以应用于 [crate] 级别，以设置可能无限递归的编译时操作的最大深度，例如宏展开或自动解引用。

它使用 [MetaNameValueStr] 语法格式 来指定递归深度。

> [!NOTE]
> `rustc` 中的默认值为 128。

```rust,compile_fail
#![recursion_limit = "4"]

macro_rules! a {
    () => { a!(1); };
    (1) => { a!(2); };
    (2) => { a!(3); };
    (3) => { a!(4); };
    (4) => { };
}

// 此展开失败，因为它需要的递归深度大于 4。
a!{}
```

```rust,compile_fail
#![recursion_limit = "1"]

// 此失败，因为它需要两个递归步骤才能自动解引用。
(|_: &u8| {})(&&&1);
```

<!-- template:attributes -->
## `type_length_limit`属性

*`type_length_limit` [属性][attributes]* 设置在单态化期间构造具体类型时允许的最大类型替换次数。

> [!NOTE]
> `rustc` 仅在 nightly `-Zenforce-type-length-limit` 标志处于活动状态时强制执行此限制。
>
> 欲了解更多信息，请参阅 [Rust PR #127670](https://github.com/rust-lang/rust/pull/127670)。

> [!EXAMPLE]
> <!-- 忽略：在没有 nightly 标志的情况下不强制执行 -->
> ```rust,ignore
> #![type_length_limit = "4"]
>
> fn f<T>(x: T) {}
>
> // 此编译失败，因为单态化为
> // `f::<((((i32,), i32), i32), i32)>` 需要多于
> // 4 个类型元素。
> f(((((1,), 2), 3), 4));
> ```

> [!NOTE]
> `rustc` 中的默认值为 `1048576`。

`type_length_limit` 属性使用 [MetaNameValueStr] 语法格式。字符串中的值必须是非负数。

`type_length_limit` 属性只能应用于 crate 根。

> [!NOTE]
> `rustc` 忽略在其他位置的使用，但会对其进行 lint 检查。这在将来可能会成为错误。

`type_length_limit` 在 [项][item] 上的首次使用才生效。

> [!NOTE]
> `rustc` 会对首次使用之后的任何使用进行 lint 检查。这在将来可能会成为错误。

[attributes]: ../attributes.md
[crate]: ../crates-and-source-files.md
