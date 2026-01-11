<!-- https://github.com/rust-lang/reference/blob/master/src/attributes.md -->
<!-- commit 68bdfd15fa9feebfbf94a06cf574de598e239198 -->

r[attributes]
# 属性

r[attributes.syntax]
```grammar,属性
InnerAttribute -> `#` `!` `[` Attr `]`

OuterAttribute -> `#` `[` Attr `]`

Attr ->
      SimplePath AttrInput?
    | `unsafe` `(` SimplePath AttrInput? `)`

AttrInput ->
      DelimTokenTree
    | `=` Expression
```

r[attributes.intro]
一个 _属性_ 是一个通用的、自由形式的元数据，根据名称、约定、语言和编译器版本进行解释。属性的建模基于 [ECMA-335] 中的属性，其 语法格式 源自 [ECMA-334] (C#)。

r[attributes.inner]
_内部属性_，写在哈希符号 (`#`) 后带一个感叹号 (`!`)，适用于声明该属性的所在形式。

> [!EXAMPLE]
> ```rust
> // 适用于 enclosing 模块或 crate 的通用元数据。
> #![crate_type = "lib"]
>
> // 内部属性适用于整个函数。
> fn some_unused_variables() {
>   #![allow(unused_variables)]
>
>   let x = ();
>   let y = ();
>   let z = ();
> }
> ```

r[attributes.outer]
_外部属性_，写在哈希符号后不带感叹号，适用于属性后面的形式。

> [!EXAMPLE]
> ```rust
> // 标记为单元测试的函数
> #[test]
> fn test_foo() {
>     /* ... */
> }
>
> // 有条件编译的模块
> #[cfg(target_os = "linux")]
> mod bar {
>     /* ... */
> }
>
> // 用于抑制警告/错误的 lint 属性
> #[allow(non_camel_case_types)]
> type int8_t = i8;
> ```

r[attributes.input]
属性由属性的路径组成，后跟一个可选的带分隔符的词法单元树，其解释由属性定义。除了宏属性之外，属性还允许输入是一个等号 (`=`) 后跟一个表达式。有关更多详细信息，请参阅下面的 [元项属性语法](#meta-item-attribute-syntax)。

r[attributes.safety]
属性的应用可能不安全。为避免在使用这些属性时出现未定义行为，必须满足某些编译器无法检查的义务。为了声明这些义务已满足，属性被 `unsafe(..)` 包裹，例如 `#[unsafe(no_mangle)]`。

以下属性是不安全的：

* [`export_name`]
* [`link_section`]
* [`naked`]
* [`no_mangle`]

r[attributes.kind]
属性可以分为以下几种：

* [内置属性][Built-in attributes]
* [过程宏属性][attribute macros]
* [派生宏辅助属性][Derive macro helper attributes]
* [工具属性](#tool-attributes)

r[attributes.allowed-position]
属性可以应用于语言中的多种形式：

* 所有 [项声明][item declarations] 接受外部属性，而 [外部块][external blocks]、[函数][functions]、[实现][implementations] 和 [模块][modules] 接受内部属性。
* 大多数 [语句][statements] 接受外部属性（有关表达式语句的限制，请参阅 [表达式属性][Expression Attributes]）。
* [块表达式][block expressions] 接受外部和内部属性，但仅当它们是 [表达式语句][expression statement] 的外部表达式或另一个块表达式的最终表达式时。
* [枚举][enum] 变体和 [结构体][struct] 及 [联合体][union] 字段接受外部属性。
* [匹配表达式分支][match expressions] 接受外部属性。
* [泛型 生命周期或类型参数][generics] 接受外部属性。
* 表达式在有限情况下接受外部属性，详见 [表达式属性][Expression Attributes]。
* [函数][functions]、[闭包][closure] 和 [函数指针][function pointer] 参数接受外部属性。这包括在函数指针和 [外部块][variadic functions] 中用 `...` 表示的可变参数上的属性。
* [内联汇编][Inline assembly] 模板字符串和操作数接受外部属性。语义上只接受某些属性；有关详细信息，请参阅 [asm.attributes.supported-attributes]。

r[attributes.meta]
## 元项属性语法

r[attributes.meta.intro]
“元项”是大多数 [内置属性][built-in attributes] 用于 [Attr] 规则的 语法格式。它具有以下语法：

r[attributes.meta.syntax]
```grammar,属性
@root MetaItem ->
      SimplePath
    | SimplePath `=` Expression
    | SimplePath `(` MetaSeq? `)`

MetaSeq ->
    MetaItemInner ( `,` MetaItemInner )* `,`?

MetaItemInner ->
      MetaItem
    | Expression
```

r[attributes.meta.literal-expr]
元项中的表达式必须宏展开为字面量表达式，其中不得包含整数或浮点类型后缀。非字面量表达式的表达式在语法上会被接受（并可传递给过程宏），但会在解析后被拒绝。

r[attributes.meta.order]
请注意，如果属性出现在另一个宏中，它将在该外部宏之后展开。例如，以下代码将首先展开 `Serialize` 过程宏，该宏必须保留 `include_str!` 调用才能被展开：

```rust ignore
#[derive(Serialize)]
struct Foo {
    #[doc = include_str!("x.md")]
    x: u32
}
```

r[attributes.meta.order-macro]
此外，属性中的宏只会在应用于该 项 的所有其他属性之后展开：

```rust ignore
#[macro_attr1] // 首先展开
#[doc = mac!()] // `mac!` 第四次展开。
#[macro_attr2] // 第二次展开
#[derive(MacroDerive1, MacroDerive2)] // 第三次展开
fn foo() {}
```

r[attributes.meta.builtin]
各种内置属性使用元项语法的不同子集来指定它们的输入。以下语法规则展示了一些常用形式：

r[attributes.meta.builtin.syntax]
```grammar,属性
@root MetaWord ->
    IDENTIFIER

MetaNameValueStr ->
    IDENTIFIER `=` (STRING_LITERAL | RAW_STRING_LITERAL)

@root MetaListPaths ->
    IDENTIFIER `(` ( SimplePath (`,` SimplePath)* `,`? )? `)`

@root MetaListIdents ->
    IDENTIFIER `(` ( IDENTIFIER (`,` IDENTIFIER)* `,`? )? `)`

@root MetaListNameValueStr ->
    IDENTIFIER `(` ( MetaNameValueStr (`,` MetaNameValueStr)* `,`? )? `)`
```

一些元项的示例如下：

样式 | 示例
------|--------
[元字][MetaWord] | `no_std`
[元名称值字符串][MetaNameValueStr] | `doc = "example"`
[元列表路径][MetaListPaths] | `allow(unused, clippy::inline_always)`
[元列表标识符][MetaListIdents] | `macro_use(foo, bar)`
[元列表名称值字符串][MetaListNameValueStr] | `link(name = "CoreFoundation", kind = "framework")`

r[attributes.activity]
## 活跃属性和惰性属性

r[attributes.activity.intro]
属性要么是活跃的，要么是惰性的。在属性处理期间，*活跃属性* 会从它们所在的表单中移除自己，而 *惰性属性* 则保留在表单上。

[`cfg`] 和 [`cfg_attr`] 属性是活跃的。[属性宏][attribute macros] 是活跃的。所有其他属性都是惰性的。

r[attributes.tool]
## 工具属性

r[attributes.tool.intro]
编译器可能允许外部工具的属性，其中每个工具都位于 [工具预导入][tool prelude] 中的自己的模块中。属性路径的第一个段是工具的名称，带有一个或多个额外段，其解释由工具决定。

r[attributes.tool.ignored]
当工具未使用时，工具的属性会被接受，而不会发出警告。当工具在使用时，工具负责处理和解释其属性。

r[attributes.tool.prelude]
如果使用了 [`no_implicit_prelude`] 属性，则工具属性不可用。

```rust
// 告诉 rustfmt 工具不要格式化以下元素。
#[rustfmt::skip]
struct S {
}

// 控制 clippy 工具的“圈复杂度”阈值。
#[clippy::cyclomatic_complexity = "100"]
pub fn f() {}
```

> [!NOTE]
> `rustc` 目前识别的工具包括 "clippy", "rustfmt", "diagnostic", "miri" 和 "rust_analyzer"。

r[attributes.builtin]
## 内置属性索引

以下是所有内置属性的索引。

- 条件编译
  - [`cfg`] --- 控制条件编译。
  - [`cfg_attr`] --- 有条件地包含属性。

- 测试
  - [`test`] --- 将函数标记为测试。
  - [`ignore`] --- 禁用测试函数。
  - [`should_panic`] --- 表示测试应生成一个 恐慌。

- 派生
  - [`derive`] --- 自动 特型 实现。
  - [`automatically_derived`] --- `derive` 创建的实现的标记。

- 宏
  - [`macro_export`] --- 导出用于跨 crate 使用的 `macro_rules` 宏。
  - [`macro_use`] --- 扩展宏可见性，或从其他 crate 导入宏。
  - [`proc_macro`] --- 定义函数式宏。
  - [`proc_macro_derive`] --- 定义派生宏。
  - [`proc_macro_attribute`] --- 定义属性宏。

- 诊断
  - [`allow`]、[`expect`]、[`warn`]、[`deny`]、[`forbid`] --- 更改默认 lint 级别。
  - [`deprecated`] --- 生成弃用通知。
  - [`must_use`] --- 对未使用的值生成 lint。
  - [`diagnostic::on_unimplemented`] --- 提示编译器在 未实现 特型 时发出特定错误消息。
  - [`diagnostic::do_not_recommend`] --- 提示编译器不要在错误消息中显示某个 特型 实现。

- ABI、链接、符号和 FFI
  - [`link`] --- 指定要与 `extern` 块链接的本地库。
  - [`link_name`] --- 指定 `extern` 块中函数或静态变量的符号名称。
  - [`link_ordinal`] --- 指定 `extern` 块中函数或静态变量的符号序号。
  - [`no_link`] --- 阻止链接外部 crate。
  - [`repr`] --- 控制类型布局。
  - [`crate_type`] --- 指定 crate 的类型（库、可执行文件等）。
  - [`no_main`] --- 禁用发出 `main` 符号。
  - [`export_name`] --- 指定函数或静态变量的导出符号名称。
  - [`link_section`] --- 指定函数或静态变量要使用的对象文件节。
  - [`no_mangle`] --- 禁用符号名称编码。
  - [`used`] --- 强制编译器将静态 项 保留在输出对象文件中。
  - [`crate_name`] --- 指定 crate 名称。

- 代码生成
  - [`inline`] --- 提示内联代码。
  - [`cold`] --- 提示函数不太可能被调用。
  - [`naked`] --- 阻止编译器发出函数序言和尾声。
  - [`no_builtins`] --- 禁用使用某些内置函数。
  - [`target_feature`] --- 配置特定于平台的代码生成。
  - [`track_caller`] --- 将父调用位置传递给 `std::panic::Location::caller()`。
  - [`instruction_set`] --- 指定用于生成函数代码的指令集。

- 文档
  - `doc` --- 指定文档。有关更多信息，请参阅 [The Rustdoc Book]。 [文档注释][Doc comments] 会转换为 `doc` 属性。

- 预导入
  - [`no_std`] --- 从 预导入 中移除 std。
  - [`no_implicit_prelude`] --- 禁用模块内的 预导入 查找。

- 模块
  - [`path`] --- 指定模块的文件名。

- 限制
  - [`recursion_limit`] --- 设置某些编译时操作的最大递归限制。
  - [`type_length_limit`] --- 设置多态类型的最大大小。

- 运行时
  - [`panic_handler`] --- 设置处理 恐慌 的函数。
  - [`global_allocator`] --- 设置全局内存分配器。
  - [`windows_subsystem`] --- 指定要链接的 Windows 子系统。

- 特性
  - `feature` --- 用于启用不稳定或实验性编译器 特性。有关 `rustc` 中实现的 特性，请参阅 [The Unstable Book]。

- 类型系统
  - [`non_exhaustive`] --- 指示将来会为类型添加更多字段/变体。

- 调试器
  - [`debugger_visualizer`] --- 嵌入一个文件，该文件指定某种类型的调试器输出。
  - [`collapse_debuginfo`] --- 控制宏调用在调试信息中编码的方式。

[Doc comments]: comments.md#文档注释
[ECMA-334]: https://www.ecma-international.org/publications-and-standards/standards/ecma-334/
[ECMA-335]: https://www.ecma-international.org/publications-and-standards/standards/ecma-335/
[Expression Attributes]: expressions.md#表达式属性
[The Rustdoc Book]: ../rustdoc/the-doc-attribute.html
[The Unstable Book]: ../unstable-book/index.html
[`allow`]: attributes/diagnostics.md#lint检查属性
[`automatically_derived`]: attributes/derive.md#the-automatically_derived-attribute
[`cfg_attr`]: conditional-compilation.md#cfg_attr属性
[`cfg`]: conditional-compilation.md#cfg属性
[`cold`]: attributes/codegen.md#cold属性
[`collapse_debuginfo`]: attributes/debugger.md#the-collapse_debuginfo-attribute
[`crate_name`]: crates-and-source-files.md#crate_name属性
[`crate_type`]: linkage.md
[`debugger_visualizer`]: attributes/debugger.md#the-debugger_visualizer-attribute
[`deny`]: attributes/diagnostics.md#lint检查属性
[`deprecated`]: attributes/diagnostics.md#deprecated属性
[`derive`]: attributes/derive.md
[`export_name`]: abi.md#the-export_name-attribute
[`expect`]: attributes/diagnostics.md#lint检查属性
[`forbid`]: attributes/diagnostics.md#lint检查属性
[`global_allocator`]: runtime.md#the-global_allocator-attribute
[`ignore`]: attributes/testing.md#ignore属性
[`inline`]: attributes/codegen.md#inline属性
[`instruction_set`]: attributes/codegen.md#the-instruction_set-attribute
[`link_name`]: items/external-blocks.md#the-link_name-attribute
[`link_ordinal`]: items/external-blocks.md#the-link_ordinal-attribute
[`link_section`]: abi.md#the-link_section-attribute
[`link`]: items/external-blocks.md#link属性
[`macro_export`]: macros-by-example.md#macro_export属性
[`macro_use`]: macros-by-example.md#macro_use属性
[`must_use`]: attributes/diagnostics.md#the-must_use-attribute
[`naked`]: attributes/codegen.md#naked属性
[`no_builtins`]: attributes/codegen.md#the-no_builtins-attribute
[`no_implicit_prelude`]: names/preludes.md#the-no_implicit_prelude-attribute
[`no_link`]: items/extern-crates.md#the-no_link-attribute
[`no_main`]: crates-and-source-files.md#no_main属性
[`no_mangle`]: abi.md#the-no_mangle-attribute
[`no_std`]: names/preludes.md#the-no_std-attribute
[`non_exhaustive`]: attributes/type_system.md#the-non_exhaustive-attribute
[`panic_handler`]: panic.md#the-panic_handler-attribute
[`path`]: items/modules.md#path属性
[`proc_macro_attribute`]: procedural-macros.md#proc_macro_attribute属性
[`proc_macro_derive`]: macro.proc.derive
[`proc_macro`]: procedural-macros.md#proc_macro属性
[`recursion_limit`]: attributes/limits.md#the-recursion_limit-attribute
[`repr`]: type-layout.md#表示法
[`should_panic`]: attributes/testing.md#the-should_panic-attribute
[`target_feature`]: attributes/codegen.md#the-target_feature-attribute
[`test`]: attributes/testing.md#test属性
[`track_caller`]: attributes/codegen.md#the-track_caller-attribute
[`type_length_limit`]: attributes/limits.md#the-type_length_limit-attribute
[`used`]: abi.md#used属性
[`warn`]: attributes/diagnostics.md#lint检查属性
[`windows_subsystem`]: runtime.md#the-windows_subsystem-attribute
[attribute macros]: procedural-macros.md#proc_macro_attribute属性
[block expressions]: expressions/block-expr.md
[built-in attributes]: #built-in-attributes-index
[derive macro helper attributes]: procedural-macros.md#派生宏辅助属性
[enum]: items/enumerations.md
[expression statement]: statements.md#表达式语句
[external blocks]: items/external-blocks.md
[functions]: items/functions.md
[generics]: items/generics.md
[implementations]: items/implementations.md
[item declarations]: items.md
[match expressions]: expressions/match-expr.md
[modules]: items/modules.md
[statements]: statements.md
[struct]: items/structs.md
[tool prelude]: names/preludes.md#工具预导入
[union]: items/unions.md
[closure]: expressions/closure-expr.md
[function pointer]: types/function-pointer.md
[variadic functions]: items/external-blocks.html#variadic-functions
[`diagnostic::on_unimplemented`]: attributes/diagnostics.md#the-diagnosticon_unimplemented-attribute
[`diagnostic::do_not_recommend`]: attributes/diagnostics.md#the-diagnosticdo_not_recommend-attribute
[Inline assembly]: inline-assembly.md
