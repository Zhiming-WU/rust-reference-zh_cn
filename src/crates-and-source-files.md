<!-- https://github.com/rust-lang/reference/blob/master/src/crates-and-source-files.md -->
<!-- commit 68bdfd15fa9feebfbf94a06cf574de598e239198 -->

r[crate]
# Crate与源文件

r[crate.syntax]
```grammar,items
@root Crate ->
    InnerAttribute*
    Item*
```

> [!NOTE]
> 尽管Rust与其他任何语言一样，既可以由解释器实现，也可以由编译器实现，但目前唯一的实现是编译器，并且该语言始终被设计为可编译的。因此，本节假定存在一个编译器。

r[crate.compile-time]
Rust的语义遵循编译期与运行期之间的 *阶段区别*[^phase-distinction]。具有 *静态解释* 的语义规则决定编译的成功或失败，而具有 *动态解释* 的语义规则决定程序在运行时的行为。

r[crate.unit]
编译模型围绕称为 _crate_ 的制品展开。每次编译都会处理一个源代码形式的单个crate，如果成功，则生成一个二进制形式的单个crate：一个可执行文件或某种库。[^cratesourcefile]

r[crate.module]
一个 _crate_ 是编译和链接的单元，也是版本控制、分发和运行时加载的单元。一个crate包含一个嵌套的[模块][module]作用域_树_。这棵树的顶层是一个匿名模块（从模块内部路径的角度来看），并且crate内的任何[项][item]都有一个规范的[模块路径][module path]，表示其在crate的模块树中的位置。

r[crate.input-source]
Rust编译器总是以单个源文件作为输入被调用，并总是生成单个输出crate。该源文件的处理可能会导致其他源文件作为模块被加载。源文件的扩展名为`.rs`。

r[crate.module-def]
一个Rust源文件描述了一个模块，该模块的名称和位置&mdash;&mdash;在当前crate的模块树中&mdash;&mdash;是从源文件外部定义的：可以通过引用源文件中的显式[Module][grammar-Module]项，或者通过crate本身的名称来定义。

r[crate.inline-module]
每个源文件都是一个模块，但并非每个模块都需要自己的源文件：[模块定义][module]可以嵌套在一个文件中。

r[crate.items]
每个源文件包含零个或多个[项][Item]定义序列，并且可以选择以任意数量应用于包含模块的[属性][attributes]开头，其中大部分会影响编译器的行为。

r[crate.attributes]
匿名crate模块可以拥有适用于整个crate的额外属性。

> [!NOTE]
> 文件内容可以由[shebang][shebang]开头。

```rust
// 指定crate名称。
#![crate_name = "projx"]

// 指定输出制品类型。
#![crate_type = "lib"]

// 开启一个警告。
// 这可以在任何模块中完成，而不仅仅是匿名crate模块。
#![warn(non_camel_case_types)]
```

r[crate.main]
## main函数

r[crate.main.general]
包含`main`[函数][function]的crate可以编译成可执行文件。

r[crate.main.restriction]
如果存在`main`函数，它必须不接受任何参数，不得声明任何[特型或生命周期边界][trait or lifetime bounds]，不得有任何[where子句][where clauses]，并且其返回类型必须实现[`Termination`]特型。

```rust
fn main() {}
```
```rust
fn main() -> ! {
    std::process::exit(0);
}
```
```rust
fn main() -> impl std::process::Termination {
    std::process::ExitCode::SUCCESS
}
```

r[crate.main.import]
`main`函数可以是一个导入，例如来自外部crate或当前crate。

```rust
mod foo {
    pub fn bar() {
        println!("Hello, world!");
    }
}
use foo::bar as main;
```

> [!NOTE]
> 标准库中实现了[`Termination`]的类型包括：
>
> *   `()`
> *   [`!`]
> *   [`Infallible`]
> *   [`ExitCode`]
> *   `Result<T, E> where T: Termination, E: Debug`

<!-- If the previous section needs updating (from "must take no arguments"
  onwards, also update it in the testing.md file -->

r[crate.uncaught-foreign-unwinding]
### 未捕获的外部unwinding

当"外部"unwind（例如从C++代码抛出的异常，或使用不同恐慌处理器的Rust代码中的`panic!`）传播超出`main`函数时，进程将安全终止。这可能以中止的形式发生，在这种情况下，不保证会执行任何`Drop`调用，并且错误输出可能不如运行时被"原生"Rust `panic`终止时那么详细。

欲了解更多信息，请参阅[恐慌文档][panic-docs]。

r[crate.no_main]
### `no_main`属性

*`no_main`[属性][attribute]* 可以应用于crate级别，以禁用为可执行二进制文件发出`main`符号。当链接到某个其他对象定义了`main`时，这会很有用。

r[crate.crate_name]
## `crate_name`属性

r[crate.crate_name.general]
*`crate_name`[属性][attribute]* 可以应用于crate级别，使用[MetaNameValueStr]语法格式指定crate的名称。

```rust
#![crate_name = "mycrate"]
```

r[crate.crate_name.restriction]
crate名称不能为空，并且只能包含[Unicode字母数字][Unicode alphanumeric]字符或`_`(U+005F)字符。

[^phase-distinction]: 这种区别也存在于解释器中。静态检查，如语法分析、类型检查和linting，无论程序何时执行，都应该在程序执行之前进行。

[^cratesourcefile]: 一个crate在某种程度上类似于ECMA-335 CLI模型中的*assembly*，SML/NJ Compilation Manager中的 *库(library)*，Owens和Flatt模块系统中的 *单元(unit)*，或Mesa中的 *配置(configuration)*。

[Unicode alphanumeric]: https://doc.rust-lang.org/std/primitive.char.html#method.is_alphanumeric
[`!`]: types/never.md
[`ExitCode`]: https://doc.rust-lang.org/std/process/struct.ExitCode.html
[`Infallible`]: https://doc.rust-lang.org/core/convert/enum.Infallible.html
[`Termination`]: https://doc.rust-lang.org/std/process/trait.Termination.html
[attribute]: attributes.md
[attributes]: attributes.md
[function]: items/functions.md
[module]: items/modules.md
[module path]: paths.md
[panic-docs]: panic.md#unwinding-across-ffi-boundaries
[shebang]: input-format.md#shebang移除
[trait or lifetime bounds]: trait-bounds.md
[where clauses]: items/generics.md#where-clauses
