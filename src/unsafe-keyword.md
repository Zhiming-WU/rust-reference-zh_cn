r[unsafe]
# `unsafe` 关键字

r[unsafe.intro]
`unsafe` 关键字用于创建或履行证明某事是安全的义务。具体来说：

- 它用于标记 *定义* 了必须在其他地方遵守的额外安全条件的代码。
  - 这包括 `unsafe fn`、`unsafe static` 和 `unsafe trait`。
- 它用于标记程序员 *断言* 满足了在其他地方定义的安全条件的代码。
  - 这包括 `unsafe {}`、`unsafe impl`、不带 [`unsafe_op_in_unsafe_fn`] 的 `unsafe fn`、`unsafe extern` 和 `#[unsafe(attr)]`。

下面将讨论这些情况。
请参阅 [关键字][keyword] 文档中的一些示例。

r[unsafe.positions]
`unsafe` 关键字可以出现在几种不同的上下文中：

- 不安全函数 (`unsafe fn`)
- 不安全块 (`unsafe {}`)
- 不安全特型 (`unsafe trait`)
- 不安全特型实现 (`unsafe impl`)
- 不安全外部块 (`unsafe extern`)
- 不安全外部静态项 (`unsafe static`)
- 不安全属性 (`#[unsafe(attr)]`)

r[unsafe.fn]
## 不安全函数 (`unsafe fn`)

r[unsafe.fn.intro]
不安全函数是在所有上下文和/或所有可能的输入下都不安全的函数。
我们说它们具有 *额外安全条件* ，这些是所有调用者必须遵守的要求，且编译器不会检查。
例如，[`get_unchecked`] 具有索引必须在范围内的额外安全条件。
不安全函数应当附带说明这些额外安全条件的文档。

r[unsafe.fn.safety]
此类函数必须带有 `unsafe` 关键字前缀，且只能从 `unsafe` 块内部调用，或者在没有启用 [`unsafe_op_in_unsafe_fn`] lint 的 `unsafe fn` 内部调用。

r[unsafe.block]
## 不安全块 (`unsafe {}`)

r[unsafe.block.intro]
代码块可以带有 `unsafe` 关键字前缀，以允许使用 [不安全性][Unsafety] 一章中定义的各种不安全操作，例如调用其他不安全函数或解引用原始指针。

r[unsafe.block.fn-body]
默认情况下，不安全函数的主体也被认为是一个不安全块；
这可以通过启用 [`unsafe_op_in_unsafe_fn`] lint 来更改。

通过将操作放入不安全块中，程序员声明他们已经负责满足该块内所有操作的额外安全条件。

不安全块是不安全函数的逻辑对等物：
不安全函数定义了调用者必须遵守的证明义务，而不安全块则表示块内调用的函数或操作的所有相关证明义务都已履行。
履行证明义务的方法有很多；
例如，可以有运行时检查或数据结构不变性，保证某些属性绝对为真；或者不安全块可能位于 `unsafe fn` 内部，在这种情况下，该块可以使用该函数的证明义务来履行块内产生的证明义务。

不安全块用于包装外部库、直接使用硬件或实现语言中没有直接提供的功能。
例如，Rust 提供了实现内存安全并发所需的语言特性，但标准库中线程和消息传递的实现使用了不安全块。

Rust 的类型系统是对动态安全要求的保守近似，因此在某些情况下，使用安全代码会有性能代价。
例如，双向链表不是树结构，在安全代码中只能用引用计数指针表示。
通过使用 `unsafe` 块将反向链接表示为原始指针，可以在不使用引用计数的情况下实现它。
（有关此特定示例的更深入探索，请参阅 ["Learn Rust With Entirely Too Many Linked Lists"](https://rust-unofficial.github.io/too-many-lists/)。）

[Unsafety]: unsafety.md

r[unsafe.trait]
## 不安全特型 (`unsafe trait`)

r[unsafe.trait.intro]
不安全特型是附带额外安全条件的特型，这些条件必须由该特型的 *实现* 遵守。
不安全特型应当附带说明这些额外安全条件的文档。

r[unsafe.trait.safety]
此类特型必须带有 `unsafe` 关键字前缀，且只能由 `unsafe impl` 块实现。

r[unsafe.impl]
## 不安全特型实现 (`unsafe impl`)

在实现不安全特型时，实现需要带有 `unsafe` 关键字前缀。
通过编写 `unsafe impl`，程序员声明他们已经负责满足特型所要求的额外安全条件。

不安全特型实现是不安全特型的逻辑对等物：不安全特型定义了实现必须遵守的证明义务，而不安全实现则表示所有相关的证明义务都已履行。

[keyword]: ../std/keyword.unsafe.html
[`get_unchecked`]: slice::get_unchecked
[`unsafe_op_in_unsafe_fn`]: ../rustc/lints/listing/allowed-by-default.html#unsafe-op-in-unsafe-fn

r[unsafe.extern]
## 不安全外部块 (`unsafe extern`)

声明 [外部块][external block] 的程序员必须确保其中包含的项的签名是正确的。如果不这样做，可能会导致未定义行为。通过编写 `unsafe extern` 来表明这一义务已经履行。

r[unsafe.extern.edition2024]
> [!EDITION-2024]
> 在 2024 版次 之前，允许在没有 `unsafe` 限定的情况下使用 `extern` 块。

[external block]: items/external-blocks.md

r[unsafe.attribute]
## 不安全属性 (`#[unsafe(attr)]`)

[不安全属性][unsafe attribute] 是指在使用该属性时必须遵守额外安全条件的属性。编译器无法检查这些条件是否已得到遵守。为了断言它们已得到遵守，这些属性必须包装在 `unsafe(..)` 中，例如 `#[unsafe(no_mangle)]`。

[unsafe attribute]: attributes.md
