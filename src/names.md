r[names]
# 名称

r[names.intro]
 一个 *实体(entity)* 是一种语言结构，它在源程序中可以以某种方式被引用，通常是通过 [路径][path] 。实体包括 [类型][types] 、 [项][items] 、 [泛型参数][generic parameters] 、 [变量绑定][variable bindings] 、 [循环标签][loop labels] 、 [生命周期][lifetimes] 、 [字段][fields] 、 [属性][attributes] 以及 [lint][lints] 。

 一个 *声明(declaration)* 是一种语法结构，它可以引入一个 *名称(name)* 来引用实体。实体名称在 [*作用域*][*scope*] 内有效 --- 即该名称可以被引用的源文本区域。

某些实体在源码中被 [显式声明](#explicitly-declared-entities) ，而某些实体作为语言或编译器扩展的一部分被 [隐式声明](#implicitly-declared-entities) 。

 [*路径*][*paths*] 用于引用实体，可能是在另一个模块或类型中。

生命周期和循环标签使用一种 [专用语法][lifetimes-and-loop-labels] ，带有一个前导引号。

名称被隔离在不同的 [*命名空间*][*namespaces*] 中，允许不同命名空间中的实体共享相同的名称而不会发生冲突。

 [*名称解析*][*Name resolution*] 是将路径、标识符和标签绑定到实体声明的编译时过程。

对某些名称的访问可能会基于它们的 [*可见性*][*visibility*] 受到限制。

r[names.explicit]
## 显式声明的实体

r[names.explicit.list]
在源码中显式引入名称的实体有：

r[names.explicit.item-decl]
* [项][Items] ：
    * [模块声明][Module declarations]
    * [外部 crate 声明][External crate declarations]
    * [使用声明][Use declarations]
    * [函数声明][Function declarations] 和 [函数参数][function parameters]
    * [类型别名][Type aliases]
    * [结构体][struct] 、 [联合体][union] 、 [枚举][enum] 、枚举变体声明以及它们的具名字段
    * [常量项声明][Constant item declarations]
    * [静态项声明][Static item declarations]
    * [特型项声明][Trait item declarations] 和它们的 [关联项][associated items]
    * [外部块项][External block items]
    * [`macro_rules` 声明][`macro_rules` declarations] 和 [匹配器元变量][matcher metavariables]
    * [实现][Implementation] 关联项

r[names.explicit.expr]
* [表达式][Expressions] ：
    * [闭包][Closure] 参数
    * [`while let`] 模式绑定
    * [`for`] 模式绑定
    * [`if let`] 模式绑定
    * [`match`] 模式绑定
    * [循环标签][Loop labels]

r[names.explicit.generics]
* [泛型参数][Generic parameters]

r[names.explicit.higher-ranked-bounds]
* [高阶特型界限][Higher ranked trait bounds]

r[names.explicit.binding]
* [`let` 语句][`let` statement] 模式绑定

r[names.explicit.macro_use]
* [`macro_use` 属性][`macro_use` attribute] 可以引入来自另一个 crate 的宏名称

r[names.explicit.macro_export]
* [`macro_export` 属性][`macro_export` attribute] 可以为宏向 crate 根部引入一个别名

r[names.explicit.macro-invocation]
此外， [宏调用][macro invocations] 和 [属性][attributes] 可以通过展开为上述项之一来引入名称。

r[names.implicit]
## 隐式声明的实体

r[names.implicit.list]
以下实体由语言隐式定义，或由编译器选项和扩展引入：

r[names.implicit.primitive-types]
* [语言预导入][Language prelude] ：
    * [布尔类型][Boolean type] --- `bool`
    * [文本类型][Textual types] --- `char` 和 `str`
    * [整数类型][Integer types] --- `i8`, `i16`, `i32`, `i64`, `i128`, `u8`, `u16`, `u32`, `u64`, `u128`
    * [机器相关整数类型][Machine-dependent integer types] --- `usize` 和 `isize`
    * [浮点类型][floating-point types] --- `f32` 和 `f64`

r[names.implicit.builtin-attributes]
* [内置属性][Built-in attributes]

r[names.implicit.prelude]
* [标准库预导入][Standard library prelude] 项、属性和宏

r[names.implicit.stdlib]
* 根模块中的 [标准库][extern-prelude] crate

r[names.implicit.extern-prelude]
* 由编译器链接的 [外部 crate][extern-prelude]

r[names.implicit.tool-attributes]
* [工具属性][Tool attributes]

r[names.implicit.lints]
* [lint][Lints] 和 [工具 lint 属性][tool lint attributes]

r[names.implicit.derive-helpers]
* [派生辅助属性][Derive helper attributes] 在项内有效，无需显式导入

r[names.implicit.lifetime-static]
* [`'static`] 生命周期

r[names.implicit.root]
此外，crate 根模块没有名称，但可以使用某些 [路径限定符][path qualifiers] 或别名来引用。

[*Name resolution*]: names/name-resolution.md
[*namespaces*]: names/namespaces.md
[*paths*]: paths.md
[*scope*]: names/scopes.md
[*visibility*]: visibility-and-privacy.md
[`'static`]: keywords.md#弱关键字
[`for`]: expressions/loop-expr.md#iterator-loops
[`if let`]: expressions/if-expr.md#if-let-patterns
[`let` statement]: statements.md#let-statements
[`macro_export` attribute]: macros-by-example.md#macro_export属性
[`macro_rules` declarations]: macros-by-example.md
[`macro_use` attribute]: macros-by-example.md#macro_use属性
[`match`]: expressions/match-expr.md
[`while let`]: expressions/loop-expr.md#while-let-patterns
[associated items]: items/associated-items.md
[attributes]: attributes.md
[Boolean type]: types/boolean.md
[Built-in attributes]: attributes.md#built-in-attributes-index
[Closure]: expressions/closure-expr.md
[Constant item declarations]: items/constant-items.md
[Derive helper attributes]: procedural-macros.md#派生宏辅助属性
[enum]: items/enumerations.md
[Expressions]: expressions.md
[extern-prelude]: names/preludes.md#extern-prelude
[External block items]: items/external-blocks.md
[External crate declarations]: items/extern-crates.md
[fields]: expressions/field-expr.md
[floating-point types]: types/numeric.md#floating-point-types
[Function declarations]: items/functions.md
[function parameters]: items/functions.md#function-parameters
[Generic parameters]: items/generics.md
[Higher ranked trait bounds]: trait-bounds.md#higher-ranked-trait-bounds
[Implementation]: items/implementations.md
[Integer types]: types/numeric.md#integer-types
[Items]: items.md
[Language prelude]: names/preludes.md#language-prelude
[lifetimes-and-loop-labels]: tokens.md#生命周期和循环标签
[lifetimes]: tokens.md#生命周期和循环标签
[Lints]: attributes/diagnostics.md#lint-check-attributes
[Loop labels]: expressions/loop-expr.md#loop-labels
[Machine-dependent integer types]: types/numeric.md#machine-dependent-integer-types
[macro invocations]: macros.md#宏调用
[matcher metavariables]: macros-by-example.md#元变量
[Module declarations]: items/modules.md
[path]: paths.md
[path qualifiers]: paths.md#path-qualifiers
[Standard library prelude]: names/preludes.md#standard-library-prelude
[Static item declarations]: items/static-items.md
[struct]: items/structs.md
[Textual types]: types/textual.md
[Tool attributes]: attributes.md#tool-attributes
[tool lint attributes]: attributes/diagnostics.md#tool-lint-attributes
[Trait item declarations]: items/traits.md
[Type aliases]: items/type-aliases.md
[types]: types.md
[union]: items/unions.md
[Use declarations]: items/use-declarations.md
[variable bindings]: patterns.md
