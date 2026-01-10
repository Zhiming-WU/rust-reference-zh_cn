r[expr.block]
# 块表达式

r[expr.block.syntax]
```grammar,expressions
BlockExpression ->
    `{`
        InnerAttribute*
        Statements?
    `}`

Statements ->
      Statement+
    | Statement+ ExpressionWithoutBlock
    | ExpressionWithoutBlock
```

r[expr.block.intro]
一个 *块表达式* ，或称 *块* ，是一个控制流表达式，也是 项 和变量声明的匿名命名空间作用域。

r[expr.block.sequential-evaluation]
作为控制流表达式，块会顺序执行其组成的非 项 声明语句，然后执行其最后的可选表达式。

r[expr.block.namespace]
作为匿名命名空间作用域， 项 声明仅在块内部有效，而由 `let` 语句声明的变量从下一条语句开始到块结束为止在作用域内有效。
有关更多详细信息，请参阅 [作用域][scopes] 章节。

r[expr.block.inner-attributes]
块的 语法格式 为 `{` ，然后是任何 [内部属性][inner attributes] ，接着是任意数量的 [语句][statements] ，然后是一个可选表达式（称为最终操作数），最后是 `}` 。

r[expr.block.statements]
通常要求语句后跟分号，但有两个例外：

1. 项 声明语句不需要后跟分号。
2. 表达式语句通常需要后跟分号，除非其外部表达式是流控制表达式。

r[expr.block.null-statement]
此外，允许语句之间存在额外的分号，但这些分号不影响语义。

r[expr.block.evaluation]
在对块表达式求值时，除 项 声明语句外的每个语句都会按顺序执行。

r[expr.block.result]
然后执行最终操作数（如果有）。

r[expr.block.type]
块的类型是最终操作数的类型，如果省略最终操作数，则为 `()` 。

```rust
# fn fn_call() {}
let _: () = {
    fn_call();
};

let five: i32 = {
    fn_call();
    5
};

assert_eq!(5, five);
```

> [!NOTE]
> 作为控制流表达式，如果块表达式是表达式语句的外部表达式，则其预期类型为 `()` ，除非它后面紧跟一个分号。

r[expr.block.value]
块始终是 [值表达式][value expressions] ，并在值表达式语境中对最后一个操作数求值。

> [!NOTE]
> 如果确实需要，这可以用于强制移动一个值。例如，以下示例在调用 `consume_self` 时失败，因为 结构体 在块表达式中已从 `s` 中移出。
>
> ```rust,compile_fail
> struct Struct;
>
> impl Struct {
>     fn consume_self(self) {}
>     fn borrow_self(&self) {}
> }
>
> fn move_by_block_expression() {
>     let s = Struct;
>
>     // 在块表达式中将值从 `s` 中移出。
>     (&{ s }).borrow_self();
>
>     // 执行失败，因为 `s` 已被移出。
>     s.consume_self();
> }
> ```

r[expr.block.async]
## `async`块

r[expr.block.async.syntax]
```grammar,expressions
AsyncBlockExpression -> `async` `move`? BlockExpression
```

r[expr.block.async.intro]
一个 *async 块* 是块表达式的一种变体，其求值结果为一个 future。

r[expr.block.async.future-result]
块的最终表达式（如果存在）决定了 future 的结果值。

r[expr.block.async.anonymous-type]
执行 async 块类似于执行闭包表达式：
它的直接效果是产生并返回一个匿名类型。

r[expr.block.async.future]
然而，闭包返回实现了一个或多个 [`std::ops::Fn`] 特型 的类型，而 async 块返回的类型实现了 [`std::future::Future`] 特型 。

r[expr.block.async.layout-unspecified]
该类型的实际数据格式未指定。

> [!NOTE]
>  rustc 生成的 future 类型大致相当于一个枚举，每个 `await` 点对应一个变体，每个变体存储从其对应点恢复所需的数据。

r[expr.block.async.edition2018]
> [!EDITION-2018]
>  async 块仅从 Rust 2018 开始可用。

r[expr.block.async.capture]
### 捕获模式

async 块使用与闭包相同的 [捕获模式][capture modes] 从其环境中捕获变量。
与闭包类似，当写成 `async { .. }` 时，每个变量的捕获模式将从块的内容中推导出来。
然而， `async move { .. }` 块会将所有引用的变量移动到生成的 future 中。

r[expr.block.async.context]
### Async语境

由于 async 块构造一个 future，它们定义了一个 **async 语境** ，而该语境又可以包含 [await 表达式][`await` expressions] 。
Async 语境由 async 块以及 async 函数体建立，后者的语义是根据 async 块定义的。

r[expr.block.async.function]
### 控制流运算符

r[expr.block.async.function.intro]
async 块就像函数边界一样，非常类似于闭包。

r[expr.block.async.function.return-try]
因此， `?` 运算符和 `return` 表达式都影响 future 的输出，而不是外层函数或其他语境。
也就是说，在 async 块内 `return <expr>` 将返回 `<expr>` 的结果作为 future 的输出。
同样地，如果 `<expr>?` 传播一个错误，该错误将作为 future 的结果传播。

r[expr.block.async.function.control-flow]
最后， `break` 和 `continue` 关键字不能用于从 async 块中跳出。
因此，以下代码是非法的：

```rust,compile_fail
loop {
    async move {
        break; // error[E0267]: 在 `async` 块内部使用 `break`
    }
}
```

r[expr.block.const]
## `const`块

r[expr.block.const.syntax]
```grammar,expressions
ConstBlockExpression -> `const` BlockExpression
```

r[expr.block.const.intro]
一个 *const 块* 是块表达式的一种变体，其主体在编译时而非运行时求值。

r[expr.block.const.context]
const 块允许你定义一个常量值，而无需定义新的 [常量项][constant items] ，因此它们有时也被称为 *内联 const* 。
它还支持类型推导，因此不需要指定类型，这与 [常量项][constant items] 不同。

r[expr.block.const.generic-params]
const 块能够引用作用域内的泛型参数，这与 [自由][free item] 常量项不同。
它们被脱糖为在作用域内带有泛型参数的常量项（类似于关联常量，但没有它们与之关联的 特型 或类型）。
例如，这段代码：

```rust
fn foo<T>() -> usize {
    const { std::mem::size_of::<T>() + 1 }
}
```

等价于：

```rust
fn foo<T>() -> usize {
    {
        struct Const<T>(T);
        impl<T> Const<T> {
            const CONST: usize = std::mem::size_of::<T>() + 1;
        }
        Const::<T>::CONST
    }
}
```

r[expr.block.const.evaluation]

如果 const 块表达式在运行时执行，则保证会对该常量求值，即使其返回值被忽略：

```rust
fn foo<T>() -> usize {
    // 如果这段代码被执行，那么断言肯定已经在编译时求值了。
    const { assert!(std::mem::size_of::<T>() > 0); }
    // 在这里，我们可以编写依赖于类型为非零大小的 unsafe 代码。
    /* ... */
    42
}
```

r[expr.block.const.not-executed]

如果 const 块表达式不在运行时执行，它可能会也可能不会被求值：
```rust,compile_fail
if false {
    // 程序构建时可能会也可能不会发生 恐慌 。
    const { panic!(); }
}
```

r[expr.block.unsafe]
## `unsafe`块

r[expr.block.unsafe.syntax]
```grammar,expressions
UnsafeBlockExpression -> `unsafe` BlockExpression
```

r[expr.block.unsafe.intro]
_ 有关何时使用 `unsafe` 的更多信息，请参阅 [unsafe 块][`unsafe` blocks] _。

代码块可以加上 `unsafe` 关键字前缀，以允许 [unsafe 操作][unsafe operations] 。
示例：

```rust
unsafe {
    let b = [13u8, 17u8];
    let a = &b[0] as *const u8;
    assert_eq!(*a, 13);
    assert_eq!(*a.offset(1), 17);
}

# unsafe fn an_unsafe_fn() -> i32 { 10 }
let a = unsafe { an_unsafe_fn() };
```

r[expr.block.label]
## 标签块表达式

标签块表达式在 [循环和其他可中断表达式][Loops and other breakable expressions] 章节中有文档说明。

r[expr.block.attributes]
## 块表达式上的属性

r[expr.block.attributes.inner-attributes]
在以下情况下，允许在块表达式的左大括号后直接使用 [内部属性][inner attributes] ：

* [函数][Function] 和 [方法][method] 体。
* 循环体（ [`loop`] 、 [`while`] 和 [`for`] ）。
* 用作 [语句][statement] 的块表达式。
* 作为 [数组表达式][array expressions] 、 [元组表达式][tuple expressions] 、
  [调用表达式][call expressions] 和元组风格 [结构体][struct] 表达式元素的块表达式。
* 作为另一个块表达式的尾随表达式的块表达式。
<!-- Keep list in sync with expressions.md -->

r[expr.block.attributes.valid]
在块表达式上有意义的属性是 [`cfg`] 和 [lint 检查属性][the lint check attributes] 。

例如，此函数在 unix 平台上返回 `true` ，在其他平台上返回 `false` 。

```rust
fn is_unix_platform() -> bool {
    #[cfg(unix)] { true }
    #[cfg(not(unix))] { false }
}
```

[`await` expressions]: await-expr.md
[`cfg`]: ../conditional-compilation.md
[`for`]: loop-expr.md#iterator-loops
[`loop`]: loop-expr.md#infinite-loops
[`unsafe` blocks]: ../unsafe-keyword.md#unsafe-blocks-unsafe-
[`while`]: loop-expr.md#predicate-loops
[array expressions]: array-expr.md
[call expressions]: call-expr.md
[capture modes]: ../types/closure.md#capture-modes
[constant items]: ../items/constant-items.md
[free item]: ../glossary.md#自由项
[function]: ../items/functions.md
[inner attributes]: ../attributes.md
[method]: ../items/associated-items.md#方法
[mutable reference]: ../types/pointer.md#mutables-references-
[scopes]: ../names/scopes.md
[shared references]: ../types/pointer.md#shared-references-
[statement]: ../statements.md
[statements]: ../statements.md
[struct]: struct-expr.md
[the lint check attributes]: ../attributes/diagnostics.md#lint-check-attributes
[tuple expressions]: tuple-expr.md
[unsafe operations]: ../unsafety.md
[value expressions]: ../expressions.md#位置表达式和值表达式
[Loops and other breakable expressions]: expr.loop.block-labels
