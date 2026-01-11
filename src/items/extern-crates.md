r[items.extern-crate]
# extern crate 声明

r[items.extern-crate.syntax]
```grammar,项
ExternCrate -> `extern` `crate` CrateRef AsClause? `;`

CrateRef -> IDENTIFIER | `self`

AsClause -> `as` ( IDENTIFIER | `_` )
```

r[items.extern-crate.intro]
一个 _`extern crate` 声明_ 指定了对外部 crate 的依赖。

r[items.extern-crate.namespace]
外部 crate 随后在 [类型命名空间][type namespace] 中作为给定的 [标识符][identifier] 绑定到声明作用域中。

r[items.extern-crate.extern-prelude]
此外，如果 `extern crate` 出现在 crate 根中，则该 crate 名称也会被添加到 [外部预导入][extern prelude] 中，使其在所有模块的作用域内自动可用。

r[items.extern-crate.as]
`as` 子句可用于将导入的 crate 绑定到不同的名称。

r[items.extern-crate.lookup]
外部 crate 在编译时解析为特定的 `soname`，并且对该 `soname` 的运行时链接要求会被传递给链接器，以便在运行时加载。`soname` 在编译时通过扫描编译器的库路径，并将提供的可选 `crate_name` 与外部 crate 编译时声明的 [`crate_name` 属性][`crate_name` attributes] 进行匹配来解析。如果没有提供 `crate_name`，则假定使用默认的 `name` 属性，该属性等于 `extern crate` 声明中给出的 [标识符][identifier]。

r[items.extern-crate.self]
可以导入 `self` crate，这会创建到当前 crate 的绑定。在这种情况下，必须使用 `as` 子句来指定要绑定到的名称。

三个 `extern crate` 声明的示例：

<!-- ignore: requires external crates -->
```rust,ignore
extern crate pcre;

extern crate std; // 等同于：extern crate std as std;

extern crate std as ruststd; // 以另一个名称链接到 'std'
```

r[items.extern-crate.name-restrictions]
在命名 Rust crate 时，禁止使用连字符。然而，Cargo 包可以使用它们。在这种情况下，当 `Cargo.toml` 没有指定 crate 名称时，Cargo 会透明地将 `-` 替换为 `_`（详见 [RFC 940]）。

示例：

<!-- ignore: requires external crates -->
```rust,ignore
// 导入 Cargo 包 hello-world
extern crate hello_world; // 连字符被替换为下划线
```

r[items.extern-crate.underscore]
## 下划线导入

外部 crate 依赖可以通过在 `extern crate foo as _` 形式中使用下划线来声明，而不将其名称绑定到作用域内。这对于只需要链接但从不引用的 crate 可能很有用，并且可以避免被报告为未使用。

r[items.extern-crate.underscore.macro_use]
[[`macro_use` 属性][`macro_use` attribute]] 照常工作，并将宏名称导入到 [[`macro_use` 预导入][`macro_use` prelude]] 中。

<!-- template:attributes -->
r[items.extern-crate.no_link]
## `no_link`属性

 *`no_link` [属性][attributes]*  可以应用于 `extern crate` 项以防止链接该 crate。

> [!NOTE]
> 这很有帮助，例如，当只需要 crate 的宏时。

> [!EXAMPLE]
> <!-- ignore: requires external crates -->
> ```rust,ignore
> #[no_link]
> extern crate other_crate;
>
> other_crate::some_macro!();
> ```

r[items.extern-crate.no_link.syntax]
`no_link` 属性使用 [MetaWord] 语法。

r[items.extern-crate.no_link.allowed-positions]
`no_link` 属性只能应用于 `extern crate` 声明。

> [!NOTE]
> `rustc` 会忽略在其他位置的使用，但会针对其发出 lint 警告。这在将来可能会变成一个错误。

r[items.extern-crate.no_link.duplicates]
在 `extern crate` 声明上只有第一次使用 `no_link` 有效。

> [!NOTE]
> `rustc` 会针对第一次之后的任何使用发出 lint 警告。这在将来可能会变成一个错误。

[identifier]: ../identifiers.md
[RFC 940]: https://github.com/rust-lang/rfcs/blob/master/text/0940-hyphens-considered-harmful.md
[`macro_use` attribute]: ../macros-by-example.md#macro_use属性
[extern prelude]: ../names/preludes.md#extern-prelude
[`macro_use` prelude]: ../names/preludes.md#macro_use-prelude
[`crate_name` attributes]: ../crates-and-source-files.md#crate_name属性
[type namespace]: ../names/namespaces.md
