<!-- https://github.com/rust-lang/reference/blob/master/src/procedural-macros.md -->
<!-- commit 68bdfd15fa9feebfbf94a06cf574de598e239198 -->

r[macro.proc]
# 过程宏

r[macro.proc.intro]
*过程宏*允许通过执行函数来创建语法扩展。
过程宏分为三种类型：

*   [函数式宏][Function-like macros] - `custom!(...)`
*   [派生宏][Derive macros] - `#[derive(CustomDerive)]`
*   [属性宏][Attribute macros] - `#[CustomAttribute]`

过程宏允许你在编译时运行操作Rust语法的代码，既可以消费Rust语法，也可以生成Rust语法。你可以将过程宏视为从一个AST到另一个AST的函数。

r[macro.proc.def]
过程宏必须定义在[crate类型][crate type]为`proc-macro`的crate根目录中。
宏不能在其定义的crate中使用，只能在另一个crate中导入后才能使用。

> [!NOTE]
> 当使用Cargo时，过程宏crate通过清单中的`proc-macro`键定义：
>
> ```toml
> [lib]
> proc-macro = true
> ```

r[macro.proc.result]
作为函数，它们必须返回语法、[恐慌][panic]，或无限循环。返回的语法会根据过程宏的种类替换或添加语法。编译器会捕获[恐慌][panic]，并将其转换为编译器错误。无限循环不会被编译器捕获，这会导致编译器挂起。

过程宏在编译期间运行，因此拥有与编译器相同的资源。例如，标准输入、错误和输出与编译器可访问的相同。同样，文件访问也相同。因此，过程宏具有与[Cargo的构建脚本][Cargo's build scripts]相同的安全问题。

r[macro.proc.error]
过程宏有两种报告错误的方式。第一种是[恐慌][panic]。第二种是发出一个[`compile_error`]宏调用。

r[macro.proc.proc_macro-crate]
## `proc_macro` crate

r[macro.proc.proc_macro-crate.intro]
过程宏crate几乎总是会链接到编译器提供的[`proc_macro` crate]。[`proc_macro` crate]提供了编写过程宏所需的类型和使其更方便的工具。

r[macro.proc.proc_macro-crate.token-stream]
这个crate主要包含[`词法单元流`][`TokenStream`]类型。过程宏操作*词法单元流*而不是AST节点，这对于编译器和过程宏来说都是一个更稳定的长期接口。*词法单元流*大致等同于`Vec<TokenTree>`，其中`TokenTree`大致可以被认为是词法单元。例如，`foo`是一个`Ident`词法单元，`.`是一个`Punct`词法单元，`1.2`是一个`Literal`词法单元。[`词法单元流`][`TokenStream`]类型与`Vec<TokenTree>`不同，克隆开销很小。

r[macro.proc.proc_macro-crate.span]
所有词法单元都带有一个关联的`Span`。`Span`是一个不透明的值，不能被修改但可以被创建。`Span`代表程序中源代码的范围，主要用于错误报告。虽然你不能修改`Span`本身，但你总是可以更改与任何词法单元*关联的*`Span`，例如通过从另一个词法单元获取`Span`。

r[macro.proc.hygiene]
## 过程宏卫生

过程宏是*不卫生*的。这意味着它们的行为就如同输出词法单元流被直接内联写入其旁边的代码一样。这意味着它受外部项影响，也影响外部导入。

宏作者需要小心，以确保他们的宏在给定此限制的情况下，尽可能在更多上下文中工作。这通常包括使用库中项的绝对路径（例如，`::std::option::Option`而不是`Option`），或者通过确保生成的函数具有不太可能与其他函数冲突的名称（如`__internal_foo`而不是`foo`）。

<!-- TODO: rule name needs improvement -->
<!-- template:attributes -->
r[macro.proc.proc_macro]
## `proc_macro`属性

r[macro.proc.proc_macro.intro]
*`proc_macro`[属性][attributes]* 定义一个[函数式][macro.invocation]过程宏。

> [!EXAMPLE]
> 此宏定义忽略其输入，并在其作用域中发出一个`answer`[函数][function]。
>
> <!-- ignore: test doesn't support proc-macro -->
> ```rust,ignore
> # #![crate_type = "proc-macro"]
> extern crate proc_macro;
> use proc_macro::TokenStream;
>
> #[proc_macro]
> pub fn make_answer(_item: TokenStream) -> TokenStream {
>     "fn answer() -> u32 { 42 }".parse().unwrap()
> }
> ```
>
> 我们可以在一个二进制crate中使用它来打印“42”到标准输出。
>
> <!-- ignore: requires external crates -->
> ```rust,ignore
> extern crate proc_macro_examples;
> use proc_macro_examples::make_answer;
>
> make_answer!();
>
> fn main() {
>     println!("{}", answer());
> }
> ```

r[macro.proc.proc_macro.syntax]
`proc_macro`属性使用[MetaWord]语法。

r[macro.proc.proc_macro.allowed-positions]
`proc_macro`属性只能应用于`fn(TokenStream) -> TokenStream`类型的`pub`[函数][function]，其中[`词法单元流`][`TokenStream`]来自[`proc_macro` crate]。它必须具有 ["Rust" ABI][items.fn.extern]。不允许使用其他函数限定符。它必须位于crate的根目录中。

r[macro.proc.proc_macro.duplicates]
`proc_macro`属性在一个函数上只能指定一次。

r[macro.proc.proc_macro.namespace]
`proc_macro`属性在crate根目录中的[宏命名空间][macro namespace]中公开定义宏，其名称与函数同名。

r[macro.proc.proc_macro.behavior]
一个函数式过程宏的函数式宏调用会将宏调用分隔符内的内容作为输入[`词法单元流`][`TokenStream`]参数传递，并用函数的输出[`词法单元流`][`TokenStream`]替换整个宏调用。

r[macro.proc.proc_macro.invocation]
函数式过程宏可以在任何宏调用位置被调用，包括：

*   [语句][Statements]
*   [表达式][Expressions]
*   [模式][Patterns]
*   [类型表达式][Type expressions]
*   [项][Item]位置，包括[`extern`块][`extern` blocks]中的项
*   固有和特型[实现][implementations]
*   [特型定义][Trait definitions]

<!-- template:attributes -->
r[macro.proc.derive]
## `proc_macro_derive`属性

r[macro.proc.derive.intro]
将 *`proc_macro_derive`[属性][attribute]* 应用于[函数][function]定义一个 *派生宏* ，它可以通过 [`derive`属性][`derive` attribute] 调用。这些宏会获得一个[结构体][struct]、[枚举][enum]或[联合体][union]定义的词法单元流，并可以在其后发出新的[项][items]。它们还可以声明和使用[派生宏辅助属性][derive macro helper attributes]。

> [!EXAMPLE]
> 这个派生宏忽略其输入，并添加定义一个函数的词法单元。
>
> <!-- ignore: test doesn't support proc-macro -->
> ```rust,ignore
> # #![crate_type = "proc-macro"]
> extern crate proc_macro;
> use proc_macro::TokenStream;
>
> #[proc_macro_derive(AnswerFn)]
> pub fn derive_answer_fn(_item: TokenStream) -> TokenStream {
>     "fn answer() -> u32 { 42 }".parse().unwrap()
> }
> ```
>
> 要使用它，我们可以这样写：
>
> <!-- ignore: requires external crates -->
> ```rust,ignore
> extern crate proc_macro_examples;
> use proc_macro_examples::AnswerFn;
>
> #[derive(AnswerFn)]
> struct Struct;
>
> fn main() {
>     assert_eq!(42, answer());
> }
> ```

r[macro.proc.derive.syntax]
`proc_macro_derive`属性的语法格式是：

```grammar,attributes
@root ProcMacroDeriveAttribute ->
    `proc_macro_derive` `(` DeriveMacroName ( `,` DeriveMacroAttributes )? `,`? `)`

DeriveMacroName -> IDENTIFIER

DeriveMacroAttributes ->
    `attributes` `(` ( IDENTIFIER (`,` IDENTIFIER)* `,`?)? `)`
```

派生宏的名称由[DeriveMacroName]给出。可选的`attributes`参数在[macro.proc.derive.attributes]中描述。

r[macro.proc.derive.allowed-positions]
`proc_macro_derive`属性只能应用于定义在crate根目录中，并带有[Rust ABI][items.fn.extern]的`pub`[函数][function]，其[类型][type]为`fn(TokenStream) -> TokenStream`，其中[`词法单元流`][`TokenStream`]来自[`proc_macro` crate]。该函数可以是`const`，并且可以使用`extern`显式指定 Rust ABI，但不能使用任何其他[限定符][FunctionQualifiers]（例如，它不能是`async`或`unsafe`）。

r[macro.proc.derive.duplicates]
`proc_macro_derive`属性在一个函数上只能使用一次。

r[macro.proc.derive.namespace]
`proc_macro_derive`属性在crate的根目录中的[宏命名空间][macro namespace]中公开定义派生宏。

r[macro.proc.derive.output]
输入[`词法单元流`][`TokenStream`]是应用`derive`属性的[项][item]的词法单元流。输出[`词法单元流`][`TokenStream`]必须是（可能为空的）一组[项][items]。这些[项][items]会在输入[项][item]之后，在同一[模块][module]或[块][block]内被追加。

r[macro.proc.derive.attributes]
### 派生宏辅助属性

r[macro.proc.derive.attributes.intro]
派生宏可以声明 *派生宏辅助属性* ，用于派生宏所应用的[项][item]的作用域内。这些[属性][attributes]是[惰性的][inert]。虽然它们的目的被声明它们的宏使用，但它们可以被任何宏看到。

r[macro.proc.derive.attributes.decl]
派生宏的辅助属性通过将其标识符添加到`proc_macro_derive`属性的`attributes`列表中来声明。

> [!EXAMPLE]
> 这声明了一个辅助属性然后忽略了它。
>
> <!-- ignore: test doesn't support proc-macro -->
> ```rust,ignore
> # #![crate_type="proc-macro"]
> # extern crate proc_macro;
> # use proc_macro::TokenStream;
> #
> #[proc_macro_derive(WithHelperAttr, attributes(helper))]
> pub fn derive_with_helper_attr(_item: TokenStream) -> TokenStream {
>     TokenStream::new()
> }
> ```
>
> 要使用它，我们可以这样写：
>
> <!-- ignore: requires external crates -->
> ```rust,ignore
> extern crate proc_macro_examples;
> use proc_macro_examples::WithHelperAttr;
>
> #[derive(WithHelperAttr)]
> struct Struct {
>     #[helper] field: (),
> }
> ```

r[macro.proc.derive.attributes.scope]
当一个派生宏调用应用于一个[项][item]时，该派生宏引入的辅助属性会在以下情况下生效：1) 用于应用于该[项][item]并在派生宏调用后词法应用的[属性][attributes]，以及 2) 用于应用于[项][item]内部的字段和变体的[属性][attributes]。

> [!NOTE]
> rustc目前允许在引入它们的宏之前使用派生辅助。这种乱序使用的派生辅助可能不会遮蔽其他属性宏。此行为已被弃用，并计划移除。
>
> <!-- ignore: requires external crates -->
> ```rust,ignore
> #[helper] // 已弃用，未来将成为硬错误。
> #[derive(WithHelperAttr)]
> struct Struct {
>     field: (),
> }
> ```
>
> 更多详情，请参见[Rust问题单#79202](https://github.com/rust-lang/rust/issues/79202)。

<!-- template:attributes -->
r[macro.proc.attribute]
## `proc_macro_attribute`属性

r[macro.proc.attribute.intro]
*`proc_macro_attribute`[属性][attributes]* 定义一个 *属性宏* ，它可以作为[外部属性][attributes]使用。

> [!EXAMPLE]
> 此属性宏接受输入流并原样发出，实际上是一个空操作属性。
>
> <!-- ignore: test doesn't support proc-macro -->
> ```rust,ignore
> # #![crate_type = "proc-macro"]
> # extern crate proc_macro;
> # use proc_macro::TokenStream;
>
> #[proc_macro_attribute]
> pub fn return_as_is(_attr: TokenStream, item: TokenStream) -> TokenStream {
>     item
> }
> ```

> [!EXAMPLE]
> 这显示了在编译器输出中，属性宏所看到的字符串化[`词法单元流`][`TokenStream`]。
>
> <!-- ignore: test doesn't support proc-macro -->
> ```rust,ignore
> // my-macro/src/lib.rs
> # extern crate proc_macro;
> # use proc_macro::TokenStream;
> #[proc_macro_attribute]
> pub fn show_streams(attr: TokenStream, item: TokenStream) -> TokenStream {
>     println!("attr: \"{attr}\"");
>     println!("item: \"{item}\"");
>     item
> }
> ```
>
> <!-- ignore: requires external crates -->
> ```rust,ignore
> // src/lib.rs
> extern crate my_macro;
>
> use my_macro::show_streams;
>
> // 示例：基本函数。
> #[show_streams]
> fn invoke1() {}
> // 输出：attr: ""
> // 输出：item: "fn invoke1() {}"
>
> // 示例：带输入的属性。
> #[show_streams(bar)]
> fn invoke2() {}
> // 输出：attr: "bar"
> // 输出：item: "fn invoke2() {}"
>
> // 示例：输入中有多个词法单元。
> #[show_streams(multiple => tokens)]
> fn invoke3() {}
> // 输出：attr: "multiple => tokens"
> // 输出：item: "fn invoke3() {}"
>
> // 示例：输入中有分隔符。
> #[show_streams { delimiters }]
> fn invoke4() {}
> // 输出：attr: "delimiters"
> // 输出：item: "fn invoke4() {}"
> ```

r[macro.proc.attribute.syntax]
`proc_macro_attribute`属性使用[MetaWord]语法。

r[macro.proc.attribute.allowed-positions]
`proc_macro_attribute`属性只能应用于`fn(TokenStream, TokenStream) -> TokenStream`类型的`pub`[函数][function]，其中[`词法单元流`][`TokenStream`]来自 [`proc_macro` crate]。它必须具有 ["Rust" ABI][items.fn.extern]。不允许使用其他函数限定符。它必须位于crate的根目录中。

r[macro.proc.attribute.duplicates]
`proc_macro_attribute`属性在一个函数上只能指定一次。

r[macro.proc.attribute.namespace]
`proc_macro_attribute`属性在crate的根目录中的[宏命名空间][macro namespace]中定义属性，其名称与函数同名。

r[macro.proc.attribute.use-positions]
属性宏只能用于：

*   [项][Items]
*   [`extern`块][`extern` blocks] 中的项
*   固有和特型 [实现][implementations]
*   [特型定义][Trait definitions]

r[macro.proc.attribute.behavior]
第一个[`词法单元流`][`TokenStream`]参数是属性名称后的带分隔符词法单元树，但不包括外部分隔符。如果应用的属性只包含属性名称，或属性名称后跟空分隔符，则[`词法单元流`][`TokenStream`]为空。

第二个[`词法单元流`][`TokenStream`]是[项][item]的其余部分，包括[项][item]上的其他[属性][attributes]。

应用属性的[项][item]会被返回的[`词法单元流`][`TokenStream`]中的零个或多个[项][items]替换。

r[macro.proc.token]
## 声明宏词法单元和过程宏词法单元

r[macro.proc.token.intro]
声明式[`macro_rules`]宏和过程宏对词法单元（或更确切地说，[`TokenTree`s][`TokenTree`s]）使用相似但不同的定义。

r[macro.proc.token.macro_rules]
[`macro_rules`]中的词法单元树（对应`tt`匹配器）定义如下：
*   带分隔符的组（`(...)`、`{...}`等）
*   语言支持的所有运算符，包括单字符和多字符的（`+`、`+=`）。
    *   请注意，此集合不包括单引号`'`。
*   字面量（`"string"`、`1`等）
    *   请注意，负号（例如`-1`）永远不会是此类字面量词法单元的一部分，而是一个单独的运算符词法单元。
*   标识符，包括关键字（`ident`、`r#ident`、`fn`）
*   生命周期（`'ident`）
*   [`macro_rules`]中的元变量替换（例如`macro_rules! mac { ($my_expr: expr) => { $my_expr } }`在`mac`展开后的`$my_expr`，无论传递的表达式如何，都将被视为单个词法单元树）

r[macro.proc.token.tree]
过程宏中的词法单元树定义如下：
*   带分隔符的组（`(...)`、`{...}`等）
*   语言支持的运算符中使用的所有标点符号（`+`，但不包括`+=`），以及单引号`'`字符（通常用于生命周期，有关生命周期拆分和合并行为，请参见下文）
*   字面量（`"string"`、`1`等）
    *   负号（例如`-1`）作为整数和浮点字面量的一部分受支持。
*   标识符，包括关键字（`ident`、`r#ident`、`fn`）

r[macro.proc.token.conversion.intro]
当词法单元流在过程宏之间传递时，会考虑这两种定义之间的不匹配。
请注意，下面的转换可能以惰性方式发生，因此如果词法单元未被实际检查，则可能不会发生转换。

r[macro.proc.token.conversion.to-proc_macro]
当传递给过程宏时
*   所有多字符运算符都会被拆分为单字符。
*   生命周期会被拆分为一个`'`字符和一个标识符。
*   关键字元变量 [`$crate`][`$crate`] 作为单个标识符传递。
*   所有其他元变量替换都表示为其底层词法单元流。
    *   此类词法单元流可能被包装到带分隔符的组（[`Group`][`Group`]）中，使用隐式分隔符（[`Delimiter::None`][`Delimiter::None`]），当需要保留解析优先级时。
    *   `tt`和`ident`替换永远不会被包装到此类组中，并且始终表示为其底层词法单元树。

r[macro.proc.token.conversion.from-proc_macro]
当从过程宏发出时
*   标点符号在适用时会被组合成多字符运算符。
*   与标识符连接的单引号`'`会被组合成生命周期。
*   负字面量会被转换为两个词法单元（`-`和字面量），可能被包装到带分隔符的组（[`Group`][`Group`]）中，使用隐式分隔符（[`Delimiter::None`][`Delimiter::None`]），当需要保留解析优先级时。

r[macro.proc.token.doc-comment]
请注意，声明宏和过程宏都不支持文档注释词法单元（例如`/// Doc`），因此当它们被传递给宏时，总是会被转换为表示其等效`#[doc = r"str"]`属性的词法单元流。

[Attribute macros]: #proc_macro_attribute属性
[Cargo's build scripts]: ../cargo/reference/build-scripts.html
[Derive macros]: macro.proc.derive
[Function-like macros]: #proc_macro属性
[`$crate`]: macro.decl.hygiene.crate
[`Delimiter::None`]: proc_macro::Delimiter::None
[`Group`]: proc_macro::Group
[`TokenStream`]: https://doc.rust-lang.org/proc_macro/struct.TokenStream.html
[`TokenTree`s]: proc_macro::TokenTree
[`derive` attribute]: attributes/derive.md
[`extern` blocks]: items/external-blocks.md
[`macro_rules`]: macros-by-example.md
[`proc_macro` crate]: proc_macro
[attribute]: attributes.md
[attributes]: attributes.md
[block]: expressions/block-expr.md
[crate type]: linkage.md
[derive macro helper attributes]: #派生宏辅助属性
[enum]: items/enumerations.md
[expressions]: expressions.md
[function]: items/functions.md
[implementations]: items/implementations.md
[inert]: attributes.md#active-and-inert-attributes
[item]: items.md
[items]: items.md
[macro namespace]: names/namespaces.md
[module]: items/modules.md
[patterns]: patterns.md
[public]: visibility-and-privacy.md
[statements]: statements.md
[struct]: items/structs.md
[trait definitions]: items/traits.md
[type expressions]: types.md#type-expressions
[type]: types.md
[union]: items/unions.md