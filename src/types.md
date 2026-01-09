r[type]
# 类型

r[type.intro]
Rust 程序中的每个变量、项和值都有一个 类型。*值* 的 *类型* 定义了其内存的解释方式以及可在该值上执行的操作。

r[type.builtin]
内置 类型与语言紧密集成，以用户定义 类型无法模拟的非平凡方式。

r[type.user-defined]
用户定义 类型具有有限的能力。

r[type.kinds]
类型 列表如下：

* 原始 类型:
    * [布尔型][Boolean] --- `bool`
    * [数值型][Numeric] --- 整型和浮点型
    * [文本型][Textual] --- `char` 和 `str`
    * [Never 类型][Never] --- `!` --- 一个没有值的 类型
* 序列 类型:
    * [元组][Tuple]
    * [数组][Array]
    * [切片][Slice]
* 用户定义 类型:
    * [结构体][Struct]
    * [枚举][Enum]
    * [联合体][Union]
* 函数 类型:
    * [函数][Functions]
    * [闭包][Closures]
* 指针 类型:
    * [引用][References]
    * [裸指针][Raw pointers]
    * [函数指针][Function pointers]
* 特型 类型:
    * [特型 对象][Trait objects]
    * [Impl 特型][Impl trait]

r[type.name]
## 类型 表达式

r[type.name.syntax]
```grammar,types
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
上述 [Type][Type] 语法规则中定义的 *类型 表达式* 是引用 类型 的 语法格式。它可以引用：

r[type.name.sequence]
* 序列 类型 ([元组][tuple]、[数组][array]、[切片][slice])。

r[type.name.path]
* [类型 路径][Type paths] 可以引用：
    * 原始 类型 ([布尔型][boolean]、[数值型][numeric]、[文本型][textual])。
    * 到某个 [项][item] 的 路径 ([结构体][struct]、[枚举][enum]、[联合体][union]、[类型 别名][type alias]、[特型][trait])。
    * [`Self` 路径][`Self` path]，其中 `Self` 是实现 类型。
    * 泛型 [类型 参数][type parameters]。

r[type.name.pointer]
* 指针 类型 ([引用][reference]、[裸指针][raw pointer]、[函数指针][function pointer])。

r[type.name.inference]
* [推断 类型][inferred type]，它要求编译器确定 类型。

r[type.name.grouped]
* [括号][Parentheses]，用于消除歧义。

r[type.name.trait]
* 特型 类型: [特型 对象][Trait objects] 和 [impl 特型][impl trait]。

r[type.name.never]
* [never 类型][never]。

r[type.name.macro-expansion]
* [宏][Macros]，其会扩展为 类型 表达式。

r[type.name.parenthesized]
### 括号类型

r[type.name.parenthesized.syntax]
```grammar,types
ParenthesizedType -> `(` Type `)`
```

r[type.name.parenthesized.intro]
在某些情况下，类型 的组合可能会产生歧义。在 类型 周围使用括号可以避免歧义。例如，[引用 类型][reference type] 中的 [类型 边界][type boundaries] 的 `+` 运算符不清楚边界应用于何处，因此需要使用括号。需要这种消歧的 语法格式 规则使用 [TypeNoBounds] 规则而不是 [Type][grammar-Type]。

```rust
# use std::any::Any;
type T<'a> = &'a (dyn Any + Send);
```

r[type.recursive]
## 递归类型

r[type.recursive.intro]
名义 类型 —— [结构体][structs]、[枚举][enumerations] 和 [联合体][unions] —— 可能是递归的。也就是说，每个 `enum` 变体或 `struct` 或 `union` 字段可以直接或间接地引用其封闭的 `enum` 或 `struct` 类型 本身。

r[type.recursive.constraint]
这种递归有一些限制：

* 递归 类型 必须在递归中包含一个名义 类型 (而不是仅仅是 [类型 别名][type aliases]，或其他结构 类型，如 [数组][arrays] 或 [元组][tuples])。因此 `type Rec = &'static [Rec]` 是不允许的。
* 递归 类型 的大小必须是有限的；换句话说，该 类型 的递归字段必须是 [指针 类型][pointer types]。

*递归 类型* 及其使用示例：

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
[Raw pointers]: types/pointer.md#raw-pointers-const-and-mut
[References]: types/pointer.md#shared-references-
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
[raw pointer]: types/pointer.md#raw-pointers-const-and-mut
[reference type]: types/pointer.md#shared-references-
[reference]: types/pointer.md#shared-references-
[structs]: types/struct.md
[trait]: types/trait-object.md
[tuples]: types/tuple.md
[type alias]: items/type-aliases.md
[type aliases]: items/type-aliases.md
[type boundaries]: trait-bounds.md
[type parameters]: types/parameters.md
[unions]: types/union.md
