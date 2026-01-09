r[expr.call]
# 调用表达式

r[expr.call.syntax]
```grammar,expressions
CallExpression -> Expression `(` CallParams? `)`

CallParams -> Expression ( `,` Expression )* `,`?
```

r[expr.call.intro]
一个  *调用表达式*  调用一个函数。
调用表达式 的 语法格式 是一个表达式（称为  *函数操作数* ），后跟一个括号括起来的、逗号分隔的表达式列表（称为  *参数操作数* ）。

r[expr.call.convergence]
如果函数最终返回，则该表达式完成。

r[expr.call.trait]
对于 [非函数类型][non-function types]，表达式 `f(...)` 会根据 函数操作数 使用以下 特型 之一的方法：

- [`Fn`] 或 [`AsyncFn`] --- 共享引用。
- [`FnMut`] 或 [`AsyncFnMut`] --- 可变引用。
- [`FnOnce`] 或 [`AsyncFnOnce`] --- 值。

r[expr.call.autoref-deref]
如果需要，将进行自动借用。
函数操作数 也会根据需要进行 [自动解引用][automatically dereferenced]。

一些 调用表达式 的示例：

```rust
# fn add(x: i32, y: i32) -> i32 { 0 }
let three: i32 = add(1i32, 2i32);
let name: &'static str = (|| "Rust")();
```

r[expr.call.desugar]
## 消除函数调用歧义

r[expr.call.desugar.fully-qualified]
所有函数调用都是更显式的 [完全限定语法][fully-qualified syntax] 的语法糖。

r[expr.call.desugar.ambiguity]
函数调用可能需要完全限定，具体取决于考虑到作用域内 项 时调用的歧义性。

> [!NOTE]
> 在过去，术语 "Unambiguous Function Call Syntax"、"Universal Function Call Syntax" 或 "UFCS" 已被用于文档、issue、RFC 和其他社区著作中。然而，这些术语缺乏描述力，并且可能会使手头的问题更加混淆。为了便于搜索，我们在这里提及它们。

r[expr.call.desugar.limits]
经常会出现几种导致方法或关联函数调用的接收者或指代对象产生歧义的情况。
这些情况可能包括：

* 多个作用域内的 特型 为相同类型定义了同名方法
* 自动解引用 (Auto-`deref`) 是不希望的；例如，区分智能指针本身的方法和指针指代对象的方法
* 不带参数的方法（如 [`default()`]）并返回类型的属性（如 [`size_of()`]）

r[expr.call.desugar.explicit-path]
为了解决歧义，程序员可以使用更具体的路径、类型或 特型 来引用其所需的方法或函数。

例如，

```rust
trait Pretty {
    fn print(&self);
}

trait Ugly {
    fn print(&self);
}

struct Foo;
impl Pretty for Foo {
    fn print(&self) {}
}

struct Bar;
impl Pretty for Bar {
    fn print(&self) {}
}
impl Ugly for Bar {
    fn print(&self) {}
}

fn main() {
    let f = Foo;
    let b = Bar;

    // 我们可以这样做，因为对于 Foo 类型，我们只有一个名为 print 的项
    f.print();
    // 更显式，而且对于 Foo 来说不是必需的
    Foo::print(&f);
    // 如果你不喜欢这种简洁方式的话
    <Foo as Pretty>::print(&f);

    // b.print(); // 错误：发现多个 'print'
    // Bar::print(&b); // 仍然是错误：发现多个 print

    // 因为作用域内的项定义了 print，所以这是必需的
    <Bar as Pretty>::print(&b);
}
```

有关更多细节和动机，请参阅 [RFC 132]。

[RFC 132]: https://github.com/rust-lang/rfcs/blob/master/text/0132-ufcs.md
[`default()`]: std::default::Default::default
[`size_of()`]: std::mem::size_of
[automatically dereferenced]: field-expr.md#automatic-dereferencing
[fully-qualified syntax]: ../paths.md#qualified-paths
[non-function types]: ../types/function-item.md
