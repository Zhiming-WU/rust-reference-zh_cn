<!-- https://github.com/rust-lang/reference/blob/master/src/names/preludes.md -->
<!-- commit 68bdfd15fa9feebfbf94a06cf574de598e239198 -->

r[names.preludes]
# 预导入

r[names.preludes.intro]
一个 *预导入* 是自动引入到 crate 中每个模块作用域的名称集合。

这些 预导入 名称本身不是模块的一部分：它们在 [名称解析][name resolution] 期间被隐式查询。例如，尽管像 [`Box`] 这样的东西在每个模块的作用域内，你也不能将其引用为 `self::Box`，因为它不是当前模块的成员。

r[names.preludes.kinds]
有几种不同的 预导入：

- [标准库预导入][Standard library prelude]
- [外部预导入][Extern prelude]
- [语言预导入][Language prelude]
- [`macro_use` 预导入][`macro_use` prelude]
- [工具预导入][Tool prelude]

r[names.preludes.std]
## 标准库预导入

r[names.preludes.std.intro]
每个 crate 都有一个标准库预导入，它由单个标准库模块中的名称组成。

r[names.preludes.std.module]
所使用的模块取决于 crate 的 版次，以及是否对 crate 应用了 [`no_std` 属性][`no_std` attribute]：

版次 | 未应用 `no_std` | 已应用 `no_std`
--------| --------------------------- | ----------------------------
2015    | [`std::prelude::rust_2015`] | [`core::prelude::rust_2015`]
2018    | [`std::prelude::rust_2018`] | [`core::prelude::rust_2018`]
2021    | [`std::prelude::rust_2021`] | [`core::prelude::rust_2021`]
2024    | [`std::prelude::rust_2024`] | [`core::prelude::rust_2024`]

> [!NOTE]
> [`std::prelude::rust_2015`] 和 [`std::prelude::rust_2018`] 的内容与 [`std::prelude::v1`] 相同。
>
> [`core::prelude::rust_2015`] 和 [`core::prelude::rust_2018`] 的内容与 [`core::prelude::v1`] 相同。

r[names.preludes.extern]
## 外部预导入

r[names.preludes.extern.intro]
在根模块中使用 [`extern crate`] 导入或提供给编译器（如使用 `rustc` 的 `--extern` 标志）的外部 crate 会被添加到 *外部预导入* 中。如果使用别名导入（如 `extern crate orig_name as new_name`），那么符号 `new_name` 将被添加到 预导入 中。

r[names.preludes.extern.core]
[`core`] crate 总是被添加到 外部预导入 中。

r[names.preludes.extern.std]
只要在 crate 根中没有指定 [`no_std` 属性][`no_std` attribute]，[`std`] crate 就会被添加。

r[names.preludes.extern.edition2018]
> [!EDITION-2018]
> 在 2015 版次中，外部预导入中的 crate 不能通过 [use 声明][use declarations] 引用，因此通常的标准做法是包含 `extern crate` 声明来将它们引入作用域。
>
> 从 2018 版次开始，[use 声明][use declarations] 可以引用 外部预导入 中的 crate，因此使用 `extern crate` 被认为是不符合习惯的。

> [!NOTE]
> 随 `rustc` 一起提供的其他 crate，例如 [`alloc`] 和 [`test`](mod@test)，在使用 Cargo 时不会通过 `--extern` 标志自动包含。即使在 2018 版次中，也必须通过 `extern crate` 声明将它们引入作用域。
>
> ```rust
> extern crate alloc;
> use alloc::rc::Rc;
> ```
>
> Cargo 仅会为过程宏 crate 将 `proc_macro` 引入 外部预导入。

<!--
See https://github.com/rust-lang/rust/issues/57288 for more about the
alloc/test limitation.
-->

<!-- template:attributes -->
r[names.preludes.extern.no_std]
### no_std属性

r[names.preludes.extern.no_std.intro]
*`no_std` [属性][attributes]* 会导致 [`std`] crate 不被自动链接，[标准库预导入][standard library prelude] 转而使用 `core` 预导入，且 [`macro_use` 预导入][`macro_use` prelude] 转而使用从 `core` crate 导出的宏。

> [!EXAMPLE]
> <!-- ignore: test infrastructure can't handle no_std -->
> ```rust,ignore
> #![no_std]
> ```

> [!NOTE]
> 当 crate 的目标平台不支持标准库，或者是有意不使用标准库的功能时，使用 `no_std` 很有用。这些功能主要是动态内存分配（例如 `Box` 和 `Vec`）以及文件和网络功能（例如 `std::fs` 和 `std::io`）。

> [!WARNING]
> 使用 `no_std` 并不阻止链接标准库。在 crate 或其依赖项之一中编写 `extern crate std` 仍然是有效的；这将导致编译器将 `std` crate 链接到程序中。

r[names.preludes.extern.no_std.syntax]
`no_std` 属性使用 [MetaWord] 语法格式。

r[names.preludes.extern.no_std.allowed-positions]
`no_std` 属性只能应用于 crate 根。

r[names.preludes.extern.no_std.duplicates]
`no_std` 属性可以在一个形式上使用任意次数。

> [!NOTE]
> `rustc` 会对第一次之后的任何使用发出 lint。

r[names.preludes.extern.no_std.module]
`no_std` 属性将 [标准库预导入][standard library prelude] 更改为使用 `core` 预导入而不是 `std` 预导入。

r[names.preludes.extern.no_std.macro_use]
默认情况下，从 `std` crate 导出的所有宏都会被添加到 [`macro_use` 预导入][`macro_use` prelude] 中。如果指定了 `no_std` 属性，那么从 `core` crate 导出的所有宏将转而被放入 [`macro_use` 预导入][`macro_use` prelude] 中。

r[names.preludes.extern.no_std.edition2018]
> [!EDITION-2018]
> 在 2018 版次之前，`std` 默认注入到 crate 根中。如果指定了 `no_std`，则注入 `core` 代替。从 2018 版次开始，无论是否指定 `no_std`，都不会将两者注入到 crate 根中。

r[names.preludes.lang]
## 语言预导入

r[names.preludes.lang.intro]
语言预导入包含了语言内置的类型和属性名称。语言预导入始终在作用域内。

r[names.preludes.lang.entities]
它包含以下内容：

* [类型命名空间][Type namespace]
    * [布尔类型][Boolean type] --- `bool`
    * [文本类型][Textual types] --- `char` 和 `str`
    * [整数类型][Integer types] --- `i8`, `i16`, `i32`, `i64`, `i128`, `u8`, `u16`, `u32`, `u64`, `u128`
    * [平台相关整数类型][Machine-dependent integer types] --- `usize` 和 `isize`
    * [浮点类型][floating-point types] --- `f32` 和 `f64`
* [宏命名空间][Macro namespace]
    * [内置属性][Built-in attributes]
    * [内置派生宏][attributes.derive.built-in]

r[names.preludes.macro_use]
## `macro_use`预导入

r[names.preludes.macro_use.intro]
`macro_use` 预导入包含了从应用了 [`macro_use` 属性][`macro_use` attribute] 的 [`extern crate`] 导入的外部 crate 中的宏。

r[names.preludes.tool]
## 工具预导入

r[names.preludes.tool.intro]
工具预导入包含了 [类型命名空间][type namespace] 中外部工具的工具名称。有关更多详细信息，请参阅 [工具属性][tool attributes] 部分。

<!-- template:attributes -->
r[names.preludes.no_implicit_prelude]
## no_implicit_prelude属性

r[names.preludes.no_implicit_prelude.intro]
*`no_implicit_prelude` [属性][attribute]* 用于防止隐式 预导入 被引入作用域。

> [!EXAMPLE]
> ```rust
> // 该属性可以应用于 crate 根以影响
> // 所有模块。
> #![no_implicit_prelude]
>
> // 或者它可以应用于一个模块，仅影响该模块
> // 及其后代。
> #[no_implicit_prelude]
> mod example {
>     // ...
> }
> ```

r[names.preludes.no_implicit_prelude.syntax]
`no_implicit_prelude` 属性使用 [MetaWord] 语法格式。

r[names.preludes.no_implicit_prelude.allowed-positions]
`no_implicit_prelude` 属性只能应用于 crate 或模块。

> [!NOTE]
> `rustc` 会忽略在其他位置的使用，但会对其发出 lint。这在未来可能会变成一个错误。

r[names.preludes.no_implicit_prelude.duplicates]
`no_implicit_prelude` 属性可以在一个形式上使用任意次数。

> [!NOTE]
> `rustc` 会对第一次之后的任何使用发出 lint。

r[names.preludes.no_implicit_prelude.excluded-preludes]
`no_implicit_prelude` 属性阻止 [标准库预导入][standard library prelude]、[外部预导入][extern prelude]、[`macro_use` 预导入][`macro_use` prelude] 和 [工具预导入][tool prelude] 被引入到该模块及其后代的作用域中。

r[names.preludes.no_implicit_prelude.lang]
`no_implicit_prelude` 属性不影响 [语言预导入][language prelude]。

r[names.preludes.no_implicit_prelude.edition2018]
> [!EDITION-2018]
> 在 2015 版次中，`no_implicit_prelude` 属性不影响 [`macro_use` 预导入][`macro_use` prelude]，并且从标准库导出的所有宏仍包含在 `macro_use` 预导入中。从 2018 版次开始，该属性确实会移除 `macro_use` 预导入。

[`extern crate`]: ../items/extern-crates.md
[`macro_use` attribute]: ../macros-by-example.md#macro_use属性
[`macro_use` prelude]: #macro_use-prelude
[`no_std` attribute]: #the-no_std-attribute
[attribute]: ../attributes.md
[Boolean type]: ../types/boolean.md
[Built-in attributes]: ../attributes.md#内置属性索引
[extern prelude]: #extern-prelude
[floating-point types]: ../types/numeric.md#浮点类型
[Integer types]: ../types/numeric.md#整数类型
[Language prelude]: #language-prelude
[Machine-dependent integer types]: ../types/numeric.md#平台相关整数类型
[Macro namespace]: namespaces.md
[name resolution]: name-resolution.md
[standard library prelude]: names.preludes.std
[Textual types]: ../types/textual.md
[tool attributes]: ../attributes.md#工具属性
[Tool prelude]: #tool-prelude
[Type namespace]: namespaces.md
[use declarations]: ../items/use-declarations.md
