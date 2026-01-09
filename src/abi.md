r[abi]
# 应用二进制接口 (ABI)

r[abi.intro]
本节记录了影响 crate 编译输出 ABI 的特性。

有关指定导出函数 ABI 的信息，请参阅 *[外部函数][extern functions]*。有关指定链接外部库 ABI 的信息，请参阅 *[外部块][external blocks]*。

r[abi.used]
## used属性

r[abi.used.intro]
*`used` 属性* 只能应用于 [`static` 项][`static` items]。此 [属性][attribute] 强制编译器在输出的目标文件（.o、.rlib 等，不包括最终的可执行文件）中保留变量，即使该变量未被 crate 中的任何其他项使用或引用。然而，链接器仍然可以自由地删除这样的项。

下面是一个示例，展示了编译器在什么条件下会将 `static` 项保留在输出的目标文件中。

``` rust
// foo.rs

// 即使未使用，也因为 #[used] 而被保留：
#[used]
static FOO: u32 = 0;

// 因为未使用而可以被删除：
#[allow(dead_code)]
static BAR: u32 = 0;

// 因为是公共可达的而被保留：
pub static BAZ: u32 = 0;

// 因为被一个公共可达的函数引用而被保留：
static QUUX: u32 = 0;

pub fn quux() -> &'static u32 {
    &QUUX
}

// 因为被一个私有的、未使用的（死代码）函数引用而可以被删除：
static CORGE: u32 = 0;

#[allow(dead_code)]
fn corge() -> &'static u32 {
    &CORGE
}
```

``` console
$ rustc -O --emit=obj --crate-type=rlib foo.rs

$ nm -C foo.o
0000000000000000 R foo::BAZ
0000000000000000 r foo::FOO
0000000000000000 R foo::QUUX
0000000000000000 T foo::quux
```

r[abi.no_mangle]
## no_mangle属性

r[abi.no_mangle.intro]
*`no_mangle` 属性* 可用于任何 [项][item] 以禁用标准的符号名混淆。该项的符号将是其名称的标识符。

r[abi.no_mangle.publicly-exported]
此外，该项将从生成的库或目标文件中公开导出，类似于 [`used` 属性](#the-used-attribute)。

r[abi.no_mangle.unsafe]
此属性是不安全的，因为不混淆的符号可能会与另一个具有相同名称的符号（或与众所周知的符号）发生冲突，从而导致未定义行为。

```rust
#[unsafe(no_mangle)]
extern "C" fn foo() {}
```

r[abi.no_mangle.edition2024]
> [!EDITION-2024]
> 在 2024 版次 之前，允许不带 `unsafe` 限定地使用 `no_mangle` 属性。

r[abi.link_section]
## link_section属性

r[abi.link_section.intro]
*`link_section` 属性* 指定了 [函数][function] 或 [static][static] 内容将放置在目标文件的哪个节中。

r[abi.link_section.syntax]
`link_section` 属性使用 [MetaNameValueStr] 语法来指定节的名称。

<!-- no_run: don't link. The format of the section name is platform-specific. -->
```rust,no_run
#[unsafe(no_mangle)]
#[unsafe(link_section = ".example_section")]
pub static VAR1: u32 = 1;
```

r[abi.link_section.unsafe]
此属性是不安全的，因为它允许用户将数据和代码放置在不期望它们的内存节中，例如将可变数据放置在只读区域中。

r[abi.link_section.edition2024]
> [!EDITION-2024]
> 在 2024 版次 之前，允许不带 `unsafe` 限定地使用 `link_section` 属性。

r[abi.export_name]
## export_name属性

r[abi.export_name.intro]
*`export_name` 属性* 指定了将在 [函数][function] 或 [static][static] 上导出的符号名称。

r[abi.export_name.syntax]
`export_name` 属性使用 [MetaNameValueStr] 语法来指定符号名称。

```rust
#[unsafe(export_name = "exported_symbol_name")]
pub fn name_in_rust() { }
```

r[abi.export_name.unsafe]
此属性是不安全的，因为具有自定义名称的符号可能会与另一个具有相同名称的符号（或与众所周知的符号）发生冲突，从而导致未定义行为。

r[abi.export_name.edition2024]
> [!EDITION-2024]
> 在 2024 版次 之前，允许不带 `unsafe` 限定地使用 `export_name` 属性。

[`static` items]: items/static-items.md
[attribute]: attributes.md
[extern functions]: items/functions.md#extern-function-qualifier
[external blocks]: items/external-blocks.md
[function]: items/functions.md
[item]: items.md
[static]: items/static-items.md
