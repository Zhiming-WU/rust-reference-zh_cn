<!-- https://github.com/rust-lang/reference/blob/master/src/expressions.md -->
<!-- commit 68bdfd15fa9feebfbf94a06cf574de598e239198 -->

r[expr]
# 表达式

r[expr.syntax]
```grammar,表达式
Expression ->
      ExpressionWithoutBlock
    | ExpressionWithBlock

ExpressionWithoutBlock ->
    OuterAttribute*
    (
        LiteralExpression
      | PathExpression
      | OperatorExpression
      | GroupedExpression
      | ArrayExpression
      | AwaitExpression
      | IndexExpression
      | TupleExpression
      | TupleIndexingExpression
      | StructExpression
      | CallExpression
      | MethodCallExpression
      | FieldExpression
      | ClosureExpression
      | AsyncBlockExpression
      | ContinueExpression
      | BreakExpression
      | RangeExpression
      | ReturnExpression
      | UnderscoreExpression
      | MacroInvocation
    )

ExpressionWithBlock ->
    OuterAttribute*
    (
        BlockExpression
      | ConstBlockExpression
      | UnsafeBlockExpression
      | LoopExpression
      | IfExpression
      | MatchExpression
    )
```

r[expr.intro]
一个表达式可能有两种角色：它总是产生一个 *值*，并且可能产生 *副作用* 。

r[expr.evaluation]
表达式 *求值为* 一个值，并在 *求值* 过程中产生副作用。

r[expr.operands]
许多表达式包含子表达式，称为该表达式的 *操作数*。

r[expr.behavior]
每种表达式的含义决定了几件事：

* 在对表达式求值时是否对操作数求值
* 对操作数求值的顺序
* 如何组合操作数的值以获得表达式的值

r[expr.structure]
通过这种方式，表达式的结构决定了执行的结构。块只是另一种表达式，因此块、语句、表达式以及再次嵌套的块可以递归地相互嵌套到任意深度。

> [!NOTE]
> 我们给表达式的操作数命名以便于讨论，但这些名称并不稳定，可能会被更改。

r[expr.precedence]
## 表达式优先级

Rust 运算符和表达式的优先级按以下顺序排列，从强到弱。同一优先级水平的二元运算符按其结合性给出的顺序进行分组。

| 运算符/表达式         | 结合性       |
|-----------------------------|---------------------|
| [路径][expr.path]          |                     |
| [方法调用][expr.method] |                     |
| [字段表达式][expr.field] | 从左到右   |
| [函数调用][expr.call], [数组索引][expr.array.index] | |
| [`?`][expr.try]             |                     |
| 一元 [`-`][expr.negate] [`!`][expr.negate] [`*`][expr.deref] [借用][expr.operator.borrow] | |
| [`as`][expr.as]             | 从左到右       |
| [`*`][expr.arith-logic] [`/`][expr.arith-logic] [`%`][expr.arith-logic] | 从左到右       |
| [`+`][expr.arith-logic] [`-`][expr.arith-logic] | 从左到右       |
| [`<<`][expr.arith-logic] [`>>`][expr.arith-logic] | 从左到右     |
| [`&`][expr.arith-logic]     | 从左到右       |
| [`^`][expr.arith-logic]     | 从左到右       |
| [<code>&#124;</code>][expr.arith-logic] | 从左到右       |
| [`==`][expr.cmp] [`!=`][expr.cmp] [`<`][expr.cmp] [`>`][expr.cmp] [`<=`][expr.cmp] [`>=`][expr.cmp] | 需要括号 |
| [`&&`][expr.bool-logic]     | 从左到右       |
| [<code>&#124;&#124;</code>][expr.bool-logic] | 从左到右       |
| [`..`][expr.range] [`..=`][expr.range] | 需要括号 |
| [`=`][expr.assign] [`+=`][expr.compound-assign] [`-=`][expr.compound-assign] [`*=`][expr.compound-assign] [`/=`][expr.compound-assign] [`%=`][expr.compound-assign] <br> [`&=`][expr.compound-assign] [<code>&#124;=</code>][expr.compound-assign] [`^=`][expr.compound-assign] [`<<=`][expr.compound-assign] [`>>=`][expr.compound-assign] | 从右到左 |
| [`return`][expr.return] [`break`][expr.loop.break] [闭包][expr.closure]  | |

r[expr.operand-order]
## 操作数的求值顺序

r[expr.operand-order.default]
以下表达式列表都以相同的方式对其操作数求值，如列表后所述。其他表达式要么不接受操作数，要么根据其各自页面的描述有条件地对其求值。

* 解引用表达式
* 错误传播表达式
* 取反表达式
* 算术和逻辑二元运算符
* 比较运算符
* 类型转换表达式
* 分组表达式
* 数组表达式
* Await 表达式
* 索引表达式
* 元组表达式
* 元组索引表达式
* 结构体表达式
* 调用表达式
* 方法调用表达式
* 字段表达式
* Break 表达式
* 范围表达式
* 返回表达式

r[expr.operand-order.operands-before-primary]
这些表达式的操作数在应用表达式的效果之前进行求值。接受多个操作数的表达式按照源代码中编写的顺序从左到右进行求值。

> [!NOTE]
> 哪些子表达式是表达式的操作数，由前一节所述的表达式优先级决定。

例如，两个 `next` 方法调用将始终按相同的顺序调用：

```rust
# // 使用 vec 而不是数组来避免引用，
# // 因为在编写此示例时
# // 还没有稳定的所有权数组迭代器。
let mut one_two = vec![1, 2].into_iter();
assert_eq!(
    (1, 2),
    (one_two.next().unwrap(), one_two.next().unwrap())
);
```

> [!NOTE]
> 由于这是递归应用的，因此这些表达式也从最内层向最外层求值，忽略同级表达式，直到没有内部子表达式。

r[expr.place-value]
## 位置表达式和值表达式

r[expr.place-value.intro]
表达式分为两个主要类别：位置表达式和值表达式；还有第三个较小的表达式类别，称为被赋值者表达式。在每个表达式中，操作数同样可能出现在位置环境或值环境中。表达式的求值取决于其自身的类别以及它所处的环境。

r[expr.place-value.place-memory-location]
*位置表达式* 是代表内存位置的表达式。

r[expr.place-value.place-expr-kinds]
这些表达式是引用局部变量的 [路径][paths]、[静态变量][static variables]、[解引用][deref] (`*expr`)、[数组索引][array indexing] 表达式 (`expr[expr]`)、[字段][field] 引用 (`expr.f`) 以及括号括起来的位置表达式。

r[expr.place-value.value-expr-kinds]
所有其他表达式都是值表达式。

r[expr.place-value.value-result]
*值表达式* 是代表实际值的表达式。

r[expr.place-value.place-context]
以下环境是 *位置表达式* 环境：

* [复合赋值][compound assignment] 表达式的左操作数。
* 一元 [借用][borrow]、[原始借用][raw borrow] 或 [解引用][deref] 运算符的操作数。
* 字段表达式的操作数。
* 数组索引表达式的被索引操作数。
* 任何 [隐式借用][implicit borrow] 的操作数。
* [let 语句][let statement] 的初始化器。
* [`if let`]、[`match`][match] 或 [`while let`] 表达式的 [审查对象][scrutinee]。
* [函数式更新][functional update] 结构体 表达式的基准。

> [!NOTE]
> 在历史上，位置表达式被称为 *左值 (lvalues)*，而值表达式被称为 *右值 (rvalues)*。

r[expr.place-value.assignee]
*被赋值者表达式* 是出现在 [赋值][assign] 表达式左操作数中的表达式。显式地，被赋值者表达式有：

- 位置表达式。
- [下划线][Underscores]。
- 被赋值者表达式的 [元组][Tuples]。
- 被赋值者表达式的 [切片][expr.array.index]。
- 被赋值者表达式的 [元组结构体][Tuple structs]。
- 被赋值者表达式的 [结构体][Structs]（带有可选命名字段）。
- [单元结构体][Unit structs]

r[expr.place-value.parenthesis]
在被赋值者表达式内部允许使用任意括号。

r[expr.move]
### 移动和复制类型

r[expr.move.intro]
当位置表达式在值表达式环境中求值，或在模式中通过值绑定时，它表示该内存位置中 *持有* 的值。

r[expr.move.copy]
如果该值的类型实现了 [`Copy`]，那么该值将被复制。

r[expr.move.requires-sized]
在其余情况下，如果该类型是 [`Sized`]，则可能会移动该值。

r[expr.move.movable-place]
只有以下位置表达式可以被移出：

* 当前未被借用的 [变量][Variables]。
* [临时值][Temporary values]。
* 可以移出且未实现 [`Drop`] 的位置表达式的 [字段][field]。
* [解引用][deref] 类型为 [`Box<T>`] 且也可以被移出的表达式的结果。

r[expr.move.deinitialization]
在移出一个求值为局部变量的位置表达式后，该位置将被去初始化，并且在重新初始化之前不能再次从中读取。

r[expr.move.place-invalid]
在所有其他情况下，尝试在值表达式环境中使用位置表达式都会报错。

r[expr.mut]
### 可变性

r[expr.mut.intro]
为了使位置表达式能被 [赋值][assign]、可变 [借用][borrow]、[隐式可变借用][implicitly mutably borrowed] 或绑定到包含 `ref mut` 的模式，它必须是 *可变的*。我们称这些为 *可变位置表达式*。相比之下，其他位置表达式称为 *不可变位置表达式*。

r[expr.mut.valid-places]
以下表达式可以是可变位置表达式环境：

* 当前未被借用的可变 [变量][Variables]。
* [可变 `static` 项][Mutable `static` items]。
* [临时值][Temporary values]。
* [字段][field]：这在可变位置表达式环境中对子表达式求值。
* `*mut T` 指针的 [解引用][deref]。
* 类型为 `&mut T` 的变量或变量字段的解引用。
  注意：这是下一条规则要求的例外。
* 实现了 `DerefMut` 的类型的解引用：
  这随后要求被解引用的值在可变位置表达式环境中求值。
* 实现了 `IndexMut` 的类型的 [数组索引][array indexing]：
  这随后在可变位置表达式环境中对被索引的值（而非索引本身）求值。

r[expr.temporary]
### 临时变量

在大多数位置表达式环境中使用值表达式时，会创建一个临时的未命名内存位置并将其初始化为该值。该表达式求值为该位置，除非被 [提升][promoted] 为 `static`。临时变量的 [销毁范围][drop scope] 通常是封闭语句的末尾。

r[expr.super-macros]
### 超级宏

r[expr.super-macros.intro]
某些内置宏可能会创建 [临时变量][temporaries]，其 [作用域][temporary scopes] 可能会被 [延长][extended]。这些临时变量是 *超级临时变量*，而这些宏是 *超级宏*。这些宏的 [调用][macro invocations] 是 *超级宏调用表达式*。这些宏的参数可能是 *超级操作数*。

> [!NOTE]
> 当超级宏调用表达式是一个 [延长表达式][extending expression] 时，它的超级操作数也是 [延长表达式][extending expressions]，并且超级临时变量的 [作用域][temporary scopes] 会被 [延长][extended]。参见 [destructors.scope.lifetime-extension.exprs]。

r[expr.super-macros.format_args]
#### `format_args!`

r[expr.super-macros.format_args.super-operands]
除了格式化字符串参数外，传递给 [`format_args!`] 的所有参数都是 *超级操作数*。

```rust,edition2024
# fn temp() -> String { String::from("") }
// 由于调用是一个延长表达式且参数是一个超级操作数，
// 内部块是一个延长表达式，
// 因此在其尾部表达式中创建的临时变量的作用域被延长了。
let _ = format_args!("{}", { &temp() }); // OK
```

r[expr.super-macros.format_args.super-temporaries]
[`format_args!`] 的超级操作数是 [隐式借用][implicitly borrowed] 的，因此是 [位置表达式环境][place expression contexts]。当 [值表达式][value expression] 被作为参数传递时，它会创建一个 *超级临时变量*。

```rust
# fn temp() -> String { String::from("") }
let x = format_args!("{}", temp());
x; // <-- 临时变量被延长，允许在此处使用。
```

[`format_args!`] 调用的展开有时会创建其他内部 *超级临时变量*。

```rust,compile_fail,E0716
let x = {
    // 此调用创建一个内部临时变量。
    let x = format_args!("{:?}", 0);
    x // <-- 临时变量被延长，允许在此处使用。
}; // <-- 临时变量在此处被销毁。
x; // 错误
```

```rust
// 此调用不创建内部临时变量。
let x = { let x = format_args!("{}", 0); x };
x; // OK
```

> [!NOTE]
> 关于 [`format_args!`] 何时会或不会创建内部临时变量的细节目前尚未指定。

r[expr.super-macros.pin]
#### `pin!`

r[expr.super-macros.pin.super-operands]
[`pin!`] 的参数是 *超级操作数*。

```rust,edition2024
# use core::pin::pin;
# fn temp() {}
// 同上 `format_args!`。
let _ = pin!({ &temp() }); // OK
```

r[expr.super-macros.pin.super-temporaries]
[`pin!`] 的参数是 [值表达式环境][value expression context]，并创建一个 *超级临时变量*。

```rust
# use core::pin::pin;
# fn temp() {}
// 参数被求值为超级临时变量。
let x = pin!(temp());
// 临时变量被延长，允许在此处使用。
x; // OK
```

r[expr.implicit-borrow]
### 隐式借用

r[expr.implicit-borrow-intro]
某些表达式会通过隐式借用将一个表达式视为位置表达式。例如，可以直接比较两个无固定大小的 [切片][slice] 是否相等，因为 `==` 运算符会隐式借用其操作数：

```rust
# let c = [1, 2, 3];
# let d = vec![1, 2, 3];
let a: &[i32];
let b: &[i32];
# a = &c;
# b = &d;
// ...
*a == *b;
// 等效形式：
::std::cmp::PartialEq::eq(&*a, &*b);
```

r[expr.implicit-borrow.application]
隐式借用可能会在以下表达式中发生：

* [方法调用][method-call] 表达式中的左操作数。
* [字段][field] 表达式中的左操作数。
* [调用表达式][call expressions] 中的左操作数。
* [数组索引][array indexing] 表达式中的左操作数。
* [解引用运算符][deref] (`*`) 的操作数。
* [比较][comparison] 的操作数。
* [复合赋值][compound assignment] 的左操作数。
* 除了格式字符串之外传递给 [`format_args!`] 的参数。

r[expr.overload]
## 重载特型

许多以下运算符和表达式也可以使用 `std::ops` 或 `std::cmp` 中的 特型 对其他类型进行重载。这些 特型 在 `core::ops` 和 `core::cmp` 中也以相同的名称存在。

r[expr.attr]
## 表达式属性

r[expr.attr.restriction]
表达式前的 [外部属性][Outer attributes] 仅在少数特定情况下允许：

* 在用作 [语句][statement] 的表达式之前。
* [数组表达式][array expressions]、[元组表达式][tuple expressions]、[调用表达式][call expressions] 以及元组风格 [结构体][struct] 表达式的元素。
* [块表达式][block expressions] 的尾部表达式。
<!-- Keep list in sync with block-expr.md -->

r[expr.attr.never-before]
在以下情况前绝不允许：
* [范围][Range] 表达式。
* 二元运算符表达式（[ArithmeticOrLogicalExpression], [ComparisonExpression], [LazyBooleanExpression], [TypeCastExpression], [AssignmentExpression], [CompoundAssignmentExpression]）。

[`Copy`]:               special-types-and-traits.md#copy
[`Drop`]:               special-types-and-traits.md#drop
[`if let`]:             expressions/if-expr.md#if-let模式
[`format_args!`]:       core::format_args
[`pin!`]:               core::pin::pin
[`Sized`]:              special-types-and-traits.md#sized
[`while let`]:          expressions/loop-expr.md#while-let模式
[array expressions]:    expressions/array-expr.md
[array indexing]:       expressions/array-expr.md#数组和切片索引表达式
[assign]:               expressions/operator-expr.md#赋值表达式
[block expressions]:    expressions/block-expr.md
[borrow]:               expressions/operator-expr.md#借用运算符
[call expressions]:     expressions/call-expr.md
[comparison]:           expressions/operator-expr.md#比较运算符
[compound assignment]:  expressions/operator-expr.md#复合赋值表达式
[deref]:                expressions/operator-expr.md#解引用运算符
[destructors]:          destructors.md
[drop scope]:           destructors.md#销毁作用域
[extended]:             destructors.scope.lifetime-extension
[extending expression]: destructors.scope.lifetime-extension.exprs
[extending expressions]: destructors.scope.lifetime-extension.exprs
[field]:                expressions/field-expr.md
[functional update]:    expressions/struct-expr.md#函数式更新语法
[implicit borrow]:      #implicit-borrows
[implicitly borrowed]:  expr.implicit-borrow
[implicitly mutably borrowed]: #implicit-borrows
[interior mutability]:  interior-mutability.md
[let statement]:        statements.md#let语句
[macro invocations]:    macro.invocation
[match]:                expressions/match-expr.md
[method-call]:          expressions/method-call-expr.md
[Mutable `static` items]: items/static-items.md#可变静态项
[Outer attributes]:     attributes.md
[paths]:                expressions/path-expr.md
[place expression contexts]: expr.place-value
[promoted]:             destructors.md#常量提升
[Range]:                expressions/range-expr.md
[raw borrow]:           expressions/operator-expr.md#原始借用运算符
[scrutinee]:            glossary.md#审查值
[slice]:                types/slice.md
[statement]:            statements.md
[static variables]:     items/static-items.md
[struct]:               expressions/struct-expr.md
[Structs]:              expr.struct
[temporaries]:          expr.temporary
[temporary scopes]:     destructors.scope.temporary
[Temporary values]:     #temporaries
[tuple expressions]:    expressions/tuple-expr.md
[Tuple structs]:        items.struct.tuple
[Tuples]:               expressions/tuple-expr.md
[Underscores]:          expressions/underscore-expr.md
[Unit structs]:         items.struct.unit
[value expression context]: expr.place-value
[value expression]:     expr.place-value
[Variables]:            variables.md
