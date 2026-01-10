r[undefined]
# 被视为未定义的行为

r[undefined.general]
Rust 代码如果表现出以下列表中的任何行为，就是不正确的。这包括 `unsafe` 块和 `unsafe` 函数中的代码。`unsafe` 仅意味着避免未定义行为的责任在程序员身上；它并不会改变 Rust 程序决不能导致未定义行为这一事实。

r[undefined.soundness]
在编写 `unsafe` 代码时，程序员有责任确保任何与 `unsafe` 代码交互的安全代码都不会触发这些行为。对于任何安全客户端都能满足此属性的 `unsafe` 代码被称为 *sound (无害的)* ；如果 `unsafe` 代码可能被安全代码误用而表现出未定义行为，则它是 *unsound (有害的)* 。

> [!WARNING]
> 以下列表并不详尽；它可能会增加或减少。对于在不安全代码中哪些允许、哪些不允许，目前还没有正式的 Rust 语义模型，因此可能还有更多行为被认为是不安全的。我们也保留在未来将列表中某些行为变为“已定义行为”的权利。换句话说，此列表并不代表任何内容在所有未来的 Rust 版本中都 *肯定* 始终是未定义的（但我们将来可能会对某些列表项做出此类承诺）。
>
> 在编写不安全代码之前，请阅读 [Rustonomicon]。

r[undefined.race]
* 数据竞争。

r[undefined.pointer-access]
* 访问（加载或存储）一个 [悬垂][dangling] 或 [基于对齐错误指针][based on a misaligned pointer] 的位置。

r[undefined.place-projection]
* 执行违反 [in-bounds 指针算术](pointer#method.offset) 要求的 位置投影 (place projection)。位置投影是指 [字段表达式][project-field]、[元组索引表达式][project-tuple] 或 [数组/切片索引表达式][project-slice]。

r[undefined.alias]
* 违反指针别名规则。精确的别名规则尚未确定，但这里有一个通用原则的大纲：
  `&T` 必须指向在其存续期间未被修改的内存（除了 [`UnsafeCell<U>`] 内部的数据），
  且 `&mut T` 必须指向在其存续期间不被任何非该引用派生的指针读取或写入，且没有其他引用指向的内存。
  出于这些规则的目的，`Box<T>` 的处理方式类似于 `&'static mut T`。
  精确的存续持续时间 (liveness duration) 尚未指定，但存在一些界限：
  * 对于引用，存续持续时间由借用检查器分配的语法生命周期作为上限；它的存续时间不能比该生命周期更 *长*。
  * 每次解引用或重新借用引用或 `Box` 时，它都被视为存续。
  * 每次将引用或 `Box` 传递给函数或从函数返回时，它都被视为存续。
  * 当引用（但不是 `Box`！）传递给函数时，它的存续时间至少与该函数调用一样长，同样，除非 `&T` 包含 [`UnsafeCell<U>`]。

  当这些类型的值作为复合类型的（嵌套）字段传递时，上述规则同样适用，但不适用于指针间接寻址后的情况。

r[undefined.immutable]
* 修改不可变字节。
  通过 [常量提升][const-promoted] 表达式可达的所有字节都是不可变的，以及通过 `static` 和 `const` 初始化器中已 [生命周期延长][lifetime-extended] 到 `'static` 的借用可达的字节也是不可变的。
  由不可变绑定或不可变 `static` 拥有的字节是不可变的，除非这些字节是 [`UnsafeCell<U>`] 的一部分。

  此外，由共享引用 [指向][pointed to] 的字节（包括通过其他引用（共享和可变）以及 `Box` 的间接指向）都是不可变的；传递性包括存储在复合类型字段中的那些引用。

  修改 (mutation) 是指与任何相关字节重叠的、多于 0 字节的任何写入（即使该写入没有改变内存内容）。

r[undefined.intrinsic]
* 通过编译器内部函数 (compiler intrinsics) 调用未定义行为。

r[undefined.target-feature]
* 执行由当前平台不支持的平台特性编译的代码（请参阅 [`target_feature`]），*除非* 平台明确记录这是安全的。

r[undefined.call]
* 使用错误的 [调用 ABI][abi] 调用函数，或者回溯 (unwind) 超过一个不允许回溯的栈帧（例如，通过调用一个导入为或转录为 `"C"` 函数或函数指针的 `"C-unwind"` 函数）。

r[undefined.invalid]
* 产生 [无效值][invalid-values]。“产生”一个值发生在任何时候：当一个值被赋值给一个位置、从一个位置读取、传递给函数/原始操作或从函数/原始操作返回时。

r[undefined.asm]
* 错误使用内联汇编。有关更多详细信息，请参阅编写使用内联汇编的代码时应遵守的 [规则][rules]。

r[undefined.runtime]
* 违反 Rust 运行时的假设。目前大多数 Rust 运行时的假设尚未明确记录。
  * 对于专门与回溯相关的假设，请参阅 [恐慌文档][unwinding-ffi]。
  * 运行时假设 Rust 栈帧在不执行该栈帧拥有的局部变量的析构函数的情况下不会被释放。这一假设可能被像 `longjmp` 这样的 C 函数违反。

> [!NOTE]
> 未定义行为会影响整个程序。例如，调用一个表现出 C 语言未定义行为的 C 函数意味着你的整个程序都包含未定义行为，这也可能影响 Rust 代码。反之亦然，Rust 中的未定义行为可能会对通过 FFI 调用其他语言执行的代码产生不利影响。

r[undefined.pointed-to]
## 指向的字节

指针或引用“指向”的字节范围由指针值和被指向类型的类型大小决定（使用 `size_of_val`）。

r[undefined.misaligned]
## 基于对齐错误指针的位置
[based on a misaligned pointer]: #places-based-on-misaligned-pointers

r[undefined.misaligned.general]
如果在位置计算过程中的最后一个 `*` 投影是在对其类型未对齐的指针上执行的，则称该位置是“基于对齐错误指针的”。（如果位置表达式中没有 `*` 投影，那么这就是访问局部变量或 `static` 的字段，rustc 将保证正确的对齐。如果有多个 `*` 投影，则其中每一个都会导致从内存中加载待解引用的指针本身，并且这些加载中的每一个都受对齐约束。请注意，由于自动解引用，在 Rust 表面语法格式中可能会省略一些 `*` 投影；我们在这里考虑的是完全展开后的位置表达式。）

例如，如果 `ptr` 的类型为 `*const S`，其中 `S` 的对齐要求为 8，那么 `ptr` 必须 8 字节对齐，否则 `(*ptr).f` 就是“基于对齐错误指针的”。即使字段 `f` 的类型是 `u8`（即对齐要求为 1 的类型），这也是成立的。换句话说，对齐要求源自被解引用的指针类型，*而不是* 正在被访问的字段类型。

r[undefined.misaligned.load-store]
请注意，基于对齐错误指针的位置仅在被加载或存储时才会导致未定义行为。

r[undefined.misaligned.raw]
在这样的位置上使用 `&raw const`/`&raw mut` 是允许的。

r[undefined.misaligned.reference]
在位置上使用 `&`/`&mut` 要求满足字段类型的对齐（否则程序将“产生无效值”），这通常比“基于对齐的指针”的要求更宽松。

r[undefined.misaligned.packed]
在字段类型可能比包含它的类型更对齐的情况下（即 `repr(packed)`），获取引用将导致编译器错误。这意味着基于对齐的指针始终足以确保新引用是对齐的，但它并不总是必要的。

r[undefined.dangling]
## 悬垂指针
[dangling]: #dangling-pointers

r[undefined.dangling.general]
如果引用/指针 [指向][points to] 的字节并不都属于同一个存活的分配（特别地，它们都必须属于 *某个* 分配），则该引用/指针是“悬垂的”。

r[undefined.dangling.zero-size]
如果大小为 0，那么指针在平凡意义上永远不会是“悬垂的”（即使它是空指针）。

r[undefined.dangling.dynamic-size]
请注意，动态大小类型（如切片和字符串）指向它们的整个范围，因此长度元数据绝不能过大非常重要。

r[undefined.dangling.alloc-limit]
特别地，Rust 值的动态大小（由 `size_of_val` 确定）绝不能超过 `isize::MAX`，因为单个分配的大小不可能大于 `isize::MAX`。

r[undefined.validity]
## 无效值
[invalid-values]: #invalid-values

r[undefined.validity.general]
Rust 编译器假设程序执行期间产生的所有值都是“有效的”，因此产生无效值会立即导致 UB。

一个值是否有效取决于其类型：

r[undefined.validity.bool]
* 一个 [`bool`] 值必须是 `false` (`0`) 或 `true` (`1`)。

r[undefined.validity.fn-pointer]
* 一个 `fn` 指针值必须非空。

r[undefined.validity.char]
* 一个 `char` 值不能是代理对 (surrogate)（即不能在 `0xD800..=0xDFFF` 范围内），且必须小于或等于 `char::MAX`。

r[undefined.validity.never]
* `!` 值绝不能存在。

r[undefined.validity.scalar]
* 整数 (`i*`/`u*`)、浮点值 (`f*`) 或原始指针必须已初始化，即不能从未初始化的内存中获得。

r[undefined.validity.str]
* `str` 值被视为类似于 `[u8]`，即它必须已初始化。

r[undefined.validity.enum]
* 一个 `enum` 必须具有有效的判别值 (discriminant)，且该判别值指示的变体 (variant) 的所有字段必须在其各自类型上有效。

r[undefined.validity.struct]
* `struct`、元组和数组要求所有字段/元素在其各自类型上有效。

r[undefined.validity.union]
* 对于 `union`，确切的有效性要求尚未确定。
  显然，所有完全在安全代码中创建的值都是有效的。
  如果联合体具有零大小字段，则每个可能的值都是有效的。
  更多细节 [仍处于讨论中](https://github.com/rust-lang/unsafe-code-guidelines/issues/438)。

r[undefined.validity.reference-box]
* 引用或 [`Box<T>`] 必须对齐且非空，它不能是 [悬垂的][dangling]，且必须指向一个有效值
  （对于动态大小类型，使用由元数据确定的被指向者的实际动态类型）。
  请注意，最后一点（关于指向有效值）仍是一个有争议的话题。

r[undefined.validity.wide]
* 宽引用、[`Box<T>`] 或原始指针的元数据必须与 unsized tail 的类型匹配：
  * `dyn Trait` 元数据必须是编译器为 `Trait` 生成的虚表 (vtable) 指针。
    （对于原始指针，这一要求仍是一个有争议的话题。）
  * 切片 (`[T]`) 元数据必须是一个有效的 `usize`。
    此外，对于宽引用和 [`Box<T>`]，如果切片元数据使得被指向值的总大小大于 `isize::MAX`，则该元数据是无效的。

r[undefined.validity.valid-range]
* 如果一个类型具有自定义的有效值范围，那么有效值必须在该范围内。
  在标准库中，这会影响 [`NonNull<T>`] 和 [`NonZero<T>`]。

  > [!NOTE]
  > `rustc` 通过不稳定的 `rustc_layout_scalar_valid_range_*` 属性来实现这一点。

r[undefined.validity.const-provenance]
* **在 [常量上下文][const contexts] 中**：除了上述描述之外，常量求值期间还适用进一步的与 物源 (provenance) 相关的要求。任何持有纯整数数据（`i*`/`u*`/`f*` 类型以及 `bool` 和 `char`、枚举判别值和切片元数据）的值都不得携带任何物源。任何持有指针数据（引用、原始指针、函数指针和 `dyn Trait` 元数据）的值必须要么不携带物源，要么所有字节必须是同一个原始指针值按正确顺序排列的碎片。

  这意味着如果指针具有物源，那么将指针（引用、原始指针或函数指针） transmute 或以其他方式重新解释为非指针类型（如整数）就是未定义行为。

  > [!EXAMPLE]
  > 以下所有情况都是 UB：
  >
  > ```rust,compile_fail
  > # use core::mem::MaybeUninit;
  > # use core::ptr;
  > // 我们不能将具有物源的指针重新解释为整数，
  > // 否则该整数的字节将具有物源。
  > const _: usize = {
  >     let ptr = &0;
  >     unsafe { (&raw const ptr as *const usize).read() }
  > };
  >
  > // 我们不能重新排列具有物源的指针的字节，
  > // 然后将它们解释为引用，因为那样持有
  > // 指针数据的值将具有顺序错误的指针碎片。
  > const _: &i32 = {
  >     let mut ptr = &0;
  >     let ptr_bytes = &raw mut ptr as *mut MaybeUninit::<u8>;
  >     unsafe { ptr::swap(ptr_bytes.add(1), ptr_bytes.add(2)) };
  >     ptr
  > };
  > ```

r[undefined.validity.undef]
**注意：** 未初始化内存对于任何具有限制有效值集合的类型也是隐式无效的。换句话说，唯一允许读取未初始化内存的情况是在 `union` 内部和“填充 (padding)”中（类型字段之间的间隙）。

[`bool`]: types/boolean.md
[`const`]: items/constant-items.md
[abi]: items/external-blocks.md#abi
[const contexts]: const-eval.const-context
[`target_feature`]: attributes/codegen.md#the-target_feature-attribute
[`UnsafeCell<U>`]: std::cell::UnsafeCell
[Rustonomicon]: ../nomicon/index.html
[`NonNull<T>`]: core::ptr::NonNull
[`NonZero<T>`]: core::num::NonZero
[place expression context]: expressions.md#位置表达式和值表达式
[rules]: inline-assembly.md#rules-for-inline-assembly
[points to]: #pointed-to-bytes
[pointed to]: #pointed-to-bytes
[project-field]: expressions/field-expr.md
[project-tuple]: expressions/tuple-expr.md#tuple-indexing-expressions
[project-slice]: expressions/array-expr.md#array-and-slice-indexing-expressions
[unwinding-ffi]: panic.md#unwinding-across-ffi-boundaries
[const-promoted]: destructors.md#constant-promotion
[lifetime-extended]: destructors.md#temporary-lifetime-extension
