<!-- https://github.com/rust-lang/reference/blob/master/src/conditional-compilation.md -->
<!-- commit 68bdfd15fa9feebfbf94a06cf574de598e239198 -->

r[cfg]
# 条件编译

r[cfg.syntax]
```grammar,configuration
ConfigurationPredicate ->
      ConfigurationOption
    | ConfigurationAll
    | ConfigurationAny
    | ConfigurationNot
    | `true`
    | `false`

ConfigurationOption ->
    IDENTIFIER ( `=` ( STRING_LITERAL | RAW_STRING_LITERAL ) )?

ConfigurationAll ->
    `all` `(` ConfigurationPredicateList? `)`

ConfigurationAny ->
    `any` `(` ConfigurationPredicateList? `)`

ConfigurationNot ->
    `not` `(` ConfigurationPredicate `)`

ConfigurationPredicateList ->
    ConfigurationPredicate (`,` ConfigurationPredicate)* `,`?
```

r[cfg.general]
*条件编译的源代码*是指仅在特定条件下才会被编译的源代码。

r[cfg.attributes-macro]
可以使用[`cfg`]和[`cfg_attr`] [属性][attributes]以及内置的[`cfg!宏`][`cfg` macro]来使源代码条件编译。

r[cfg.conditional]
是否进行编译可以取决于被编译的crate的目标架构、传递给编译器的任意值，以及下面进一步描述的其他因素。

r[cfg.predicate]
每种形式的条件编译都接受一个 _配置断言_ ，该断言求值为真或假。该断言是以下形式之一：

r[cfg.predicate.option]
* 一个配置选项。如果该选项已设置，则断言为真；如果未设置，则为假。

r[cfg.predicate.all]
* `all()`，带有一个逗号分隔的配置断言列表。如果所有给定的断言都为真，或者列表为空，则它为真。

r[cfg.predicate.any]
* `any()`，带有一个逗号分隔的配置断言列表。如果至少有一个给定的断言为真，则它为真。如果没有断言，则它为假。

r[cfg.predicate.not]
* `not()`，带有一个配置断言。如果其断言为假，则它为真；如果其断言为真，则它为假。

r[cfg.predicate.literal]
* `true`或`false`字面量，分别总是为真或假。

r[cfg.option-spec]
_配置选项_ 可以是名称，也可以是键值对，它们要么已设置，要么未设置。

r[cfg.option-name]
名称写作单个标识符，例如`unix`。

r[cfg.option-key-value]
键值对写作一个标识符、`=`，然后是一个字符串，例如`target_arch = "x86_64"`。

> [!NOTE]
> `=`周围的空白符会被忽略，所以`foo="bar"`和`foo = "bar"`是等效的。

r[cfg.option-key-uniqueness]
键不需要是唯一的。例如，`feature = "std"`和`feature = "serde"`可以同时设置。

r[cfg.options.set]
## 配置选项设置

r[cfg.options.general]
哪些配置选项被设置是在crate编译期间静态确定的。

r[cfg.options.target]
一些选项是 _编译器设置的_，基于编译的相关数据。

r[cfg.options.other]
其他选项是 _任意设置的_，基于代码外部传递给编译器的输入。

r[cfg.options.crate]
无法在被编译的crate的源代码内部设置配置选项。

> [!NOTE]
> 对于`rustc`，任意设置的配置选项使用[`--cfg`]标志设置。指定目标的配置值可以使用`rustc --print cfg --target $TARGET`显示。
>
> 键为`feature`的配置选项是[Cargo][cargo-feature]用于指定编译时选项和可选依赖项的约定。

r[cfg.target_arch]
### `target_arch`

r[cfg.target_arch.gen]
键值选项，设置为目标CPU架构一次。该值类似于平台目标三元组的第一个元素，但不完全相同。

r[cfg.target_arch.values]
示例值：

*   `"x86"`
*   `"x86_64"`
*   `"mips"`
*   `"powerpc"`
*   `"powerpc64"`
*   `"arm"`
*   `"aarch64"`

r[cfg.target_feature]
### `target_feature`

r[cfg.target_feature.general]
键值选项，为当前编译目标可用的每个平台特性设置。

r[cfg.target_feature.values]
示例值：

*   `"avx"`
*   `"avx2"`
*   `"crt-static"`
*   `"rdrand"`
*   `"sse"`
*   `"sse2"`
*   `"sse4.1"`

有关可用特性的更多详细信息，请参阅[`target_feature`属性][`target_feature` attribute]。

r[cfg.target_feature.crt_static]
`target_feature`选项还提供了一个额外的`crt-static`特性，用于指示[静态 C 运行时][static C runtime]是否可用。

r[cfg.target_os]
### `target_os`

r[cfg.target_os.general]
键值选项，设置为目标的操作系统一次。该值类似于平台目标三元组的第二个和第三个元素。

r[cfg.target_os.values]
示例值：

*   `"windows"`
*   `"macos"`
*   `"ios"`
*   `"linux"`
*   `"android"`
*   `"freebsd"`
*   `"dragonfly"`
*   `"openbsd"`
*   `"netbsd"`
*   `"none"`（通常用于嵌入式目标）

r[cfg.target_family]
### `target_family`

r[cfg.target_family.general]
键值选项，提供目标的更通用描述，例如目标通常所属的操作系统或架构家族。可以设置任意数量的`target_family`键值对。

r[cfg.target_family.values]
示例值：

*   `"unix"`
*   `"windows"`
*   `"wasm"`
*   `"unix"`和`"wasm"`两者

r[cfg.target_family.unix]
### `unix`和`windows`

如果设置了`target_family = "unix"`，则`unix`被设置。

r[cfg.target_family.windows]
如果设置了`target_family = "windows"`，则`windows`被设置。

r[cfg.target_env]
### `target_env`

r[cfg.target_env.general]
键值选项，包含关于目标平台ABI或`libc`的进一步区分信息。出于历史原因，此值仅在实际需要区分时才定义为非空字符串。因此，例如，在许多GNU平台上，此值将为空。此值类似于平台目标三元组的第四个元素。一个区别是，像`gnueabihf`这样的嵌入式ABI将简单地将`target_env`定义为`"gnu"`。

r[cfg.target_env.values]
示例值：

*   `""`
*   `"gnu"`
*   `"msvc"`
*   `"musl"`
*   `"sgx"`
*   `"sim"`
*   `"macabi"`

r[cfg.target_abi]
### `target_abi`

r[cfg.target_abi.general]
键值选项，包含关于目标ABI的进一步区分信息。

r[cfg.target_abi.disambiguation]
出于历史原因，此值仅在实际需要区分时才定义为非空字符串。因此，例如，在许多GNU平台上，此值将为空。

r[cfg.target_abi.values]
示例值：

*   `""`
*   `"llvm"`
*   `"eabihf"`
*   `"abi64"`

r[cfg.target_endian]
### `target_endian`

键值选项，根据目标CPU的字节序设置为`"little"`或`"big"`。

r[cfg.target_pointer_width]
### `target_pointer_width`

r[cfg.target_pointer_width.general]
键值选项，设置为目标指针宽度（以位为单位）一次。

r[cfg.target_pointer_width.values]
示例值：

*   `"16"`
*   `"32"`
*   `"64"`

r[cfg.target_vendor]
### `target_vendor`

r[cfg.target_vendor.general]
键值选项，设置为目标供应商一次。

r[cfg.target_vendor.values]
示例值：

*   `"apple"`
*   `"fortanix"`
*   `"pc"`
*   `"unknown"`

r[cfg.target_has_atomic]
### `target_has_atomic`

r[cfg.target_has_atomic.general]
键值选项，为目标支持原子加载、存储和比较并交换操作的每个位宽设置。

r[cfg.target_has_atomic.stdlib]
当此cfg存在时，所有稳定的[`core::sync::atomic`](https://doc.rust-lang.org/core/sync/atomic/index.html)API都可用于相关的原子位宽。

r[cfg.target_has_atomic.values]
可能的值：

*   `"8"`
*   `"16"`
*   `"32"`
*   `"64"`
*   `"128"`
*   `"ptr"`

r[cfg.test]
### `test`

在编译测试harness时启用。通过`rustc`使用[`--test`]标志来完成。有关测试支持的更多信息，请参见[测试][Testing]。

r[cfg.debug_assertions]
### `debug_assertions`

默认情况下在没有优化的情况下编译时启用。这可用于在开发中启用额外的调试代码，但在生产中不启用。例如，它控制标准库的[`debug_assert!`](https://doc.rust-lang.org/core/macro.debug_assert.html)宏的行为。

r[cfg.proc_macro]
### `proc_macro`

当被编译的crate使用`proc_macro` [crate类型][crate type]编译时设置。

r[cfg.panic]
### `panic`

r[cfg.panic.general]
键值选项，根据[恐慌策略][panic strategy]设置。请注意，将来可能会添加更多值。

r[cfg.panic.values]
示例值：

*   `"abort"`
*   `"unwind"`

[panic strategy]: panic.md#panic-strategy

## 条件编译的形式

<!-- template:attributes -->
r[cfg.attr]
### `cfg`属性

r[cfg.attr.intro]
*`cfg`[属性][attribute]* 根据配置断言有条件地包含其所附加的形式。

> [!EXAMPLE]
> ```rust
> // 该函数只在为 macOS 编译时才会被包含在构建中
> #[cfg(target_os = "macos")]
> fn macos_only() {
>   // ...
> }
>
> // 该函数只在 foo 或 bar 被定义时才会被包含
> #[cfg(any(foo, bar))]
> fn needs_foo_or_bar() {
>   // ...
> }
>
> // 该函数只在为 32 位架构的类 Unix 操作系统编译时才会被包含
> #[cfg(all(unix, target_pointer_width = "32"))]
> fn on_32bit_unix() {
>   // ...
> }
>
> // 该函数只在 foo 未定义时才会被包含
> #[cfg(not(foo))]
> fn needs_not_foo() {
>   // ...
> }
>
> // 该函数只在恐慌策略设置为 unwound 时才会被包含
> #[cfg(panic = "unwind")]
> fn when_unwinding() {
>   // ...
> }
> ```

r[cfg.attr.syntax]
`cfg`属性的语法是：

```grammar,configuration
@root CfgAttribute -> `cfg` `(` ConfigurationPredicate `)`
```

r[cfg.attr.allowed-positions]
`cfg`属性可以在允许属性的任何地方使用。

r[cfg.attr.duplicates]
`cfg`属性可以用于一个形式任意多次。如果任何`cfg`断言为假，则附加了这些属性的形式将不会被包含，[cfg.attr.crate-level-attrs]中描述的情况除外。

r[cfg.attr.effect]
如果断言为真，则该形式将被重写为不带`cfg`属性。如果任何断言为假，则该形式将从源代码中删除。

r[cfg.attr.crate-level-attrs]
当crate级别的`cfg`有一个假断言时，crate本身仍然存在。任何在`cfg`之前的crate属性会被保留，而任何在`cfg`之后的crate属性以及所有后续的crate内容都会被移除。

> [!EXAMPLE]
> 不移除前面属性的行为允许您执行诸如包含`#![no_std]`以避免链接`std`之类的操作，即使`#![cfg(...)]`已经移除了crate的内容。例如：
>
> <!-- ignore: test infrastructure can't handle no_std -->
> ```rust,ignore
> // 即使 crate 级别的 `cfg` 属性为 false，此 `no_std` 属性也会被保留。
> #![no_std]
> #![cfg(false)]
>
> // 该函数未被包含。
> pub fn example() {}
> ```

<!-- template:attributes -->
r[cfg.cfg_attr]
### `cfg_attr`属性

r[cfg.cfg_attr.intro]
*`cfg_attr`[属性][attribute]* 根据配置断言有条件地包含属性。

> [!EXAMPLE]
> 以下模块将根据目标平台，位于`linux.rs`或`windows.rs`。
>
> <!-- ignore: `mod` needs multiple files -->
> ```rust,ignore
> #[cfg_attr(target_os = "linux", path = "linux.rs")]
> #[cfg_attr(windows, path = "windows.rs")]
> mod os;
> ```

r[cfg.cfg_attr.syntax]
`cfg_attr`属性的语法是：

```grammar,configuration
@root CfgAttrAttribute -> `cfg_attr` `(` ConfigurationPredicate `,` CfgAttrs? `)`

CfgAttrs -> Attr (`,` Attr)* `,`?
```

r[cfg.cfg_attr.allowed-positions]
`cfg_attr`属性可以在允许属性的任何地方使用。

r[cfg.cfg_attr.duplicates]
`cfg_attr`属性可以用于一个形式任意多次。

r[cfg.cfg_attr.attr-restriction]
[`crate_type`]和[`crate_name`]属性不能与`cfg_attr`一起使用。

r[cfg.cfg_attr.behavior]
当配置断言为真时，`cfg_attr`会展开为断言后列出的属性。

r[cfg.cfg_attr.attribute-list]
可以列出零个、一个或多个属性。多个属性将分别展开为单独的属性。

> [!EXAMPLE]
> <!-- ignore: fake attributes -->
> ```rust,ignore
> #[cfg_attr(feature = "magic", sparkles, crackles)]
> fn bewitched() {}
>
> // 当`magic`特性标志被启用时，上面代码将展开为：
> #[sparkles]
> #[crackles]
> fn bewitched() {}
> ```

> [!NOTE]
> `cfg_attr`可以展开为另一个`cfg_attr`。例如，`#[cfg_attr(target_os = "linux", cfg_attr(feature = "multithreaded", some_other_attribute))]`是有效的。这个例子等同于`#[cfg_attr(all(target_os = "linux", feature = "multithreaded"), some_other_attribute)]`。

r[cfg.macro]
### `cfg!宏`

内置的`cfg!宏`接受一个配置断言，并在断言为真时求值为`true`字面量，在断言为假时求值为`false`字面量。

例如：

```rust
let machine_kind = if cfg!(unix) {
  "unix"
} else if cfg!(windows) {
  "windows"
} else {
  "unknown"
};

println!("I'm running on a {} machine!", machine_kind);
```

[Testing]: attributes/testing.md
[`--cfg`]: https://doc.rust-lang.org/rustc/command-line-arguments.html#--cfg-configure-the-compilation-environment
[`--test`]: https://doc.rust-lang.org/rustc/command-line-arguments.html#--test-build-a-test-harness
[`cfg`]: #cfg属性
[`cfg` macro]: #cfg宏
[`cfg_attr`]: #cfg_attr属性
[`crate_name`]: crates-and-source-files.md#crate_name属性
[`crate_type`]: linkage.md
[`target_feature` attribute]: attributes/codegen.md#the-target_feature-attribute
[attribute]: attributes.md
[attributes]: attributes.md
[cargo-feature]: https://doc.rust-lang.org/cargo/reference/features.html
[crate type]: linkage.md
[static C runtime]: linkage.md#static-and-dynamic-c-runtimes
