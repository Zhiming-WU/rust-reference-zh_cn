<!-- https://github.com/rust-lang/reference/blob/master/src/paths.md -->
<!-- commit 68bdfd15fa9feebfbf94a06cf574de598e239198 -->

r[paths]
# 路径

r[paths.intro]
一个 *路径* 是由 `::` 词法单元分隔的一个或多个路径段组成的序列。路径用于引用 [项][items]、值、 [类型][types]、 [宏][macros] 和 [属性][attributes]。

两个仅由标识符段组成的简单路径示例：

<!-- ignore: syntax fragment -->
```rust,ignore
x;
x::y::z;
```

## 路径的类型

r[paths.simple]
### 简单路径

r[paths.simple.syntax]
```grammar,路径
SimplePath ->
    `::`? SimplePathSegment (`::` SimplePathSegment)*

SimplePathSegment ->
    IDENTIFIER | `super` | `self` | `crate` | `$crate`
```

r[paths.simple.intro]
简单路径用于 [可见性][visibility] 标记、 [属性][attributes]、 [声明宏][mbe] 和 [`use`] 项。例如：

```rust
use std::io::{self, Write};
mod m {
    #[clippy::cyclomatic_complexity = "0"]
    pub (in super) fn f1() {}
}
```

r[paths.expr]
### 表达式中的路径

r[paths.expr.syntax]
```grammar,路径
PathInExpression ->
    `::`? PathExprSegment (`::` PathExprSegment)*

PathExprSegment ->
    PathIdentSegment (`::` GenericArgs)?

PathIdentSegment ->
    IDENTIFIER | `super` | `self` | `Self` | `crate` | `$crate`

GenericArgs ->
      `<` `>`
    | `<` ( GenericArg `,` )* GenericArg `,`? `>`

GenericArg ->
    Lifetime | Type | GenericArgsConst | GenericArgsBinding | GenericArgsBounds

GenericArgsConst ->
      BlockExpression
    | LiteralExpression
    | `-` LiteralExpression
    | SimplePathSegment

GenericArgsBinding ->
    IDENTIFIER GenericArgs? `=` Type

GenericArgsBounds ->
    IDENTIFIER GenericArgs? `:` TypeParamBounds
```

r[paths.expr.intro]
表达式中的路径允许指定带有泛型参数的路径。它们被用于 [表达式][expressions] 和 [模式][patterns] 的各个地方。

r[paths.expr.turbofish]
在泛型参数的开口 `<` 之前需要 `::` 词法单元，以避免与小于运算符产生歧义。这被通俗地称为 "turbofish" 语法格式。

```rust
(0..10).collect::<Vec<_>>();
Vec::<u8>::with_capacity(1024);
```

r[paths.expr.argument-order]
泛型参数的顺序限制为生命周期参数，然后是类型参数，接着是常量参数，最后是等值约束。

r[paths.expr.complex-const-params]
常量参数必须用大括号括起来，除非它们是 [字面量][literal]、 [推断常量][inferred const] 或单段路径。 [推断常量][inferred const] 不得用大括号括起来。

```rust
mod m {
    pub const C: usize = 1;
}
const C: usize = m::C;
fn f<const N: usize>() -> [u8; N] { [0; N] }

let _ = f::<1>(); // 字面量。
let _: [_; 1] = f::<_>(); // 推断常量。
let _: [_; 1] = f::<(((_)))>(); // 推断常量。
let _ = f::<C>(); // 单段路径。
let _ = f::<{ m::C }>(); // 多段路径必须用大括号包围。
```

```rust,compile_fail
fn f<const N: usize>() -> [u8; N] { [0; _] }
let _: [_; 1] = f::<{ _ }>();
//                    ^ ERROR `_` not allowed here
```

> [!NOTE]
> 在泛型参数列表中， [推断常量][inferred const] 被解析为 [推断类型][InferredType]，但在语义上被视为一种单独的 [常量泛型参数][const generic argument]。

r[paths.expr.impl-trait-params]
对应于 `impl Trait` 类型的合成类型参数是隐式的，不能显式指定。

r[paths.qualified]
## 限定路径

r[paths.qualified.syntax]
```grammar,路径
QualifiedPathInExpression -> QualifiedPathType (`::` PathExprSegment)+

QualifiedPathType -> `<` Type (`as` TypePath)? `>`

QualifiedPathInType -> QualifiedPathType (`::` TypePathSegment)+
```

r[paths.qualified.intro]
全限定路径允许消除 [特型实现][trait implementations] 的路径歧义，并用于指定 [规范路径](#canonical-paths)。当用于类型规范时，它支持使用下面指定的类型语法格式。

```rust
struct S;
impl S {
    fn f() { println!("S"); }
}
trait T1 {
    fn f() { println!("T1 f"); }
}
impl T1 for S {}
trait T2 {
    fn f() { println!("T2 f"); }
}
impl T2 for S {}
S::f();  // 调用固有实现。
<S as T1>::f();  // 调用 T1 特型函数。
<S as T2>::f();  // 调用 T2 特型函数。
```

r[paths.type]
### 类型中的路径

r[paths.type.syntax]
```grammar,路径
TypePath -> `::`? TypePathSegment (`::` TypePathSegment)*

TypePathSegment -> PathIdentSegment (`::`? (GenericArgs | TypePathFn))?

TypePathFn -> `(` TypePathFnInputs? `)` (`->` TypeNoBounds)?

TypePathFnInputs -> Type (`,` Type)* `,`?
```

r[paths.type.intro]
类型路径用于类型定义、特型界限、类型参数界限和限定路径中。

r[paths.type.turbofish]
虽然在泛型参数之前允许使用 `::` 词法单元，但它不是必需的，因为不像在 [PathInExpression] 中那样存在歧义。

```rust
# mod ops {
#     pub struct Range<T> {f1: T}
#     pub trait Index<T> {}
#     pub struct Example<'a> {f1: &'a i32}
# }
# struct S;
impl ops::Index<ops::Range<usize>> for S { /*...*/ }
fn i<'a>() -> impl Iterator<Item = ops::Example<'a>> {
    // ...
#    const EXAMPLE: Vec<ops::Example<'static>> = Vec::new();
#    EXAMPLE.into_iter()
}
type G = std::boxed::Box<dyn std::ops::FnOnce(isize) -> isize>;
```

r[paths.qualifiers]
## 路径限定符

路径可以用各种前导限定符来表示，以改变其解析方式的含义。

r[paths.qualifiers.global-root]
### `::`

r[paths.qualifiers.global-root.intro]
以 `::` 开头的路径被认为是 *全局路径*，其路径段的解析起始位置因版次而异。路径中的每个标识符必须解析为一个项。

r[paths.qualifiers.global-root.edition2018]
> [!EDITION-2018]
> 在 2015 版次中，标识符从 "crate 根"（2018 版次中的 `crate::`）开始解析，它包含各种不同的项，包括外部 crate、默认 crate（如 `std` 或 `core`）以及 crate 顶层的项（包括 `use` 导入）。
>
> 从 2018 版次开始，以 `::` 开头的路径从 [外部预导入][extern prelude] 中的 crate 开始解析。也就是说，它们后面必须紧跟一个 crate 的名称。

```rust
pub fn foo() {
    // 在 2018 版次中，这通过外部预导入访问 `std`。
    // 在 2015 版次中，这通过 crate 根访问 `std`。
    let now = ::std::time::Instant::now();
    println!("{:?}", now);
}
```

```rust,edition2015
// 2015 版次
mod a {
    pub fn foo() {}
}
mod b {
    pub fn foo() {
        ::a::foo(); // 调用 a 的 foo 函数
        // 在 Rust 2018 中，`::a` 将被解释为 crate `a`。
    }
}
# fn main() {}
```

r[paths.qualifiers.mod-self]
### `self`

r[paths.qualifiers.mod-self.intro]
`self` 相对于当前模块解析路径。

r[paths.qualifiers.mod-self.restriction]
`self` 只能用作第一段，且前面不能有 `::`。

r[paths.qualifiers.self-pat]
在方法体中，仅由单个 `self` 段组成的路径解析为该方法的 self 参数。

```rust
fn foo() {}
fn bar() {
    self::foo();
}
struct S(bool);
impl S {
  fn baz(self) {
        self.0;
    }
}
# fn main() {}
```

r[paths.qualifiers.type-self]
### `Self`

r[paths.qualifiers.type-self.intro]
首字母大写的 `Self` 用于引用当前正在实现或定义的类型。它可以用于以下情况：

r[paths.qualifiers.type-self.trait]
* 在 [特型][trait] 定义中，它引用实现该特型的类型。

r[paths.qualifiers.type-self.impl]
* 在 [实现][implementation] 中，它引用正在被实现的类型。
  在实现元组或单元 [结构体][struct] 时，它还在 [值命名空间][value namespace] 中引用构造函数。

r[paths.qualifiers.type-self.type]
* 在 [结构体][struct]、 [枚举][enumeration] 或 [联合体][union] 的定义中，它引用正在被定义的类型。
  定义不允许无限递归（必须有间接寻址）。

r[paths.qualifiers.type-self.scope]
`Self` 的作用域行为类似于泛型参数；有关更多详细信息，请参阅 [`Self` 作用域][`Self` scope] 部分。

r[paths.qualifiers.type-self.allowed-positions]
`Self` 只能用作第一段，且前面不能有 `::`。

r[paths.qualifiers.type-self.no-generics]
`Self` 路径不能包含泛型参数（如 `Self::<i32>`）。

```rust
trait T {
    type Item;
    const C: i32;
    // `Self` 将是实现 `T` 的任何类型。
    fn new() -> Self;
    // `Self::Item` 将是实现中的类型别名。
    fn f(&self) -> Self::Item;
}
struct S;
impl T for S {
    type Item = i32;
    const C: i32 = 9;
    fn new() -> Self {           // `Self` 是类型 `S`。
        S
    }
    fn f(&self) -> Self::Item {  // `Self::Item` 是类型 `i32`。
        Self::C                  // `Self::C` 是常量值 `9`。
    }
}

// `Self` 在特型定义的泛型作用域内，
// 用以引用正在定义的类型。
trait Add<Rhs = Self> {
    type Output;
    // `Self` 也可以引用
    // 正在实现的类型的关联项。
    fn add(self, rhs: Rhs) -> Self::Output;
}

struct NonEmptyList<T> {
    head: T,
    // 结构体可以引用自身（只要它不是
    // 无限递归的）。
    tail: Option<Box<Self>>,
}
```

r[paths.qualifiers.super]
### `super`

r[paths.qualifiers.super.intro]
路径中的 `super` 解析为父模块。

r[paths.qualifiers.super.allowed-positions]
它只能用于路径的前导段，可能在初始的 `self` 段之后。

```rust
mod a {
    pub fn foo() {}
}
mod b {
    pub fn foo() {
        super::a::foo(); // 调用 a 的 foo 函数
    }
}
# fn main() {}
```

r[paths.qualifiers.super.repetition]
`super` 可以在第一个 `super` 或 `self` 之后重复多次，以引用祖先模块。

```rust
mod a {
    fn foo() {}

    mod b {
        mod c {
            fn foo() {
                super::super::foo(); // 调用 a 的 foo 函数
                self::super::super::foo(); // 调用 a 的 foo 函数
            }
        }
    }
}
# fn main() {}
```

r[paths.qualifiers.crate]
### `crate`

r[paths.qualifiers.crate.intro]
`crate` 相对于当前 crate 解析路径。

r[paths.qualifiers.crate.allowed-positions]
`crate` 只能用作第一段，且前面不能有 `::`。

```rust
fn foo() {}
mod a {
    fn bar() {
        crate::foo();
    }
}
# fn main() {}
```

r[paths.qualifiers.macro-crate]
### `$crate`

r[paths.qualifiers.macro-crate.allowed-positions]
[`$crate`] 仅在 [宏转录器][macro transcribers] 中使用，并且只能用作第一段，且前面不能有 `::`。

r[paths.qualifiers.macro-crate.hygiene]
[`$crate`] 将展开为访问宏定义所在 crate 顶层项的路径，无论宏在哪个 crate 中被调用。

```rust
pub fn increment(x: u32) -> u32 {
    x + 1
}

#[macro_export]
macro_rules! inc {
    ($x:expr) => ( $crate::increment($x) )
}
# fn main() { }
```

r[paths.canonical]
## 规范路径

r[paths.canonical.intro]
在模块或实现中定义的项具有一个 *规范路径*，该路径对应于它在其 crate 中定义的位置。

r[paths.canonical.alias]
指向这些项的所有其他路径都是别名。

r[paths.canonical.def]
规范路径被定义为一个 *路径前缀* 加上项本身定义的路径段。

r[paths.canonical.non-canonical]
[实现][implementations] 和 [使用声明][use declarations] 没有规范路径，尽管实现定义的项确实有规范路径。在块表达式中定义的项没有规范路径。在没有规范路径的模块中定义的项没有规范路径。在引用了没有规范路径的项（例如作为实现类型、正在实现的特型、类型参数或类型参数界限）的实现中定义的关联项，没有规范路径。

r[paths.canonical.module-prefix]
模块的路径前缀是该模块的规范路径。

r[paths.canonical.bare-impl-prefix]
对于固有实现，它是正在实现的项的规范路径，并用 <span class="parenthetical">尖括号 (`<>`)</span> 包围。

r[paths.canonical.trait-impl-prefix]
对于 [特型实现][trait implementations]，它是正在实现的项的规范路径，后跟 `as`，再后跟该特型的规范路径，全部用 <span class="parenthetical">尖括号 (`<>`)</span> 包围。

r[paths.canonical.local-canonical-path]
规范路径仅在给定的 crate 内有意义。不存在跨 crate 的全局命名空间；项的规范路径仅在其 crate 内标识它。

```rust
// 注释显示了项的规范路径。

mod a { // crate::a
    pub struct Struct; // crate::a::Struct

    pub trait Trait { // crate::a::Trait
        fn f(&self); // crate::a::Trait::f
    }

    impl Trait for Struct {
        fn f(&self) {} // <crate::a::Struct as crate::a::Trait>::f
    }

    impl Struct {
        fn g(&self) {} // <crate::a::Struct>::g
    }
}

mod without { // crate::without
    fn canonicals() { // crate::without::canonicals
        struct OtherStruct; // 无

        trait OtherTrait { // 无
            fn g(&self); // 无
        }

        impl OtherTrait for OtherStruct {
            fn g(&self) {} // 无
        }

        impl OtherTrait for crate::a::Struct {
            fn g(&self) {} // 无
        }

        impl crate::a::Trait for OtherStruct {
            fn f(&self) {} // 无
        }
    }
}

# fn main() {}
```

[`$crate`]: macro.decl.hygiene.crate
[implementations]: items/implementations.md
[items]: items.md
[literal]: expressions/literal-expr.md
[use declarations]: items/use-declarations.md
[`Self` scope]: names/scopes.md#self作用域
[`use`]: items/use-declarations.md
[attributes]: attributes.md
[const generic argument]: items.generics.const.argument
[enumeration]: items/enumerations.md
[expressions]: expressions.md
[extern prelude]: names/preludes.md#外部预导入
[implementation]: items/implementations.md
[inferred const]: items.generics.const.inferred
[macro transcribers]: macros-by-example.md
[macros]: macros.md
[mbe]: macros-by-example.md
[patterns]: patterns.md
[struct]: items/structs.md
[trait implementations]: items/implementations.md#特型实现
[trait]: items/traits.md
[traits]: items/traits.md
[types]: types.md
[union]: items/unions.md
[value namespace]: names/namespaces.md
[visibility]: visibility-and-privacy.md
