r[lang-types]
# 特殊类型和特型

r[lang-types.intro]
 [标准库][the standard library] 中存在的某些类型和 特型 是 Rust 编译器所熟知的。本章记录了这些类型和 特型 的特殊 特性 。

r[lang-types.box]
## `Box<T>`

r[lang-types.box.intro]
[`Box<T>`] 具有一些 Rust 目前不允许用户定义类型使用的特殊 特性 。

r[lang-types.box.deref]
*  `Box<T>` 的 [解引用运算符][dereference operator] 会产生一个可以被移出的位置。这意味着 `*` 运算符和  `Box<T>` 的析构函数是内置在语言中的。

r[lang-types.box.receiver]
*  [方法][Methods] 可以接收 `Box<Self>` 作为接收者。

r[lang-types.box.fundamental]
*  在与 `T` 相同的 crate 中，可以为 `Box<T>` 实现某个 特型 ，而 [孤儿规则][orphan rules] 会阻止其他 泛型 类型这样做。

<!-- Editor Note: This is nowhere close to an exhaustive list -->

r[lang-types.rc]
## `Rc<T>`

r[lang-types.rc.receiver]
[方法][Methods] 可以接收 [`Rc<Self>`] 作为接收者。

r[lang-types.arc]
## `Arc<T>`

r[lang-types.arc.receiver]
[方法][Methods] 可以接收 [`Arc<Self>`] 作为接收者。

r[lang-types.pin]
## `Pin<P>`

r[lang-types.pin.receiver]
[方法][Methods] 可以接收 [`Pin<P>`] 作为接收者。

r[lang-types.unsafe-cell]
## `UnsafeCell<T>`

r[lang-types.unsafe-cell.interior-mut]
[`std::cell::UnsafeCell<T>`] 用于 [内部可变性][interior mutability] 。它确保编译器不会对这些类型执行错误的优化。

r[lang-types.unsafe-cell.read-only-alloc]
它还确保具有内部可变性类型的 [`static` 项][`static` items] 不会被放置在标记为只读的内存中。

r[lang-types.phantom-data]
## `PhantomData<T>`

[`std::marker::PhantomData<T>`] 是一种零大小、最小对齐的类型，为了 [型变][variance] 、 [析构检查][drop check] 和 [自动特型](#auto-traits) 的目的，它被视为拥有一个 `T` 。

r[lang-types.ops]
## 运算符特型

[`std::ops`] 和 [`std::cmp`] 中的 特型 用于重载 [运算符][operators] 、 [索引表达式][indexing expressions] 和 [调用表达式][call expressions] 。

r[lang-types.deref]
## `Deref`和`DerefMut`

除了重载一元 `*` 运算符外， [`Deref`] 和 [`DerefMut`] 还用于 [方法解析][method resolution] 和 [解引用强转][deref coercions] 。

r[lang-types.drop]
## `Drop`

[`Drop`] 特型 提供了一个 [析构函数][destructor] ，每当该类型的值要被销毁时就会运行。

r[lang-types.copy]
## `Copy`

r[lang-types.copy.intro]
[`Copy`] 特型 会改变实现它的类型的语义。

r[lang-types.copy.behavior]
类型实现 `Copy` 的值在赋值时会被复制而不是移动。

r[lang-types.copy.constraint]
`Copy` 只能为未实现 `Drop` 且其字段均为 `Copy` 的类型实现。对于枚举，这意味着所有变体的所有字段都必须是 `Copy` 。对于 联合体 ，这意味着所有变体都必须是 `Copy` 。

r[lang-types.copy.builtin-types]
`Copy` 由编译器为以下内容实现：

r[lang-types.copy.tuple]
*  `Copy` 类型的 [元组][Tuples]

r[lang-types.copy.fn-pointer]
*  [函数指针][Function pointers]

r[lang-types.copy.fn-item]
*  [函数项][Function items]

r[lang-types.copy.closure]
*  未捕获任何值或仅捕获 `Copy` 类型值的 [闭包][Closures]

r[lang-types.clone]
## `Clone`

r[lang-types.clone.intro]
[`Clone`] 特型 是 `Copy` 的父特型，因此它也需要编译器生成的实现。

r[lang-types.clone.builtin-types]
它由编译器为以下类型实现：

r[lang-types.clone.builtin-copy]
* 具有内置 `Copy` 实现的类型（见上文）

r[lang-types.clone.tuple]
*  `Clone` 类型的 [元组][Tuples]

r[lang-types.clone.closure]
*  仅捕获 `Clone` 类型的值或不从环境中捕获值的 [闭包][Closures]

r[lang-types.send]
## `Send`

[`Send`] 特型 表示该类型的值可以安全地从一个线程发送到另一个线程。

r[lang-types.sync]
## `Sync`

r[lang-types.sync.intro]
[`Sync`] 特型 表示该类型的值可以安全地在多个线程之间共享。

r[lang-types.sync.static-constraint]
此 特型 必须为不可变 [`static` 项][`static` items] 中使用的所有类型实现。

r[lang-types.termination]
## `Termination`

[`Termination`] 特型 表示 [main 函数][main function] 和 [测试函数][test functions] 可接受的返回类型。

r[lang-types.auto-traits]
## 自动特型

[`Send`] 、 [`Sync`] 、 [`Unpin`] 、 [`UnwindSafe`] 和 [`RefUnwindSafe`] 特型 是 _自动特型_ 。自动特型具有特殊属性。

r[lang-types.auto-traits.auto-impl]
如果未针对给定类型的自动 特型 编写显式实现或否定实现，则编译器会根据以下规则自动实现它：

r[lang-types.auto-traits.builtin-composite]
*  如果 `T` 实现了该 特型 ，则 `&T` 、 `&mut T` 、 `*const T` 、 `*mut T` 、 `[T; n]` 和 `[T]` 也会实现。

r[lang-types.auto-traits.fn-item-pointer]
* 函数项类型和函数指针会自动实现该 特型 。

r[lang-types.auto-traits.aggregate]
*  结构体 、枚举、 联合体 和元组的所有字段都实现了该 特型 ，则它们也会实现。

r[lang-types.auto-traits.closure]
*  如果闭包所有捕获的类型都实现了该 特型 ，则该闭包也会实现。通过共享引用捕获 `T` 并通过值捕获 `U` 的闭包将实现 `&T` 和 `U` 都实现的任何自动 特型 。

r[lang-types.auto-traits.generic-impl]
对于 泛型 类型（将上述内置类型视为关于 `T` 的 泛型 ），如果存在 泛型 实现，那么对于那些本可以使用该实现但因不满足必要 特型 约束而无法使用的类型，编译器不会自动实现它。例如，标准库为所有 `T` 为 `Sync` 的 `&T` 实现了 `Send` ；这意味着如果 `T` 是 `Send` 但不是 `Sync` ，编译器将不会为 `&T` 实现 `Send` 。

r[lang-types.auto-traits.negative]
自动 特型 也可以具有否定实现（在标准库文档中显示为 `impl !AutoTrait for T` ），这些实现会覆盖自动实现。例如 `*mut T` 具有 `Send` 的否定实现，因此即使 `T` 是 `Send` ， `*mut T` 也不是 `Send` 。目前还没有稳定且通用的方法来指定额外的否定实现；它们仅存在于标准库中。

r[lang-types.auto-traits.trait-object-marker]
自动 特型 可以作为附加约束添加到任何 [特型对象][trait object] 中，即使通常只允许一个 特型 。例如， `Box<dyn Debug + Send + UnwindSafe>` 是一个有效类型。

r[lang-types.sized]
## `Sized`

r[lang-types.sized.intro]
[`Sized`] 特型 表示该类型的大小在编译时是已知的；也就是说，它不是 [动态大小类型][dynamically sized type] 。

r[lang-types.sized.implicit-sized]
[类型参数][Type parameters] （ 特型 中的 `Self` 除外）默认是 `Sized` 的， [关联类型][associated types] 也是如此。

r[lang-types.sized.implicit-impl]
`Sized` 总是由编译器自动实现，而不是通过 [实现项][implementation items] 实现。

r[lang-types.sized.relaxation]
这些隐式的 `Sized` 约束可以通过使用特殊的 `?Sized` 约束来放宽。

[`Arc<Self>`]: std::sync::Arc
[`Deref`]: std::ops::Deref
[`DerefMut`]: std::ops::DerefMut
[`Pin<P>`]: std::pin::Pin
[`Rc<Self>`]: std::rc::Rc
[`RefUnwindSafe`]: std::panic::RefUnwindSafe
[`Termination`]: std::process::Termination
[`UnwindSafe`]: std::panic::UnwindSafe
[`Unpin`]: std::marker::Unpin

[Arrays]: types/array.md
[associated types]: items/associated-items.md#associated-types
[call expressions]: expressions/call-expr.md
[deref coercions]: type-coercions.md#coercion-types
[dereference operator]: expressions/operator-expr.md#the-dereference-operator
[destructor]: destructors.md
[drop check]: ../nomicon/dropck.html
[dynamically sized type]: dynamically-sized-types.md
[Function pointers]: types/function-pointer.md
[Function items]: types/function-item.md
[implementation items]: items/implementations.md
[indexing expressions]: expressions/array-expr.md#array-and-slice-indexing-expressions
[interior mutability]: interior-mutability.md
[main function]: crates-and-source-files.md#main函数
[Methods]: items/associated-items.md#associated-functions-and-methods
[method resolution]: expressions/method-call-expr.md
[operators]: expressions/operator-expr.md
[orphan rules]: items/implementations.md#trait-implementation-coherence
[`static` items]: items/static-items.md
[test functions]: attributes/testing.md#the-test-attribute
[the standard library]: std
[trait object]: types/trait-object.md
[Tuples]: types/tuple.md
[Type parameters]: types/parameters.md
[variance]: subtyping.md#variance
[Closures]: types/closure.md
