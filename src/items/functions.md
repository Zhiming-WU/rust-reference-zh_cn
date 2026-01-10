r[items.fn]
# 函数

r[items.fn.syntax]
```grammar,items
Function ->
    FunctionQualifiers `fn` IDENTIFIER GenericParams?
        `(` FunctionParameters? `)`
        FunctionReturnType? WhereClause?
        ( BlockExpression | `;` )

FunctionQualifiers -> `const`? `async`?[^async-edition] ItemSafety?[^extern-qualifiers] (`extern` Abi?)?

ItemSafety -> `safe`[^extern-safe] | `unsafe`

Abi -> STRING_LITERAL | RAW_STRING_LITERAL

FunctionParameters ->
      SelfParam `,`?
    | (SelfParam `,`)? FunctionParam (`,` FunctionParam)* `,`?

SelfParam -> OuterAttribute* ( ShorthandSelf | TypedSelf )

ShorthandSelf -> (`&` | `&` Lifetime)? `mut`? `self`

TypedSelf -> `mut`? `self` `:` Type

FunctionParam -> OuterAttribute* ( FunctionParamPattern | `...` | Type[^fn-param-2015] )

FunctionParamPattern -> PatternNoTopAlt `:` ( Type | `...` )

FunctionReturnType -> `->` Type
```

[^async-edition]: `async` 限定符在 2015 版次中是不允许的。

[^extern-safe]: `safe` 函数限定符在语义上仅允许在 `extern` 块内部。

[^extern-qualifiers]: *与 Rust 2024 之前的版次相关* ：在 `extern` 块内部，仅当 `extern` 被限定为 `unsafe` 时，才允许使用 `safe` 或 `unsafe` 函数限定符。

[^fn-param-2015]: 在 2015 版次中，仅包含类型的函数参数仅允许出现在 [特型项][trait item] 的关联函数中。

r[items.fn.intro]
一个 _函数 (function)_ 由一个 [块][block]（即函数的 _主体 (body)_ ）、一个名称、一组参数以及一个输出类型组成。除了名称以外，这些都是可选的。

r[items.fn.namespace]
函数使用关键字 `fn` 声明，它在所在的模块或块的 [值命名空间][value namespace] 中定义给定的名称。

r[items.fn.signature]
函数可以通过参数声明一组 *输入* [*变量*][variables]，调用者通过这些参数将实参传递给函数；函数还可以声明在完成后返回给调用者的值的 *输出* [*类型*][type]。

r[items.fn.implicit-return]
如果未显式说明输出类型，则默认为 [单元类型][unit type]。

r[items.fn.fn-item-type]
当被引用时，一个 _函数_ 会产生一个对应的零大小的 [*函数项类型*][*function item type*] 的一等 *值* ，当它被调用时，会执行对该函数的直接调用。

例如，这是一个简单的函数：
```rust
fn answer_to_life_the_universe_and_everything() -> i32 {
    return 42;
}
```

r[items.fn.safety-qualifiers]
`safe` 函数在语义上仅允许在 [`extern` 块][`extern` block] 中使用。

r[items.fn.params]
## 函数参数

r[items.fn.params.intro]
函数参数是不可反驳的 [模式][patterns]，因此任何在没有 else 的 `let` 绑定中有效的模式作为参数也同样有效：

```rust
fn first((value, _): (i32, i32)) -> i32 { value }
```

r[items.fn.params.self-pat]
如果第一个参数是一个 [SelfParam]，这表明该函数是一个 [方法][method]。

r[items.fn.params.self-restriction]
带有 self 参数的函数只能作为 [特型][trait] 或 [实现][implementation] 中的 [关联函数][associated function] 出现。

r[items.fn.params.varargs]
带有 `...` 词法单元 的参数表示 [变参函数][variadic function]，且只能用作 [外部块][external block] 函数的最后一个参数。变参参数可以有一个可选的标识符，例如 `args: ...`。

r[items.fn.body]
## 函数主体

r[items.fn.body.intro]
函数的主体块在概念上被包装在另一个块中，该块首先绑定参数模式，然后 `return` 函数主体的值。这意味着如果块的尾随表达式被求值，它最终会返回给调用者。像往常一样，函数主体内显式的返回表达式如果被执行，将短路该隐式返回。

例如，上述函数的行为就像它被写成：

<!-- ignore: example expansion -->
```rust,ignore
// argument_0 是从调用者传递的实际第一个参数
let (value, _) = argument_0;
return {
    value
};
```

r[items.fn.body.bodyless]
没有主体块的函数以分号结尾。这种形式只能出现在 [特型][trait] 或 [外部块][external block] 中。

r[items.fn.generics]
## 泛型函数

r[items.fn.generics.intro]
一个 _泛型函数_ 允许在其签名中出现一个或多个 _参数化类型_ 。每个类型参数必须在函数名之后、由尖括号包围且以逗号分隔的列表中显式声明。

```rust
// foo 对 A 和 B 是泛型的

fn foo<A, B>(x: A, y: B) {
# }
```

r[items.fn.generics.param-names]
在函数签名和主体内部，类型参数的名称可以用作类型名称。

r[items.fn.generics.param-bounds]
可以为类型参数指定 [特型][Trait] 绑定，以允许在该类型的值上调用具有该 特型 的方法。这是使用 `where` 语法指定的：

```rust
# use std::fmt::Debug;
fn foo<T>(x: T) where T: Debug {
# }
```

r[items.fn.generics.mono]
当引用泛型函数时，其类型会根据引用的上下文进行实例化。例如，在这里调用 `foo` 函数：

```rust
use std::fmt::Debug;

fn foo<T>(x: &[T]) where T: Debug {
    // 细节已省略
}

foo(&[1, 2]);
```

将使用 `i32` 实例化类型参数 `T`。

r[items.fn.generics.explicit-arguments]
类型参数也可以在函数名之后的尾随 [路径][path] 组件中显式提供。如果没有足够的上下文来确定类型参数，这可能是必要的。例如，`mem::size_of::<u32>() == 4`。

r[items.fn.extern]
## Extern函数限定符

r[items.fn.extern.intro]
`extern` 函数限定符允许提供可以使用特定 ABI 调用的函数 _定义_ ：

<!-- ignore: fake ABI -->
```rust,ignore
extern "ABI" fn foo() { /* ... */ }
```

r[items.fn.extern.def]
这些通常与 [外部块][external block] 项结合使用，后者提供函数 _声明_ ，可用于调用函数而无需提供其 _定义_ ：

<!-- ignore: fake ABI -->
```rust,ignore
unsafe extern "ABI" {
  unsafe fn foo(); /* 没有主体 */
  safe fn bar(); /* 没有主体 */
}
unsafe { foo() };
bar();
```

r[items.fn.extern.default-abi]
当函数 项 的 `FunctionQualifiers` 中省略了 `"extern" Abi?*` 时，将分配 ABI `"Rust"`。例如：

```rust
fn foo() {}
```

等同于：

```rust
extern "Rust" fn foo() {}
```

r[items.fn.extern.foreign-call]
函数可以被外部代码调用，并且使用与 Rust 不同的 ABI 允许提供可以从其他编程语言（如 C）调用的函数：

```rust
// 声明一个具有 "C" ABI 的函数
extern "C" fn new_i32() -> i32 { 0 }

// 声明一个具有 "stdcall" ABI 的函数
# #[cfg(any(windows, target_arch = "x86"))]
extern "stdcall" fn new_i32_stdcall() -> i32 { 0 }
```

r[items.fn.extern.default-extern]
就像 [外部块][external block] 一样，当使用 `extern` 关键字并省略 `"ABI"` 时，使用的 ABI 默认为 `"C"`。也就是说，这段代码：

```rust
extern fn new_i32() -> i32 { 0 }
let fptr: extern fn() -> i32 = new_i32;
```

等同于：

```rust
extern "C" fn new_i32() -> i32 { 0 }
let fptr: extern "C" fn() -> i32 = new_i32;
```

r[items.fn.extern.unwind]
### 展开(Unwinding)

r[items.fn.extern.unwind.intro]
大多数 ABI 字符串有两种变体，一种带有 `-unwind` 后缀，另一种没有。`Rust` ABI 总是允许展开，因此没有 `Rust-unwind` ABI。ABI 的选择与运行时的 [恐慌处理器][panic handler] 一起决定了从函数中展开时的行为。

r[items.fn.extern.unwind.behavior]
下表说明了展开操作到达每种类型的 ABI 边界（使用相应 ABI 字符串的函数声明或定义）时的行为。请注意，Rust 运行时不受完全发生在另一种语言运行时内部的任何展开的影响，也无法对其产生影响，即在未到达 Rust ABI 边界的情况下抛出和捕获的展开。

`panic`-unwind 列是指通过 `panic!` 宏和类似的标准库机制进行的 [恐慌][panicking]，以及任何其他导致恐慌的 Rust 操作，例如数组索引越界或整数溢出。

“展开” ABI 类别是指 `"Rust"`（未标记为 `extern` 的 Rust 函数的隐式 ABI）、`"C-unwind"` 以及任何名称中带有 `-unwind` 的 ABI。“非展开” ABI 类别是指所有其他 ABI 字符串，包括 `"C"` 和 `"stdcall"`。

原生展开是针对每个目标定义的。在支持抛出和捕获 C++ 异常的目标上，它指的是用于实现此功能的机制。某些平台实现了一种被称为 [“强制展开”][forced-unwinding] 的展开形式；Windows 上的 `longjmp` 和 `glibc` 中的 `pthread_exit` 就是这样实现的。强制展开被明确排除在表中的“原生展开”列之外。

| 恐慌运行时 | ABI | `panic`-unwind | 原生展开 (非强制) |
| -------------- | ------------  | ------------------------------------- | -----------------------  |
| `panic=unwind` | 展开型 | 展开 | 展开 |
| `panic=unwind` | 非展开型 | 中止 (见下文注释) | [未定义行为][undefined behavior] |
| `panic=abort`  | 展开型 | `panic` 在不展开的情况下中止 | 中止 |
| `panic=abort`  | 非展开型 | `panic` 在不展开的情况下中止 | [未定义行为][undefined behavior] |

r[items.fn.extern.abort]
在 `panic=unwind` 的情况下，当 恐慌 被非展开 ABI 边界转变为中止时，要么不运行任何析构函数 (`Drop` 调用)，要么运行直到 ABI 边界为止的所有析构函数。具体发生这两种行为中的哪一种是未指定的。

有关跨 FFI 边界展开的其他注意事项和限制，请参阅 [恐慌文档中的相关部分][panic-ffi]。

[forced-unwinding]: https://rust-lang.github.io/rfcs/2945-c-unwind-abi.html#forced-unwinding
[panic handler]: ../panic.md#the-panic_handler-attribute
[panic-ffi]: ../panic.md#unwinding-across-ffi-boundaries
[panicking]: ../panic.md
[undefined behavior]: ../behavior-considered-undefined.md

r[items.fn.const]
## Const函数

有关 const 函数的定义，请参阅 [const 函数][const functions]。

r[items.fn.async]
## Async函数

r[items.fn.async.intro]
函数可以被限定为 async，这也可以与 `unsafe` 限定符结合使用：

```rust
async fn regular_example() { }
async unsafe fn unsafe_example() { }
```

r[items.fn.async.future]
Async 函数在调用时不执行任何操作：相反，它们将参数捕获到一个 future 中。当被轮询时，该 future 将执行函数的主体。

r[items.fn.async.desugar-brief]
一个 async 函数大致相当于一个返回 [`impl Future`][`impl Future`] 并且以 [`async move` 块][async-blocks] 作为其主体的函数：

```rust
// 源码
async fn example(x: &str) -> usize {
    x.len()
}
```

大致相当于：

```rust
# use std::future::Future;
// 脱糖后
fn example<'a>(x: &'a str) -> impl Future<Output = usize> + 'a {
    async move { x.len() }
}
```

r[items.fn.async.desugar]
实际的脱糖过程更为复杂：

r[items.fn.async.lifetime-capture]
- 脱糖中的返回类型被假定为捕获了 `async fn` 声明中的所有生命周期参数。这可以在上面的脱糖示例中看到，它显式地超过了 `'a` 的生命周期，因此捕获了 `'a`。

r[items.fn.async.param-capture]
- 主体中的 [`async move` 块][async-blocks] 捕获了所有函数参数，包括那些未使用或绑定到 `_` 模式的参数。这确保了函数参数的销毁顺序与函数非 async 时相同，不同之处在于销毁发生在返回的 future 被完全 await 之后。

[async-blocks]: ../expressions/block-expr.md#async-blocks
[`impl Future`]: ../types/impl-trait.md

[`impl Future`]: ../types/impl-trait.md

r[items.fn.async.edition2018]
> [!EDITION-2018]
> Async 函数仅从 Rust 2018 开始可用。

r[items.fn.async.safety]
### 结合`async`和`unsafe`

r[items.fn.async.safety.intro]
声明一个既是 async 又是 unsafe 的函数是合法的。生成的函数调用时是不安全的，并且（像任何 async 函数一样）返回一个 future。这个 future 只是一个普通的 future，因此 “await” 它不需要 `unsafe` 上下文：

```rust
// 返回一个在 await 时解引用 `x` 的 future。
//
// 健全性条件：在生成的 future 完成之前，`x` 必须可以安全地解引用。
async unsafe fn unsafe_example(x: *const i32) -> i32 {
  *x
}

async fn safe_example() {
    // 最初调用该函数需要 `unsafe` 块：
    let p = 22;
    let future = unsafe { unsafe_example(&p) };

    // 但这里不需要 `unsafe` 块。这将
    // 读取 `p` 的值：
    let q = future.await;
}
```

请注意，这种行为是脱糖为返回 `impl Future` 的函数的结果 —— 在这种情况下，我们脱糖到的函数是一个 `unsafe` 函数，但返回值保持不变。

Unsafe 在 async 函数上的使用方式与在其他函数上的使用方式完全相同：它表明该函数对调用者施加了一些额外的义务以确保健全性。与任何其他 unsafe 函数一样，这些条件可能会延伸到初始调用本身之外 —— 例如，在上面的代码片段中，`unsafe_example` 函数接受一个指针 `x` 作为参数，然后 (在被 await 时) 解引用该指针。这意味着 `x` 必须在 future 完成执行之前保持有效，而确保这一点是调用者的责任。

r[items.fn.attributes]
## 函数上的属性

r[items.fn.attributes.intro]
函数允许使用 [外部属性][attributes]。[内部属性][attributes] 允许直接在函数主体 [块][block] 内部的 `{` 之后使用。

这个例子展示了函数上的内部属性。该函数仅使用单词 “Example” 进行文档说明。

```rust
fn documented() {
    #![doc = "Example"]
}
```

> [!NOTE]
> 除了 lint 之外，在函数 项 上仅使用外部属性是符合习惯的。

r[items.fn.attributes.builtin-attributes]
对函数有意义的属性有：

- [`cfg_attr`]
- [`cfg`]
- [`cold`]
- [`deprecated`]
- [`doc`]
- [`export_name`]
- [`inline`]
- [`link_section`]
- [`must_use`]
- [`no_mangle`]
- [Lint 检查属性][Lint check attributes]
- [过程宏属性][Procedural macro attributes]
- [测试属性][Testing attributes]

r[items.fn.param-attributes]
## 函数参数上的属性

r[items.fn.param-attributes.intro]
函数参数允许使用 [外部属性][attributes]，允许的 [内置属性][built-in attributes] 限于 `cfg`、`cfg_attr`、`allow`、`warn`、`deny` 和 `forbid`。

```rust
fn len(
    #[cfg(windows)] slice: &[u16],
    #[cfg(not(windows))] slice: &[u8],
) -> usize {
    slice.len()
}
```

r[items.fn.param-attributes.parsed-attributes]
应用于 项 的 过程宏 属性所使用的惰性辅助属性也是允许的，但要注意不要在最终的 `TokenStream` 中包含这些惰性属性。

例如，以下代码定义了一个在任何地方都没有正式定义的惰性 `some_inert_attribute` 属性，而 `some_proc_macro_attribute` 过程宏 负责检测它的存在并将其从输出词法单元流中移除。

<!-- ignore: requires proc macro -->
```rust,ignore
#[some_proc_macro_attribute]
fn foo_oof(#[some_inert_attribute] arg: u8) {
}
```

[const contexts]: ../const_eval.md#const-context
[const functions]: ../const_eval.md#const-functions
[external block]: external-blocks.md
[path]: ../paths.md
[block]: ../expressions/block-expr.md
[variables]: ../variables.md
[type]: ../types.md#type-expressions
[unit type]: ../types/tuple.md
[*function item type*]: ../types/function-item.md
[Trait]: traits.md
[attributes]: ../attributes.md
[`cfg`]: ../conditional-compilation.md#cfg属性
[`cfg_attr`]: ../conditional-compilation.md#cfg_attr属性
[lint check attributes]: ../attributes/diagnostics.md#lint-check-attributes
[procedural macro attributes]: macro.proc.attribute
[testing attributes]: ../attributes/testing.md
[`cold`]: ../attributes/codegen.md#the-cold-attribute
[`inline`]: ../attributes/codegen.md#the-inline-attribute
[`deprecated`]: ../attributes/diagnostics.md#the-deprecated-attribute
[`doc`]: ../../rustdoc/the-doc-attribute.html
[`must_use`]: ../attributes/diagnostics.md#the-must_use-attribute
[patterns]: ../patterns.md
[`export_name`]: ../abi.md#the-export_name-attribute
[`link_section`]: ../abi.md#the-link_section-attribute
[`no_mangle`]: ../abi.md#the-no_mangle-attribute
[built-in attributes]: ../attributes.md#built-in-attributes-index
[trait item]: traits.md
[method]: associated-items.md#方法
[associated function]: associated-items.md#associated-functions-and-methods
[implementation]: implementations.md
[value namespace]: ../names/namespaces.md
[variadic function]: external-blocks.md#variadic-functions
[`extern` block]: external-blocks.md
