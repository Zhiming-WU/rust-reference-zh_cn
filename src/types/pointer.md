r[type.pointer]
# 指针类型

r[type.pointer.intro]
所有指针都是显式的一等值。
它们可以被移动或复制，存储进数据 结构体 中，并从函数中返回。

r[type.pointer.reference]
## 引用 (`&` 和 `&mut`)

r[type.pointer.reference.syntax]
```grammar,类型
ReferenceType -> `&` Lifetime? `mut`? TypeNoBounds
```

r[type.pointer.reference.shared]
### 共享引用 (`&`)

r[type.pointer.reference.shared.intro]
共享引用指向由其他值拥有的内存。

r[type.pointer.reference.shared.constraint-mutation]
当创建一个值的共享引用时，它会阻止该值的直接修改。
[内部可变性][Interior mutability] 在某些情况下为此提供了例外。
顾名思义，一个值可以存在任意数量的共享引用。
共享引用类型写为 `&type` ，或者在需要指定显式生命周期时写为 `&'a type` 。

r[type.pointer.reference.shared.copy]
复制引用是一个 “浅层” 操作：
它只涉及复制指针本身，也就是说，指针是 `Copy` 的。
释放引用对它所指向的值没有影响，但引用一个 [临时值][temporary value] 会在引用本身的作用域内使其保持存活。

r[type.pointer.reference.mut]
### 可变引用 (`&mut`)

r[type.pointer.reference.mut.intro]
可变引用指向由其他值拥有的内存。
可变引用类型写为 `&mut type` 或 `&'a mut type` 。

r[type.pointer.reference.mut.copy]
可变引用（尚未被借用的）是访问其指向的值的唯一方式，因此它不是 `Copy` 的。

r[type.pointer.raw]
## 裸指针 (`*const` 和 `*mut`)

r[type.pointer.raw.syntax]
```grammar,类型
RawPointerType -> `*` ( `mut` | `const` ) TypeNoBounds
```

r[type.pointer.raw.intro]
裸指针是没有安全或存活性保证的指针。
裸指针写为 `*const T` 或 `*mut T` 。
例如 `*const i32` 表示一个指向 32 位整数的裸指针。

r[type.pointer.raw.copy]
复制或丢弃裸指针对任何其他值的生命周期都没有影响。

r[type.pointer.raw.safety]
解引用裸指针是一个 [`unsafe` 操作][`unsafe` operation] 。

这也可以通过重借用 (`&*` 或 `&mut *` ) 来将裸指针转换为引用。
通常不鼓励使用裸指针；
它们的存在是为了支持与外部代码的互操作性，以及编写性能关键型或底层函数。

r[type.pointer.raw.cmp]
比较裸指针时，是按它们的地址进行比较，而不是按它们指向的内容进行比较。
在将裸指针与 [动态大小类型][dynamically sized types] 进行比较时，它们的附加数据也会被比较。

r[type.pointer.raw.constructor]
裸指针可以使用 `&raw const` 直接创建为 `*const` 指针，使用 `&raw mut` 直接创建为 `*mut` 指针。

r[type.pointer.smart]
## 智能指针

标准库包含除引用和裸指针之外的其他 “智能指针” 类型。

r[type.pointer.validity]
## 位有效性

r[type.pointer.validity.pointer-fragment]
尽管在大多数平台上生成的机器代码中，指针和引用与 `usize` 相似，
但目前尚未确定将引用或指针类型转换为非指针类型的语义。
因此，将指针或引用类型 `P` 转换为 `[u8; size_of::<P>()]` 可能是不合法的。

r[type.pointer.validity.raw]
对于瘦裸指针（即针对 `T: Sized` 的 `P = *const T` 或 `P = *mut T` ），
反向（从整数或整数数组转换为 `P` ）始终是有效的。
然而，通过这种转换产生的指针可能无法被解引用（即使 `T` 的大小为零也不行）。

[Interior mutability]: ../interior-mutability.md
[`unsafe` operation]: ../unsafety.md
[dynamically sized types]: ../dynamically-sized-types.md
[temporary value]: ../expressions.md#temporaries
