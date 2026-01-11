<!-- https://github.com/rust-lang/reference/blob/master/src/types/textual.md -->
<!-- commit 68bdfd15fa9feebfbf94a06cf574de598e239198 -->

r[type.text]
# 文本类型

r[type.text.intro]
`char` 和 `str` 类型保存文本数据。

r[type.text.char-value]
`char` 类型的值是一个 [Unicode 标量值][Unicode scalar value] （即不是代理对（surrogate）的码位），表示为 0x0000 到 0xD7FF 或 0xE000 到 0x10FFFF 范围内的 32 位无符号字。

r[type.text.char-precondition]
创建一个超出此范围的 `char` 会立即导致 [未定义行为][undefined behavior] 。 `[char]` 实际上是一个长度为 1 的 UCS-4 / UTF-32 字符串。

r[type.text.str-value]
`str` 类型的值的表示方式与 `[u8]` 相同，即 8 位无符号字节的切片。然而，Rust 标准库对 `str` 做了额外的假设：在 `str` 上操作的方法假定并确保其中的数据是有效的 UTF-8。使用非 UTF-8 缓冲区调用 `str` 方法可能会在现在或将来导致 [未定义行为][undefined behavior] 。

r[type.text.str-unsized]
由于 `str` 是 [动态大小类型][dynamically sized type] ，它只能通过指针类型实例化，例如 `&str` 。 `&str` 的布局与 `&[u8]` 的布局相同。

r[type.text.layout]
## 布局和位有效性

r[type.layout.char-layout]
`char` 在所有平台上保证具有与 `u32` 相同的大小和对齐。

r[type.layout.char-validity]
`char` 的每个字节都保证被初始化（换句话说， `transmute::<char, [u8; size_of::<char>()]>(...)` 总是健全的——但由于某些位模式是无效的 `char` ，反向操作并不总是健全的）。

[Unicode scalar value]: http://www.unicode.org/glossary/#unicode_scalar_value
[undefined behavior]: ../behavior-considered-undefined.md
[dynamically sized type]: ../dynamically-sized-types.md
