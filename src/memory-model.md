r[memory]
# 内存模型

> [!WARNING]
> Rust 的 内存模型 尚未完善且未完全确定。

r[memory.bytes]
## 字节

r[memory.bytes.intro]
Rust 中最基本的内存单位是字节。

> [!NOTE]
> 虽然字节通常被映射到硬件字节，但 Rust 使用一种“抽象”的字节概念，可以做出硬件中不存在的区分，例如未初始化，或存储指针的一部分。这些区分会影响你的程序是否具有未定义行为，因此它们仍然对编译后的 Rust 程序的行为产生实际影响。

r[memory.bytes.contents]
每个字节可能具有以下值之一：

r[memory.bytes.init]
* 一个包含 `u8` 值和可选的 [来源][std::ptr#provenance] 的已初始化字节，

r[memory.bytes.uninit]
* 一个未初始化字节。

> [!NOTE]
> 以上列表尚未保证是详尽的。
