<!-- https://github.com/rust-lang/reference/blob/master/src/types.md -->
<!-- commit 68bdfd15fa9feebfbf94a06cf574de598e239198 -->

r[type]
# 类型

r[type.intro]
Rust 程序中的每个变量、 项 和值都有一个类型。 *值* 的 _类型_ 定义了对持有它的内存的解释以及可以对该值执行的操作。

r[type.builtin]
内置类型紧密地集成到语言中，其方式非常复杂，无法在用户定义类型中模拟。

r[type.user-defined]
用户定义类型的能力有限。

r[type.kinds]
类型列表如下：

* 原语类型：
    * [布尔型][Boolean] --- `bool`
    * [数值型][Numeric] --- 整数和浮点数
    * [文本型][Textual] --- `char` 和 `str`
    * [Never][Never] --- `!` --- 一种没有值的类型
* 序列类型：
    * [元组][Tuple]
    * [数组][Array]
    * [切片][Slice]
* 用户定义类型：
    * [结构体][Struct]
    * [枚举][Enum]
    * [联合体][Union]
* 函数类型：
    * [函数][Functions]
    * [闭包][Closures]
* 指针类型：
    * [引用][References]
    * [裸指针][Raw pointers]
    * [函数指针][Function pointers]
* 特型类型：
    * [特型对象][Trait objects]
    * [impl 特型][Impl trait]

r[type.name]
## 类型表达式

r[type.name.syntax]
```grammar,类型
Type ->
      TypeNoBounds
    | ImplTraitType
    | TraitObjectType

TypeNoBounds ->
      ParenthesizedType
    | ImplTraitTypeOneBound
    | TraitObjectTypeOneBound
    | TypePath
    | TupleType
    | NeverType
    | RawPointerType
    | ReferenceType
    | ArrayType
    | SliceType
    | InferredType
    | QualifiedPathInType
    | BareFunctionType
    | MacroInvocation
```

r[type.name.intro]
上面 [类型][Type] 语法格式规则中定义的 _类型表达式_ 是引用类型的语法格式。它可以引用：

r[type.name.sequence]
* 序列类型（ [元组][tuple]、 [数组][array]、 [切片][slice] ）。

r[type.name.path]
* [类型路径][Type paths] 可以引用：
    * 原语类型（ [布尔型][boolean]、 [数值型][numeric]、 [文本型][textual] ）。
    * 指向 [项][item] 的路径（ [结构体][struct]、 [枚举][enum]、 [联合体][union]、 [类型别名][type alias]、 [特型][trait] ）。
    * [`Self` 路径][`Self` path] ，其中 `Self` 是实现类型。
    * 泛型 [类型参数][type parameters] 。

r[type.name.pointer]
* 指针类型（ [引用][reference]、 [裸指针][raw pointer]、 [函数指针][function pointer] ）。

r[type.name.inference]
* [推断类型][inferred type] ，它要求编译器确定类型。

r[type.name.grouped]
* 用于消除歧义的 [括号][Parentheses] 。

r[type.name.trait]
* 特型类型： [特型对象][Trait objects] 和 [impl 特型][impl trait] 。

r[type.name.never]
* [never][never] 类型。

r[type.name.macro-expansion]
* 展开为类型表达式的 [宏][Macros] 。

r[type.name.parenthesized]
### 括号类型

r[type.name.parenthesized.syntax]
```grammar,类型
ParenthesizedType -> `(` Type `)`
```

r[type.name.parenthesized.intro]
在某些情况下，类型的组合可能会产生歧义。在类型周围使用括号以避免歧义。例如， [引用类型][reference type] 中用于 [特型边界][type boundaries] 的 `+` 运算符不清楚边界应用于何处，因此需要使用括号。需要这种消除歧义的语法格式规则使用 [TypeNoBounds] 规则而不是 [类型][grammar-Type] 。

```rust
# use std::any::Any;
type T<'a> = &'a (dyn Any + Send);
```

r[type.recursive]
## 递归类型

r[type.recursive.intro]
标称类型 —— [结构体][structs] 、 [枚举][enumerations] 和 [联合体][unions] —— 可能是递归的。也就是说，每个 `enum` 变体或 `struct` 或 `union` 字段可以直接或间接地引用包围它的 `enum` 或 `struct` 类型本身。

r[type.recursive.constraint]
这种递归有限制：

* 递归类型必须在递归中包含一个标称类型（不只是 [类型别名][type aliases] ，或者其他结构化类型，如 [数组][arrays] 或 [元组][tuples] ）。因此 `type Rec = &'static [Rec]` 是不允许的。
* 递归类型的大小必须是有限的；换句话说，类型的递归字段必须是 [指针类型][pointer types] 。

一个 *递归* 类型及其使用的示例：

```rust
enum List<T> {
    Nil,
    Cons(T, Box<List<T>>)
}

let a: List<i32> = List::Cons(7, Box::new(List::Cons(13, Box::new(List::Nil))));
```

[Array]: types/array.md
[Boolean]: types/boolean.md
[Closures]: types/closure.md
[Enum]: types/enum.md
[Function pointers]: types/function-pointer.md
[Functions]: types/function-item.md
[Impl trait]: types/impl-trait.md
[Macros]: macros.md
[Numeric]: types/numeric.md
[Parentheses]: #parenthesized-types
[Raw pointers]: types/pointer.md#裸指针const和mut
[References]: types/pointer.md#共享引用
[Slice]: types/slice.md
[Struct]: types/struct.md
[Textual]: types/textual.md
[Trait objects]: types/trait-object.md
[Tuple]: types/tuple.md
[Type paths]: paths.md#paths-in-types
[Union]: types/union.md
[`Self` path]: paths.md#self-1
[arrays]: types/array.md
[enumerations]: types/enum.md
[function pointer]: types/function-pointer.md
[inferred type]: types/inferred.md
[item]: items.md
[never]: types/never.md
[pointer types]: types/pointer.md
[raw pointer]: types/pointer.md#裸指针const和mut
[reference type]: types/pointer.md#共享引用
[reference]: types/pointer.md#共享引用
[structs]: types/struct.md
[trait]: types/trait-object.md
[tuples]: types/tuple.md
[type alias]: items/type-aliases.md
[type aliases]: items/type-aliases.md
[type boundaries]: trait-bounds.md
[type parameters]: types/parameters.md
[unions]: types/union.md
