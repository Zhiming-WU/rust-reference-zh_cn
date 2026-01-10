r[layout]
# 类型布局

r[layout.intro]
类型的布局是其大小、对齐方式以及其字段的相对偏移量。对于枚举，判别式如何布局和解释也是类型布局的一部分。

r[layout.guarantees]
类型布局可能会在每次编译时发生变化。我们不打算准确记录具体的操作，而只记录目前所保证的内容。

请注意，即使布局相同的类型在跨函数边界传递的方式上仍可能有所不同。关于类型的函数调用 ABI 兼容性，请参阅 [此处][fn-abi-compatibility]。

r[layout.properties]
## 大小和对齐

所有值都有对齐方式和大小。

r[layout.properties.align]
值的 *对齐方式* 指明了存储该值时哪些地址是有效的。对齐方式为 `n` 的值必须存储在 `n` 的倍数的地址上。例如，对齐方式为 2 的值必须存储在偶数地址上，而对齐方式为 1 的值可以存储在任何地址上。对齐方式以字节为单位，且必须至少为 1，并且始终是 2 的幂。值的对齐方式可以通过 [`align_of_val`] 函数检查。

r[layout.properties.size]
值的 *大小* 是具有该 项 类型的数组中连续元素之间的字节偏移量，包括对齐填充。值的大小始终是其对齐方式的倍数。请注意，某些类型是零大小的；0 被认为是任何对齐方式的倍数（例如，在某些平台上，类型 `[u16; 0]` 的大小为 0，对齐方式为 2）。值的大小可以通过 [`size_of_val`] 函数检查。

r[layout.properties.sized]
如果某种类型的所有值都具有相同的大小和对齐方式，且两者在编译时都是已知的，则该类型实现了 [`Sized`] 特型，并且可以通过 [`size_of`] 和 [`align_of`] 函数进行检查。不是 [`Sized`] 的类型被称为 [动态大小类型][dynamically sized types] 。由于 `Sized` 类型的所有值都共享相同的大小和对齐方式，因此我们分别将这些共享值称为类型的大小和类型的对齐方式。

r[layout.primitive]
## 原语数据布局

r[layout.primitive.size]
大多数原语的大小如下表所示。

| 类型              | `size_of::<Type>()`|
|--                 |--                  |
| `bool`            | 1                  |
| `u8` / `i8`       | 1                  |
| `u16` / `i16`     | 2                  |
| `u32` / `i32`     | 4                  |
| `u64` / `i64`     | 8                  |
| `u128` / `i128`   | 16                 |
| `usize` / `isize` | 见下文              |
| `f32`             | 4                  |
| `f64`             | 8                  |
| `char`            | 4                  |

r[layout.primitive.size-int]
`usize` 和 `isize` 的大小足以包含目标平台上的每个地址。例如，在 32 位目标上，这是 4 字节，而在 64 位目标上，这是 8 字节。

r[layout.primitive.align]
原语的对齐方式取决于平台。
在大多数情况下，它们的对齐方式等于它们的大小，但可能会更小。
特别地，`i128` 和 `u128` 通常对齐到 4 或 8 字节，尽管它们的大小是 16；在许多 32 位平台上，`i64`、 `u64` 和 `f64` 仅对齐到 4 字节，而不是 8 字节。

r[layout.pointer]
## 指针和引用布局

r[layout.pointer.intro]
指针和引用具有相同的布局。指针或引用的可变性不会改变布局。

r[layout.pointer.thin]
指向定长类型的指针具有与 `usize` 相同的大小和对齐方式。

r[layout.pointer.unsized]
指向不定长类型的指针是定长的。其大小和对齐方式保证至少等于指针的大小和对齐方式。

> [!NOTE]
> 尽管你不应该依赖这一点，但目前所有指向 <abbr title="Dynamically Sized Types">DST</abbr> 的指针的大小都是 `usize` 大小的两倍，并且具有相同的对齐方式。

r[layout.array]
## 数组布局

数组 `[T; N]` 的大小为 `size_of::<T>() * N` ，并且具有与 `T` 相同的对齐方式。数组的布局使得数组中从零开始的第 `n` 个元素相对于数组起始位置的偏移量为 `n * size_of::<T>()` 字节。

r[layout.slice]
## 切片布局

切片与其切取的数组部分具有相同的布局。

> [!NOTE]
> 这是关于原始 `[T]` 类型的，而不是指向切片的指针（ `&[T]` 、 `Box<[T]>` 等）。

r[layout.str]
## `str`布局

字符串切片是字符的 UTF-8 表示，其布局与 `[u8]` 类型的切片相同。引用 `&str` 与引用 `&[u8]` 具有相同的布局。

r[layout.tuple]
## 元组布局

r[layout.tuple.general]
元组根据 [`Rust` 表示法][`Rust`] 进行布局。

r[layout.tuple.unit]
此处的例外是单元元组（ `()` ），它作为零大小类型，保证大小为 0，对齐方式为 1。

r[layout.trait-object]
## 特型对象布局

特型对象与该特型对象所属的值具有相同的布局。

> [!NOTE]
> 这是关于原始特型对象类型的，而不是指向特型对象的指针（ `&dyn Trait` 、 `Box<dyn Trait>` 等）。

r[layout.closure]
## 闭包布局

闭包没有布局保证。

r[layout.repr]
## 表示法

r[layout.repr.intro]
所有用户定义的复合类型（ 结构体 、 枚举 和 联合体 ）都有一个 *表示法* ，用于指定该类型的布局。

r[layout.repr.kinds]
类型可能的表示法有：

- [`Rust`] （默认）
- [`C`]
- [原语表示法][primitive representations]
- [`transparent`]

r[layout.repr.attribute]
可以通过对其应用 `repr` 属性来更改类型的表示法。以下示例显示了一个具有 `C` 表示法的结构体。

```rust
#[repr(C)]
struct ThreeInts {
    first: i16,
    second: i8,
    third: i32
}
```

r[layout.repr.align-packed]
可以使用 `align` 和 `packed` 修饰符分别提高或降低对齐方式。它们会修改属性中指定的表示法。如果没有指定表示法，则修改默认表示法。

```rust
// 默认表示法，对齐方式降至 2。
#[repr(packed(2))]
struct PackedStruct {
    first: i16,
    second: i8,
    third: i32
}

// C 表示法，对齐方式升至 8
#[repr(C, align(8))]
struct AlignedStruct {
    first: i16,
    second: i8,
    third: i32
}
```

> [!NOTE]
> 由于表示法是 项 上的属性，因此表示法不依赖于泛型参数。具有相同名称的任何两个类型都具有相同的表示法。例如，`Foo<Bar>` 和 `Foo<Baz>` 都具有相同的表示法。

r[layout.repr.inter-field]
类型的表示法可以改变字段之间的填充，但不会改变字段本身的布局。例如，一个具有 `C` 表示法且包含具有 `Rust` 表示法的结构体 `Inner` 的结构体，其内部 `Inner` 的布局不会改变。

<a id="the-default-representation"></a>
r[layout.repr.rust]
### `Rust`表示法

r[layout.repr.rust.intro]
`Rust` 表示法是没有 `repr` 属性的标称类型的默认表示法。通过 `repr` 属性显式使用此表示法与完全省略该属性的效果保证相同。

r[layout.repr.rust.layout]
此表示法做出的唯一数据布局保证是那些为了健全性所必需的。它们是：

 1. 字段正确对齐。
 2. 字段不重叠。
 3. 类型的对齐方式至少是其字段的最大对齐方式。

r[layout.repr.rust.alignment]
正式地说，第一个保证意味着任何字段的偏移量都可以被该字段的对齐方式整除。

r[layout.repr.rust.field-storage]
第二个保证意味着字段可以按序排列，使得任何字段的偏移量加上大小小于或等于排序中下一个字段的偏移量。该顺序不必与类型声明中指定字段的顺序相同。

请注意，第二个保证并不意味着字段具有不同的地址：零大小类型可能与同一结构体中的其他字段具有相同的地址。

r[layout.repr.rust.unspecified]
此表示法没有做出其他数据布局保证。

r[layout.repr.c]
### `C`表示法

r[layout.repr.c.intro]
`C` 表示法是为双重目的而设计的。一个目的是创建可与 C 语言互操作的类型。第二个目的是创建可以安全地执行依赖于数据布局的操作的类型，例如将值重新解释为不同的类型。

由于这种双重目的，可能会创建一些对于与 C 编程语言交互没有用处的类型。

r[layout.repr.c.constraint]
此表示法可以应用于结构体、联合体和枚举。例外是 [零变体枚举][zero-variant enums] ，对其使用 `C` 表示法会导致错误。

r[layout.repr.c.struct]
#### `#[repr(C)]`结构体

r[layout.repr.c.struct.align]
结构体的对齐方式是其中对齐程度最高的字段的对齐方式。

r[layout.repr.c.struct.size-field-offset]
字段的大小和偏移量由以下算法确定。

从 0 字节的当前偏移量开始。

对于结构体中按声明顺序排列的每个字段，首先确定该字段的大小和对齐方式。如果当前偏移量不是该字段对齐方式的倍数，则在当前偏移量中添加填充字节，直到它是该字段对齐方式的倍数。该字段的偏移量就是当前的偏移量。然后将当前偏移量增加该字段的大小。

最后，结构体的大小是当前偏移量向上舍入到结构体对齐方式的最近倍数。

这是用伪代码描述的该算法。

<!-- ignore: pseudocode -->
```rust,ignore
/// 返回在 `offset` 之后需要的填充量，以确保后续地址将对齐到 `alignment`。
fn padding_needed_for(offset: usize, alignment: usize) -> usize {
    let misalignment = offset % alignment;
    if misalignment > 0 {
        // 向上舍入到 `alignment` 的下一个倍数
        alignment - misalignment
    } else {
        // 已经是 `alignment` 的倍数
        0
    }
}

struct.alignment = struct.fields().map(|field| field.alignment).max();

let current_offset = 0;

for field in struct.fields_in_declaration_order() {
    // 增加当前偏移量，使其成为该字段对齐方式的倍数。
    // 对于第一个字段，这始终为零。
    // 跳过的字节被称为填充字节。
    current_offset += padding_needed_for(current_offset, field.alignment);

    struct[field].offset = current_offset;

    current_offset += field.size;
}

struct.size = current_offset + padding_needed_for(current_offset, struct.alignment);
```

> [!WARNING]
> 为了清晰起见，此伪代码使用了一种忽略溢出问题的朴素算法。要在实际代码中执行内存布局计算，请使用 [`Layout`] 。

> [!NOTE]
> 此算法可以生成零大小的结构体。在 C 语言中，像 `struct Foo { }` 这样的空结构体声明是非法的。然而，gcc 和 clang 都支持启用此类结构体的选项，并将其大小指定为零。相比之下，C++ 给空结构体的大小为 1，除非它们是被继承的，或者是具有 `[[no_unique_address]]` 属性的字段，在这种情况下它们不会增加结构体的总体大小。

r[layout.repr.c.union]
#### `#[repr(C)]`联合体

r[layout.repr.c.union.intro]
使用 `#[repr(C)]` 声明的联合体将具有与目标平台 C 语言中等效的 C 联合体声明相同的大小和对齐方式。

r[layout.repr.c.union.size-align]
联合体的大小为其所有字段的最大大小向上舍入到其对齐方式的结果，其对齐方式为其所有字段的最大对齐方式。这些最大值可能来自不同的字段。

```rust
#[repr(C)]
union Union {
    f1: u16,
    f2: [u8; 4],
}

assert_eq!(std::mem::size_of::<Union>(), 4);  // 来自 f2
assert_eq!(std::mem::align_of::<Union>(), 2); // 来自 f1

#[repr(C)]
union SizeRoundedUp {
   a: u32,
   b: [u16; 3],
}

assert_eq!(std::mem::size_of::<SizeRoundedUp>(), 8);  // 来自 b 的大小为 6，
                                                      // 根据 a 的对齐方式
                                                      // 向上舍入为 8。
assert_eq!(std::mem::align_of::<SizeRoundedUp>(), 4); // 来自 a
```

r[layout.repr.c.enum]
#### `#[repr(C)]`无字段枚举

对于 [无字段枚举][field-less enums] ， `C` 表示法具有目标平台 C ABI 的默认 `enum` 大小和对齐方式。

> [!NOTE]
> C 中的枚举表示法是实现定义的，所以这实际上是一个“最佳猜测”。特别地，当感兴趣的 C 代码使用某些标志编译时，这可能是不正确的。

> [!WARNING]
> C 语言中的 `enum` 与使用此表示法的 Rust [无字段枚举][field-less enums] 之间存在关键区别。C 中的 `enum` 大多是一个 `typedef` 加上一些命名常量；换句话说， `enum` 类型的一个对象可以持有任何整数值。例如，在 `C` 中这常用于位标志。相比之下，Rust 的 [无字段枚举][field-less enums] 只能合法地持有判别式的值，其他任何值都是 [未定义行为][undefined behavior] 。因此，在 FFI 中使用无字段枚举来建模 C 的 `enum` 通常是错误的。

r[layout.repr.c.adt]
#### `#[repr(C)]`带字段枚举

带字段的 `repr(C)` 枚举的表示法是具有两个字段的 `repr(C)` 结构体，在 C 中也称为“标签联合”：

r[layout.repr.c.adt.tag]
- 删除所有字段的枚举的 `repr(C)` 版本（“标签”）

r[layout.repr.c.adt.fields]
- 每个具有字段的变体的字段对应的 `repr(C)` 结构体的 `repr(C)` 联合体（“有效负载”）

> [!NOTE]
> 由于 `repr(C)` 结构体和联合体的表示方式，如果一个变体只有一个字段，直接将该字段放入联合体或将其包装在结构体中没有区别；因此，任何希望操作此类 `enum` 表示法的系统都可以使用对他们来说更方便或更一致的形式。

```rust
// 此 Enum 与 ... 具有相同的表示法
#[repr(C)]
enum MyEnum {
    A(u32),
    B(f32, u64),
    C { x: u32, y: u8 },
    D,
 }

// ... 此结构体。
#[repr(C)]
struct MyEnumRepr {
    tag: MyEnumDiscriminant,
    payload: MyEnumFields,
}

// 这是判别式枚举。
#[repr(C)]
enum MyEnumDiscriminant { A, B, C, D }

// 这是变体联合体。
#[repr(C)]
union MyEnumFields {
    A: MyAFields,
    B: MyBFields,
    C: MyCFields,
    D: MyDFields,
}

#[repr(C)]
#[derive(Copy, Clone)]
struct MyAFields(u32);

#[repr(C)]
#[derive(Copy, Clone)]
struct MyBFields(f32, u64);

#[repr(C)]
#[derive(Copy, Clone)]
struct MyCFields { x: u32, y: u8 }

// 此结构体可以省略（它是一个零大小类型），并且它必须存在于
// C/C++ 头文件中。
#[repr(C)]
#[derive(Copy, Clone)]
struct MyDFields;
```

r[layout.repr.primitive]
### 原语表示法

r[layout.repr.primitive.intro]
*原语表示法* 是与原语整数类型同名的表示法。即： `u8` 、 `u16` 、 `u32` 、 `u64` 、 `u128` 、 `usize` 、 `i8` 、 `i16` 、 `i32` 、 `i64` 、 `i128` 和 `isize` 。

r[layout.repr.primitive.constraint]
原语表示法只能应用于枚举，并且根据枚举是否有字段而具有不同的行为。对于 [零变体枚举][zero-variant enums] ，使用原语表示法是错误的。将两个原语表示法结合在一起是错误的。

r[layout.repr.primitive.enum]
#### 无字段枚举的原语表示法

对于 [无字段枚举][field-less enums] ，原语表示法将大小和对齐方式设置为与同名原语类型相同。例如，具有 `u8` 表示法的无字段枚举只能具有 0 到 255（含）之间的判别式。

r[layout.repr.primitive.adt]
#### 带字段枚举的原语表示法

原语表示法枚举的表示法是每个带字段变体的 `repr(C)` 结构体的 `repr(C)` 联合体。联合体中每个结构体的第一个字段是删除所有字段的枚举的原语表示法版本（“标签”），其余字段是该变体的字段。

> [!NOTE]
> 如果标签在联合体中被赋予了自己的成员，这种表示法不会改变，只要这能让你的操作更清晰（尽管为了遵循 C++ 标准，标签成员应该包装在一个 `struct` 中）。

```rust
// 此 enum 与 ... 具有相同的表示法
#[repr(u8)]
enum MyEnum {
    A(u32),
    B(f32, u64),
    C { x: u32, y: u8 },
    D,
 }

// ... 此联合体。
#[repr(C)]
union MyEnumRepr {
    A: MyVariantA,
    B: MyVariantB,
    C: MyVariantC,
    D: MyVariantD,
}

// 这是判别式枚举。
#[repr(u8)]
#[derive(Copy, Clone)]
enum MyEnumDiscriminant { A, B, C, D }

#[repr(C)]
#[derive(Clone, Copy)]
struct MyVariantA(MyEnumDiscriminant, u32);

#[repr(C)]
#[derive(Clone, Copy)]
struct MyVariantB(MyEnumDiscriminant, f32, u64);

#[repr(C)]
#[derive(Clone, Copy)]
struct MyVariantC { tag: MyEnumDiscriminant, x: u32, y: u8 }

#[repr(C)]
#[derive(Clone, Copy)]
struct MyVariantD(MyEnumDiscriminant);
```

r[layout.repr.primitive-c]
#### 将带字段枚举的原语表示法与`#[repr(C)]`结合

对于带字段的枚举，也可以将 `repr(C)` 和原语表示法结合使用（例如 `repr(C, u8)` ）。这会通过将判别式枚举的表示法更改为所选原语来修改 [`repr(C)`] 。因此，如果你选择了 `u8` 表示法，则判别式枚举的大小和对齐方式将为 1 字节。

[前面][`repr(C)`] 示例中的判别式枚举随后变为：

```rust
#[repr(C, u8)] // 添加了 `u8`
enum MyEnum {
    A(u32),
    B(f32, u64),
    C { x: u32, y: u8 },
    D,
 }

// ...

#[repr(u8)] // 所以这里使用 `u8` 代替 `C`
enum MyEnumDiscriminant { A, B, C, D }

// ...
```

例如，对于 `repr(C, u8)` 枚举，不可能有 257 个唯一的判别式（“标签”），而仅具有 `repr(C)` 属性的相同枚举编译时不会有任何问题。

除了 `repr(C)` 之外，使用原语表示法还可以改变枚举的大小（相对于 `repr(C)` 形式）：

```rust
#[repr(C)]
enum EnumC {
    Variant0(u8),
    Variant1,
}

#[repr(C, u8)]
enum Enum8 {
    Variant0(u8),
    Variant1,
}

#[repr(C, u16)]
enum Enum16 {
    Variant0(u8),
    Variant1,
}

// C 表示法的大小取决于平台
assert_eq!(std::mem::size_of::<EnumC>(), 8);
// 在 Enum8::Variant0 中，判别式占一个字节，值占一个字节
assert_eq!(std::mem::size_of::<Enum8>(), 2);
// 在 Enum16::Variant0 中，判别式占两个字节，值占一个字节，
// 加上一个字节的填充。
assert_eq!(std::mem::size_of::<Enum16>(), 4);
```

[`repr(C)`]: #reprc-enums-with-fields

r[layout.repr.alignment]
### 对齐修饰符

r[layout.repr.alignment.intro]
`align` 和 `packed` 修饰符可以分别用于提高或降低 结构体 和 联合体 的对齐方式。 `packed` 还可以改变字段之间的填充（尽管它不会改变任何字段内部的填充）。 `align` 和 `packed` 本身不提供有关结构体布局或枚举变体布局中字段顺序的保证，尽管它们可以与确实提供此类保证的表示法（如 `C` ）结合使用。

r[layout.repr.alignment.constraint-alignment]
对齐方式被指定为一个整数参数，形式为 `#[repr(align(x))]` 或 `#[repr(packed(x))]` 。对齐值必须是 1 到 2<sup>29</sup> 之间的 2 的幂。对于 `packed` ，如果没有给出值，如 `#[repr(packed)]` ，则值为 1。

r[layout.repr.alignment.align]
对于 `align` ，如果指定的对齐方式小于没有 `align` 修饰符的类型的对齐方式，则对齐方式不受影响。

r[layout.repr.alignment.packed]
对于 `packed` ，如果指定的对齐方式大于没有 `packed` 修饰符的类型的对齐方式，则对齐方式和布局不受影响。

r[layout.repr.alignment.packed-fields]
为了定位字段，每个字段的对齐方式是指定对齐方式和该字段类型对齐方式中的较小者。

r[layout.repr.alignment.packed-padding]
字段间填充保证是满足每个字段（可能已改变的）对齐方式所需的最小值（尽管请注意， `packed` 本身不提供有关字段排序的任何保证）。这些规则的一个重要结果是，具有 `#[repr(packed(1))]` （或 `#[repr(packed)]` ）的类型将没有字段间填充。

r[layout.repr.alignment.constraint-exclusive]
`align` 和 `packed` 修饰符不能应用于同一类型，并且 `packed` 类型不能传递性地包含另一个 `align` 过的类型。 `align` 和 `packed` 只能应用于 [`Rust`] 和 [`C`] 表示法。

r[layout.repr.alignment.enum]
`align` 修饰符也可以应用于 `enum` 。
应用时，对 `enum` 对齐方式的影响与该 `enum` 被包装在具有相同 `align` 修饰符的新类型 `struct` 中相同。

> [!NOTE]
> 不允许对非对齐字段进行引用，因为这是 [未定义行为][undefined behavior] 。当字段由于对齐修饰符而非对齐时，请考虑使用以下选项来进行引用和解引用：
>
> ```rust
> #[repr(packed)]
> struct Packed {
>     f1: u8,
>     f2: u16,
> }
> let mut e = Packed { f1: 1, f2: 2 };
> // 不要创建对字段的引用，而是将值复制到局部变量中。
> let x = e.f2;
> // 或者在像 `println!` 这样创建引用的情况下，使用大括号将其更改为值的副本。
> println!("{}", {e.f2});
> // 或者如果你需要一个指针，使用用于读取和写入的非对齐方法，
> // 而不是直接解构指针。
> let ptr: *const u16 = &raw const e.f2;
> let value = unsafe { ptr.read_unaligned() };
> let mut_ptr: *mut u16 = &raw mut e.f2;
> unsafe { mut_ptr.write_unaligned(3) }
> ```

r[layout.repr.transparent]
### `transparent`表示法

r[layout.repr.transparent.constraint-field]
`transparent` 表示法只能用于 [结构体][structs] 或具有单个变体的 [枚举][enumerations] ，该变体具有：
- 任意数量的大小为 0 且对齐方式为 1 的字段（例如 [`PhantomData<T>`] ），以及
- 至多一个其他字段。

r[layout.repr.transparent.layout-abi]
具有此表示法的结构体和枚举具有与唯一的非大小 0 非对齐 1 字段（如果存在）相同的布局和 ABI，否则为单元布局。

这与 `C` 表示法不同，因为具有 `C` 表示法的结构体始终具有 `C` `struct` 的 ABI，而具有 `transparent` 表示法且具有原语字段的结构体将具有该原语字段的 ABI。

r[layout.repr.transparent.constraint-exclusive]
由于这种表示法将类型布局委托给另一种类型，因此它不能与任何其他表示法一起使用。

[`align_of_val`]: std::mem::align_of_val
[`size_of_val`]: std::mem::size_of_val
[`align_of`]: std::mem::align_of
[`size_of`]: std::mem::size_of
[`Sized`]: std::marker::Sized
[`Copy`]: std::marker::Copy
[dynamically sized types]: dynamically-sized-types.md
[field-less enums]: items/enumerations.md#field-less-enum
[fn-abi-compatibility]: ../core/primitive.fn.md#abi-compatibility
[enumerations]: items/enumerations.md
[zero-variant enums]: items/enumerations.md#zero-variant-enums
[undefined behavior]: behavior-considered-undefined.md
[`PhantomData<T>`]: special-types-and-traits.md#phantomdatat
[`Rust`]: #rust表示法
[`C`]: #c表示法
[primitive representations]: #原语表示法
[structs]: items/structs.md
[`transparent`]: #transparent表示法
[`Layout`]: std::alloc::Layout
