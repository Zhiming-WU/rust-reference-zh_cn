r[items.static]
# 静态项

r[items.static.syntax]
```grammar,项
StaticItem ->
    ItemSafety?[^extern-safety] `static` `mut`? IDENTIFIER `:` Type ( `=` Expression )? `;`
```

[^extern-safety]: `safe` 和 `unsafe` 函数限定符在语义上仅允许在 `extern` 块中使用。

r[items.static.intro]
一个  *静态项*  与 [常量项][constant] 类似，不同之处在于它表示程序中由初始化表达式初始化的分配。指向该 静态项 的所有引用和裸指针都指向同一个分配。

r[items.static.lifetime]
静态项 具有 `static` 生命周期，它比 Rust 程序中的所有其他生命周期都要长。 静态项 在程序结束时不会调用 [`drop`][`drop`]。

r[items.static.storage-disjointness]
如果 `static` 的大小至少为 1 字节，则此分配与所有其他此类 `static` 分配以及堆分配和栈分配变量是不相交的。然而，不可变  *静态项*  的存储可以与自身没有唯一地址的分配重叠，例如 [提升项][promoteds] 和 [`const` 项][constant]。

r[items.static.namespace]
静态声明在它所在的模块或块的 [值命名空间][value namespace] 中定义了一个静态值。

r[items.static.init]
静态初始化器是一个在编译时求值的 [常量表达式][constant expression]。静态初始化器可以引用并读取其他 静态项。从可变 静态项 读取时，它们读取该 静态项 的初始值。

r[items.static.read-only]
包含非 [内部可变][interior mutable] 类型的非 `mut` 静态项 可能会被放置在只读内存中。

r[items.static.safety]
对 静态项 的所有访问都是安全的，但对 静态项 有一些限制：

r[items.static.sync]
* 类型必须具有 [`Sync`](https://doc.rust-lang.org/core/marker/trait.Sync.html) 特型 绑定以允许线程安全访问。

r[items.static.init.omission]
初始化表达式在 [外部块][external block] 中必须省略，而在自由 静态项 中必须提供。

r[items.static.safety-qualifiers]
从语义上讲，`safe` 和 `unsafe` 限定符仅在 [外部块][external block] 中使用时才被允许。

r[items.static.generics]
## 静态项与泛型

在 泛型 作用域（例如在全覆盖实现或默认实现）中定义的 静态项 将导致仅定义一个 静态项，就好像静态定义从当前作用域提取到了模块中一样。每一单态化实例将  *不*  有一个 项。

这段代码：

```rust
use std::sync::atomic::{AtomicUsize, Ordering};

trait Tr {
    fn default_impl() {
        static COUNTER: AtomicUsize = AtomicUsize::new(0);
        println!("default_impl: counter was {}", COUNTER.fetch_add(1, Ordering::Relaxed));
    }

    fn blanket_impl();
}

struct Ty1 {}
struct Ty2 {}

impl<T> Tr for T {
    fn blanket_impl() {
        static COUNTER: AtomicUsize = AtomicUsize::new(0);
        println!("blanket_impl: counter was {}", COUNTER.fetch_add(1, Ordering::Relaxed));
    }
}

fn main() {
    <Ty1 as Tr>::default_impl();
    <Ty2 as Tr>::default_impl();
    <Ty1 as Tr>::blanket_impl();
    <Ty2 as Tr>::blanket_impl();
}
```

打印：

```text
default_impl: counter was 0
default_impl: counter was 1
blanket_impl: counter was 0
blanket_impl: counter was 1
```

r[items.static.mut]
## 可变静态项

r[items.static.mut.intro]
如果 静态项 使用 `mut` 关键字声明，则允许程序对其进行修改。Rust 的目标之一是使并发错误难以发生，而这显然是竞争条件或其他错误的一个非常大的来源。

r[items.static.mut.safety]
出于这个原因，在读取或写入可变静态变量时都需要 `unsafe` 块。应注意确保对可变静态项的修改相对于在同一进程中运行的其他线程是安全的。

r[items.static.mut.extern]
然而，可变静态项仍然非常有用。它们可以与 C 库一起使用，也可以在 `extern` 块中从 C 库绑定。

```rust
# fn atomic_add(_: *mut u32, _: u32) -> u32 { 2 }

static mut LEVELS: u32 = 0;

// 这违反了无共享状态的想法，且这在内部不防御竞争，因此这个函数是 `unsafe` 的
unsafe fn bump_levels_unsafe() -> u32 {
    unsafe {
        let ret = LEVELS;
        LEVELS += 1;
        return ret;
    }
}

// 作为 `bump_levels_unsafe` 的替代方案，假设我们有一个返回旧值的 atomic_add 函数，那么这个函数就是安全的。
// 只有在没有其他代码以非原子方式访问该静态项的情况下，此函数才是安全的。
// 如果此类访问是可能的（例如在 `bump_levels_unsafe` 中），那么这将需要是 `unsafe` 的，
// 以向调用者指示他们仍必须防范并发访问。
fn bump_levels_safe() -> u32 {
    unsafe {
        return atomic_add(&raw mut LEVELS, 1);
    }
}
```

r[items.static.mut.sync]
可变静态项具有与普通 静态项 相同的限制，但类型不必实现 `Sync` 特型。

r[items.static.alternate]
## 使用静态项或常量项

使用 常量项 还是 静态项 可能会令人困惑。通常情况下，应优先使用 常量项 而非 静态项，除非满足以下条件之一：

* 正在存储大量数据。
* 需要 静态项 的单地址属性。
* 需要内部可变性。

[constant]: constant-items.md
[`drop`]: ../destructors.md
[constant expression]: ../const_eval.md#常量表达式
[external block]: external-blocks.md
[interior mutable]: ../interior-mutability.md
[value namespace]: ../names/namespaces.md
[promoteds]: ../destructors.md#constant-promotion
