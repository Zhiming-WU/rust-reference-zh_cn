r[names.namespaces]
# 命名空间

r[names.namespaces.intro]
一个 *命名空间* 是已声明 [名称][names] 的逻辑分组。名称根据其引用的实体种类被隔离到不同的命名空间中。命名空间允许一个命名空间中出现的名称与另一个命名空间中的相同名称不发生冲突。

存在几个不同的命名空间，每个命名空间包含不同种类的实体。名称的使用将根据上下文在不同的命名空间中寻找该名称的声明，如 [名称解析][name resolution] 章节所述。

r[names.namespaces.kinds]
以下是命名空间列表及其对应的实体：

* 类型命名空间
    * [模块声明][Module declarations]
    * [外部 crate 声明][External crate declarations]
    * [外部 crate 预导入][External crate prelude] 项
    * [结构体][Struct] 、 [联合体][union] 、 [枚举][enum] 、枚举变体声明
    * [特型项声明][Trait item declarations]
    * [类型别名][Type aliases]
    * [关联类型声明][Associated type declarations]
    * 内置类型： [布尔型][boolean] 、 [数值型][numeric] 和 [文本型][textual]
    * [泛型类型参数][Generic type parameters]
    * [`Self` 类型][`Self` type]
    * [工具属性模块][Tool attribute modules]
* 值命名空间
    * [函数声明][Function declarations]
    * [常量项声明][Constant item declarations]
    * [静态项声明][Static item declarations]
    * [结构体构造函数][Struct constructors]
    * [枚举变体构造函数][Enum variant constructors]
    * [`Self` 构造函数][`Self` constructors]
    * [泛型常量参数][Generic const parameters]
    * [关联常量声明][Associated const declarations]
    * [关联函数声明][Associated function declarations]
    * 本地绑定 --- [`let`] 、 [`if let`] 、 [`while let`] 、 [`for`] 、 [`match`] 分支、 [函数参数][function parameters] 、 [闭包参数][closure parameters]
    * 捕获的 [闭包][closure] 变量
* 宏命名空间
    * [`macro_rules` 声明][`macro_rules` declarations]
    * [内置属性][Built-in attributes]
    * [工具属性][Tool attributes]
    * [类函数过程宏][Function-like procedural macros]
    * [派生宏][Derive macros]
    * [派生宏辅助属性][Derive macro helpers]
    * [属性宏][Attribute macros]
* 生命周期命名空间
    * [泛型生命周期参数][Generic lifetime parameters]
* 标签命名空间
    * [循环标签][Loop labels]
    * [块标签][Block labels]

关于如何在不同命名空间中使用重叠名称而不会产生歧义的示例：

```rust
// Foo 在类型命名空间中引入了一个类型，在值命名空间中引入了一个构造函数。
struct Foo(u32);

// Foo 宏在宏命名空间中声明。
macro_rules! Foo {
    () => {};
}

// f 参数类型中的 Foo 指向类型命名空间中的 Foo。
// 'Foo 在生命周期命名空间中引入了一个新的生命周期。
fn example<'Foo>(f: Foo) {
    // Foo 指向值命名空间中的 Foo 构造函数。
    let ctor = Foo;
    // Foo 指向宏命名空间中的 Foo 宏。
    Foo!{}
    // 'Foo 在标签命名空间中引入了一个标签。
    'Foo: loop {
        // 'Foo 指向 'Foo 生命周期参数，而 Foo 指向类型命名空间。
        let x: &'Foo Foo;
        // 'Foo 指向该标签。
        break 'Foo;
    }
}
```

r[names.namespaces.without]
## 无命名空间的具名实体

以下实体具有显式名称，但这些名称不属于任何特定的命名空间。

### 字段

r[names.namespaces.without.fields]
尽管结构体、枚举和联合体字段是具名的，但这些具名字段并不存在于显式命名空间中。它们只能通过 [字段表达式][field expression] 访问，该表达式仅检查正在访问的特定类型的字段名称。

### Use声明

r[names.namespaces.without.use]
一个 [use声明][use declaration] 具有导入到作用域中的具名别名，但 `use` 项本身并不属于特定命名空间。相反，它可以根据正在导入的项种类，将别名引入到多个命名空间中。

r[names.namespaces.sub-namespaces]
## 子命名空间

r[names.namespaces.sub-namespaces.intro]
宏命名空间分为两个子命名空间：一个用于 [感叹号风格宏][bang-style macros] ，另一个用于 [属性][attributes] 。
当解析属性时，作用域内的任何感叹号风格宏都将被忽略。
反之，解析感叹号风格宏将忽略作用域内的属性宏。
这可以防止一种风格遮蔽另一种风格。

例如， [`cfg` 属性][`cfg` attribute] 和 [`cfg` 宏][`cfg` macro] 是宏命名空间中具有相同名称的两个不同实体，但它们仍然可以在各自的上下文中使用。

<!-- ignore: requires external crates -->
> [!NOTE]
> `use` 导入仍然不能在模块或块中创建相同名称的重复绑定，无论子命名空间如何。
>
> ```rust,ignore
> #[macro_export]
> macro_rules! mymac {
>     () => {};
> }
>
> use myattr::mymac; // error[E0252]: 名称 `mymac` 被多次定义。
> ```

[`cfg` attribute]: ../conditional-compilation.md#cfg属性
[`cfg` macro]: ../conditional-compilation.md#cfg宏
[`for`]: ../expressions/loop-expr.md#iterator-loops
[`if let`]: ../expressions/if-expr.md#if-let-patterns
[`let`]: ../statements.md#let-statements
[`macro_rules` declarations]: ../macros-by-example.md
[`match`]: ../expressions/match-expr.md
[`Self` constructors]: ../paths.md#self-1
[`Self` type]: ../paths.md#self-1
[`use` import]: ../items/use-declarations.md
[`while let`]: ../expressions/loop-expr.md#while-let-patterns
[Associated const declarations]: ../items/associated-items.md#associated-constants
[Associated function declarations]: ../items/associated-items.md#associated-functions-and-methods
[Associated type declarations]: ../items/associated-items.md#associated-types
[Attribute macros]: ../procedural-macros.md#proc_macro_attribute属性
[attributes]: ../attributes.md
[bang-style macros]: ../macros.md
[Block labels]: expr.loop.block-labels
[boolean]: ../types/boolean.md
[Built-in attributes]: ../attributes.md#built-in-attributes-index
[closure parameters]: ../expressions/closure-expr.md
[closure]: ../expressions/closure-expr.md
[Constant item declarations]: ../items/constant-items.md
[Derive macro helpers]: ../procedural-macros.md#派生宏辅助属性
[Derive macros]: macro.proc.derive
[entity]: ../glossary.md#entity
[Enum variant constructors]: ../items/enumerations.md
[enum]: ../items/enumerations.md
[External crate declarations]: ../items/extern-crates.md
[External crate prelude]: preludes.md#extern-prelude
[field expression]: ../expressions/field-expr.md
[Function declarations]: ../items/functions.md
[function parameters]: ../items/functions.md#function-parameters
[Function-like procedural macros]: ../procedural-macros.md#proc_macro属性
[Generic const parameters]: ../items/generics.md#const-generics
[Generic lifetime parameters]: ../items/generics.md
[Generic type parameters]: ../items/generics.md
[Loop labels]: ../expressions/loop-expr.md#loop-labels
[Module declarations]: ../items/modules.md
[name resolution]: name-resolution.md
[names]: ../names.md
[numeric]: ../types/numeric.md
[Static item declarations]: ../items/static-items.md
[Struct constructors]: ../items/structs.md
[Struct]: ../items/structs.md
[textual]: ../types/textual.md
[Tool attribute modules]: ../attributes.md#tool-attributes
[Tool attributes]: ../attributes.md#tool-attributes
[Trait item declarations]: ../items/traits.md
[Type aliases]: ../items/type-aliases.md
[union]: ../items/unions.md
[use declaration]: ../items/use-declarations.md
