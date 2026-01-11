<!-- https://github.com/rust-lang/reference/blob/master/src/unsafety.md -->
<!-- commit 68bdfd15fa9feebfbf94a06cf574de598e239198 -->

r[safety]
# 不安全性

r[safety.intro]
不安全操作是指那些可能会违反 Rust 静态语义中内存安全保证的操作。

r[safety.unsafe-ops]
以下语言层级的特性不能在 Rust 的安全子集中使用：

r[safety.unsafe-deref]
- 解引用 [原始指针][raw pointer]。

r[safety.unsafe-static]
- 读取或写入 [可变][mutable] 或不安全的 [外部][external] 静态变量。

r[safety.unsafe-union-access]
- 访问 [`联合体`][`union`] 的字段，除了对其进行赋值。

r[safety.unsafe-call]
- 调用不安全函数。

r[safety.unsafe-target-feature-call]
- 在没有启用相同特性的 `target_feature` 属性的函数中，调用标有 [`target_feature`][attributes.codegen.target_feature] 的安全函数（请参阅 [attributes.codegen.target_feature.safety-restrictions]）。

r[safety.unsafe-impl]
- 实现 [不安全特型][unsafe trait]。

r[safety.unsafe-extern]
- 声明 [`外部`][`extern`] 块[^extern-2024]。

r[safety.unsafe-attribute]
- 将 [不安全属性][unsafe attribute] 应用于 [项][item]。

[^extern-2024]: 在 2024 版次 之前，允许在不使用 `unsafe` 的情况下声明外部块。

[`extern`]: items/external-blocks.md
[`union`]: items/unions.md
[mutable]: items/static-items.md#可变静态项
[external]: items/external-blocks.md
[raw pointer]: types/pointer.md
[unsafe trait]: items/traits.md#不安全特型
[unsafe attribute]: attributes.md
