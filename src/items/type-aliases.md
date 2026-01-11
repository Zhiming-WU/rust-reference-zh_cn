<!-- https://github.com/rust-lang/reference/blob/master/src/items/type-aliases.md -->
<!-- commit 68bdfd15fa9feebfbf94a06cf574de598e239198 -->

r[items.type]
# 类型别名

r[items.type.syntax]
```grammar,项
TypeAlias ->
    `type` IDENTIFIER GenericParams? ( `:` TypeParamBounds )?
        WhereClause?
        ( `=` Type WhereClause?)? `;`
```

r[items.type.intro]
 *类型别名*  为其所在的模块或代码块的  [类型命名空间][type namespace]  中的现有  [类型][type]  定义一个新名称。
类型别名使用关键字 `type` 声明。
每个值都有一个单一、具体的类型，但可能实现几个不同的 特型，并且可能与几种不同的类型约束兼容。

例如，下面将类型 `Point` 定义为类型 `(u8, u8)`（即 8 位无符号整数对的类型）的同义词：

```rust
type Point = (u8, u8);
let p: Point = (41, 68);
```

r[items.type.constructor-alias]
指向 元组结构体 或 单元结构体 的类型别名不能用于限定该类型的构造函数：

```rust,compile_fail
struct MyStruct(u32);

use MyStruct as UseAlias;
type TypeAlias = MyStruct;

let _ = UseAlias(5); // 正常
let _ = TypeAlias(5); // 无法工作
```

r[items.type.associated-type]
当类型别名不作为  [关联类型][associated type]  使用时，必须包含一个  [类型][grammar-Type]  且不得包含 [TypeParamBounds]。

r[items.type.associated-trait]
当类型别名在  [特型][trait]  中作为  [关联类型][associated type]  使用时，不得包含  [类型][grammar-Type]  规范，但可以包含 [TypeParamBounds]。

r[items.type.associated-impl]
当类型别名在  [特型实现][trait impl]  中作为  [关联类型][associated type]  使用时，必须包含一个  [类型][grammar-Type]  规范，且不得包含 [TypeParamBounds]。

r[items.type.deprecated]
在  [特型实现][trait impl]  的类型别名中，等号之前的 Where 子句（如 `type TypeAlias<T> where T: Foo = Bar<T>`）已被弃用。等号之后的 Where 子句（如 `type TypeAlias<T> = Bar<T> where T: Foo`）是首选。

[associated type]: associated-items.md#关联类型
[trait impl]: implementations.md#特型实现
[trait]: traits.md
[type namespace]: ../names/namespaces.md
[type]: ../types.md
