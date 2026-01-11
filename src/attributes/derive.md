<!-- https://github.com/rust-lang/reference/blob/master/src/attributes/derive.md -->
<!-- commit 68bdfd15fa9feebfbf94a06cf574de598e239198 -->

r[attributes.derive]
# 派生

r[attributes.derive.intro]
*`derive` [属性][attributes]* 调用一个或多个 [派生宏][derive macros]，允许为数据结构自动生成新的 [项][items]。你可以使用 [过程宏][procedural macros] 创建 `derive` 宏。

> [!EXAMPLE]
> [`PartialEq`][macro@PartialEq] 派生宏 为 `Foo<T> where T: PartialEq` 发出一个 [`PartialEq`] 的 [实现][implementation]。[`Clone`][macro@Clone] 派生宏 为 [`Clone`] 也做同样的事情。
>
> ```rust
> #[derive(PartialEq, Clone)]
> struct Foo<T> {
>     a: i32,
>     b: T,
> }
> ```
>
> 生成的 `impl` 项 等同于：
>
> ```rust
> # struct Foo<T> { a: i32, b: T }
> impl<T: PartialEq> PartialEq for Foo<T> {
>     fn eq(&self, other: &Foo<T>) -> bool {
>         self.a == other.a && self.b == other.b
>     }
> }
>
> impl<T: Clone> Clone for Foo<T> {
>     fn clone(&self) -> Self {
>         Foo { a: self.a.clone(), b: self.b.clone() }
>     }
> }
> ```

r[attributes.derive.syntax]
`derive` 属性使用 [MetaListPaths] 语法格式 来指定要调用的 [派生宏][derive macros] 的路径列表。

r[attributes.derive.allowed-positions]
`derive` 属性只能应用于 [结构体][items.struct]、[枚举][items.enum] 和 [联合体][items.union]。

r[attributes.derive.duplicates]
`derive` 属性可以在一个 项 上使用任意次数。所有属性中列出的所有 派生宏 都会被调用。

r[attributes.derive.stdlib]
`derive` 属性在标准库 预导入 中作为 [`core::prelude::v1::derive`] 导出。

r[attributes.derive.built-in]
内置派生在 [语言预导入][names.preludes.lang] 中定义。内置派生的列表是：

- [`Clone`]
- [`Copy`]
- [`Debug`]
- [`Default`]
- [`Eq`]
- [`Hash`]
- [`Ord`]
- [`PartialEq`]
- [`PartialOrd`]

r[attributes.derive.built-in-automatically_derived]
内置派生在它们生成的 实现 上包含 [`automatically_derived` 属性][attributes.derive.automatically_derived]。

r[attributes.derive.behavior]
在宏扩展期间，对于派生列表中的每个元素，相应的 派生宏 扩展为零个或多个 [项][items]。

<!-- template:attributes -->
r[attributes.derive.automatically_derived]
## `automatically_derived` 属性

r[attributes.derive.automatically_derived.intro]
*`automatically_derived` [属性][attributes]* 用于注解一个 [实现][implementation]，以表明它是由 [派生宏][derive macro] 自动创建的。它没有直接作用，但工具和诊断 lint 可能会用它来检测这些自动生成的 实现。

> [!EXAMPLE]
> 给定 `struct Example` 上的 [`#[derive(Clone)]`][macro@Clone]，[派生宏][derive macro] 可能会生成：
>
> ```rust
> # struct Example;
> #[automatically_derived]
> impl ::core::clone::Clone for Example {
>     #[inline]
>     fn clone(&self) -> Self {
>         Example
>     }
> }
> ```

r[attributes.derive.automatically_derived.syntax]
`automatically_derived` 属性使用 [MetaWord] 语法格式。

r[attributes.derive.automatically_derived.allowed-positions]
`automatically_derived` 属性只能应用于 [实现][implementation]。

> [!NOTE]
> `rustc` 在其他位置的使用会被忽略，但会发出 lint 警告。这在将来可能会变为错误。

r[attributes.derive.automatically_derived.duplicates]
在一个 实现 上多次使用 `automatically_derived` 与使用一次效果相同。

> [!NOTE]
> `rustc` 对第一次使用之后的任何使用都会发出 lint 警告。

r[attributes.derive.automatically_derived.behavior]
`automatically_derived` 属性没有行为。

[items]: ../items.md
[derive macro]: macro.proc.derive
[derive macros]: macro.proc.derive
[implementation]: ../items/implementations.md
[items]: ../items.md
[procedural macros]: macro.proc.derive
