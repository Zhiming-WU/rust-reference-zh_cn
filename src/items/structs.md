r[items.struct]
# 结构体

r[items.struct.syntax]
```grammar,items
Struct ->
      StructStruct
    | TupleStruct

StructStruct ->
    `struct` IDENTIFIER GenericParams? WhereClause? ( `{` StructFields? `}` | `;` )

TupleStruct ->
    `struct` IDENTIFIER GenericParams? `(` TupleFields? `)` WhereClause? `;`

StructFields -> StructField (`,` StructField)* `,`?

StructField -> OuterAttribute* Visibility? IDENTIFIER `:` Type

TupleFields -> TupleField (`,` TupleField)* `,`?

TupleField -> OuterAttribute* Visibility? Type
```

r[items.struct.intro]
 *结构体*  是一个名义  [结构体类型][struct type] ，使用关键字 `struct` 定义。

r[items.struct.namespace]
结构体声明在其所在的模块或代码块的  [类型命名空间][type namespace]  中定义给定的名称。

一个  `struct`  项  及其用法的示例：

```rust
struct Point {x: i32, y: i32}
let p = Point {x: 10, y: 11};
let px: i32 = p.x;
```

r[items.struct.tuple]
 *元组结构体*  是一个名义  [元组类型][tuple type] ，也是使用关键字 `struct` 定义的。
除了定义类型外，它还在  [值命名空间][value namespace]  中定义了一个同名的构造函数。
构造函数是一个可以调用来创建该 结构体 新实例的函数。
例如：

```rust
struct Point(i32, i32);
let p = Point(10, 11);
let px: i32 = match p { Point(x, _) => x };
```

r[items.struct.unit]
 *类单元结构体*  是不包含任何字段的 结构体，通过完全省略字段列表来定义。此类 结构体 隐式地定义了一个与其类型同名的  [常量][constant] 。例如：

```rust
struct Cookie;
let c = [Cookie, Cookie {}, Cookie, Cookie {}];
```

等价于

```rust
struct Cookie {}
const Cookie: Cookie = Cookie {};
let c = [Cookie, Cookie {}, Cookie, Cookie {}];
```

r[items.struct.layout]
结构体 的精确内存布局并未指定。可以使用  [`repr` 属性][`repr` attribute]  指定特定的布局。

[`repr` attribute]: ../type-layout.md#representations
[constant]: constant-items.md
[struct type]: ../types/struct.md
[tuple type]: ../types/tuple.md
[type namespace]: ../names/namespaces.md
[value namespace]: ../names/namespaces.md
