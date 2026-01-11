<!-- https://github.com/rust-lang/reference/blob/master/src/keywords.md -->
<!-- commit 68bdfd15fa9feebfbf94a06cf574de598e239198 -->

r[lex.keywords]
# 关键字

Rust 将关键字分为三类：

* [严格关键字](#严格关键字)
* [保留关键字](#保留关键字)
* [弱关键字](#弱关键字)

r[lex.keywords.strict]
## 严格关键字

r[lex.keywords.strict.intro]
这些关键字只能在其正确的上下文中使用。它们不能用作以下内容的名称：

* [项][Items]
* [变量][Variables]和函数参数
* 字段和[变体][variants]
* [类型参数][Type parameters]
* 生命周期参数或[循环标签][loop labels]
* [宏][Macros]或[属性][attributes]
* [宏占位符][Macro placeholders]
* [crate][Crates]

r[lex.keywords.strict.list]
以下关键字在所有版次中都存在：

- `_`
- `as`
- `async`
- `await`
- `break`
- `const`
- `continue`
- `crate`
- `dyn`
- `else`
- `enum`
- `extern`
- `false`
- `fn`
- `for`
- `if`
- `impl`
- `in`
- `let`
- `loop`
- `match`
- `mod`
- `move`
- `mut`
- `pub`
- `ref`
- `return`
- `self`
- `Self`
- `static`
- `struct`
- `super`
- `trait`
- `true`
- `type`
- `unsafe`
- `use`
- `where`
- `while`

r[lex.keywords.strict.edition2018]
> [!EDITION-2018]
> 以下关键字在 2018 版次中添加：
>
> - `async`
> - `await`
> - `dyn`

r[lex.keywords.reserved]
## 保留关键字

r[lex.keywords.reserved.intro]
这些关键字尚未投入使用，但已为将来保留。它们与严格关键字具有相同的限制。这样做的理由是通过禁止当前程序使用这些关键字，使其与 Rust 的未来版本向前兼容。

r[lex.keywords.reserved.list]
- `abstract`
- `become`
- `box`
- `do`
- `final`
- `gen`
- `macro`
- `override`
- `priv`
- `try`
- `typeof`
- `unsized`
- `virtual`
- `yield`

r[lex.keywords.reserved.edition2018]
> [!EDITION-2018]
> `try`关键字在 2018 版次中作为保留关键字添加。

r[lex.keywords.reserved.edition2024]
> [!EDITION-2024]
> `gen`关键字在 2024 版次中作为保留关键字添加。

r[lex.keywords.weak]
## 弱关键字

r[lex.keywords.weak.intro]
这些关键字仅在特定上下文中有特殊含义。例如，可以使用`union`这个名字声明一个变量或方法。

- `'static`
- `macro_rules`
- `raw`
- `safe`
- `union`

r[lex.keywords.weak.macro_rules]
* `macro_rules`用于创建自定义[宏][macros]。

r[lex.keywords.weak.union]
* `union`用于声明一个[联合体][union]，并且仅在联合体声明中使用时才作为关键字。

r[lex.keywords.weak.lifetime-static]
* `'static`用于静态生命周期，不能用作[泛型生命周期参数][generic lifetime parameter]或[循环标签][loop label]

  ```compile_fail
  // error[E0262]: invalid lifetime parameter name: `'static`
  fn invalid_lifetime_parameter<'static>(s: &'static str) -> &'static str { s }
  ```

r[lex.keywords.weak.safe]
* `safe`用于函数和静态量，在[外部块][external blocks]中有特殊含义。

r[lex.keywords.weak.raw]
* `raw`用于[原始借用运算符][raw borrow operators]，并且仅在匹配原始借用运算符形式（例如`&raw const expr`或`&raw mut expr`）时才作为关键字。

r[lex.keywords.weak.dyn.edition2018]
> [!EDITION-2018]
> 在 2015 版次中，[`dyn`]在类型位置后跟不以`::`或`<`开头的路径、生命周期、问号、`for`关键字或左括号时是一个关键字。
>
> 从 2018 版次开始，`dyn`已被提升为严格关键字。

[items]: items.md
[Variables]: variables.md
[Type parameters]: types/parameters.md
[loop labels]: expressions/loop-expr.md#循环标签
[Macros]: macros.md
[attributes]: attributes.md
[Macro placeholders]: macros-by-example.md
[Crates]: crates-and-source-files.md
[union]: items/unions.md
[variants]: items/enumerations.md
[`dyn`]: types/trait-object.md
[loop label]: expressions/loop-expr.md#循环标签
[generic lifetime parameter]: items/generics.md
[external blocks]: items/external-blocks.md
[raw borrow operators]: expressions/operator-expr.md#原始借用运算符