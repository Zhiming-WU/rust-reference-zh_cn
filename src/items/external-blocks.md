<!-- https://github.com/rust-lang/reference/blob/master/src/items/external-blocks.md -->
<!-- commit 68bdfd15fa9feebfbf94a06cf574de598e239198 -->

r[items.extern]
# 外部块

r[items.extern.syntax]
```grammar,项
ExternBlock ->
    `unsafe`?[^unsafe-2024] `extern` Abi? `{`
        InnerAttribute*
        ExternalItem*
    `}`

ExternalItem ->
    OuterAttribute* (
        MacroInvocationSemi
      | Visibility? StaticItem
      | Visibility? Function
    )
```

[^unsafe-2024]: 从 2024 版次 开始，`unsafe` 关键字在语义上是必需的。

r[items.extern.intro]
外部块提供在当前 crate 中未  *定义*  而是  *声明*  的 项 的声明，是 Rust 外部函数接口 (FFI) 的基础。这些类似于未经检查的导入。

r[items.extern.allowed-kinds]
外部块中允许两种 项 *声明* ： [函数][functions] 和 [静态项][statics]。

r[items.extern.safety]
仅允许在 [`unsafe` 上下文][`unsafe` context] 中调用在外部块中声明的 不安全 函数或访问 不安全 静态项。

r[items.extern.namespace]
外部块在其所在的模块或块的 [值命名空间][value namespace] 中定义其函数和静态项。

r[items.extern.unsafe-required]
从语义上讲，`unsafe` 关键字必须出现在外部块的 `extern` 关键字之前。

r[items.extern.edition2024]
> [!EDITION-2024]
> 在 2024 版次 之前，`unsafe` 关键字是可选的。只有当外部块本身被标记为 `unsafe` 时，才允许使用 `safe` 和 `unsafe` 项限定符。

r[items.extern.fn]
## 函数

r[items.extern.fn.body]
外部块中的函数声明方式与其它 Rust 函数相同，不同之处在于它们不能有函数体，而是以分号结尾。

r[items.extern.fn.param-patterns]
参数中不允许使用模式，只能使用 [IDENTIFIER] 或 `_`。

r[items.extern.fn.qualifiers]
允许使用 `safe` 和 `unsafe` 函数限定符，但不允许使用其它函数限定符（例如 `const`、`async`、`extern`）。

r[items.extern.fn.foreign-abi]
外部块中的函数可以被 Rust 代码调用，就像在 Rust 中定义的函数一样。Rust 编译器会自动在 Rust ABI 和外部 ABI 之间进行转换。

r[items.extern.fn.safety]
在 extern 块中声明的函数隐式地是 `unsafe` 的，除非存在 `safe` 函数限定符。

r[items.extern.fn.fn-ptr]
当强制转换为函数指针时，在 extern 块中声明的函数具有类型 `extern "abi" for<'l1, ..., 'lm> fn(A1, ..., An) -> R`，其中 `'l1`、... `'lm` 是其生命周期参数，`A1`、...、`An` 是其参数的声明类型，而 `R` 是声明的返回类型。

r[items.extern.static]
## 静态项

r[items.extern.static.intro]
外部块中的 静态项 的声明方式与外部块之外的 [静态项][statics] 相同，不同之处在于它们没有初始化其值的表达式。

r[items.extern.static.safety]
除非在 extern 块中声明的 静态项 被限定为 `safe`，否则访问该 项 是 `unsafe` 的，无论它是否可变，因为由于某些任意（例如 C）代码负责初始化该 静态项，因此没有任何东西能保证该 静态项 内存处的位模式对于声明它的类型是有效的。

r[items.extern.static.mut]
外部 静态项 既可以是不可变的，也可以是可变的，就像外部块之外的 [静态项][statics] 一样。

r[items.extern.static.read-only]
在执行任何 Rust 代码之前， *必须*  初始化不可变 静态项。仅在 Rust 代码从中读取之前初始化 静态项 是不够的。一旦 Rust 代码运行，修改不可变 静态项（从 Rust 内部或外部）就是 未定义行为 (UB)，除非修改发生在 `UnsafeCell` 内部的字节上。

r[items.extern.abi]
## ABI

r[items.extern.abi.intro]
`extern` 关键字后面可以跟一个可选的 [ABI][ABI] 字符串。ABI 指定了块中函数的调用约定。调用约定定义了函数的低级接口，例如参数如何放置在寄存器或栈中，返回值如何传递，以及谁负责清理栈。

> [!EXAMPLE]
> ```rust
> // Windows API 接口。
> unsafe extern "system" { /* ... */ }
> ```

r[items.extern.abi.default]
如果未指定 ABI 字符串，则默认为 `"C"`。

> [!NOTE]
> 不带显式 ABI 的 `extern` 语法正在逐步淘汰，因此最好始终显式写入 ABI。
>
> 有关更多详细信息，请参阅 [Rust 问题 #134986](https://github.com/rust-lang/rust/issues/134986)。

r[items.extern.abi.standard]
所有平台都支持以下 ABI 字符串：

r[items.extern.abi.rust]
* `unsafe extern "Rust"` --- Rust 函数和闭包的原生调用约定。当声明函数而不使用 [`extern fn`][`extern fn`] 时，这是默认值。Rust ABI 不提供稳定性保证。

r[items.extern.abi.c]
* `unsafe extern "C"` --- "C" ABI 与目标平台的主流 C 编译器选择的默认 ABI 匹配。

r[items.extern.abi.system]
* `unsafe extern "system"` --- 这等同于 `extern "C"`，但在 Windows x86_32 上除外，对于非变长参数函数，它等同于 `"stdcall"`，而对于变长参数函数，它等同于 `"C"`。

  > [!NOTE]
  > 由于 Windows 上正确的底层 ABI 与目标平台相关，因此在尝试链接不使用显式定义 ABI 的 Windows API 函数时，最好使用 `extern "system"`。

r[items.extern.abi.unwind]
* `extern "C-unwind"` 和 `extern "system-unwind"` --- 分别与 `"C"` 和 `"system"` 相同，但在被调用者展开（通过 恐慌 或抛出 C++ 风格异常）时具有 [不同的行为][unwind-behavior]。

r[items.extern.abi.platform]
还有一些平台特定的 ABI 字符串：

r[items.extern.abi.cdecl]
* `unsafe extern "cdecl"` --- 通常用于 x86_32 C 代码的调用约定。
  * 仅适用于 x86_32 目标。
  * 对应于 MSVC 的 `__cdecl` 以及 GCC 和 clang 的 `__attribute__((cdecl))`。

  > [!NOTE]
  > 有关详细信息，请参阅：
  >
  > - <https://learn.microsoft.com/en-us/cpp/cpp/cdecl>
  > - <https://en.wikipedia.org/wiki/X86_calling_conventions#cdecl>

r[items.extern.abi.stdcall]
* `unsafe extern "stdcall"` --- 通常由 x86_32 上的 [Win32 API][win32 api] 使用的调用约定。
  * 仅适用于 x86_32 目标。
  * 对应于 MSVC 的 `__stdcall` 以及 GCC 和 clang 的 `__attribute__((stdcall))`。

  > [!NOTE]
  > 有关详细信息，请参阅：
  >
  > - <https://learn.microsoft.com/en-us/cpp/cpp/stdcall>
  > - <https://en.wikipedia.org/wiki/X86_calling_conventions#stdcall>

r[items.extern.abi.win64]
* `unsafe extern "win64"` --- Windows x64 ABI。
  * 仅适用于 x86_64 目标。
  * "win64" 与 Windows x86_64 目标上的 "C" ABI 相同。
  * 对应于 GCC 和 clang 的 `__attribute__((ms_abi))`。

  > [!NOTE]
  > 有关详细信息，请参阅：
  >
  > - <https://learn.microsoft.com/en-us/cpp/build/x64-software-conventions>
  > - <https://en.wikipedia.org/wiki/X86_calling_conventions#Microsoft_x64_calling_convention>

r[items.extern.abi.sysv64]
* `unsafe extern "sysv64"` --- System V ABI。
  * 仅适用于 x86_64 目标。
  * "sysv64" 与非 Windows x86_64 目标上的 "C" ABI 相同。
  * 对应于 GCC 和 clang 的 `__attribute__((sysv_abi))`。

  > [!NOTE]
  > 有关详细信息，请参阅：
  >
  > - <https://wiki.osdev.org/System_V_ABI>
  > - <https://en.wikipedia.org/wiki/X86_calling_conventions#System_V_AMD64_ABI>

r[items.extern.abi.aapcs]
* `unsafe extern "aapcs"` --- ARM 的软浮点 ABI。
  * 仅适用于 ARM32 目标。
  * "aapcs" 与软浮点 ARM32 上的 "C" ABI 相同。
  * 对应于 clang 的 `__attribute__((pcs("aapcs")))`。

  > [!NOTE]
  > 有关详细信息，请参阅：
  >
  > - [Arm 过程调用标准](https://developer.arm.com/documentation/107656/0101/Getting-started-with-Armv8-M-based-systems/Procedure-Call-Standard-for-Arm-Architecture--AAPCS-)

r[items.extern.abi.fastcall]
* `unsafe extern "fastcall"` --- stdcall 的 “快速” 变体，通过寄存器传递某些参数。
  * 仅适用于 x86_32 目标。
  * 对应于 MSVC 的 `__fastcall` 以及 GCC 和 clang 的 `__attribute__((fastcall))`。

  > [!NOTE]
  > 有关详细信息，请参阅：
  >
  > - <https://learn.microsoft.com/en-us/cpp/cpp/fastcall>
  > - <https://en.wikipedia.org/wiki/X86_calling_conventions#Microsoft_fastcall>

r[items.extern.abi.thiscall]
* `unsafe extern "thiscall"` --- 通常在 x86_32 MSVC 上用于 C++ 类成员函数的调用约定。
  * 仅适用于 x86_32 目标。
  * 对应于 MSVC 的 `__thiscall` 以及 GCC 和 clang 的 `__attribute__((thiscall))`。

  > [!NOTE]
  > 有关详细信息，请参阅：
  >
  > - <https://en.wikipedia.org/wiki/X86_calling_conventions#thiscall>
  > - <https://learn.microsoft.com/en-us/cpp/cpp/thiscall>

r[items.extern.abi.efiapi]
* `unsafe extern "efiapi"` --- 用于 [UEFI][UEFI] 函数的 ABI。
  * 仅适用于 x86 和 ARM 目标（32 位和 64 位）。

r[items.extern.abi.platform-unwind-variants]
与 `"C"` 和 `"system"` 类似，大多数平台特定的 ABI 字符串也具有 [对应的 `-unwind` 变体][unwind-behavior]；具体而言，它们是：

* `"aapcs-unwind"`
* `"cdecl-unwind"`
* `"fastcall-unwind"`
* `"stdcall-unwind"`
* `"sysv64-unwind"`
* `"thiscall-unwind"`
* `"win64-unwind"`

r[items.extern.variadic]
## 变长参数函数

外部块中的函数可以通过指定 `...` 作为最后一个参数来成为变长的。变长参数可以可选地指定一个标识符。

```rust
unsafe extern "C" {
    unsafe fn foo(...);
    unsafe fn bar(x: i32, ...);
    unsafe fn with_name(format: *const u8, args: ...);
    // 安全性：此函数保证它根本不会访问变长参数。
    safe fn ignores_variadic_arguments(x: i32, ...);
}
```

> [!WARNING]
> 不应在 `extern` 块中的函数上使用 `safe` 限定符，除非该函数保证它根本不会访问变长参数。向变长参数函数传递非预期数量的参数或非预期类型的参数可能会导致 [未定义行为][undefined]。

r[items.extern.variadic.conventions]
变长参数只能在具有以下 ABI 字符串或其对应 [`-unwind` 变体][items.fn.extern.unwind] 的 `extern` 块中指定：

- `"aapcs"`
- `"C"`
- `"cdecl"`
- `"efiapi"`
- `"system"`
- `"sysv64"`
- `"win64"`

r[items.extern.attributes]
## 外部块上的属性

r[items.extern.attributes.intro]
以下 [属性][attributes] 控制外部块的行为。

r[items.extern.attributes.link]
### `link`属性

r[items.extern.attributes.link.intro]
 *`link` 属性*  指定了编译器应针对 `extern` 块内的 项 进行链接的原生库的名称。

r[items.extern.attributes.link.syntax]
它使用 [MetaListNameValueStr] 语法来指定其输入。`name` 键是要链接的原生库的名称。`kind` 键是一个可选值，它指定了库的类型，具有以下可能的值：

r[items.extern.attributes.link.dylib]
- `dylib` --- 表示动态库。如果未指定 `kind`，则这是默认值。

r[items.extern.attributes.link.static]
- `static` --- 表示静态库。

r[items.extern.attributes.link.framework]
- `framework` --- 表示 macOS 框架。这仅对 macOS 目标有效。

r[items.extern.attributes.link.raw-dylib]
- `raw-dylib` --- 表示一个动态库，编译器将生成一个导入库来与其链接（详见下文的 [`dylib` 与 `raw-dylib` 比较][`dylib` versus `raw-dylib`]）。这仅对 Windows 目标有效。

r[items.extern.attributes.link.name-requirement]
如果指定了 `kind`，则必须包含 `name` 键。

r[items.extern.attributes.link.modifiers]
可选的 `modifiers` 参数是一种为要链接的库指定链接修饰符的方法。

r[items.extern.attributes.link.modifiers.syntax]
修饰符指定为以逗号分隔的字符串，每个修饰符前缀为 `+` 或 `-`，分别表示启用或禁用该修饰符。

r[items.extern.attributes.link.modifiers.multiple]
目前不支持在单个 `link` 属性中指定多个 `modifiers` 参数，或者在同一个 `modifiers` 参数中指定多个相同的修饰符。 \
示例：`#[link(name = "mylib", kind = "static", modifiers = "+whole-archive")]`。

r[items.extern.attributes.link.wasm_import_module]
当从宿主环境导入符号时，`wasm_import_module` 键可用于为 `extern` 块中的 项 指定 [WebAssembly 模块][WebAssembly module] 名称。如果未指定 `wasm_import_module`，则默认模块名称为 `env`。

<!-- ignore: requires extern linking -->
```rust,ignore
#[link(name = "crypto")]
unsafe extern {
    // …
}

#[link(name = "CoreFoundation", kind = "framework")]
unsafe extern {
    // …
}

#[link(wasm_import_module = "foo")]
unsafe extern {
    // …
}
```

r[items.extern.attributes.link.empty-block]
在空的外部块上添加 `link` 属性是有效的。您可以使用它来满足代码中其他地方（包括上游 crate）外部块的链接要求，而不是在每个外部块中都添加该属性。

r[items.extern.attributes.link.modifiers.bundle]
#### 链接修饰符：`bundle`

r[items.extern.attributes.link.modifiers.bundle.allowed-kinds]
此修饰符仅与 `static` 链接类型兼容。使用任何其他类型都将导致编译器错误。

r[items.extern.attributes.link.modifiers.bundle.behavior]
在构建 rlib 或 staticlib 时，`+bundle` 意味着原生静态库将被打包到 rlib 或 staticlib 归档中，然后在链接最终二进制文件期间从中检索。

r[items.extern.attributes.link.modifiers.bundle.behavior-negative]
在构建 rlib 时，`-bundle` 意味着原生静态库被 “按名称” 注册为该 rlib 的依赖项，并且仅在链接最终二进制文件期间才包含其中的目标文件，在最终链接期间也会执行按该名称的文件搜索。 \
在构建 staticlib 时，`-bundle` 意味着原生静态库根本不包含在归档中，某些更高级别的构建系统将需要在稍后链接最终二进制文件期间添加它。

r[items.extern.attributes.link.modifiers.bundle.no-effect]
在构建可执行文件或动态库等其他目标时，此修饰符没有效果。

r[items.extern.attributes.link.modifiers.bundle.default]
此修饰符的默认值为 `+bundle`。

有关此修饰符的更多实现细节，可以在 [rustc 的 `bundle` 文档][`bundle` documentation for rustc] 中找到。

r[items.extern.attributes.link.modifiers.whole-archive]
#### 链接修饰符：`whole-archive`

r[items.extern.attributes.link.modifiers.whole-archive.allowed-kinds]
此修饰符仅与 `static` 链接类型兼容。使用任何其他类型都将导致编译器错误。

r[items.extern.attributes.link.modifiers.whole-archive.behavior]
`+whole-archive` 意味着静态库作为整个归档进行链接，而不丢弃任何目标文件。

r[items.extern.attributes.link.modifiers.whole-archive.default]
此修饰符的默认值为 `-whole-archive`。

有关此修饰符的更多实现细节，可以在 [rustc 的 `whole-archive` 文档][`whole-archive` documentation for rustc] 中找到。

r[items.extern.attributes.link.modifiers.verbatim]
### 链接修饰符：`verbatim`

r[items.extern.attributes.link.modifiers.verbatim.allowed-kinds]
此修饰符与所有链接类型兼容。

r[items.extern.attributes.link.modifiers.verbatim.behavior]
`+verbatim` 意味着 rustc 本身不会向库名称添加任何目标平台指定的库前缀或后缀（如 `lib` 或 `.a`），并将尽力向链接器请求同样的内容。

r[items.extern.attributes.link.modifiers.verbatim.behavior-negative]
`-verbatim` 意味着 rustc 在将库名称传递给链接器之前，会向其添加目标平台特定的前缀和后缀，或者不会阻止链接器隐式添加它。

r[items.extern.attributes.link.modifiers.verbatim.default]
此修饰符的默认值为 `-verbatim`。

有关此修饰符的更多实现细节，可以在 [rustc 的 `verbatim` 文档][`verbatim` documentation for rustc] 中找到。

r[items.extern.attributes.link.kind-raw-dylib]
#### `dylib`与 `raw-dylib`比较

r[items.extern.attributes.link.kind-raw-dylib.intro]
在 Windows 上，与动态库链接要求向链接器提供一个导入库：这是一个特殊的静态库，它声明了动态库导出的所有符号，其方式使链接器知道它们必须在运行时动态加载。

r[items.extern.attributes.link.kind-raw-dylib.import]
指定 `kind = "dylib"` 会指示 Rust 编译器链接一个基于 `name` 键的导入库。链接器随后将使用其正常的库解析逻辑来查找该导入库。或者，指定 `kind = "raw-dylib"` 会指示编译器在编译期间生成一个导入库，并将该导入库提供给链接器。

r[items.extern.attributes.link.kind-raw-dylib.platform-specific]
`raw-dylib` 仅在 Windows 上受支持。针对其他平台使用它将导致编译器错误。

r[items.extern.attributes.link.import_name_type]
#### `import_name_type`键

r[items.extern.attributes.link.import_name_type.intro]
在 x86 Windows 上，函数名称会被 “修饰” (decorated)（即添加特定的前缀和/或后缀）以指示其调用约定。例如，一个名为 `fn1` 且没有参数的 `stdcall` 调用约定函数将被修饰为 `_fn1@0`。然而，[PE 格式][PE Format] 也确实允许名称没有前缀或不被修饰。此外，MSVC 和 GNU 工具链对相同的调用约定使用不同的修饰，这意味着默认情况下，某些 Win32 函数无法通过 GNU 工具链使用 `raw-dylib` 链接类型进行调用。

r[items.extern.attributes.link.import_name_type.values]
为了允许这些差异，在使用 `raw-dylib` 链接类型时，您还可以使用以下值之一指定 `import_name_type` 键，以更改生成的导入库中函数的命名方式：

* `decorated`：函数名称将使用 MSVC 工具链格式进行完整修饰。
* `noprefix`：函数名称将使用 MSVC 工具链格式进行修饰，但跳过前导的 `?`、`@` 或可选的 `_`。
* `undecorated`：函数名称将不被修饰。

r[items.extern.attributes.link.import_name_type.default]
如果未指定 `import_name_type` 键，则函数名称将使用目标工具链的格式进行完整修饰。

r[items.extern.attributes.link.import_name_type.variables]
变量永远不会被修饰，因此 `import_name_type` 键对它们在生成的导入库中的命名方式没有影响。

r[items.extern.attributes.link.import_name_type.platform-specific]
`import_name_type` 键仅在 x86 Windows 上受支持。针对其他平台使用它将导致编译器错误。

<!-- template:attributes -->
r[items.extern.attributes.link_name]
### `link_name`属性

r[items.extern.attributes.link_name.intro]
 *`link_name` [属性][attributes]*  可以应用于外部块内的声明，以指定要为给定函数或静态项导入的符号。

> [!EXAMPLE]
> ```rust
> unsafe extern "C" {
>     #[link_name = "actual_symbol_name"]
>     safe fn name_in_rust();
> }
> ```

r[items.extern.attributes.link_name.syntax]
`link_name` 属性使用 [MetaNameValueStr] 语法。

r[items.extern.attributes.link_name.allowed-positions]
`link_name` 属性只能应用于外部块中的函数或 静态项。

> [!NOTE]
> `rustc` 忽略在其他位置的使用，但会针对其发出 lint。这在将来可能会变成错误。

r[items.extern.attributes.link_name.duplicates]
只有 项 上最后一次使用的 `link_name` 才会生效。

> [!NOTE]
> `rustc` 会针对最后一次使用之前的任何使用发出 lint。这在将来可能会变成错误。

r[items.extern.attributes.link_name.link_ordinal]
`link_name` 属性不得与 [`link_ordinal`][`link_ordinal`] 属性一起使用。

r[items.extern.attributes.link_ordinal]
### `link_ordinal`属性

r[items.extern.attributes.link_ordinal.intro]
 *`link_ordinal` 属性*  可以应用于外部块内的声明，以指示生成要链接的导入库时要使用的数字序数。序数是 Windows 上动态库导出的每个符号的唯一编号，在加载库以查找该符号时可以使用序数，而不必按名称查找。

> [!WARNING]
> `link_ordinal` 仅应在已知符号序数稳定的情况下使用：如果符号的序数在构建其包含的二进制文件时未显式设置，则系统会自动为其分配一个序数，并且分配的序数可能会在二进制文件的不同构建之间发生变化。

```rust
# #[cfg(all(windows, target_arch = "x86"))]
#[link(name = "exporter", kind = "raw-dylib")]
unsafe extern "stdcall" {
    #[link_ordinal(15)]
    safe fn imported_function_stdcall(i: i32);
}
```

r[items.extern.attributes.link_ordinal.allowed-kinds]
此属性仅与 `raw-dylib` 链接类型一起使用。使用任何其他类型都将导致编译器错误。

r[items.extern.attributes.link_ordinal.exclusive]
将此属性与 `link_name` 属性一起使用将导致编译器错误。

r[items.extern.attributes.fn-parameters]
### 函数参数上的属性

外部函数参数上的属性遵循与 [常规函数参数][regular function parameters] 相同的规则和限制。

[ABI]: glossary.abi
[PE Format]: https://learn.microsoft.com/windows/win32/debug/pe-format#import-name-type
[UEFI]: https://uefi.org/specifications
[WebAssembly module]: https://webassembly.github.io/spec/core/syntax/modules.html
[`bundle` documentation for rustc]: ../../rustc/command-line-arguments.html#linking-modifiers-bundle
[`dylib` versus `raw-dylib`]: #dylib-versus-raw-dylib
[`extern fn`]: items.fn.extern
[`unsafe` context]: ../unsafe-keyword.md
[`verbatim` documentation for rustc]: ../../rustc/command-line-arguments.html#linking-modifiers-verbatim
[`whole-archive` documentation for rustc]: ../../rustc/command-line-arguments.html#linking-modifiers-whole-archive
[attributes]: ../attributes.md
[functions]: functions.md
[regular function parameters]: functions.md#函数参数上的属性
[statics]: static-items.md
[unwind-behavior]: functions.md#展开
[value namespace]: ../names/namespaces.md
[win32 api]: https://learn.microsoft.com/en-us/windows/win32/api/
[`link_ordinal`]: items.extern.attributes.link_ordinal
