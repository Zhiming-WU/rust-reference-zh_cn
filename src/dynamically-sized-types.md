r[dynamic-sized]
# 动态大小类型

r[dynamic-sized.intro]
大多数类型在编译时具有固定的大小，并实现了 [`Sized`][sized] 特型。大小仅在运行时已知的类型被称为 _动态大小类型_ (_DST_) ，或者非正式地称为不定长类型。 [切片][Slices]、 [特型对象][trait objects] 和 [str] 是 <abbr title="dynamically sized types">DST</abbr> 的示例。

r[dynamic-sized.restriction]
这类类型只能在某些情况下使用：

r[dynamic-sized.pointer-types]
* 指向 <abbr title="dynamically sized types">DST</abbr> 的 [指针类型][Pointer types] 是定长的，但其大小是定长类型指针的两倍
    * 指向切片和 `str` 的指针还存储了元素的数量。
    * 指向特型对象的指针还存储了一个指向 vtable 的指针。

r[dynamic-sized.question-sized]
* <abbr title="dynamically sized types">DST</abbr> 可以作为具有特殊 `?Sized` 边界的泛型类型参数的类型参数提供。当相应的关联类型声明具有 `?Sized` 边界时，它们也可以用于关联类型定义。默认情况下，任何类型参数或关联类型都具有 `Sized` 边界，除非使用 `?Sized` 将其放宽。

r[dynamic-sized.trait-impl]
* 可以为 <abbr title="dynamically sized types">DST</abbr> 实现特型。与泛型类型参数不同，在特型定义中 `Self: ?Sized` 是默认的。

r[dynamic-sized.struct-field]
* 结构体可以将 <abbr title="dynamically sized type">DST</abbr> 作为其最后一个字段；这使得结构体本身也成为一个 <abbr title="dynamically sized type">DST</abbr> 。

> [!NOTE]
> [变量][Variables]、 函数参数、 [常量][const] 项 和 [静态][static] 项 必须是 `Sized` 的。

[sized]: special-types-and-traits.md#sized
[Slices]: types/slice.md
[str]: types/textual.md
[trait objects]: types/trait-object.md
[Pointer types]: types/pointer.md
[Variables]: variables.md
[const]: items/constant-items.md
[static]: items/static-items.md
