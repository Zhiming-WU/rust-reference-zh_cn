<!-- https://github.com/rust-lang/reference/blob/master/src/trait-bounds.md -->
<!-- commit 68bdfd15fa9feebfbf94a06cf574de598e239198 -->

r[bound]
# 特型与生命周期界限

r[bound.syntax]
```grammar,杂项
TypeParamBounds -> TypeParamBound ( `+` TypeParamBound )* `+`?

TypeParamBound -> Lifetime | TraitBound | UseBound

TraitBound ->
      ( `?` | ForLifetimes )? TypePath
    | `(` ( `?` | ForLifetimes )? TypePath `)`

LifetimeBounds -> ( Lifetime `+` )* Lifetime?

Lifetime ->
      LIFETIME_OR_LABEL
    | `'static`
    | `'_`

UseBound -> `use` UseBoundGenericArgs

UseBoundGenericArgs ->
      `<` `>`
    | `<` ( UseBoundGenericArg `,`)* UseBoundGenericArg `,`? `>`

UseBoundGenericArg ->
      Lifetime
    | IDENTIFIER
    | `Self`
```

r[bound.intro]
`[特型][Trait]` 和生命周期界限为 `[泛型项][generic]` 提供了一种限制哪些类型和生命周期可以用作其参数的方法。可以在 `[where 子句][where clause]` 中的任何类型上提供界限。对于某些常见情况，也有较短的形式：

* 在声明 `[泛型参数][generic]` 后编写的界限：`fn f<A: Copy>() {}` 与 `fn f<A>() where A: Copy {}` 相同。
* 在特型声明中作为 `[父特型][supertraits]` ：`trait Circle : Shape {}` 等同于 `trait Circle where Self : Shape {}` 。
* 在特型声明中作为 `[关联类型][associated types]` 上的界限：`trait A { type B: Copy; }` 等同于 `trait A where Self::B: Copy { type B; }` 。

r[bound.satisfaction]
使用 项 时必须满足其上的界限。在对泛型 项 进行类型检查和借用检查时，界限可用于确定某个类型是否实现了某个特型。例如，给定 `Ty: Trait`

* 在泛型函数的函数体中，可以在 `Ty` 值上调用来自 `Trait` 的方法。同样，也可以使用 `Trait` 上的关联常量。
* 可以使用来自 `Trait` 的关联类型。
* 具有 `T: Trait` 界限的泛型函数和类型可以使用 `Ty` 作为 `T` 。

```rust
# type Surface = i32;
trait Shape {
    fn draw(&self, surface: Surface);
    fn name() -> &'static str;
}

fn draw_twice<T: Shape>(surface: Surface, sh: T) {
    sh.draw(surface);           // 可以调用方法，因为 T: Shape
    sh.draw(surface);
}

fn copy_and_draw_twice<T: Copy>(surface: Surface, sh: T) where T: Shape {
    let shape_copy = sh;        // 不会移动 sh，因为 T: Copy
    draw_twice(surface, sh);    // 可以使用泛型函数，因为 T: Shape
}

struct Figure<S: Shape>(S, S);

fn name_figure<U: Shape>(
    figure: Figure<U>,          // 类型 Figure<U> 是良构的，因为 U: Shape
) {
    println!(
        "Figure of two {}",
        U::name(),              // 可以使用关联函数
    );
}
```

r[bound.trivial]
不使用 项 的参数或 `[高阶生命周期][higher-ranked lifetimes]` 的界限会在定义 项 时进行检查。此类界限若为假则是错误的。

r[bound.special]
在使用 项 时，还会检查某些泛型类型的 `[`Copy`]` 、 `[`Clone`]` 和 `[`Sized`]` 界限，即使该使用并未提供具体类型。在可变引用、 `[特型对象][trait object]` 或 `[切片][slice]` 上将 `Copy` 或 `Clone` 作为界限是错误的。在特型对象或切片上将 `Sized` 作为界限是错误的。

```rust,compile_fail
struct A<'a, T>
where
    i32: Default,           // 允许，但没用
    i32: Iterator,          // 错误：`i32` 不是迭代器
    &'a mut T: Copy,        // （使用时）错误：特型界限未满足
    [T]: Sized,             // （使用时）错误：大小在编译时无法确定
{
    f: &'a T,
}
struct UsesA<'a, T>(A<'a, T>);
```

r[bound.trait-object]
特型和生命周期界限也用于命名 `[特型对象][trait objects]` 。

r[bound.sized]
## `?Sized`

`?` 仅用于放宽针对 `[类型参数][type parameters]` 或 `[关联类型][associated types]` 的隐式 `[`Sized`]` 特型界限。 `?Sized` 不可用作其他类型的界限。

r[bound.lifetime]
## 生命周期界限

r[bound.lifetime.intro]
生命周期界限可以应用于类型或其他生命周期。

r[bound.lifetime.outlive-lifetime]
界限 `'a: 'b` 通常读作 `'a` *长于* `'b` 。 `'a: 'b` 表示 `'a` 的持续时间至少与 `'b` 一样长，因此只要 `&'b ()` 有效，引用 `&'a ()` 就是有效的。

```rust
fn f<'a, 'b>(x: &'a i32, mut y: &'b i32) where 'a: 'b {
    y = x;                      // &'a i32 是 &'b i32 的子类型，因为 'a: 'b
    let r: &'b &'a i32 = &&0;   // &'b &'a i32 是良构的，因为 'a: 'b
}
```

r[bound.lifetime.outlive-type]
`T: 'a` 表示 `T` 的所有生命周期参数都 *长于* `'a` 。例如，如果 `'a` 是一个不受约束的生命周期参数，则 `i32: 'static` 和 `&'static str: 'a` 被满足，但 `Vec<&'a ()>: 'static` 不满足。

r[bound.higher-ranked]
## 高阶特型界限

r[bound.higher-ranked.syntax]
```grammar,杂项
ForLifetimes -> `for` GenericParams
```

r[bound.higher-ranked.intro]
特型界限可以是生命周期上的 *高阶* 界限。这些界限指定了 *对于所有* 生命周期都成立的界限。例如，像 `for<'a> &'a T: PartialEq<i32>` 这样的界限需要类似如下的实现

```rust
# struct T;
impl<'a> PartialEq<i32> for &'a T {
    // ...
#    fn eq(&self, other: &i32) -> bool {true}
}
```

并可用于将具有任何生命周期的 `&'a T` 与 `i32` 进行比较。

此处只能使用高阶界限，因为引用的生命周期比函数上任何可能的生命周期参数都要短：

```rust
fn call_on_ref_zero<F>(f: F) where for<'a> F: Fn(&'a i32) {
    let zero = 0;
    f(&zero);
}
```

r[bound.higher-ranked.trait]
高阶生命周期也可以紧接在特型之前指定：唯一的区别是生命周期参数的 [作用域][hrtb-scopes] ，它仅延伸到后续特型的末尾，而不是整个界限。此函数等同于上一个函数。

```rust
fn call_on_ref_zero<F>(f: F) where F: for<'a> Fn(&'a i32) {
    let zero = 0;
    f(&zero);
}
```

r[bound.implied]
## 隐含界限

r[bound.implied.intro]
类型为良构所需的生命周期界限有时会被推断出来。

```rust
fn requires_t_outlives_a<'a, T>(x: &'a T) {}
```

要求类型参数 `T` *长于* `'a` ，以便 `&'a T` 类型是良构的。这是被推断出来的，因为函数签名包含类型 `&'a T` ，该类型仅在 `T: 'a` 成立时才有效。

r[bound.implied.context]
隐含界限会被添加到函数的所有参数和输出中。在 `requires_t_outlives_a` 内部，即使没有显式指定，也可以假设 `T: 'a` 成立：

```rust
fn requires_t_outlives_a_not_implied<'a, T: 'a>() {}

fn requires_t_outlives_a<'a, T>(x: &'a T) {
    // 这可以编译，因为 `T: 'a` 是由
    // 引用类型 `&'a T` 隐含的。
    requires_t_outlives_a_not_implied::<'a, T>();
}
```

```rust,compile_fail,E0309
# fn requires_t_outlives_a_not_implied<'a, T: 'a>() {}
fn not_implied<'a, T>() {
    // 这会报错，因为 `T: 'a` 不是由
    // 函数签名隐含的。
    requires_t_outlives_a_not_implied::<'a, T>();
}
```

r[bound.implied.trait]
只有生命周期界限是隐含的，特型界限仍必须显式添加。因此，以下示例会导致错误：

```rust,compile_fail,E0277
use std::fmt::Debug;
struct IsDebug<T: Debug>(T);
// 错误[E0277]：`T` 未实现 `Debug`
fn doesnt_specify_t_debug<T>(x: IsDebug<T>) {}
```

r[bound.implied.def]
对于任何类型的类型定义和 impl 块，也会推断出生命周期界限：

```rust
struct Struct<'a, T> {
    // 这要求 `T: 'a` 是良构的，
    // 由编译器推断。
    field: &'a T,
}

enum Enum<'a, T> {
    // 这要求 `T: 'a` 是良构的，
    // 由编译器推断。
    //
    // 注意，即使仅使用
    // `Enum::OtherVariant`，也要求 `T: 'a`。
    SomeVariant(&'a T),
    OtherVariant,
}

trait Trait<'a, T: 'a> {}

// 这会报错，因为 `T: 'a` 未由 impl 标题中的任何类型隐含。
//     impl<'a, T> Trait<'a, T> for () {}

// 这可以编译，因为 `T: 'a` 是由 self 类型 `&'a T` 隐含的。
impl<'a, T> Trait<'a, T> for &'a T {}
```

r[bound.use]
## `use`界限

某些界限列表可能包含 `use<..>` 界限，以控制哪些泛型参数会被 `impl Trait` [抽象返回类型][abstract return type] 捕获。有关更多详细信息，请参阅 [精确捕获][precise capturing] 。

[abstract return type]: types/impl-trait.md#抽象返回类型
[arrays]: types/array.md
[associated types]: items/associated-items.md#关联类型
[hrtb-scopes]: names/scopes.md#高阶特型界限作用域
[supertraits]: items/traits.md#父特型
[generic]: items/generics.md
[higher-ranked lifetimes]: #higher-ranked-trait-bounds
[precise capturing]: types/impl-trait.md#精确捕获
[slice]: types/slice.md
[Trait]: items/traits.md#特型界限
[trait object]: types/trait-object.md
[trait objects]: types/trait-object.md
[type parameters]: types/parameters.md
[where clause]: items/generics.md#where子句
