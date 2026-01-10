r[panic]
# 恐慌

r[panic.intro]
Rust 提供了一种机制来防止函数正常返回，而是引发 "恐慌 (panic)"。这是对错误条件的响应，通常在遇到错误的上下文中被认为无法恢复。

r[panic.lang-ops]
一些语言结构，例如越界的 [数组索引][array indexing]，会自动引发恐慌。

r[panic.control]
还有一些语言特性提供了一定程度的恐慌行为控制：

*  [_恐慌处理器 (panic handler)_][panic handler] 定义了恐慌的行为。
* [FFI ABI](items/functions.md#unwinding) 可能会改变恐慌的行为。

> [!NOTE]
> 标准库提供了通过 [panic! 宏][panic!] 显式引发恐慌的能力。

r[panic.panic_handler]
## panic_handler属性

r[panic.panic_handler.intro]
*panic_handler 属性* 可以应用于函数以定义恐慌的行为。

r[panic.panic_handler.allowed-positions]
panic_handler 属性只能应用于具有签名 `fn(&PanicInfo) -> !` 的函数。

> [!NOTE]
> [PanicInfo 结构体][`PanicInfo`] 包含了有关恐慌发生位置的信息。

r[panic.panic_handler.unique]
在依赖图中必须有且仅有一个 panic_handler 函数。

下面展示了一个记录恐慌消息然后停止线程的 panic_handler 函数。

<!-- ignore: test infrastructure can't handle no_std -->
```rust,ignore
#![no_std]

use core::fmt::{self, Write};
use core::panic::PanicInfo;

struct Sink {
    // ..
#    _0: (),
}
#
# impl Sink {
#     fn new() -> Sink { Sink { _0: () }}
# }
#
# impl fmt::Write for Sink {
#     fn write_str(&mut self, _: &str) -> fmt::Result { Ok(()) }
# }

#[panic_handler]
fn panic(info: &PanicInfo) -> ! {
    let mut sink = Sink::new();

    // 将 "panicked at '$reason', src/main.rs:27:4" 记录到某个 `sink`
    let _ = writeln!(sink, "{}", info);

    loop {}
}
```

r[panic.panic_handler.std]
### 标准行为

r[panic.panic_handler.std.kinds]
`std` 提供了两种不同的恐慌处理器：

* `unwind` --- 展开栈，并且是潜在可恢复的。
* `abort` ---- 中止进程，且是不可恢复的。

并非所有 target 都提供 `unwind` 处理器。

> [!NOTE]
> 链接 `std` 时使用的恐慌处理器可以通过 [`-C panic`] 命令行标志设置。大多数 target 的默认值是 `unwind`。
>
> 标准库的恐慌行为可以在运行时使用 [`std::panic::set_hook`] 函数进行修改。

r[panic.panic_handler.std.no_std]
链接 [`no_std`] 二进制文件、dylib、cdylib 或 staticlib 时，需要指定你自己的恐慌处理器。

r[panic.strategy]
## 恐慌策略

r[panic.strategy.intro]
_恐慌策略 (panic strategy)_ 定义了一个 crate 构建时支持的恐慌行为类型。

> [!NOTE]
> 恐慌策略可以在 `rustc` 中使用 [`-C panic`] 命令行标志进行选择。
>
> 当生成二进制文件、dylib、cdylib 或 staticlib 且链接 `std` 时， `-C panic` 命令行标志也会影响使用哪个 [恐慌处理器][panic handler]。

> [!NOTE]
> 当使用 `abort` 恐慌策略编译代码时，优化器可能会假设跨 Rust 栈帧的展开是不可能的，这可以提高代码大小和运行时速度。

> [!NOTE]
> 有关链接具有不同恐慌策略的 crate 的限制，请参见 [link.unwinding]。其中一个暗示是，使用 `unwind` 策略构建的 crate 可以使用 `abort` 恐慌处理器，但 `abort` 策略不能使用 `unwind` 恐慌处理器。

r[panic.unwind]
## 展开

r[panic.unwind.intro]
引发恐慌可能是可恢复的或不可恢复的，尽管可以配置（通过选择不展开的恐慌处理器）为始终不可恢复。（反之则不然：`unwind` 处理器不保证所有恐慌都是可恢复的，仅保证通过 `panic!` 宏及类似的标准库机制引发的恐慌是可恢复的。）

r[panic.unwind.destruction]
当发生恐慌时，`unwind` 处理器会 "展开 (unwind)" Rust 栈帧，就像 C++ 的 `throw` 展开 C++ 栈帧一样，直到恐慌到达恢复点（例如在线程边界处）。这意味着当恐慌遍历 Rust 栈帧时，这些栈帧中 [实现 Drop][destructors] 的活跃对象将调用其 `drop` 方法。因此，当恢复正常执行时，不再可访问的对象将被 "清理" 掉，就像它们正常超出作用域一样。

> [!NOTE]
> 只要保留了这种资源清理的保证， "展开" 的实现可以不实际使用 target 平台 C++ 所使用的机制。

> [!NOTE]
> 标准库提供了两种从恐慌中恢复的机制：[`std::panic::catch_unwind`]（允许在发生恐慌的线程内恢复）和 [`std::thread::spawn`]（自动为生成的线程设置恐慌恢复，以便其他线程可以继续运行）。

r[panic.unwind.ffi]
### 跨FFI边界展开

r[panic.unwind.ffi.intro]
可以使用 [适当的 ABI 声明][unwind-abi] 跨 FFI 边界进行展开。虽然在某些情况下很有用，但这为未定义行为创造了独特的机会，特别是在涉及多个语言运行时时。

r[panic.unwind.ffi.undefined]
使用错误的 ABI 进行展开是未定义行为：

* 从通过不带展开的 ABI（如 `"C"`、`"system"` 等）声明的函数声明或指针调用的外部函数，引发指向 Rust 代码的展开。（例如，这种情况发生在用 C++ 编写的此类函数抛出未捕获且传播到 Rust 的异常时。）
* 在不支持展开的代码（例如使用 GCC 或 Clang 并配合 `-fno-exceptions` 编译的代码）中调用会展开（带有 `extern "C-unwind"` 或其他允许展开的 ABI）的 Rust `extern` 函数。

r[panic.unwind.ffi.catch-foreign]
使用 [`std::panic::catch_unwind`]、[`std::thread::JoinHandle::join`] 或让外部展开操作传播到 Rust `main()` 函数或线程根部之外来捕获它（例如 C++ 异常），将产生以下两种行为之一，且未指定具体会发生哪种：

* 进程中止。
* 函数返回一个包含不透明类型的 [`Result::Err`]。

> [!NOTE]
> 就此项保证而言，使用不同的 Rust 标准库实例编译或链接的 Rust 代码算作 "外部异常" 。因此，如果一个使用了 `panic!` 且链接到一个版本 Rust 标准库的库，被一个使用不同版本标准库的应用程序调用，即使该库仅在子线程中使用，也可能导致整个应用程序中止。

r[panic.unwind.ffi.dispose-panic]
目前对于外部运行时尝试处理或重新抛出 Rust `panic` 负载时的行为没有任何保证。换句话说，起源于 Rust 运行时的展开必须要么导致进程终止，要么被同一个运行时捕获。

[`-C panic`]: https://doc.rust-lang.org/rustc/codegen-options/index.html#panic
[`no_std`]: names/preludes.md#the-no_std-attribute
[`PanicInfo`]: core::panic::PanicInfo
[array indexing]: expressions/array-expr.md#array-and-slice-indexing-expressions
[attribute]: attributes.md
[destructors]: destructors.md
[panic handler]: #the-panic_handler-attribute
[runtime]: runtime.md
[unwind-abi]: items/functions.md#unwinding
