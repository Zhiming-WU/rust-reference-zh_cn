<!-- https://github.com/rust-lang/reference/blob/master/src/interior-mutability.md -->
<!-- commit 68bdfd15fa9feebfbf94a06cf574de598e239198 -->

r[interior-mut]
# 内部可变性

r[interior-mut.intro]
有时一个类型在拥有多个别名时需要被修改。在 Rust 中，这是通过一种被称为 _内部可变性_ 的模式实现的。

r[interior-mut.shared-ref]
如果一个类型的内部状态可以通过指向它的 [共享引用][shared reference] 来改变，那么该类型就具有内部可变性。

r[interior-mut.no-constraint]
这违背了通常的 [要求][ub] ，即共享引用指向的值不能被修改。

r[interior-mut.unsafe-cell]
[`std::cell::UnsafeCell<T>`] 类型是唯一允许禁用此要求的方式。当 `UnsafeCell<T>` 被不可变地起别名时，修改它包含的 `T` ，或者获取它的一个可变引用，仍然是安全的。

r[interior-mut.mut-unsafe-cell]
与所有其他类型一样，拥有多个 `&mut UnsafeCell<T>` 别名是未定义行为。

r[interior-mut.abstraction]
其他具有内部可变性的类型可以通过使用 `UnsafeCell<T>` 作为字段来创建。标准库提供了多种提供安全内部可变性 API 的类型。

r[interior-mut.ref-cell]
例如， [`std::cell::RefCell<T>`] 使用运行时借用检查来确保围绕多个引用的通常规则。

r[interior-mut.atomic]
[`std::sync::atomic`] 模块包含了一些包装值的类型，这些值只能通过原子操作访问，从而允许该值在线程间共享和修改。

[shared reference]: types/pointer.md#共享引用
[ub]: behavior-considered-undefined.md
