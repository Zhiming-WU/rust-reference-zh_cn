r[link]
# 链接

> [!NOTE]
> 本节更多是从编译器而非语言本身的角度进行描述。

r[link.intro]
编译器支持静态和动态链接 crate 的多种方法。本节将探讨链接 crate 的各种方法，有关原生库的更多信息可以在 [本书的 FFI 章节][ffi] 中找到。

[ffi]: ../book/ch19-01-unsafe-rust.html#using-extern-functions-to-call-external-code

r[link.type]
在一次编译会话中，编译器可以通过使用命令行标志或 `crate_type` 属性生成多个产物。如果指定了一个或多个命令行标志，则所有的 `crate_type` 属性都将被忽略，而只构建通过命令行指定的产物。

r[link.bin]
* `--crate-type=bin`, `#![crate_type = "bin"]` - 将生成一个可运行的可执行文件。这要求 crate 中有一个 `main` 函数，该函数将在程序开始执行时运行。这将链接所有的 Rust 和原生依赖项，生成一个单一的可分发二进制文件。这是默认的 crate 类型。

r[link.lib]
* `--crate-type=lib`, `#![crate_type = "lib"]` - 将生成一个 Rust 库。这是一个模棱两可的概念，因为库可以以多种形式表现。这种通用的 `lib` 选项的目的是生成 "compiler recommended" 风格的库。输出的库始终可以被 rustc 使用，但库的具体类型可能会不时发生变化。其余输出类型都是库的不同变体，`lib` 类型可以看作是其中之一的别名 (but the actual one is compiler-defined)。

r[link.dylib]
* `--crate-type=dylib`, `#![crate_type = "dylib"]` - 将生成一个动态 Rust 库。这与 `lib` 输出类型的不同之处在于它强制生成动态库。生成的动态库可以用作其他库和/或可执行文件的依赖项。这种输出类型将在 Linux 上创建 `*.so` 文件，在 macOS 上创建 `*.dylib` 文件，在 Windows 上创建 `*.dll` 文件。

r[link.staticlib]
* `--crate-type=staticlib`, `#![crate_type = "staticlib"]` - 将生成一个静态系统库。这与其他库输出的不同之处在于，编译器永远不会尝试链接到 `staticlib` 输出。这种输出类型的目的是创建一个包含所有本地 crate 代码以及所有上游依赖项的静态库。这种输出类型将在 Linux, macOS 和 Windows (MinGW) 上创建 `*.a` 文件，并在 Windows (MSVC) 上创建 `*.lib` 文件。建议在将 Rust 代码链接到现有的非 Rust 应用程序等情况下使用此格式，因为它不会对其他 Rust 代码产生动态依赖。

  请注意，静态库可能具有的任何动态依赖项 (such as dependencies on system libraries, or dependencies on Rust libraries that are compiled as dynamic libraries) 在从某处链接该静态库时必须手动指定。 `--print=native-static-libs` 标志可能对此有所帮助。

  请注意，由于生成的静态库包含所有依赖项的代码 (including the standard library) ，并且还导出了它们的所有公共符号，因此将静态库链接到可执行文件或共享库可能需要特别注意。在共享库的情况下，必须通过例如链接器或符号版本脚本、导出符号列表 (macOS) 或模块定义文件 (Windows) 来限制导出符号的列表。此外，可以删除未使用的节，以删除未实际使用的所有依赖项代码 (e.g. `--gc-sections` or `-dead_strip` for macOS)。

r[link.cdylib]
* `--crate-type=cdylib`, `#![crate_type = "cdylib"]` - 将生成一个动态系统库。这用于在编译要从另一种语言加载的动态库时使用。这种输出类型将在 Linux 上创建 `*.so` 文件，在 macOS 上创建 `*.dylib` 文件，在 Windows 上创建 `*.dll` 文件。

r[link.rlib]
* `--crate-type=rlib`, `#![crate_type = "rlib"]` - 将生成一个 "Rust library" 文件。这用作中间产物，可以被认为是 "static Rust library" 。与 `staticlib` 文件不同，这些 `rlib` 文件在未来的链接中由编译器解释。这本质上意味着 `rustc` 将在 `rlib` 文件中寻找元数据，就像在动态库中寻找元数据一样。这种输出形式用于生成静态链接的可执行文件以及 `staticlib` 输出。

r[link.proc-macro]
* `--crate-type=proc-macro`, `#![crate_type = "proc-macro"]` - 生成的输出未指定，但如果为其提供了 `-L` 路径，则编译器会将输出产物识别为宏，并且可以为程序加载它。使用此 crate 类型编译的 crate 只能导出 [过程宏][procedural macros]。编译器将自动设置 `proc_macro` [配置选项][configuration option]。crate 始终使用与编译器本身构建时相同的 target 进行编译。例如，如果您在带有 `x86_64` CPU 的 Linux 上执行编译器，则 target 将是 `x86_64-unknown-linux-gnu`，即使该 crate 是为不同 target 构建的另一个 crate 的依赖项。

r[link.repetition]
请注意，这些输出是可堆叠的，即如果指定了多个，则编译器将生成每种形式的输出，而无需重新编译。但是，这仅适用于通过相同方法指定的输出。如果仅指定了 `crate_type` 属性，则它们都将被构建，但如果指定了一个或多个 `--crate-type` 命令行标志，则仅构建那些输出。

r[link.dependency]
对于所有这些不同种类的输出，如果 crate A 依赖于 crate B，则编译器可能会在整个系统中找到各种不同形式的 B。然而，编译器寻找的唯一形式是 `rlib` 格式和动态库格式。对于依赖库的这两个选项，编译器必须在某个时间点在两种格式之间做出选择。考虑到这一点，编译器在确定将使用哪种格式的依赖项时遵循以下规则：

r[link.dependency-staticlib]
1. 如果正在生成静态库，则要求所有上游依赖项都必须以 `rlib` 格式可用。这一要求源于动态库无法转换为静态格式的原因。

   请注意，无法将原生动态依赖项链接到静态库，在这种情况下，将打印有关所有未链接的原生动态依赖项的警告。

r[link.dependency-rlib]

2. 如果正在生成 `rlib` 文件，则对上游依赖项可用的格式没有限制。仅要求所有上游依赖项都可用于读取元数据。

   这是因为 `rlib` 文件不包含其任何上游依赖项。让所有 `rlib` 文件都包含 `libstd.rlib` 的副本是非常低效的！

r[link.dependency-prefer-dynamic]

3. 如果正在生成可执行文件且未指定 `-C prefer-dynamic` 标志，则首先尝试寻找 `rlib` 格式的依赖项。如果某些依赖项在 rlib 格式下不可用，则尝试动态链接 (see below)。

r[link.dependency-dynamic]

4. 如果正在生成动态库或正在动态链接的可执行文件，则编译器将尝试协调以 rlib 或 dylib 格式提供的依赖项，以创建最终产品。

   编译器的主要目标是确保一个库在任何产物中都不会出现超过一次。例如，如果动态库 B 和 C 都静态链接到库 A，那么一个 crate 就不能同时链接到 B 和 C，因为那样就会有两个 A 的副本。编译器允许混合 rlib 和 dylib 格式，但必须满足这一限制。

   编译器目前没有提供暗示库应以何种格式链接的方法。在动态链接时，编译器将尝试最大化动态依赖项，同时仍允许通过 rlib 链接一些依赖项。

   对于大多数情况，如果进行动态链接，建议所有库都以 dylib 形式可用。对于其他情况，如果编译器无法确定每个库应以哪种格式链接，它将发出警告。

通常，对于所有编译需求，`--crate-type=bin` 或 `--crate-type=lib` 应该就足够了，其他选项仅在需要对 crate 输出格式进行更精细控制时可用。

r[link.crt]
## 静态和动态C运行时

r[link.crt.intro]
标准库通常力求根据需要为 target 支持静态链接和动态链接的 C 运行时。例如 `x86_64-pc-windows-msvc` 和 `x86_64-unknown-linux-musl` target 通常带有两种运行时，用户可以选择他们喜欢的。编译器中的所有 target 都有链接到 C 运行时的默认模式。通常 target 默认是动态链接的，但也有些例外是默认静态链接的，例如：

* `arm-unknown-linux-musleabi`
* `arm-unknown-linux-musleabihf`
* `armv7-unknown-linux-musleabihf`
* `i686-unknown-linux-musl`
* `x86_64-unknown-linux-musl`

r[link.crt.crt-static]
C 运行时的链接配置为遵循 `crt-static` target 特性。这些 target 特性通常在命令行中通过编译器的标志进行配置。例如，要启用静态运行时，您可以执行：

```sh
rustc -C target-feature=+crt-static foo.rs
```

而要动态链接到 C 运行时，您可以执行：

```sh
rustc -C target-feature=-crt-static foo.rs
```

r[link.crt.ineffective]
不支持在 C 运行时链接方式之间切换的 target 将忽略此标志。建议在编译器成功后检查生成的二进制文件，以确保它按预期链接。

r[link.crt.target_feature]
crate 也可以了解 C 运行时是如何被链接的。例如，MSVC 上的代码需要根据链接的运行时进行不同的编译 (e.g. with `/MT` or `/MD`) 。这目前通过 [`cfg` 属性 `target_feature` 选项][`cfg` attribute `target_feature` option] 导出：

```rust
#[cfg(target_feature = "crt-static")]
fn foo() {
    println!("the C runtime should be statically linked");
}

#[cfg(not(target_feature = "crt-static"))]
fn foo() {
    println!("the C runtime should be dynamically linked");
}
```

另请注意，Cargo 构建脚本可以通过 [环境变量][cargo] 了解此特性。在构建脚本中，您可以通过以下方式检测链接：

```rust
use std::env;

fn main() {
    let linkage = env::var("CARGO_CFG_TARGET_FEATURE").unwrap_or(String::new());

    if linkage.contains("crt-static") {
        println!("the C runtime will be statically linked");
    } else {
        println!("the C runtime will be dynamically linked");
    }
}
```

[cargo]: ../cargo/reference/environment-variables.html#environment-variables-cargo-sets-for-build-scripts

要在本地使用此特性，您通常会使用 `RUSTFLAGS` 环境变量通过 Cargo 为编译器指定标志。例如，要在 MSVC 上编译静态链接的二进制文件，您可以执行：

```sh
RUSTFLAGS='-C target-feature=+crt-static' cargo build --target x86_64-pc-windows-msvc
```

r[link.foreign-code]
## 混合Rust和外部代码库

r[link.foreign-code.foreign-linkers]
如果您混合使用 Rust 和外部代码 (e.g. C, C++) 并希望制作包含两种代码类型的单个二进制文件，则最终二进制链接有两种方法：

* 使用 `rustc`。使用 `rustc` 参数 `-L <directory>` 和 `-l<library>` 传递任何非 Rust 库，和/或在 Rust 代码中使用 `#[link]` 指令。如果您需要链接 `*.o` 文件，可以使用 `-Clink-arg=file.o`。
* 使用您的外部链接器。在这种情况下，您首先需要生成一个 Rust `staticlib` target 并将其传递给您的外部链接器调用。如果您需要链接多个 Rust 子系统，您将需要生成一个 *单个* `staticlib`，可能需要使用大量的 `extern crate` 语句来包含多个 Rust `rlib`。多个 Rust `staticlib` 文件很可能会冲突。

目前不支持将 `rlib` 直接传递到您的外部链接器中。

> [!NOTE]
> 对于本节而言，使用不同的 Rust 运行时实例编译或链接的 Rust 代码也算作 "foreign code" 。

r[link.unwinding]
### 禁止的链接和恐慌展开

r[link.unwinding.intro]
仅当二进制文件根据以下规则一致地构建时，才能使用恐慌展开。

r[link.unwinding.potential]
如果满足以下任何条件，则 Rust 产物被称为 *潜在展开的* ：
- 该产物使用 [`unwind` 恐慌处理器][panic.panic_handler]。
- 该产物包含一个使用 `unwind` [恐慌策略][panic strategy] 构建的 crate，该 crate 使用 `-unwind` ABI 调用函数。
- 该产物对在另一个拥有独立的 Rust 运行时副本的 Rust 产物中运行的代码进行 "Rust" ABI 调用，并且该另一个产物是潜在展开的。

> [!NOTE]
> 此定义涵盖了 Rust 产物内部的 "Rust" ABI 调用是否会发生展开。

r[link.unwinding.prohibited]
如果一个 Rust 产物是潜在展开的，那么它的所有 crate 都必须使用 `unwind` [恐慌策略][panic strategy] 构建。否则，展开可能会导致未定义行为。

> [!NOTE]
> 如果您使用 `rustc` 进行链接，这些规则会自动执行。如果您 *不* 使用 `rustc` 进行链接，则必须注意确保在整个二进制文件中一致地处理展开。不使用 `rustc` 链接包括使用 `dlopen` 或类似设施，其中链接由系统运行时完成，而不涉及 `rustc`。这种情况仅在混合使用具有不同 [`-C panic`] 标志的代码时才会发生，因此大多数用户无需担心这一点。

> [!NOTE]
> 为了保证无论链接时使用何种恐慌运行时，库都是健全的 (and linkable with `rustc`) ，可以使用 [`ffi_unwind_calls` lint] 。该 lint 会标记对 `-unwind` 外部函数或函数指针的任何调用。

[`cfg` attribute `target_feature` option]: conditional-compilation.md#target_feature
[`ffi_unwind_calls` lint]: ../rustc/lints/listing/allowed-by-default.html#ffi-unwind-calls
[configuration option]: conditional-compilation.md
[procedural macros]: procedural-macros.md
[panic strategy]: panic.md#panic-strategy
[`-C panic`]: ../rustc/codegen-options/index.html#panic
