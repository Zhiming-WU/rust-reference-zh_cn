<!-- https://github.com/rust-lang/reference/blob/master/src/items/traits.md -->
<!-- commit 68bdfd15fa9feebfbf94a06cf574de598e239198 -->

r[items.traits]
# 特型

r[items.traits.syntax]
```grammar,项
Trait ->
    `unsafe`? `trait` IDENTIFIER GenericParams? ( `:` TypeParamBounds? )? WhereClause?
    `{`
        InnerAttribute*
        AssociatedItem*
    `}`
```

r[items.traits.intro]
一个 _特型_ 描述了一个类型可以实现的抽象接口。此接口由 [关联项][associated items] 组成，共有三种：

- [函数](associated-items.md#关联函数和方法)
- [类型](associated-items.md#关联类型)
- [常量](associated-items.md#关联常量)

r[items.traits.namespace]
特型 声明在它所在的模块或块的 [类型命名空间][type namespace] 中定义了一个 特型。

r[items.traits.associated-item-namespaces]
关联项 在其各自的命名空间中被定义为 特型 的成员。关联类型 定义在 类型命名空间 中。关联常量 和 关联函数 定义在 值命名空间 中。

r[items.traits.self-param]
所有 特型 都定义了一个隐式的 类型参数 `Self`，它指向 “实现此接口的类型” 。 特型 还可以包含额外的 类型参数。这些 类型参数（包括 `Self`）可以像 [往常一样][generics] 受到其他 特型 等的约束。

r[items.traits.impls]
特型 通过单独的 [实现][implementations] 为特定类型实现。

r[items.traits.associated-item-decls]
特型 函数可以通过用分号替换函数体来省略它。这表示 实现 必须定义该函数。如果 特型 函数定义了函数体，则该定义对于任何未覆盖它的 实现 都充当默认值。类似地，关联常量 可以省略等号和表达式，以表示 实现 必须定义常量值。 关联类型 绝不能定义类型，类型只能在 实现 中指定。

```rust
// 具有定义和不具有定义的特型关联项示例。
trait Example {
    const CONST_NO_DEFAULT: i32;
    const CONST_WITH_DEFAULT: i32 = 99;
    type TypeNoDefault;
    fn method_without_default(&self);
    fn method_with_default(&self) {}
}
```

r[items.traits.const-fn]
特型 函数不允许是 [`const`]。

r[items.traits.bounds]
## 特型界限

泛型 项 可以使用特型作为其类型参数的 [界限][bounds] 。

r[items.traits.generic]
## 泛型特型

可以为 特型 指定 类型参数 以使其成为 泛型。这些出现在 特型 名称之后，使用与 [泛型函数][generic functions] 相同的语法。

```rust
trait Seq<T> {
    fn len(&self) -> u32;
    fn elt_at(&self, n: u32) -> T;
    fn iter<F>(&self, f: F) where F: Fn(T);
}
```

<a id="object-safety"></a>
r[items.traits.dyn-compatible]
## Dyn兼容性

r[items.traits.dyn-compatible.intro]
一个 dyn 兼容的 特型 可以作为 [特型对象][trait object] 的基础 特型。如果一个 特型 具备以下特质，则它是 *dyn 兼容的 (dyn compatible)* ：

r[items.traits.dyn-compatible.supertraits]
* 所有 [父特型][supertraits] 也必须是 dyn 兼容的。

r[items.traits.dyn-compatible.sized]
* `Sized` 不能是一个 [父特型][supertraits]。换句话说，它不能要求 `Self: Sized`。

r[items.traits.dyn-compatible.associated-consts]
* 它不能有任何 关联常量。

r[items.traits.dyn-compatible.associated-types]
* 它不能有任何带有 泛型 的 关联类型。

r[items.traits.dyn-compatible.associated-functions]
* 所有 关联函数 必须要么可以从 特型对象 分派，要么是明确的不可分派：
    * 可分派函数必须：
        * 不具有任何 类型参数（尽管允许生命周期参数）。
        * 是一个除了在 接收者 类型中之外不使用 `Self` 的 [方法][method]。
        * 拥有以下类型之一的 接收者：
            * `&Self` (即 `&self`)
            * `&mut Self` (即 `&mut self`)
            * [`Box<Self>`]
            * [`Rc<Self>`]
            * [`Arc<Self>`]
            * [`Pin<P>`] 其中 `P` 是上述类型之一
        * 不具有 不透明返回类型；即，
            * 不是 `async fn`（它具有隐藏的 `Future` 类型）。
            * 不具有 返回位置 `impl Trait` 类型 (`fn example(&self) -> impl Trait`)。
        * 不具有 `where Self: Sized` 界限（`Self` 的 接收者 类型 (即 `self`) 隐含了这一点）。
    * 明确的不可分派函数要求：
        * 具有 `where Self: Sized` 界限（`Self` 的 接收者 类型 (即 `self`) 隐含了这一点）。

r[items.traits.dyn-compatible.async-traits]
* [`AsyncFn`]、[`AsyncFnMut`] 和 [`AsyncFnOnce`] 特型 不是 dyn 兼容的。

> [!NOTE]
> 这个概念以前被称为 *对象安全 (object safety)* 。

```rust
# use std::rc::Rc;
# use std::sync::Arc;
# use std::pin::Pin;
// dyn 兼容方法的示例。
trait TraitMethods {
    fn by_ref(self: &Self) {}
    fn by_ref_mut(self: &mut Self) {}
    fn by_box(self: Box<Self>) {}
    fn by_rc(self: Rc<Self>) {}
    fn by_arc(self: Arc<Self>) {}
    fn by_pin(self: Pin<&Self>) {}
    fn with_lifetime<'a>(self: &'a Self) {}
    fn nested_pin(self: Pin<Arc<Self>>) {}
}
# struct S;
# impl TraitMethods for S {}
# let t: Box<dyn TraitMethods> = Box::new(S);
```

```rust,compile_fail
// 此特型是 dyn 兼容的，但这些方法不能在特型对象上分派。
trait NonDispatchable {
    // 非方法不能被分派。
    fn foo() where Self: Sized {}
    // Self 类型在运行时才已知。
    fn returns(&self) -> Self where Self: Sized;
    // `other` 可能是接收者的不同具体类型。
    fn param(&self, other: Self) where Self: Sized {}
    // 泛型与虚表 (vtable) 不兼容。
    fn typed<T>(&self, x: T) where Self: Sized {}
}

struct S;
impl NonDispatchable for S {
    fn returns(&self) -> Self where Self: Sized { S }
}
let obj: Box<dyn NonDispatchable> = Box::new(S);
obj.returns(); // 错误：不能通过 Self 返回值调用
obj.param(S);  // 错误：不能通过 Self 参数调用
obj.typed(1);  // 错误：不能通过泛型类型调用
```

```rust,compile_fail
# use std::rc::Rc;
// dyn 不兼容特型的示例。
trait DynIncompatible {
    const CONST: i32 = 1;  // 错误：不能拥有关联常量

    fn foo() {}  // 错误：没有 Sized 的关联函数
    fn returns(&self) -> Self; // 错误：返回类型中存在 Self
    fn typed<T>(&self, x: T) {} // 错误：具有泛型类型参数
    fn nested(self: Rc<Box<Self>>) {} // 错误：嵌套接收者不能被分派
}

struct S;
impl DynIncompatible for S {
    fn returns(&self) -> Self { S }
}
let obj: Box<dyn DynIncompatible> = Box::new(S); // 错误
```

```rust,compile_fail
// `Self: Sized` 特型是 dyn 不兼容的。
trait TraitWithSize where Self: Sized {}

struct S;
impl TraitWithSize for S {}
let obj: Box<dyn TraitWithSize> = Box::new(S); // 错误
```

```rust,compile_fail
// 如果 `Self` 是一个类型参数，则 dyn 不兼容。
trait Super<A> {}
trait WithSelf: Super<Self> where Self: Sized {}

struct S;
impl<A> Super<A> for S {}
impl WithSelf for S {}
let obj: Box<dyn WithSelf> = Box::new(S); // 错误：不能使用 `Self` 类型参数
```

r[items.traits.supertraits]
## 父特型

r[items.traits.supertraits.intro]
**父特型 (Supertraits)** 是指为了让某个类型实现特定 特型 而必须先为该类型实现的 特型。此外，任何被 特型 限界的 [泛型][generics] 或 [特型对象][trait object] 都可以访问其 父特型 的 关联项。

r[items.traits.supertraits.decl]
父特型 通过 特型 的 `Self` 类型上的 特型界限 来声明，并以此类推，包括那些 特型界限 中声明的 特型 的 父特型。 特型 成为其自身的 父特型 是一个错误。

r[items.traits.supertraits.subtrait]
拥有 父特型 的 特型 被称为其 父特型 的 **子特型 (subtrait)** 。

以下是声明 `Shape` 为 `Circle` 的 父特型 的示例。

```rust
trait Shape { fn area(&self) -> f64; }
trait Circle: Shape { fn radius(&self) -> f64; }
```

以下是相同的示例，只是使用了 [where 子句][where clauses]。

```rust
trait Shape { fn area(&self) -> f64; }
trait Circle where Self: Shape { fn radius(&self) -> f64; }
```

下一个示例使用来自 `Shape` 的 `area` 函数为 `radius` 提供默认实现。

```rust
# trait Shape { fn area(&self) -> f64; }
trait Circle where Self: Shape {
    fn radius(&self) -> f64 {
        // A = pi * r^2
        // 所以在代数上，
        // r = sqrt(A / pi)
        (self.area() / std::f64::consts::PI).sqrt()
    }
}
```

下一个示例在 泛型 参数上调用 父特型 方法。

```rust
# trait Shape { fn area(&self) -> f64; }
# trait Circle: Shape { fn radius(&self) -> f64; }
fn print_area_and_radius<C: Circle>(c: C) {
    // 这里我们调用来自 `Circle` 的父特型 `Shape` 的 area 方法。
    println!("Area: {}", c.area());
    println!("Radius: {}", c.radius());
}
```

类似地，这里是一个在 特型对象 上调用 父特型 方法的示例。

```rust
# trait Shape { fn area(&self) -> f64; }
# trait Circle: Shape { fn radius(&self) -> f64; }
# struct UnitCircle;
# impl Shape for UnitCircle { fn area(&self) -> f64 { std::f64::consts::PI } }
# impl Circle for UnitCircle { fn radius(&self) -> f64 { 1.0 } }
# let circle = UnitCircle;
let circle = Box::new(circle) as Box<dyn Circle>;
let nonsense = circle.radius() * circle.area();
```

r[items.traits.safety]
## 不安全特型

r[items.traits.safety.intro]
以 `unsafe` 关键字开头的 特型 项 表示 *实现* 该 特型 可能是 [不安全][unsafe] 的。使用正确实现的 不安全特型 是安全的。 [特型实现][trait implementation] 也必须以 `unsafe` 关键字开头。

[`Sync`] 和 [`Send`] 是 不安全特型 的示例。

r[items.traits.params]
## 参数模式

r[items.traits.params.patterns-no-body]
在没有函数体的 关联函数 中，参数仅允许 [标识符][IDENTIFIER] 或 `_` [通配符模式][WildcardPattern]，以及 [Self 参数][SelfParam] 允许的形式。 `mut` [标识符][IDENTIFIER] 目前是允许的，但它已被弃用，将来会变成一个硬错误。
<!-- https://github.com/rust-lang/rust/issues/35203 -->

```rust
trait T {
    fn f1(&self);
    fn f2(x: Self, _: i32);
}
```

```rust,compile_fail,E0642
trait T {
    fn f2(&x: &i32); // 错误：在没有函数体的函数中不允许使用模式
}
```

r[items.traits.params.patterns-with-body]
带有函数体的 关联函数 中的参数仅允许 不可驳模式。

```rust
trait T {
    fn f1((a, b): (i32, i32)) {} // OK：是不可驳模式
}
```

```rust,compile_fail,E0005
trait T {
    fn f1(123: i32) {} // 错误：模式是可驳的
    fn f2(Some(x): Option<i32>) {} // 错误：模式是可驳的
}
```

r[items.traits.params.pattern-required.edition2018]
> [!EDITION-2018]
> 在 2018 版次 之前，关联函数参数的模式是可选的：
>
> ```rust,edition2015
> // 2015 版次
> trait T {
>     fn f(i32); // OK：参数标识符不是必需的
> }
> ```
>
> 从 2018 版次 开始，模式不再是可选的。

r[items.traits.params.restriction-patterns.edition2018]
> [!EDITION-2018]
> 在 2018 版次 之前，带有函数体的 关联函数 中的参数仅限于以下几种 模式：
>
> * [标识符][IDENTIFIER]
> * `mut` [标识符][IDENTIFIER]
> * [`_`][WildcardPattern]
> * `&` [标识符][IDENTIFIER]
> * `&&` [标识符][IDENTIFIER]
>
> ```rust,edition2015,compile_fail,E0642
> // 2015 版次
> trait T {
>     fn f1((a, b): (i32, i32)) {} // 错误：不允许使用该模式
> }
> ```
>
> 从 2018 开始，所有不可驳模式都是允许的，如 [items.traits.params.patterns-with-body] 中所述。

r[items.traits.associated-visibility]
## 项可见性

r[items.traits.associated-visibility.intro]
特型项 在语法格式上允许 [可见性][Visibility] 注解，但在验证 特型 时会被拒绝。这允许 项 在使用它们的不同上下文中以统一的语法格式进行解析。例如，空的 `vis` 宏片段说明符可以用于 特型项，其中该宏规则可以用于允许可见性的其他情况。

```rust
macro_rules! create_method {
    ($vis:vis $name:ident) => {
        $vis fn $name(&self) {}
    };
}

trait T1 {
    // 允许空的 `vis`。
    create_method! { method_of_t1 }
}

struct S;

impl S {
    // 这里允许可见性。
    create_method! { pub method_of_s }
}

impl T1 for S {}

fn main() {
    let s = S;
    s.method_of_t1();
    s.method_of_s();
}
```

[WildcardPattern]: ../patterns.md#通配符模式
[bounds]: ../trait-bounds.md
[trait object]: ../types/trait-object.md
[associated items]: associated-items.md
[method]: associated-items.md#方法
[supertraits]: #父特型
[implementations]: implementations.md
[generics]: generics.md
[where clauses]: generics.md#where子句
[generic functions]: functions.md#泛型函数
[unsafe]: ../unsafety.md
[trait implementation]: implementations.md#特型实现
[`Send`]: ../special-types-and-traits.md#send
[`Sync`]: ../special-types-and-traits.md#sync
[`Arc<Self>`]: ../special-types-and-traits.md#arct
[`Box<Self>`]: ../special-types-and-traits.md#boxt
[`Pin<P>`]: ../special-types-and-traits.md#pinp
[`Rc<Self>`]: ../special-types-and-traits.md#rct
[`async`]: functions.md#async函数
[`const`]: functions.md#const函数
[type namespace]: ../names/namespaces.md
