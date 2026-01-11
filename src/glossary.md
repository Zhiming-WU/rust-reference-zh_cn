<!-- https://github.com/rust-lang/reference/blob/master/src/glossary.md -->
<!-- commit 68bdfd15fa9feebfbf94a06cf574de598e239198 -->

# 词汇表

r[glossary.ast]
### 抽象语法树

“抽象语法树”，或“AST”，是编译器编译程序时，程序结构的中间表示。

### 对齐

值的 对齐 方式指定了值倾向于从哪个地址开始。始终是 2 的幂。对值的引用必须对齐。
[更多][alignment]。

r[glossary.abi]
### 应用程序二进制接口(ABI)

一个 *应用程序二进制接口* (ABI) 定义了编译后的代码如何与其它编译后的代码交互。通过 [`extern` 块][`extern` blocks] 和 [`extern fn`][`extern fn`]，*ABI 字符串* 影响：

- **调用约定**：函数参数如何传递，返回值如何返回（例如，在寄存器中或在栈上），以及谁负责清理栈。
- **展开**：是否允许栈展开。例如，`"C-unwind"` ABI 允许跨 FFI 边界展开，而 `"C"` ABI 不允许。

### 元数

元数 指的是函数或运算符接受的参数数量。例如，`f(2, 3)` 和 `g(4, 6)` 的 元数 是 2，而 `h(8, 2, 6)` 的 元数 是 3。`!` 运算符的 元数 是 1。

### 数组

数组，有时也称为 固定大小数组 或 内联数组，是一个描述元素集合的值，每个元素通过程序在运行时计算的索引来选择。它占据一片连续的内存区域。

### 关联项

关联项 是与另一个 [项][item] 关联的 [项][item]。关联项 在 [实现][implementations] 中定义，并在 [特型][traits] 中声明。只有函数、常量和类型别名可以关联。与 [自由项](#自由项) 形成对比。

### 全局实现

任何类型以 [未覆盖类型](#未覆盖类型) 方式出现的实现。`impl<T> Foo for T`、`impl<T> Bar<T> for T`、`impl<T> Bar<Vec<T>> for T` 和 `impl<T> Bar<T> for Vec<T>` 都被认为是 全局实现。然而，`impl<T> Bar<Vec<T>> for Vec<T>` 不是 全局实现，因为出现在此 `impl` 中的所有 `T` 实例都被 `Vec` 覆盖。

### 界限

界限 是对类型或 特型 的约束。例如，如果对函数接受的参数设置了 界限，则传递给该函数的类型必须遵守该约束。

### 组合器

组合器 是高阶函数，它们只应用函数和先前定义的 组合器，以从其参数中提供结果。它们可以用于以模块化的方式管理控制流。

### crate

crate 是编译和链接的单元。有不同的 [crate 类型][types of crates]，例如库或可执行文件。crate 可以链接并引用其他库 crate，称为 外部 crate。一个 crate 拥有一个自包含的 [模块][modules] 树，从一个名为 crate 根 的未命名根模块开始。[项][items] 可以通过在 crate 根 中将其标记为公共来对其他 crate 可见，包括通过公共模块的 [路径][paths] 。
[更多][crate]。

### 调度

调度 是一种机制，用于在涉及 多态 时确定实际运行哪个特定版本的代码。调度的主要形式有两种：静态调度 和 动态调度。Rust 通过使用 [特型对象][type.trait-object] 支持 动态调度。

### 动态大小类型(DST)

动态大小类型 (DST) 是一种没有静态已知大小或对齐的 类型。

### 实体

一个 [*实体*][*entity*] 是一种语言构造，可以在源程序中以某种方式引用，通常通过 [路径][paths] 来引用。实体 包括 [类型][types]、[项][items]、[泛型参数][generic parameters]、[变量绑定][variable bindings]、[循环标签][loop labels]、[生命周期][lifetimes]、[字段][fields]、[属性][attributes] 和 [lint][lints]。

### 表达式

表达式 是值、常量、变量、运算符和函数的组合，它们评估为一个单一的值，有或没有副作用。

例如，`2 + (3 * 4)` 是一个返回 14 的 表达式。

### 自由项

一个不属于 [实现][implementation] 的 [项][item]，例如 *自由函数* 或 *自由常量*。与 [关联项](#关联项) 形成对比。

### 基本特型

基本特型 是指为现有类型添加它的实现会造成 破坏性变更 的 特型。`Fn` 特型和 `Sized` 是 基本特型。

### 基本类型构造器

基本类型构造器 是一种类型，对其实现 [全局实现](#全局实现) 会造成 破坏性变更。`&`、`&mut`、`Box` 和 `Pin` 是 基本的。

任何时候，如果类型 `T` 被认为是 [局部](#局部类型) 的，那么 `&T`、`&mut T`、`Box<T>` 和 `Pin<T>` 也被认为是 局部 的。基本类型构造器 不能 [覆盖](#未覆盖类型) 其他类型。任何时候使用术语“覆盖类型”时，`&T`、`&mut T`、`Box<T>` 和 `Pin<T>` 中的 `T` 都不被认为是 覆盖的。

### 可居住的

如果一个类型有构造器，从而可以被 实例化，那么它就是 可居住的。一个 可居住的 类型在“空”的意义上并非“空”，即可以存在该类型的值。与 [不可居住的](#不可居住的) 相反。

### 固有实现

一个适用于名义类型而非 特型-类型对 的 [实现][implementation]。
[更多][inherent implementation]。

### 固有方法

在 [固有实现][inherent implementation] 中定义而不是在 特型实现 中定义的 [方法][method]。

### 已初始化

如果一个变量已被 赋值 且之后未被 移出，则它就是 已初始化 的。所有其他内存位置都被认为是 未初始化 的。只有 非安全 Rust 才能创建未初始化的内存位置。

### 局部特型

在当前 crate 中定义的 `trait`。特型定义 的 局部性 独立于应用的 类型参数。给定 `trait Foo<T, U>`，`Foo` 始终是 局部 的，无论为 `T` 和 `U` 替换什么类型。

### 局部类型

在当前 crate 中定义的 `struct`、`enum` 或 `union`。这不受应用的 类型参数 影响。`struct Foo` 被认为是 局部 的，但 `Vec<Foo>` 不是。`LocalType<ForeignType>` 是 局部 的。类型别名 不影响 局部性。

### 模块

模块 是一个包含零个或多个 [项][items] 的容器。模块 以树形结构组织，从一个未命名的根模块开始，该模块称为 crate 根 或 根模块。[路径][Paths] 可用于引用其他模块中的 项，这可能受到 [可见性规则][visibility rules] 的限制。
[更多][modules]

### 名称

一个 [*名称*][*name*] 是一个 [标识符][identifier] 或 [生命周期或循环标签][lifetime or loop label]，它引用一个 [实体](#实体)。*名称绑定* 是指一个 实体声明 引入了一个与该 实体 关联的 标识符 或 标签。[路径][Paths]、标识符 和 [标签][labels] 用于引用一个 实体。

### 名称解析

[*名称解析*][*Name resolution*] 是将 [路径][paths]、[标识符][identifiers] 和 [标签][labels] 与 [实体](#实体) 声明 关联起来的 编译时过程。

### 命名空间

一个 *命名空间* 是根据名称所引用的 [实体](#实体) 类型，对已声明的 [名称](#名称) 进行的 逻辑分组。命名空间 允许一个 名称 在一个 命名空间 中出现，而不会与另一个 命名空间 中的同名 名称 冲突。

在 命名空间 内部，名称 以 层次结构 的方式组织，其中 层次结构 的每个级别都有自己的一组 命名实体。

### 名义类型

可以直接通过 路径 引用的 类型。具体来说是 [枚举][enums]、[结构体][structs]、[联合体][unions] 和 [特型对象类型][trait object types]。

### dyn兼容特型

可以在 [特型对象类型][trait object types] (`dyn Trait`) 中使用的 [特型][traits] 。
只有遵循特定 [规则][dyn compatibility] 的 特型 才是 *dyn 兼容* 的。

这些以前被称为 *对象安全特型*。

### 路径

一个 [*路径*][*path*] 是一系列一个或多个 路径段，用于引用当前 [作用域](#作用域) 或 [命名空间](#命名空间) 层次结构 其它级别中的 [实体](#实体)。

### 预导入

预导入，或称 Rust 预导入，是一个小型的 项 集合——主要是 特型——它们被导入到每个 crate 的每个 模块 中。预导入 中的 特型 是普遍存在的。

### 作用域

一个 [*作用域*][*scope*] 是源代码中一个 命名实体 可以通过该名称被引用的区域。

### 审查值

审查值 是在 `match` 表达式和类似的 模式匹配构造 中被匹配的 表达式。例如，在 `match x { A => 1, B => 2 }` 中，表达式 `x` 是 审查值。

### 大小

值 的 大小 有两种定义。

第一种是存储该 值 必须分配多少 内存。

第二种是具有该 项类型 的 数组 中连续 元素 之间的 字节 偏移量。

它是 对齐 的倍数，包括零。大小 可能因 编译器版本（随着新 优化 的出现）和 目标平台（类似于 `usize` 在不同平台上的差异）而变化。

[更多][alignment]。

### 切片

切片 是一个对 连续序列 的 动态大小视图，写作 `[T]`。

它通常以其借用形式出现，可以是 可变 的或 共享 的。共享 切片类型 是 `&[T]`，而 可变 切片类型 是 `&mut [T]`，其中 `T` 代表 元素类型。

### 语句

语句 是编程语言中最小的独立元素，它命令计算机执行一个 动作。

### 字符串字面量

字符串字面量 是直接存储在 最终二进制文件 中的 字符串，因此在 `'static` 持续时间 内有效。

它的类型是 `'static` 持续时间 借用字符串切片，`&'static str`。

### 字符串切片

字符串切片 是 Rust 中最原始的 字符串类型，写作 `str`。它通常以其借用形式出现，可以是 可变 的或 共享 的。共享 字符串切片类型 是 `&str`，而 可变 字符串切片类型 是 `&mut str`。

字符串切片 始终是有效的 UTF-8。

### 特型

特型 是一种语言项，用于描述 类型 必须提供的 功能。它允许 类型 对其 行为 做出某些承诺。

泛型函数 和 泛型结构体 可以使用 特型 来 约束 或 限定它们接受的 类型。

### 涡轮鱼

表达式 中带有 泛型参数 的 路径 必须在开头的尖括号前加上 `::`。结合 泛型 的尖括号，这看起来像一条鱼 `::<>`。因此，这种 语法格式 被俗称为 涡轮鱼 语法格式。

示例：

```rust
let ok_num = Ok::<_, ()>(5); // 好的数字
let vec = [1, 2, 3].iter().map(|n| n * 2).collect::<Vec<_>>(); // 向量
```

这个 `::` 前缀是必需的，用于消除 逗号分隔列表 中带有 多个比较 的 泛型路径 的歧义。有关缺少前缀会产生 模糊性 的示例，请参阅 [涡轮鱼的堡垒][turbofish test]。

### 未覆盖类型

未覆盖类型 是不作为另一个 类型 的 参数 出现的 类型。例如，`T` 是 未覆盖 的，但 `Vec<T>` 中的 `T` 是 覆盖的。这仅与 类型参数 相关。

### 未定义行为

未定义行为 是未指定 的 编译时 或 运行时行为。这可能导致（但不限于）：进程终止或损坏；不适当、不正确或意外的计算；或平台特定结果。
[更多][undefined-behavior]。

### 不可居住的

如果一个 类型 没有 构造器，从而永远不能被 实例化，那么它就是 不可居住的。一个 不可居住的 类型在“空”的意义上是“空”的，即该 类型 没有 值。一个 不可居住的 类型 的典型例子是 [never 类型][never type] `!`，或者一个没有 变体 的枚举 `enum Never { }`。与 [可居住的](#可居住的) 相反。

[`extern` blocks]: items.extern
[`extern fn`]: items.fn.extern
[alignment]: type-layout.md#大小和对齐
[associated item]: #associated-item
[attributes]: attributes.md
[*entity*]: names.md
[crate]: crates-and-source-files.md
[dyn compatibility]: items/traits.md#dyn兼容性
[enums]: items/enumerations.md
[fields]: expressions/field-expr.md
[free item]: #free-item
[generic parameters]: items/generics.md
[identifier]: identifiers.md
[identifiers]: identifiers.md
[implementation]: items/implementations.md
[implementations]: items/implementations.md
[inherent implementation]: items/implementations.md#固有实现
[item]: items.md
[items]: items.md
[labels]: tokens.md#生命周期和循环标签
[lifetime or loop label]: tokens.md#生命周期和循环标签
[lifetimes]: tokens.md#生命周期和循环标签
[lints]: attributes/diagnostics.md#lint检查属性
[loop labels]: tokens.md#生命周期和循环标签
[method]: items/associated-items.md#方法
[modules]: items/modules.md
[*Name resolution*]: names/name-resolution.md
[*name*]: names.md
[*namespace*]: names/namespaces.md
[never type]: types/never.md
[*path*]: paths.md
[Paths]: paths.md
[*scope*]: names/scopes.md
[structs]: items/structs.md
[trait object types]: types/trait-object.md
[traits]: items/traits.md
[turbofish test]: https://github.com/rust-lang/rust/blob/1.58.0/src/test/ui/parser/bastion-of-the-turbofish.rs
[types of crates]: linkage.md
[types]: types.md
[undefined-behavior]: behavior-considered-undefined.md
[unions]: items/unions.md
[variable bindings]: patterns.md
[visibility rules]: visibility-and-privacy.md
