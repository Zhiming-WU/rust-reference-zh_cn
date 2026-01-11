r[expr.method]
# 方法调用表达式

r[expr.method.syntax]
```grammar,表达式
MethodCallExpression -> Expression `.` PathExprSegment `(`CallParams? `)`
```

r[expr.method.intro]
一个 _方法调用_ 由一个表达式（ *接收者* ）、一个点、一个表达式路径段和一个括号括起来的表达式列表组成。

r[expr.method.target]
方法调用被解析为特定 特型 上的关联 [方法][methods]，如果左侧的准确 `self` 类型已知，则静态分派到方法；如果左侧表达式是一个间接的 [特型对象][trait objects]。

```rust
let pi: Result<f32, _> = "3.14".parse();
let log_pi = pi.unwrap_or(1.0).log(2.72);
# assert!(1.14 < log_pi && log_pi < 1.15)
```

r[expr.method.autoref-deref]
在查找方法调用时，接收者可能会被自动解引用或借用，以便调用方法。
这比其他函数需要更复杂的查找过程，因为可能有多个可能的方法可以调用。使用以下程序：

r[expr.method.candidate-receivers]
第一步是构建候选接收者类型列表。
通过重复 [解引用][dereference] 接收者表达式的类型来获取这些类型，将遇到的每个类型添加到列表中，最后尝试进行 [不定尺寸强制转换][unsized coercion]，如果成功则添加结果类型。

r[expr.method.candidate-receivers-refs]
然后，对于每个候选 `T`，紧接着 `T` 之后将 `&T` 和 `&mut T` 添加到列表中。

例如，如果接收者的类型为 `Box<[i32;2]>`，那么候选类型将是 `Box<[i32;2]>`、`&Box<[i32;2]>`、`&mut Box<[i32;2]>`、`[i32; 2]`（通过解引用）、`&[i32; 2]`、`&mut [i32; 2]`、`[i32]`（通过不定尺寸强制转换）、`&[i32]`，以及最后的 `&mut [i32]`。

r[expr.method.candidate-search]
然后，对于每个候选类型 `T`，在以下位置搜索具有该类型接收者的 [可见][visible] 方法：

1. `T` 的固有方法（直接在 `T` 上实现的方法）。
1. 由 `T` 实现的任何 [可见][visible] 特型 提供的方法。
   如果 `T` 是类型参数，则首先查找由 `T` 上的 特型 约束提供的方法。
   然后查找作用域内所有剩余的方法。

> [!NOTE]
> 查找按顺序对每个类型进行，这偶尔会导致令人惊讶的结果。下面的代码将打印 "In trait impl!"，因为首先查找 `&self` 方法，在找到结构体的 `&mut self` 方法之前就找到了 特型 方法。
>
> ```rust
> struct Foo {}
>
> trait Bar {
>   fn bar(&self);
> }
>
> impl Foo {
>   fn bar(&mut self) {
>     println!("In struct impl!")
>   }
> }
>
> impl Bar for Foo {
>   fn bar(&self) {
>     println!("In trait impl!")
>   }
> }
>
> fn main() {
>   let mut f = Foo{};
>   f.bar();
> }
> ```

r[expr.method.ambiguous-target]
如果这导致多个可能的候选者，则是一个错误，必须将接收者 [转换][disambiguate call] 为适当的接收者类型以进行方法调用。

r[expr.method.receiver-constraints]
此过程不考虑接收者的可变性或生命周期，也不考虑方法是否为 `unsafe`。
一旦查找到方法，如果由于这些原因之一（或多个）而无法调用，结果将是编译器错误。

r[expr.method.ambiguous-search]
如果到达某一步存在多个可能的方法，例如泛型方法或 特型 被视为相同，则这是一个编译器错误。
这些情况需要使用 [消除歧义的函数调用语法][disambiguating function call syntax] 进行方法和函数调用。

r[expr.method.edition2021]
> [!EDITION-2021]
> 在 2021 版次之前，在搜索可见方法期间，如果候选接收者类型是 [数组类型][array type]，则会忽略标准库 [`IntoIterator`] 特型 提供的方法。
>
> 为此目的使用的 版次 由代表方法名的 词法单元 决定。
>
> 此特殊情况将来可能会被删除。

> [!WARNING]
> 对于 [特型对象][trait objects]，如果存在与 特型 方法同名的固有方法，则在尝试在方法调用表达式中调用该方法时会给出编译器错误。
> 相反，你可以使用 [消除歧义的函数调用语法][disambiguating function call syntax] 来调用该方法，在这种情况下，它调用的是 特型 方法，而不是固有方法。
> 没有办法调用固有方法。
> 只要不在 特型对象 上定义与 特型 方法同名的固有方法就没事。

[visible]: ../visibility-and-privacy.md
[array type]: ../types/array.md
[trait objects]: ../types/trait-object.md
[disambiguate call]: call-expr.md#disambiguating-function-calls
[disambiguating function call syntax]: call-expr.md#disambiguating-function-calls
[dereference]: operator-expr.md#the-dereference-operator
[methods]: ../items/associated-items.md#方法
[unsized coercion]: ../type-coercions.md#unsized-coercions
[`IntoIterator`]: std::iter::IntoIterator
