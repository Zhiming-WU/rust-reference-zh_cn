<!-- https://github.com/rust-lang/reference/blob/master/src/types/impl-trait.md -->
<!-- commit 68bdfd15fa9feebfbf94a06cf574de598e239198 -->

r[type.impl-trait]
# Impl trait

r[type.impl-trait.syntax]
```grammar,类型
ImplTraitType -> `impl` TypeParamBounds

ImplTraitTypeOneBound -> `impl` TraitBound
```

r[type.impl-trait.intro]
`impl Trait` 提供了指定实现了特定特型的未命名但具体类型的方法。
它可以出现在两类地方：参数位置（在这里它可以作为函数的匿名泛型参数）和返回位置（在这里它可以作为抽象返回类型）。

```rust
trait Trait {}
# impl Trait for () {}

// 参数位置：匿名类型参数
fn foo(arg: impl Trait) {
}

// 返回位置：抽象返回类型
fn bar() -> impl Trait {
}
```
r[type.impl-trait.param]
## 匿名类型参数

> [!NOTE]
> 这通常被称为 “参数位置的 impl Trait” 。（术语 “parameter” 在这里更准确，但 “参数位置的 impl Trait” 是该特性开发期间使用的措辞，并且在实现的某些部分中仍然保留。）

r[type.impl-trait.param.intro]
函数可以使用 `impl` 后跟一组特型界限来声明一个具有匿名类型的参数。
调用者必须提供一个满足匿名类型参数所声明的界限的类型，并且函数只能使用通过匿名类型参数的特型界限可用的方法。

例如，这两种形式几乎是等价的：

```rust
trait Trait {}

// 泛型类型参数
fn with_generic_type<T: Trait>(arg: T) {
}

// 参数位置的 impl Trait
fn with_impl_trait(arg: impl Trait) {
}
```

r[type.impl-trait.param.generic]
也就是说，参数位置的 `impl Trait` 是类似于 `<T: Trait>` 的泛型类型参数的语法糖，只是该类型是匿名的，并且不会出现在 [GenericParams] 列表中。

> [!NOTE]
> 对于函数参数，泛型类型参数和 `impl Trait` 并不完全等价。对于像 `<T: Trait>` 这样的泛型参数，调用者可以选择在调用处使用 [GenericArgs] 显式指定 `T` 的泛型参数，例如 `foo::<usize>(1)` 。将参数从其中一种形式更改为另一种形式可能会对函数的调用者构成破坏性变更，因为这改变了泛型参数的数量。

r[type.impl-trait.return]
## 抽象返回类型

> [!NOTE]
> 这通常被称为 “返回位置的 impl Trait” 。

r[type.impl-trait.return.intro]
函数可以使用 `impl Trait` 来返回一个抽象返回类型。
这些类型代表另一个具体类型，调用者只能使用由指定的 `Trait` 声明的方法。

r[type.impl-trait.return.constraint-body]
函数中每个可能的返回值必须解析为相同的具体类型。

返回位置的 `impl Trait` 允许函数返回一个非装箱（unboxed）的抽象类型。
这在 [闭包][closures] 和迭代器中特别有用。
例如，闭包具有唯一的、不可写的类型。
以前，从函数返回闭包的唯一方法是使用 [特型对象][trait object] ：

```rust
fn returns_closure() -> Box<dyn Fn(i32) -> i32> {
    Box::new(|x| x + 1)
}
```

这可能会因堆分配和动态分派而产生性能损失。
当时无法完全指定闭包的类型，只能使用 `Fn` 特型。
这意味着特型对象是必需的。
然而，通过 `impl Trait` ，可以更简单地编写：

```rust
fn returns_closure() -> impl Fn(i32) -> i32 {
    |x| x + 1
}
```

这也避免了使用装箱特型对象的缺点。

类似地，迭代器的具体类型可能会变得非常复杂，包含链中所有先前迭代器的类型。
返回 `impl Iterator` 意味着函数只将其返回类型暴露为 `Iterator` 特型界限，而不是显式指定涉及的所有其他迭代器类型。

r[type.impl-trait.return-in-trait]
## 特型和特型实现中的返回位置 `impl Trait`

r[type.impl-trait.return-in-trait.intro]
特型中的函数也可以使用 `impl Trait` 作为匿名关联类型的语法。

r[type.impl-trait.return-in-trait.desugaring]
特型中关联函数的返回类型里的每个 `impl Trait` 都会被脱糖为一个匿名的关联类型。出现在实现函数的签名中的返回类型用于确定该关联类型的值。

r[type.impl-trait.generic-captures]
## 捕获

每个返回位置的 `impl Trait` 抽象类型背后都有一些隐藏的具体类型。为了让这个具体类型使用泛型参数，该泛型参数必须被抽象类型 *捕获* 。

r[type.impl-trait.generic-capture.auto]
## 自动捕获

r[type.impl-trait.generic-capture.auto.intro]
返回位置的 `impl Trait` 抽象类型会自动捕获所有作用域内的泛型参数，包括泛型类型、常量和生命周期参数（包括高阶参数）。

r[type.impl-trait.generic-capture.edition2024]
> [!EDITION-2024]
> 在 2024 版次 之前，在自由函数以及固有实现的关联函数和方法上，未出现在抽象返回类型界限中的泛型生命周期参数不会被自动捕获。

r[type.impl-trait.generic-capture.precise]
## 精确捕获

r[type.impl-trait.generic-capture.precise.use]
由返回位置 `impl Trait` 抽象类型捕获的泛型参数集合可以通过 [`use<..>` 界限][`use<..>` bound] 进行显式控制。如果存在，则仅捕获 `use<..>` 界限中列出的泛型参数。例如：

```rust
fn capture<'a, 'b, T>(x: &'a (), y: T) -> impl Sized + use<'a, T> {
  //                                      ~~~~~~~~~~~~~~~~~~~~~~~
  //                                     仅捕获 `'a` 和 `T` 。
  (x, y)
}
```

r[type.impl-trait.generic-capture.precise.constraint-single]
目前，界限列表中只能出现一个 `use<..>` 界限，必须包含所有作用域内的类型和常量泛型参数，并且必须包含出现在抽象类型其他界限中的所有生命周期参数。

r[type.impl-trait.generic-capture.precise.constraint-lifetime]
在 `use<..>` 界限内，任何存在的生命周期参数必须出现在所有类型和常量泛型参数之前，并且如果省略生命周期 (`'_`) 被允许出现在 `impl Trait` 返回类型中，则它可以存在。

r[type.impl-trait.generic-capture.precise.constraint-param-impl-trait]
因为所有作用域内的类型参数都必须按名称包含，所以 `use<..>` 界限不能用于使用了参数位置 `impl Trait` 的项的签名中，因为这些项的作用域内有匿名类型参数。

r[type.impl-trait.generic-capture.precise.constraint-in-trait]
特型定义中关联函数中存在的任何 `use<..>` 界限必须包含特型的所有泛型参数，包括特型隐式的 `Self` 泛型类型参数。

## 泛型和返回位置 `impl Trait` 之间的区别

在参数位置， `impl Trait` 在语义上与泛型类型参数非常相似。
然而，两者在返回位置有显著区别。
使用 `impl Trait` ，与泛型类型参数不同，函数选择返回类型，而调用者不能选择返回类型。

函数：

```rust
# trait Trait {}
fn foo<T: Trait>() -> T {
    // ...
# panic!()
}
```

允许调用者确定返回类型 `T` ，并且函数返回该类型。

函数：

```rust
# trait Trait {}
# impl Trait for () {}
fn foo() -> impl Trait {
    // ...
}
```

不允许调用者确定返回类型。
相反，函数选择返回类型，但只承诺它将实现 `Trait` 。

r[type.impl-trait.constraint]
## 限制

`impl Trait` 只能作为非 `extern` 函数的参数或返回类型出现。
它不能作为 `let` 绑定的类型、字段类型，或出现在类型别名中。

[`use<..>` bound]: ../trait-bounds.md#use界限
[closures]: closure.md
[trait object]: trait-object.md
