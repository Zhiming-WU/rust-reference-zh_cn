<!-- https://github.com/rust-lang/reference/blob/master/src/items/associated-items.md -->
<!-- commit 68bdfd15fa9feebfbf94a06cf574de598e239198 -->

r[items.associated]
# 关联项

r[items.associated.syntax]
```grammar,项
AssociatedItem ->
    OuterAttribute* (
        MacroInvocationSemi
      | ( Visibility? ( TypeAlias | ConstantItem | Function ) )
    )
```

r[items.associated.intro]
 *关联项 (Associated Items)*  是在 [特型][traits] 中声明或在 [实现][implementations] 中定义的 项。它们之所以被称为关联项，是因为它们定义在一个关联类型上 —— 即实现中的类型。

r[items.associated.kinds]
它们是可以在模块中声明的 项 的一个子集。具体来说，包括 [关联函数][associated functions]（包括方法）、[关联类型][associated types] 和 [关联常量][associated constants]。

[associated functions]: #关联函数和方法
[associated types]: #关联类型
[associated constants]: #关联常量

r[items.associated.related]
当关联项在逻辑上与被关联的 项 相关时，使用关联项非常有用。例如，`Option` 上的 `is_some` 方法本质上与 Options 相关，因此应该被关联。

r[items.associated.decl-def]
每种关联项都有两种形式：包含实际实现的定义，以及为定义声明签名的声明。

r[items.associated.trait-items]
正是这些声明构成了 特型 的契约，以及 泛型 类型上可用的功能。

r[items.associated.fn]
## 关联函数和方法

r[items.associated.fn.intro]
 *关联函数 (Associated functions)*  是与类型关联的 [函数][functions]。

r[items.associated.fn.decl]
 *关联函数声明*  为关联函数定义声明一个签名。它的写法与函数 项 类似，只是函数体被替换为 `;`。

r[items.associated.name]
标识符是函数的名称。

r[items.associated.same-signature]
关联函数的 泛型、参数列表、返回类型和 where 子句必须与关联函数声明的相同。

r[items.associated.fn.def]
 *关联函数定义*  定义了一个与另一个类型关联的函数。它的写法与 [函数项][function item] 相同。

> [!NOTE]
> 一个常见的例子是名为 `new` 的关联函数，它返回一个与之关联的类型的值。

```rust
struct Struct {
    field: i32
}

impl Struct {
    fn new() -> Struct {
        Struct {
            field: 0i32
        }
    }
}

fn main () {
    let _struct = Struct::new();
}
```

r[items.associated.fn.qualified-self]
当关联函数在 特型 上声明时，也可以使用指向该 特型 并在其后追加 特型 名称的 [路径][path] 来调用该函数。当发生这种情况时，它会被替换为 `<_ as Trait>::function_name`。

```rust
trait Num {
    fn from_i32(n: i32) -> Self;
}

impl Num for f64 {
    fn from_i32(n: i32) -> f64 { n as f64 }
}

// 在这种情况下，这 4 种写法都是等价的。
let _: f64 = Num::from_i32(42);
let _: f64 = <_ as Num>::from_i32(42);
let _: f64 = <f64 as Num>::from_i32(42);
let _: f64 = f64::from_i32(42);
```

r[items.associated.fn.method]
### 方法

r[items.associated.fn.method.intro]
第一个参数名为 `self` 的关联函数被称为  *方法 (methods)* ，可以使用 [方法调用运算符][method call operator] 调用，例如 `x.foo()`，也可以使用通常的函数调用 记法。

r[items.associated.fn.method.self-ty]
如果指定了 `self` 参数的类型，则它仅限于解析为由以下语法生成的类型（其中 `'lt` 表示某个任意生命周期）：

```text
P = &'lt S | &'lt mut S | Box<S> | Rc<S> | Arc<S> | Pin<P>
S = Self | P
```

此语法中的 `Self` 终结符表示解析为实现类型的类型。这也可以包括上下文类型别名 `Self`、其他类型别名，或解析为实现类型的关联类型投影。

```rust
# use std::rc::Rc;
# use std::sync::Arc;
# use std::pin::Pin;
// 在结构体 `Example` 上实现的方法示例。
struct Example;
type Alias = Example;
trait Trait { type Output; }
impl Trait for Example { type Output = Example; }
impl Example {
    fn by_value(self: Self) {}
    fn by_ref(self: &Self) {}
    fn by_ref_mut(self: &mut Self) {}
    fn by_box(self: Box<Self>) {}
    fn by_rc(self: Rc<Self>) {}
    fn by_arc(self: Arc<Self>) {}
    fn by_pin(self: Pin<&Self>) {}
    fn explicit_type(self: Arc<Example>) {}
    fn with_lifetime<'a>(self: &'a Self) {}
    fn nested<'a>(self: &mut &'a Arc<Rc<Box<Alias>>>) {}
    fn via_projection(self: <Example as Trait>::Output) {}
}
```

r[associated.fn.method.self-pat-shorthands]
可以使用简写语法而不指定类型，它们具有以下等价形式：

简写                     | 等价形式
----------------------|-----------
`self`                | `self: Self`
`&'lifetime self`     | `self: &'lifetime Self`
`&'lifetime mut self` | `self: &'lifetime mut Self`

> [!NOTE]
> 生命周期可以，且通常在这种简写中被省略。

r[associated.fn.method.self-pat-mut]
如果 `self` 参数带有 `mut` 前缀，它就会变成一个可变变量，类似于使用 `mut` [标识符模式][identifier pattern] 的常规参数。例如：

```rust
trait Changer: Sized {
    fn change(mut self) {}
    fn modify(mut self: Box<Self>) {}
}
```

作为 特型 上方法的示例，请考虑以下内容：

```rust
# type Surface = i32;
# type BoundingBox = i32;
trait Shape {
    fn draw(&self, surface: Surface);
    fn bounding_box(&self) -> BoundingBox;
}
```

这定义了一个具有两个方法的 特型。在 特型 处于作用域内时，所有具有该 特型 [实现][implementations] 的值都可以调用其 `draw` 和 `bounding_box` 方法。

```rust
# type Surface = i32;
# type BoundingBox = i32;
# trait Shape {
#     fn draw(&self, surface: Surface);
#     fn bounding_box(&self) -> BoundingBox;
# }
#
struct Circle {
    // ...
}

impl Shape for Circle {
    // ...
#   fn draw(&self, _: Surface) {}
#   fn bounding_box(&self) -> BoundingBox { 0i32 }
}

# impl Circle {
#     fn new() -> Circle { Circle{} }
# }
#
let circle_shape = Circle::new();
let bounding_box = circle_shape.bounding_box();
```

r[items.associated.fn.params.edition2018]
> [!EDITION-2018]
> 在 2015 版次中，可以使用匿名参数声明 特型 方法（例如 `fn foo(u8)`）。从 2018 版次开始，这种做法已被弃用并被视为错误。所有参数都必须有一个参数名称。

r[items.associated.fn.param-attributes]
#### 方法参数上的属性

方法参数上的属性遵循与 [常规函数参数][regular function parameters] 相同的规则和限制。

r[items.associated.type]
## 关联类型

r[items.associated.type.intro]
 *关联类型 (Associated types)*  是与另一个类型关联的 [类型别名][type aliases]。

r[items.associated.type.restrictions]
关联类型不能在 [固有实现][inherent implementations] 中定义，也不能在 特型 中给出默认实现。

r[items.associated.type.decl]
 *关联类型声明*  为关联类型定义声明一个签名。它的写法有以下形式之一，其中 `Assoc` 是关联类型的名称，`Params` 是以逗号分隔的类型、生命周期或常量参数列表，`Bounds` 是以加号分隔的关联类型必须满足的特型界限列表，而 `WhereBounds` 是以逗号分隔的参数必须满足的界限列表：

<!-- ignore: illustrative example forms -->
```rust,ignore
type Assoc;
type Assoc: Bounds;
type Assoc<Params>;
type Assoc<Params>: Bounds;
type Assoc<Params> where WhereBounds;
type Assoc<Params>: Bounds where WhereBounds;
```

r[items.associated.type.name]
标识符是声明的类型别名的名称。

r[items.associated.type.impl-fulfillment]
可选的特型界限必须由类型别名的实现满足。

r[items.associated.type.sized]
关联类型上有一个隐式的 [`Sized`][`Sized`] 界限，可以使用特殊的 `?Sized` 界限来放宽。

r[items.associated.type.def]
 *关联类型定义*  为 特型 在类型上的实现定义了一个类型别名。

r[items.associated.type.def.restriction]
它们的写法类似于  *关联类型声明* ，但不能包含 `Bounds`，而是必须包含一个 `Type`：

<!-- ignore: illustrative example forms -->
```rust,ignore
type Assoc = Type;
type Assoc<Params> = Type; // 这里的类型 `Type` 可以引用 `Params`
type Assoc<Params> = Type where WhereBounds;
type Assoc<Params> where WhereBounds = Type; // 已弃用，推荐使用上述形式
```

r[items.associated.type.alias]
如果类型 `Item` 具有来自 特型 `Trait` 的关联类型 `Assoc`，那么 `<Item as Trait>::Assoc` 就是一个类型，它是关联类型定义中指定的类型的别名。

r[items.associated.type.param]
此外，如果 `Item` 是一个 类型参数，那么 `Item::Assoc` 可以用在类型参数中。

r[items.associated.type.generic]
关联类型可以包含 [泛型参数][generic parameters] 和 [where 子句][where clauses]；这些通常被称为  *泛型关联类型 (generic associated types)*  或  *GATs* 。如果类型 `Thing` 具有来自 特型 `Trait` 且带有泛型 `<'a>` 的关联类型 `Item`，则该类型可以被命名为 `<Thing as Trait>::Item<'x>`，其中 `'x` 是作用域内的某个生命周期。在这种情况下，在 impl 上的关联类型定义中，凡是出现 `'a` 的地方都会使用 `'x`。

```rust
trait AssociatedType {
    // 关联类型声明
    type Assoc;
}

struct Struct;

struct OtherStruct;

impl AssociatedType for Struct {
    // 关联类型定义
    type Assoc = OtherStruct;
}

impl OtherStruct {
    fn new() -> OtherStruct {
        OtherStruct
    }
}

fn main() {
    // 使用关联类型以 <Struct as AssociatedType>::Assoc 形式引用 OtherStruct
    let _other_struct: OtherStruct = <Struct as AssociatedType>::Assoc::new();
}
```

带有泛型和 where 子句的关联类型示例：

```rust
struct ArrayLender<'a, T>(&'a mut [T; 16]);

trait Lend {
    // 泛型关联类型声明
    type Lender<'a> where Self: 'a;
    fn lend<'a>(&'a mut self) -> Self::Lender<'a>;
}

impl<T> Lend for [T; 16] {
    // 泛型关联类型定义
    type Lender<'a> = ArrayLender<'a, T> where Self: 'a;

    fn lend<'a>(&'a mut self) -> Self::Lender<'a> {
        ArrayLender(self)
    }
}

fn borrow<'a, T: Lend>(array: &'a mut T) -> <T as Lend>::Lender<'a> {
    array.lend()
}

fn main() {
    let mut array = [0usize; 16];
    let lender = borrow(&mut array);
}
```

### 关联类型容器示例

考虑以下 `Container` 特型 的示例。请注意，该类型可在方法签名中使用：

```rust
trait Container {
    type E;
    fn empty() -> Self;
    fn insert(&mut self, elem: Self::E);
}
```

为了让一个类型实现这个 特型，它不仅必须为每个方法提供实现，还必须指定类型 `E`。下面是为标准库类型 `Vec` 实现 `Container` 的示例：

```rust
# trait Container {
#     type E;
#     fn empty() -> Self;
#     fn insert(&mut self, elem: Self::E);
# }
impl<T> Container for Vec<T> {
    type E = T;
    fn empty() -> Vec<T> { Vec::new() }
    fn insert(&mut self, x: T) { self.push(x); }
}
```

### `Bounds`和`WhereBounds`之间的关系

在这个例子中：

```rust
# use std::fmt::Debug;
trait Example {
    type Output<T>: Ord where T: Debug;
}
```

给定关联类型的引用如 `<X as Example>::Output<Y>`，关联类型本身必须是 `Ord`，且类型 `Y` 必须是 `Debug`。

r[items.associated.type.generic-where-clause]
### 泛型关联类型上必需的 where子句

r[items.associated.type.generic-where-clause.intro]
特型 上的泛型关联类型声明目前可能需要一系列 where 子句，这取决于 特型 中的函数以及 GAT 的使用方式。这些规则将来可能会放宽；更新可以在 [泛型关联类型计划仓库 (generic associated types initiative repository)](https://rust-lang.github.io/generic-associated-types-initiative/explainer/required_bounds.html) 中找到。

r[items.associated.type.generic-where-clause.valid-fn]
简而言之，为了最大限度地允许 impl 中关联类型的定义，这些 where 子句是必需的。为此，对于 GAT 作为输入或输出出现的任何函数，凡是 *可以被证明在函数上成立* 的子句（使用函数或 特型 的参数），也必须写在 GAT 自身之上。

```rust
trait LendingIterator {
    type Item<'x> where Self: 'x;
    fn next<'a>(&'a mut self) -> Self::Item<'a>;
}
```

在上面 `next` 函数中，由于 `&'a mut self` 的隐含界限，我们可以证明 `Self: 'a`；因此，我们必须在 GAT 自身上写下等效的界限：`where Self: 'x`。

r[items.associated.type.generic-where-clause.intersection]
当 特型 中有多个函数使用该 GAT 时，将使用来自不同函数的界限的  *交集 (intersection)* ，而不是并集。

```rust
trait Check<T> {
    type Checker<'x>;
    fn create_checker<'a>(item: &'a T) -> Self::Checker<'a>;
    fn do_check(checker: Self::Checker<'_>);
}
```

在这个例子中，`type Checker<'a>;` 上不需要任何界限。虽然我们知道在 `create_checker` 上 `T: 'a`，但在 `do_check` 上我们并不知道这一点。但是，如果 `do_check` 被注释掉，那么 `Checker` 上就需要 `where T: 'x` 界限。

r[items.associated.type.generic-where-clause.forward]
关联类型上的界限也会传播必需的 where 子句。

```rust
trait Iterable {
    type Item<'a> where Self: 'a;
    type Iterator<'a>: Iterator<Item = Self::Item<'a>> where Self: 'a;
    fn iter<'a>(&'a self) -> Self::Iterator<'a>;
}
```

这里，由于 `iter` 的缘故，`Item` 上需要 `where Self: 'a`。然而，由于 `Item` 被用于 `Iterator` 的界限中，`where Self: 'a` 子句在那里也是必需的。

r[items.associated.type.generic-where-clause.static]
最后，在 特型 中的 GAT 上显式使用的任何 `'static` 都不计入必需界限。

```rust
trait StaticReturn {
    type Y<'a>;
    fn foo(&self) -> Self::Y<'static>;
}
```

r[items.associated.const]
## 关联常量

r[items.associated.const.intro]
 *关联常量 (Associated constants)*  是与类型关联的 [常量][constants]。

r[items.associated.const.decl]
 *关联常量声明*  为关联常量定义声明一个签名。它的写法是 `const`，然后是一个标识符，然后是 `:`，然后是一个类型，最后以 `;` 结束。

r[items.associated.const.name]
标识符是在路径中使用的常量的名称。类型是定义必须实现的类型。

r[items.associated.const.def]
 *关联常量定义*  定义了一个与类型关联的常量。它的写法与 [常量项][constant item] 相同。

r[items.associated.const.eval]
关联常量定义仅在被引用时才进行 [常量求值][constant evaluation]。此外，包含 [泛型参数][generic parameters] 的定义会在单态化后进行求值。

```rust,compile_fail
struct Struct;
struct GenericStruct<const ID: i32>;

impl Struct {
    // 定义不会立即求值
    const PANIC: () = panic!("compile-time panic");
}

impl<const ID: i32> GenericStruct<ID> {
    // 定义不会立即求值
    const NON_ZERO: () = if ID == 0 {
        panic!("contradiction")
    };
}

fn main() {
    // 引用 Struct::PANIC 会导致编译错误
    let _ = Struct::PANIC;

    // 正常，ID 不是 0
    let _ = GenericStruct::<1>::NON_ZERO;

    // 求值 ID=0 的 NON_ZERO 时产生编译错误
    let _ = GenericStruct::<0>::NON_ZERO;
}
```

### 关联常量示例

一个基本示例：

```rust
trait ConstantId {
    const ID: i32;
}

struct Struct;

impl ConstantId for Struct {
    const ID: i32 = 1;
}

fn main() {
    assert_eq!(1, Struct::ID);
}
```

使用默认值：

```rust
trait ConstantIdDefault {
    const ID: i32 = 1;
}

struct Struct;
struct OtherStruct;

impl ConstantIdDefault for Struct {}

impl ConstantIdDefault for OtherStruct {
    const ID: i32 = 5;
}

fn main() {
    assert_eq!(1, Struct::ID);
    assert_eq!(5, OtherStruct::ID);
}
```

[`Arc<Self>`]: ../special-types-and-traits.md#arct
[`Box<Self>`]: ../special-types-and-traits.md#boxt
[`Pin<P>`]: ../special-types-and-traits.md#pinp
[`Rc<Self>`]: ../special-types-and-traits.md#rct
[`Sized`]: ../special-types-and-traits.md#sized
[traits]: traits.md
[type aliases]: type-aliases.md
[inherent implementations]: implementations.md#固有实现
[identifier]: ../identifiers.md
[identifier pattern]: ../patterns.md#标识符模式
[implementations]: implementations.md
[type]: ../types.md#类型表达式
[constants]: constant-items.md
[constant item]: constant-items.md
[functions]: functions.md
[function item]: ../types/function-item.md
[method call operator]: ../expressions/method-call-expr.md
[path]: ../paths.md
[regular function parameters]: functions.md#函数参数上的属性
[generic parameters]: generics.md
[where clauses]: generics.md#where子句
[constant evaluation]: ../const_eval.md
