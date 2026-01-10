r[destructors]
# 析构函数

r[destructors.intro]
当一个 [已初始化][initialized]&#32;[变量][variable] 或 [临时变量][temporary] 离开 [作用域](#drop-scopes) 时，其 *析构函数* 会被运行，或者说它被 *销毁(dropped)*。 [赋值][Assignment] 也会运行其左侧操作数（如果已初始化）的析构函数。如果一个变量已被部分初始化，则仅销毁其已初始化的字段。

r[destructors.operation]
类型 `T` 的析构函数由以下部分组成：

1. 如果 `T: Drop`，则调用 [`<T as core::ops::Drop>::drop`](core::ops::Drop::drop)
2. 递归运行其所有字段的析构函数。
    * [结构体][struct] 的字段按声明顺序销毁。
    * 活跃 [枚举变体][enum variant] 的字段按声明顺序销毁。
    * [元组][tuple] 的字段按顺序销毁。
    * [数组][array] 或拥有所有权的 [切片][slice] 的元素从第一个元素到最后一个元素依次销毁。
    * [闭包][closure] 通过移动(move)捕获的变量按未指定的顺序销毁。
    * [特型对象][Trait objects] 运行底层类型的析构函数。
    * 其他类型不会导致进一步的销毁操作。

r[destructors.drop_in_place]
如果必须手动运行析构函数（例如在实现自己的智能指针时），可以使用 [`core::ptr::drop_in_place`] 。

一些示例：

```rust
struct PrintOnDrop(&'static str);

impl Drop for PrintOnDrop {
    fn drop(&mut self) {
        println!("{}", self.0);
    }
}

let mut overwritten = PrintOnDrop("drops when overwritten");
overwritten = PrintOnDrop("drops when scope ends"); // 覆盖时销毁

let tuple = (PrintOnDrop("Tuple first"), PrintOnDrop("Tuple second"));

let moved;
// 赋值时不运行析构函数。
moved = PrintOnDrop("Drops when moved");
// 现在销毁，但之后变为未初始化状态。
moved;

// 未初始化不会销毁。
let uninitialized: PrintOnDrop;

// 部分移动后，仅销毁剩余字段。
let mut partial_move = (PrintOnDrop("first"), PrintOnDrop("forgotten"));
// 执行部分移动，仅保留 `partial_move.0` 为初始化状态。
core::mem::forget(partial_move.1);
// 当 partial_move 的作用域结束时，仅销毁第一个字段。
```

r[destructors.scope]
## 销毁作用域

r[destructors.scope.intro]
每个变量或临时变量都与一个 *销毁作用域* 相关联。当控制流离开销毁作用域时，与该作用域相关的所有变量都将按声明（对于变量）或创建（对于临时变量）的逆序进行销毁。

r[destructors.scope.desugaring]
可以通过使用 [`match`] 、 [`loop`] 和 `break` 将 [`for`] 、 [`if`] 和 [`while`] 表达式替换为等效表达式来确定销毁作用域。

r[destructors.scope.operators]
重载运算符与内置运算符不作区分，且不考虑 [绑定模式][binding modes] 。

r[destructors.scope.list]
给定一个函数或闭包，存在以下销毁作用域：

r[destructors.scope.function]
* 整个函数

r[destructors.scope.statement]
* 每个 [语句][statement]

r[destructors.scope.expression]
* 每个 [表达式][expression]

r[destructors.scope.block]
* 每个块，包括函数体
    * 在 [块表达式][block expression] 的情况下，块的作用域和表达式的作用域是同一个作用域。

r[destructors.scope.match-arm]
* `match` 表达式的每个分支(arm)

r[destructors.scope.nesting]
销毁作用域按如下方式相互嵌套。当同时离开多个作用域时（例如从函数返回时），变量按从内向外的顺序销毁。

r[destructors.scope.nesting.function]
* 整个函数作用域是最外层作用域。

r[destructors.scope.nesting.function-body]
* 函数体块包含在整个函数的作用域内。

r[destructors.scope.nesting.expr-statement]
* 表达式语句中表达式的父作用域是该语句的作用域。

r[destructors.scope.nesting.let-initializer]
* [`let` 语句][`let` statement] 的初始化器的父作用域是该 `let` 语句的作用域。

r[destructors.scope.nesting.statement]
* 语句作用域的父作用域是包含该语句的块的作用域。

r[destructors.scope.nesting.match-guard]
* `match` 守卫(guard)表达式的父作用域是该守卫所属分支的作用域。

r[destructors.scope.nesting.match-arm]
* `match` 表达式中 `=>` 之后表达式的父作用域是该表达式所属分支的作用域。

r[destructors.scope.nesting.match]
* 分支作用域的父作用域是该分支所属 `match` 表达式的作用域。

r[destructors.scope.nesting.other]
* 所有其他作用域的父作用域是直接包围它们的表达式的作用域。

r[destructors.scope.params]
### 函数参数的作用域

所有函数参数都在整个函数体的作用域内，因此在评估函数时最后被销毁。每个实际的函数参数都在该参数模式中引入的任何绑定之后被销毁。

```rust
# struct PrintOnDrop(&'static str);
# impl Drop for PrintOnDrop {
#     fn drop(&mut self) {
#         println!("drop({})", self.0);
#     }
# }
// 先销毁 `y`，然后是第二个参数，接着是 `x`，最后是第一个参数
fn patterns_in_parameters(
    (x, _): (PrintOnDrop, PrintOnDrop),
    (_, y): (PrintOnDrop, PrintOnDrop),
) {}

// 销毁顺序是 3 2 0 1
patterns_in_parameters(
    (PrintOnDrop("0"), PrintOnDrop("1")),
    (PrintOnDrop("2"), PrintOnDrop("3")),
);
```

r[destructors.scope.bindings]
### 局部变量的作用域

r[destructors.scope.bindings.intro]
在 `let` 语句中声明的局部变量与包含该 `let` 语句的块的作用域相关联。在 `match` 表达式中声明的局部变量与它们所声明的 `match` 分支作用域相关联。

```rust
# struct PrintOnDrop(&'static str);
# impl Drop for PrintOnDrop {
#     fn drop(&mut self) {
#         println!("drop({})", self.0);
#     }
# }
let declared_first = PrintOnDrop("Dropped last in outer scope");
{
    let declared_in_block = PrintOnDrop("Dropped in inner scope");
}
let declared_last = PrintOnDrop("Dropped first in outer scope");
```

r[destructors.scope.bindings.patterns]
模式中的变量按模式内声明顺序的逆序销毁。

```rust
# struct PrintOnDrop(&'static str);
# impl Drop for PrintOnDrop {
#     fn drop(&mut self) {
#         println!("drop({})", self.0);
#     }
# }
let (declared_first, declared_last) = (
    PrintOnDrop("Dropped last"),
    PrintOnDrop("Dropped first"),
);
```

r[destructors.scope.bindings.or-patterns]
出于销毁顺序的目的， [或模式][or-patterns] 按第一个子模式给出的顺序声明绑定。

```rust
# struct PrintOnDrop(&'static str);
# impl Drop for PrintOnDrop {
#     fn drop(&mut self) {
#         println!("drop({})", self.0);
#     }
# }
// 先销毁 `x` 后销毁 `y`。
fn or_pattern_drop_order<T>(
    (Ok([x, y]) | Err([y, x])): Result<[T; 2], [T; 2]>
//   ^^^^^^^^^^   ^^^^^^^^^^^ 这是第二个子模式。
//   |
//   这是第一个子模式。
//
//   在第一个子模式中，`x` 在 `y` 之前声明。由于它是
//   第一个子模式，即使匹配的是第二个子模式（其中绑定的声明顺序相反），
//   也会使用该顺序。
) {}

// 这里我们匹配第一个子模式，销毁按第一个子模式中的声明顺序发生。
or_pattern_drop_order(Ok([
    PrintOnDrop("Declared first, dropped last"),
    PrintOnDrop("Declared last, dropped first"),
]));

// 这里我们匹配第二个子模式，销毁仍按第一个子模式中的声明顺序发生。
or_pattern_drop_order(Err([
    PrintOnDrop("Declared last, dropped first"),
    PrintOnDrop("Declared first, dropped last"),
]));
```

r[destructors.scope.temporary]
### 临时作用域

r[destructors.scope.temporary.intro]
表达式的 *临时作用域* 是指当表达式用于 [位置上下文][place context] 时，用于持有该表达式结果的临时变量的作用域，除非该表达式被 [提升][promoted] 。

r[destructors.scope.temporary.enclosing]
除生命周期延长外，表达式的临时作用域是包含该表达式的最小作用域，且为以下之一：

* 整个函数。
* 一个语句。
* [`if`] 、 [`while`] 或 [`loop`] 表达式的体部。
* `if` 表达式的 `else` 块。
* `if` 或 `while` 表达式中非模式匹配的条件表达式，或 `match` 守卫。
* match 分支的体表达式。
* [惰性布尔表达式][lazy boolean expression] 的每个操作数。
* [`if`] 的模式匹配条件及随后的体部 ([destructors.scope.temporary.edition2024])。
* [`while`] 的模式匹配条件和循环体。
* 块的尾随表达式的整体 ([destructors.scope.temporary.edition2024])。

> [!NOTE]
> `match` 表达式的 [受查表达式][scrutinee] 不是临时作用域，因此受查表达式中的临时变量可以在 `match` 表达式之后销毁。例如， `match 1 { ref mut z => z };` 中 `1` 的临时变量存活到语句结束。

> [!NOTE]
> [解构赋值][destructuring assignment] 的脱糖限制了其赋值操作数（右侧值）的临时作用域。有关详细信息，请参阅 [expr.assign.destructure.tmp-scopes] 。

r[destructors.scope.temporary.edition2024]
> [!EDITION-2024]
> 2024 版次增加了两条新的临时作用域收窄规则： `if let` 临时变量在 `else` 块之前销毁，块的尾随表达式的临时变量在尾随表达式评估后立即销毁。

一些示例：

```rust
# #![allow(irrefutable_let_patterns)]
# struct PrintOnDrop(&'static str);
# impl Drop for PrintOnDrop {
#     fn drop(&mut self) {
#         println!("drop({})", self.0);
#     }
# }
let local_var = PrintOnDrop("local var");

// 一旦条件评估完毕就会被销毁
if PrintOnDrop("If condition").0 == "If condition" {
    // 在块结束时销毁
    PrintOnDrop("If body").0
} else {
    unreachable!()
};

if let "if let scrutinee" = PrintOnDrop("if let scrutinee").0 {
    PrintOnDrop("if let consequent").0
    // `if let consequent` 在此处销毁
}
// `if let scrutinee` 在此处销毁
else {
    PrintOnDrop("if let else").0
    // `if let else` 在此处销毁
};

while let x = PrintOnDrop("while let scrutinee").0 {
    PrintOnDrop("while let loop body").0;
    break;
    // `while let loop body` 在此处销毁。
    // `while let scrutinee` 在此处销毁。
}

// 在第一个 || 之前销毁
(PrintOnDrop("first operand").0 == ""
// 在 ) 之前销毁
|| PrintOnDrop("second operand").0 == "")
// 在 ; 之前销毁
|| PrintOnDrop("third operand").0 == "";

// 受查表达式在函数结束时、局部变量之前销毁
// （因为这是函数体块的尾随表达式）。
match PrintOnDrop("Matched value in final expression") {
    // 一旦条件评估完毕就会被销毁
    _ if PrintOnDrop("guard condition").0 == "" => (),
    _ => (),
}
```

r[destructors.scope.operands]
### 操作数

在评估表达式的其他操作数时，也会创建临时变量来持有该表达式的操作数结果。这些临时变量与具有该操作数的表达式的作用域相关联。由于一旦表达式评估完毕，临时变量就会被移走，因此销毁它们没有影响，除非表达式的一个操作数跳出了表达式、返回或发生了 [恐慌][panic] 。

```rust
# struct PrintOnDrop(&'static str);
# impl Drop for PrintOnDrop {
#     fn drop(&mut self) {
#         println!("drop({})", self.0);
#     }
# }
loop {
    // 元组表达式未完成评估，因此操作数按逆序销毁
    (
        PrintOnDrop("Outer tuple first"),
        PrintOnDrop("Outer tuple second"),
        (
            PrintOnDrop("Inner tuple first"),
            PrintOnDrop("Inner tuple second"),
            break,
        ),
        PrintOnDrop("Never created"),
    );
}
```

r[destructors.scope.const-promotion]
### 常量提升

当一个值表达式可以写在常量中并被借用，且该借用可以在表达式最初编写的地方被解引用而不会改变运行时行为时，就会发生将该表达式提升到 `'static` 槽位的操作。也就是说，被提升的表达式可以在编译时评估，且结果值不包含 [内部可变性][interior mutability] 或 [析构函数][destructors] （这些属性尽可能根据值来确定，例如 `&None` 始终具有类型 `&'static Option<_>` ，因为它不包含任何被禁止的内容）。

r[destructors.scope.lifetime-extension]
### 临时生命周期延长

> [!NOTE]
> 临时生命周期延长的具体规则可能会发生变化。此处仅描述当前行为。

r[destructors.scope.lifetime-extension.let]
在 `let` 语句中，表达式的临时作用域有时会 *延长* 到包含该 `let` 语句的块的作用域。根据某些语法规则，当通常的临时作用域太小时会执行此操作。例如：

```rust
let x = &mut 0;
// 通常临时变量现在已经被销毁了，但 `0` 的临时变量
// 存活到块结束。
println!("{}", x);
```

r[destructors.scope.lifetime-extension.static]
生命周期延长也适用于 `static` 和 `const` 项 ，这使得临时变量存活到程序结束。例如：

```rust
const C: &Vec<i32> = &Vec::new();
// 通常这将是一个悬空引用，因为 `Vec` 仅存在于 `C` 的
// 初始化表达式内部，但这里的借用被延长了生命周期，
// 因此它实际上具有 `'static` 生命周期。
println!("{:?}", C);
```

r[destructors.scope.lifetime-extension.sub-expressions]
如果 [借用][borrow] 、 [解引用表达式][dereference expression] 、 [字段表达式][field expression] 或 [元组索引表达式][tuple indexing expression] 具有延长的临时作用域，则其操作数也具有。如果 [索引表达式][indexing expression] 具有延长的临时作用域，则被索引的表达式也具有延长的临时作用域。

r[destructors.scope.lifetime-extension.patterns]
#### 基于模式的延长

r[destructors.scope.lifetime-extension.patterns.extending]
一个 *延长模式* 是以下之一：

* 通过引用或可变引用绑定的 [标识符模式][identifier pattern] 。

  ```rust
  # fn temp() {}
  let ref x = temp(); // 通过引用绑定。
  # x;
  let ref mut x = temp(); // 通过可变引用绑定。
  # x;
  ```

* [结构体模式][struct pattern] 、 [元组模式][tuple pattern] 、 [元组结构体模式][tuple struct pattern] 、 [切片模式][slice pattern] 或 [或模式][or-patterns] ，其中至少一个直接子模式是延长模式。

  ```rust
  # use core::sync::atomic::{AtomicU64, Ordering::Relaxed};
  # static X: AtomicU64 = AtomicU64::new(0);
  struct W<T>(T);
  # impl<T> Drop for W<T> { fn drop(&mut self) { X.fetch_add(1, Relaxed); } }
  let W { 0: ref x } = W(()); // 结构体模式。
  # x;
  let W(ref x) = W(()); // 元组结构体模式。
  # x;
  let (W(ref x),) = (W(()),); // 元组模式。
  # x;
  let [W(ref x), ..] = [W(())]; // 切片模式。
  # x;
  let (Ok(W(ref x)) | Err(&ref x)) = Ok(W(())); // 或模式。
  # x;
  //
  // 以上所有临时变量在这里仍然存活。
  # assert_eq!(0, X.load(Relaxed));
  ```

因此 `ref x` 、 `V(ref x)` 和 `[ref x, y]` 都是延长模式，但 `x` 、 `&ref x` 和 `&(ref x,)` 则不是。

r[destructors.scope.lifetime-extension.patterns.let]
如果 `let` 语句中的模式是延长模式，则初始化器表达式的临时作用域将被延长。

```rust
# fn temp() {}
// 这是一个延长模式，因此临时作用域被延长。
let ref x = *&temp(); // 正常
# x;
```

```rust,compile_fail,E0716
# fn temp() {}
// 这既不是延长模式也不是延长表达式，
// 因此临时变量在分号处销毁。
let &ref x = *&&temp(); // 错误
# x;
```

```rust
# fn temp() {}
// 这不是延长模式，但它是一个延长表达式，
// 因此临时变量存活超过了 `let` 语句。
let &ref x = &*&temp(); // 正常
# x;
```

r[destructors.scope.lifetime-extension.exprs]
#### 基于表达式的延长

r[destructors.scope.lifetime-extension.exprs.extending]
对于带有初始化器的 let 语句， *延长表达式* 是以下表达式之一：

* 初始化器表达式。
* 延长 [借用][borrow] 表达式的操作数。
* 延长 [超级宏调用][super macro call] 表达式的 [超级操作数][super operands] 。
* 延长 [数组表达式][array expression] 、 [转换表达式][cast expression] 、 [大括号结构体表达式][struct expression] 或 [元组表达式][tuple expression] 的操作数。
* 延长 [元组结构体][tuple struct] 或 [元组枚举变体][tuple enum variant] 构造器表达式的参数。
* 延长 [块表达式][block expression] 的最终表达式（异步块表达式 [async block expression] 除外）。
* 延长 [`if`] 表达式的 `consequent` 、 `else if` 或 `else` 块的最终表达式。
* 延长 [`match`] 表达式的分支表达式。

> [!NOTE]
> [解构赋值][destructuring assignment] 的脱糖使其赋值操作数（右侧值）成为新引入块内的延长表达式。有关详细信息，请参阅 [expr.assign.destructure.tmp-ext] 。

因此 `&mut 0` 、 `(&1, &mut 2)` 和 `Some(&mut 3)` 中的借用表达式都是延长表达式。而 `&0 + &1` 和 `f(&mut 0)` 中的借用则不是。

r[destructors.scope.lifetime-extension.exprs.borrows]
延长 [借用][borrow] 表达式的操作数其 [临时作用域][temporary scope] 被 [延长][extended] 。

r[destructors.scope.lifetime-extension.exprs.super-macros]
延长 [超级宏调用][super macro call] 表达式的 [超级临时变量][super temporaries] 其 [作用域][temporary scopes] 被 [延长][extended] 。

> [!NOTE]
> `rustc` 不将延长 [数组][array] 表达式的 [数组重复操作数][array repeat operands] 视为延长表达式。是否应该这样做是一个悬而未决的问题。
>
> 有关详细信息，请参阅 [Rust issue #146092](https://github.com/rust-lang/rust/issues/146092) 。

#### 示例

以下是表达式具有延长临时作用域的一些示例：

```rust,edition2024
# use core::pin::pin;
# use core::sync::atomic::{AtomicU64, Ordering::Relaxed};
# static X: AtomicU64 = AtomicU64::new(0);
# #[derive(Debug)] struct S;
# impl Drop for S { fn drop(&mut self) { X.fetch_add(1, Relaxed); } }
# const fn temp() -> S { S }
let x = &temp(); // 借用的操作数。
# x;
let x = &raw const *&temp(); // 原生借用的操作数。
# assert_eq!(X.load(Relaxed), 0);
let x = &temp() as &dyn Send; // 转换的操作数。
# x;
let x = (&*&temp(),); // 元组构造器的操作数。
# x;
struct W<T>(T);
let x = W(&temp()); // 元组结构体构造器的参数。
# x;
let x = Some(&temp()); // 元组枚举变体构造器的参数。
# x;
let x = { [Some(&temp())] }; // 块的最终表达式。
# x;
let x = const { &temp() }; // `const` 块的最终表达式。
# x;
let x = unsafe { &temp() }; // `unsafe` 块的最终表达式。
# x;
let x = if true { &temp() } else { &temp() };
//              ^^^^^^^^^^^^^^^^^^^^^^^^^^^^
//           `if`/`else` 块的最终表达式。
# x;
let x = match () { _ => &temp() }; // `match` 分支表达式。
# x;
let x = pin!(temp()); // 超级宏调用表达式的超级操作数。
# x;
let x = pin!({ &mut temp() }); // 同上。
# x;
let x = format_args!("{:?}", temp()); // 同上。
# x;
//
// 以上所有临时变量在这里仍然存活。
# assert_eq!(0, X.load(Relaxed));
```

以下是表达式不具有延长临时作用域的一些示例：

```rust,compile_fail,E0716
# fn temp() {}
// 函数调用的参数不是延长表达式。临时变量在分号处销毁。
let x = core::convert::identity(&temp()); // 错误
# x;
```

```rust,compile_fail,E0716
# fn temp() {}
# trait Use { fn use_temp(&self) -> &Self { self } }
# impl Use for () {}
// 方法调用的接收者不是延长表达式。
let x = (&temp()).use_temp(); // 错误
# x;
```

```rust,compile_fail,E0716
# fn temp() {}
// match 表达式的受查表达式不是延长表达式。
let x = match &temp() { x => x }; // 错误
# x;
```

```rust,compile_fail,E0515
# fn temp() {}
// `async` 块的最终表达式不是延长表达式。
let x = async { &temp() }; // 错误
# x;
```

```rust,compile_fail,E0515
# fn temp() {}
// 闭包的最终表达式不是延长表达式。
let x = || &temp(); // 错误
# x;
```

```rust,compile_fail,E0716
# fn temp() {}
// 循环 break 的操作数不是延长表达式。
let x = loop { break &temp() }; // 错误
# x;
```

```rust,compile_fail,E0716
# fn temp() {}
// 带有标签的 break 的操作数不是延长表达式。
let x = 'a: { break 'a &temp() }; // 错误
# x;
```

```rust,edition2024,compile_fail,E0716
# use core::pin::pin;
# fn temp() {}
// 仅当调用是延长表达式时，`pin!` 的参数才是延长表达式。
// 由于它不是，内部块不是延长表达式，因此其尾随表达式中的
// 临时变量会立即销毁。
pin!({ &temp() }); // 错误
```

```rust,edition2024,compile_fail,E0716
# fn temp() {}
// 同上。
format_args!("{:?}", { &temp() }); // 错误
```

r[destructors.forget]
## 不运行析构函数

r[destructors.manually-suppressing]
### 手动抑制析构函数

[`core::mem::forget`] 可用于防止运行变量的析构函数，而 [`core::mem::ManuallyDrop`] 提供了一个包装器来防止变量或字段被自动销毁。

> [!NOTE]
> 通过 [`core::mem::forget`] 或其他手段防止析构函数运行是安全的，即使它的类型不是 `'static` 。除了本文档定义的保证运行析构函数的地方外，为了健全性，类型 *不可* 安全地依赖于析构函数的运行。

r[destructors.process-termination]
### 无需展开的进程终止

有一些方法可以在不进行 [展开][unwinding] 的情况下终止进程，在这种情况下将不运行析构函数。

标准库提供了 [`std::process::exit`] 和 [`std::process::abort`] 来显式执行此操作。此外，如果 [恐慌处理器][panic.panic_handler.std] 设置为 `abort` ，发生恐慌时将始终终止进程而不运行析构函数。

还有一个需要注意的情况：当恐慌达到 [非展开 ABI 边界][non-unwinding ABI boundary] 时，要么不运行任何析构函数，要么运行直到该 ABI 边界为止的所有析构函数。

[Assignment]: expressions/operator-expr.md#assignment-expressions
[binding modes]: patterns.md#binding-modes
[closure]: types/closure.md
[destructors]: destructors.md
[destructuring assignment]: expr.assign.destructure
[expression]: expressions.md
[identifier pattern]: patterns.md#identifier-patterns
[initialized]: glossary.md#initialized
[interior mutability]: interior-mutability.md
[lazy boolean expression]: expressions/operator-expr.md#lazy-boolean-operators
[non-unwinding ABI boundary]: items/functions.md#unwinding
[panic]: panic.md
[place context]: expressions.md#place-expressions-and-value-expressions
[promoted]: destructors.md#constant-promotion
[scrutinee]: glossary.md#scrutinee
[statement]: statements.md
[temporary]: expressions.md#temporaries
[unwinding]: panic.md#unwinding
[variable]: variables.md

[array]: types/array.md
[enum variant]: types/enum.md
[slice]: types/slice.md
[struct]: types/struct.md
[Trait objects]: types/trait-object.md
[tuple]: types/tuple.md

[or-patterns]: patterns.md#or-patterns
[slice pattern]: patterns.md#slice-patterns
[struct pattern]: patterns.md#struct-patterns
[tuple pattern]: patterns.md#tuple-patterns
[tuple struct pattern]: patterns.md#tuple-struct-patterns
[tuple struct]: type.struct.tuple
[tuple enum variant]: type.enum.declaration

[array expression]: expressions/array-expr.md#array-expressions
[array repeat operands]: expr.array.repeat-operand
[async block expression]: expr.block.async
[block expression]: expressions/block-expr.md
[borrow]: expr.operator.borrow
[cast expression]: expressions/operator-expr.md#type-cast-expressions
[dereference expression]: expressions/operator-expr.md#the-dereference-operator
[extended]: destructors.scope.lifetime-extension
[field expression]: expressions/field-expr.md
[indexing expression]: expressions/array-expr.md#array-and-slice-indexing-expressions
[struct expression]: expressions/struct-expr.md
[super macro call]: expr.super-macros
[super operands]: expr.super-macros
[super temporaries]: expr.super-macros
[temporary scope]: destructors.scope.temporary
[temporary scopes]: destructors.scope.temporary
[tuple expression]: expressions/tuple-expr.md#tuple-expressions
[tuple indexing expression]: expressions/tuple-expr.md#tuple-indexing-expressions

[`for`]: expressions/loop-expr.md#iterator-loops
[`if let`]: expressions/if-expr.md#if-let-patterns
[`if`]: expressions/if-expr.md#if-expressions
[`let` statement]: statements.md#let-statements
[`loop`]: expressions/loop-expr.md#infinite-loops
[`match`]: expressions/match-expr.md
[`while let`]: expressions/loop-expr.md#while-let-patterns
[`while`]: expressions/loop-expr.md#predicate-loops
