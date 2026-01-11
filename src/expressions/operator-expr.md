r[expr.operator]
# 运算符表达式

r[expr.operator.syntax]
```grammar,表达式
OperatorExpression ->
      BorrowExpression
    | DereferenceExpression
    | TryPropagationExpression
    | NegationExpression
    | ArithmeticOrLogicalExpression
    | ComparisonExpression
    | LazyBooleanExpression
    | TypeCastExpression
    | AssignmentExpression
    | CompoundAssignmentExpression
```

r[expr.operator.intro]
Rust 语言为内置类型定义了运算符。

r[expr.operator.trait]
下面许多运算符也可以使用 `std::ops` 或 `std::cmp` 中的 特型 来重载。

r[expr.operator.int-overflow]
## 溢出

r[expr.operator.int-overflow.intro]
在调试模式下编译时，整数运算符在溢出时会产生 恐慌 。
可以使用 `-C debug-assertions` 和 `-C overflow-checks` 编译器标志来更直接地控制此行为。
以下情况被视为溢出：

r[expr.operator.int-overflow.binary-arith]
* 当 `+` 、 `*` 或二元 `-` 产生的值大于最大值，或小于可存储的最小值时。

r[expr.operator.int-overflow.unary-neg]
* 对任何有符号整数类型的最小负值应用一元 `-` ，除非操作数是 [字面量表达式][literal expression] （或单独存在于一个或多个 [分组表达式][grouped expression] 中的字面量表达式）。

r[expr.operator.int-overflow.div]
* 使用 `/` 或 `%` ，其中左侧参数是有符号整数类型的最小整数，而右侧参数是 `-1` 。
  由于历史原因，即使禁用了 `-C overflow-checks` ，也会进行这些检查。

r[expr.operator.int-overflow.shift]
* 使用 `<<` 或 `>>` ，其中右侧参数大于或等于左侧参数类型中的位数，或者是负数。

> [!NOTE]
> 一元 `-` 后的字面量表达式的例外情况意味着诸如 `-128_i8` 或 `let j: i8 = -(128)` 之类的形式永远不会导致 恐慌 ，并且具有预期的值 -128。
>
> 在这些情况下，字面量表达式已经具有其类型的最小负值（例如， `128_i8` 的值为 -128），因为整数字面量会根据 [整数字面量表达式][literal expression] 中的描述被截断为其类型。
>
> 由于补码溢出约定，这些最小负值的取负操作保持原值不变。
>
> 在 `rustc` 中，这些最小负数表达式也会被 `overflowing_literals` lint 检查忽略。

r[expr.operator.borrow]
## 借用运算符

r[expr.operator.borrow.syntax]
```grammar,表达式
BorrowExpression ->
      (`&`|`&&`) Expression
    | (`&`|`&&`) `mut` Expression
    | (`&`|`&&`) `raw` `const` Expression
    | (`&`|`&&`) `raw` `mut` Expression
```

r[expr.operator.borrow.intro]
 `&` （共享借用）和 `&mut` （可变借用）运算符是一元前缀运算符。

r[expr.operator.borrow.result]
当应用于 [位置表达式][place expression] 时，此表达式会产生一个指向该值所引用的位置的引用（指针）。

r[expr.operator.borrow.lifetime]
在引用的持续时间内，内存位置也处于借用状态。
对于共享借用（ `&` ），这意味着该位置不可被修改，但可以被读取或再次共享。
对于可变借用（ `&mut` ），在借用过期之前，不能以任何方式访问该位置。

r[expr.operator.borrow.mut]
 `&mut` 在可变位置表达式语境中对其操作数求值。

r[expr.operator.borrow.temporary]
如果 `&` 或 `&mut` 运算符应用于 [值表达式][value expression] ，则会创建一个 [临时值][temporary value] 。

这些运算符不能被重载。

```rust
{
    // 创建一个值为 7 的临时变量，持续作用域为当前块。
    let shared_reference = &7;
}
let mut array = [-2, 3, 9];
{
    // 在此作用域内可变地借用 `array`。
    // `array` 只能通过 `mutable_reference` 使用。
    let mutable_reference = &mut array;
}
```

r[expr.borrow.and-and-syntax]
尽管 `&&` 是单个 词法单元 （ [惰性 '与' 运算符](#lazy-boolean-operators) ），但在借用表达式的语境中使用时，它的作用相当于两次借用：

```rust
// 含义相同：
let a = &&  10;
let a = & & 10;

// 含义相同：
let a = &&&&  mut 10;
let a = && && mut 10;
let a = & & & & mut 10;
```

r[expr.borrow.raw]
### 原始借用运算符

r[expr.borrow.raw.intro]
 `&raw const` 和 `&raw mut` 是 _原始借用运算符_ 。

r[expr.borrow.raw.place]
这些运算符的操作数表达式在位置表达式语境中求值。

r[expr.borrow.raw.result]
 `&raw const expr` 随后会创建一个指向给定位置的 `*const T` 类型的常量原始指针，而 `&raw mut expr` 会创建一个 `*mut T` 类型的可变原始指针。

r[expr.borrow.raw.invalid-ref]
每当位置表达式可能求得的位置未正确对齐，或者根据其类型未存储有效值，或者创建引用会引入不正确的别名假设时，必须使用原始借用运算符而不是借用运算符。
在这些情况下，使用借用运算符会因创建无效引用而导致 [未定义行为][undefined behavior] ，但仍可以构造原始指针。

以下是通过 `packed` 结构体 创建指向未对齐位置的原始指针的示例：

```rust
#[repr(packed)]
struct Packed {
    f1: u8,
    f2: u16,
}

let packed = Packed { f1: 1, f2: 2 };
// `&packed.f2` 会创建一个未对齐的引用，因此会导致未定义行为！
let raw_f2 = &raw const packed.f2;
assert_eq!(unsafe { raw_f2.read_unaligned() }, 2);
```

以下是创建指向不包含有效值的位置的原始指针的示例：

```rust
use std::mem::MaybeUninit;

struct Demo {
    field: bool,
}

let mut uninit = MaybeUninit::<Demo>::uninit();
// `&uninit.as_mut().field` 会创建一个对未初始化 `bool` 的引用，
// 因此会导致未定义行为！
let f1_ptr = unsafe { &raw mut (*uninit.as_mut_ptr()).field };
unsafe { f1_ptr.write(true); }
let init = unsafe { uninit.assume_init() };
```

r[expr.deref]
## 解引用运算符

r[expr.deref.syntax]
```grammar,表达式
DereferenceExpression -> `*` Expression
```

r[expr.deref.intro]
 `*` （解引用）运算符也是一元前缀运算符。

r[expr.deref.result]
当应用于 [指针](../types/pointer.md) 时，它表示指向的位置。

r[expr.deref.mut]
如果表达式的类型是 `&mut T` 或 `*mut T` ，并且是局部变量、局部变量的（嵌套）字段或可变 [位置表达式][place expression] ，则可以对产生的内存位置进行赋值。

r[expr.deref.safety]
对原始指针进行解引用需要 `unsafe` 。

r[expr.deref.traits]
在 [不可变位置表达式语境](../expressions.md#mutability) 中，对于非指针类型， `*x` 等价于 `*std::ops::Deref::deref(&x)` ；在可变位置表达式语境中，等价于 `*std::ops::DerefMut::deref_mut(&mut x)` 。

```rust
let x = &7;
assert_eq!(*x, 7);
let y = &mut 9;
*y = 11;
assert_eq!(*y, 11);
```

r[expr.try]
## try传播表达式

r[expr.try.syntax]
```grammar,表达式
TryPropagationExpression -> Expression `?`
```

r[expr.try.intro]
try 传播表达式使用内部表达式的值和 [`Try`] 特型 来决定是产生一个值（如果产生，产生什么值），还是向调用者返回一个值（如果返回，返回什么值）。

> [!EXAMPLE]
> ```rust
> # use std::num::ParseIntError;
> fn try_to_parse() -> Result<i32, ParseIntError> {
>     let x: i32 = "123".parse()?; // `x` 是 `123`。
>     let y: i32 = "24a".parse()?; // 立即返回 `Err()`。
>     Ok(x + y)                    // 不运行。
> }
>
> let res = try_to_parse();
> println!("{res:?}");
> # assert!(res.is_err())
> ```
>
> ```rust
> fn try_option_some() -> Option<u8> {
>     let val = Some(1)?;
>     Some(val)
> }
> assert_eq!(try_option_some(), Some(1));
>
> fn try_option_none() -> Option<u8> {
>     let val = None?;
>     Some(val)
> }
> assert_eq!(try_option_none(), None);
> ```
>
> ```rust
> use std::ops::ControlFlow;
>
> pub struct TreeNode<T> {
>     value: T,
>     left: Option<Box<TreeNode<T>>>,
>     right: Option<Box<TreeNode<T>>>,
> }
>
> impl<T> TreeNode<T> {
>     pub fn traverse_inorder<B>(&self, f: &mut impl FnMut(&T) -> ControlFlow<B>) -> ControlFlow<B> {
>         if let Some(left) = &self.left {
>             left.traverse_inorder(f)?;
>         }
>         f(&self.value)?;
>         if let Some(right) = &self.right {
>             right.traverse_inorder(f)?;
>         }
>         ControlFlow::Continue(())
>     }
> }
> #
> # fn main() {
> #     let n = TreeNode {
> #         value: 1,
> #         left: Some(Box::new(TreeNode{value: 2, left: None, right: None})),
> #         right: None,
> #     };
> #     let v = n.traverse_inorder(&mut |t| {
> #         if *t == 2 {
> #             ControlFlow::Break("found")
> #         } else {
> #             ControlFlow::Continue(())
> #         }
> #     });
> #     assert_eq!(v, ControlFlow::Break("found"));
> # }
> ```

> [!NOTE]
>  [`Try`] 特型 目前不稳定，因此不能为用户类型实现。
>
> try 传播表达式目前大致等价于：
>
> ```rust
> # #![ feature(try_trait_v2) ]
> # fn example() -> Result<(), ()> {
> # let expr = Ok(());
> match core::ops::Try::branch(expr) {
>     core::ops::ControlFlow::Continue(val) => val,
>     core::ops::ControlFlow::Break(residual) =>
>         return core::ops::FromResidual::from_residual(residual),
> }
> # Ok(())
> # }
> ```

> [!NOTE]
> try 传播运算符有时被称为 _问号运算符_ 、 _`?` 运算符_ 或 _try 运算符_ 。

r[expr.try.restricted-types]
try 传播运算符可以应用于具有以下类型的表达式：

- [`Result<T, E>`]
    - `Result::Ok(val)` 求值为 `val` 。
    - `Result::Err(e)` 返回 `Result::Err(From::from(e))` 。
- [`Option<T>`]
    - `Option::Some(val)` 求值为 `val` 。
    - `Option::None` 返回 `Option::None` 。
- [`ControlFlow<B, C>`][core::ops::ControlFlow]
    - `ControlFlow::Continue(c)` 求值为 `c` 。
    - `ControlFlow::Break(b)` 返回 `ControlFlow::Break(b)` 。
- [`Poll<Result<T, E>>`][core::task::Poll]
    - `Poll::Ready(Ok(val))` 求值为 `Poll::Ready(val)` 。
    - `Poll::Ready(Err(e))` 返回 `Poll::Ready(Err(From::from(e)))` 。
    - `Poll::Pending` 求值为 `Poll::Pending` 。
- [`Poll<Option<Result<T, E>>>`][`core::task::Poll`]
    - `Poll::Ready(Some(Ok(val)))` 求值为 `Poll::Ready(Some(val))` 。
    - `Poll::Ready(Some(Err(e)))` 返回 `Poll::Ready(Some(Err(From::from(e))))` 。
    - `Poll::Ready(None)` 求值为 `Poll::Ready(None)` 。
    - `Poll::Pending` 求值为 `Poll::Pending` 。

r[expr.negate]
## 求负运算符

r[expr.negate.syntax]
```grammar,表达式
NegationExpression ->
      `-` Expression
    | `!` Expression
```

r[expr.negate.intro]
这是最后两个一元运算符。

r[expr.negate.results]
下表总结了它们在原始类型上的行为，以及用于为其他类型重载这些运算符的 特型 。
请记住，有符号整数始终使用补码表示。
所有这些运算符的操作数都在 [值表达式语境][value expression] 中求值，因此会被移动或复制。

| 符号 | 整数     | `bool`        | 浮点数 | 重载特型  |
|--------|-------------|-------------- |----------------|--------------------|
| `-`    | 取负*   |               | 取负       | `std::ops::Neg`    |
| `!`    | 按位取反 | [逻辑非][Logical NOT] |                | `std::ops::Not`    |

\* 仅适用于有符号整数类型。

以下是这些运算符的一些示例：

```rust
let x = 6;
assert_eq!(-x, -6);
assert_eq!(!x, -7);
assert_eq!(true, !false);
```

r[expr.arith-logic]
## 算术和逻辑二元运算符

r[expr.arith-logic.syntax]
```grammar,表达式
ArithmeticOrLogicalExpression ->
      Expression `+` Expression
    | Expression `-` Expression
    | Expression `*` Expression
    | Expression `/` Expression
    | Expression `%` Expression
    | Expression `&` Expression
    | Expression `|` Expression
    | Expression `^` Expression
    | Expression `<<` Expression
    | Expression `>>` Expression
```

r[expr.arith-logic.intro]
二元运算符表达式都使用中缀记法编写。

r[expr.arith-logic.behavior]
下表总结了算术和逻辑二元运算符在原始类型上的行为，以及用于为其他类型重载这些运算符的 特型 。
请记住，有符号整数始终使用补码表示。
所有这些运算符的操作数都在 [值表达式语境][value expression] 中求值，因此会被移动或复制。

| 符号 | 整数                 | `bool`        | 浮点数 | 重载特型  | 重载复合赋值特型 |
|--------|-------------------------|---------------|----------------|--------------------| ------------------------------------- |
| `+`    | 加法                |               | 加法       | `std::ops::Add`    | `std::ops::AddAssign`                 |
| `-`    | 减法             |               | 减法    | `std::ops::Sub`    | `std::ops::SubAssign`                 |
| `*`    | 乘法          |               | 乘法 | `std::ops::Mul`    | `std::ops::MulAssign`                 |
| `/`    | 除法*†              |               | 除法       | `std::ops::Div`    | `std::ops::DivAssign`                 |
| `%`    | 取余**†            |               | 取余      | `std::ops::Rem`    | `std::ops::RemAssign`                 |
| `&`    | 按位与             | [逻辑与][Logical AND] |                | `std::ops::BitAnd` | `std::ops::BitAndAssign`              |
| `|` | 按位或 | [逻辑或][Logical OR]  |                | `std::ops::BitOr`  | `std::ops::BitOrAssign`               |
| `^`    | 按位异或             | [逻辑异或][Logical XOR] |                | `std::ops::BitXor` | `std::ops::BitXorAssign`              |
| `<<`   | 左移              |               |                | `std::ops::Shl`    | `std::ops::ShlAssign`                 |
| `>>`   | 右移***          |               |                | `std::ops::Shr`    |  `std::ops::ShrAssign`                |

\* 整数除法向零舍入。

\*\* Rust 使用由 [截断除法](https://en.wikipedia.org/wiki/Modulo_operation#Variants_of_the_definition) 定义的余数。给定 `remainder = dividend % divisor` ，余数将与被除数具有相同的符号。

\*\*\* 有符号整数类型为算术右移，无符号整数类型为逻辑右移。

† 对于整数类型，除以零会产生 恐慌 。

以下是正在使用的这些运算符的示例。

```rust
assert_eq!(3 + 6, 9);
assert_eq!(5.5 - 1.25, 4.25);
assert_eq!(-5 * 14, -70);
assert_eq!(14 / 3, 4);
assert_eq!(100 % 7, 2);
assert_eq!(0b1010 & 0b1100, 0b1000);
assert_eq!(0b1010 | 0b1100, 0b1110);
assert_eq!(0b1010 ^ 0b1100, 0b110);
assert_eq!(13 << 3, 104);
assert_eq!(-10 >> 2, -3);
```

r[expr.cmp]
## 比较运算符

r[expr.cmp.syntax]
```grammar,表达式
ComparisonExpression ->
      Expression `==` Expression
    | Expression `!=` Expression
    | Expression `>` Expression
    | Expression `<` Expression
    | Expression `>=` Expression
    | Expression `<=` Expression
```

r[expr.cmp.intro]
比较运算符也为原始类型和标准库中的许多类型定义。

r[expr.cmp.paren-chaining]
在链式调用比较运算符时需要括号。例如，表达式 `a == b == c` 是无效的，可以写成 `(a == b) == c` 。

r[expr.cmp.trait]
与算术和逻辑运算符不同，重载这些运算符的 特型 被更广泛地用于展示一个类型如何被比较，并且很可能被使用这些 特型 作为界限的函数假设为定义了实际的比较。
标准库中的许多函数和 声明宏 随后可以使用该假设（尽管不能以此来确保安全性）。

r[expr.cmp.place]
与上面的算术和逻辑运算符不同，这些运算符隐式地对其操作数进行共享借用，并在 [位置表达式语境][place expression] 中对它们求值：

```rust
# let a = 1;
# let b = 1;
a == b;
// 等价于
::std::cmp::PartialEq::eq(&a, &b);
```

这意味着操作数不需要被移出。

r[expr.cmp.behavior]

| 符号 | 含义                  | 重载方法         |
|--------|--------------------------|----------------------------|
| `==`   | 等于                    | `std::cmp::PartialEq::eq`  |
| `!=`   | 不等于                | `std::cmp::PartialEq::ne`  |
| `>`    | 大于             | `std::cmp::PartialOrd::gt` |
| `<`    | 小于                | `std::cmp::PartialOrd::lt` |
| `>=`   | 大于等于 | `std::cmp::PartialOrd::ge` |
| `<=`   | 小于等于    | `std::cmp::PartialOrd::le` |

以下是正在使用的比较运算符示例。

```rust
assert!(123 == 123);
assert!(23 != -12);
assert!(12.5 > 12.2);
assert!([1, 2, 3] < [1, 3, 4]);
assert!('A' <= 'B');
assert!("World" >= "Hello");
```

r[expr.bool-logic]
## 惰性布尔运算符

r[expr.bool-logic.syntax]
```grammar,表达式
LazyBooleanExpression ->
      Expression `||` Expression
    | Expression `&&` Expression
```

r[expr.bool-logic.intro]
运算符 `||` 和 `&&` 可以应用于布尔类型的操作数。
 `||` 运算符表示逻辑‘或’， `&&` 运算符表示逻辑‘与’。

r[expr.bool-logic.conditional-evaluation]
它们与 `|` 和 `&` 的不同之处在于，只有当左侧操作数尚未确定表达式结果时，才会对右侧操作数求值。
也就是说，只有当左侧操作数求值为 `false` 时， `||` 才会对其右侧操作数求值；而只有当左侧操作数求值为 `true` 时， `&&` 才会对其右侧操作数求值。

```rust
let x = false || true; // true
let y = false && panic!(); // false, 不会对 `panic!()` 求值
```

r[expr.as]
## 类型转换表达式

r[expr.as.syntax]
```grammar,表达式
TypeCastExpression -> Expression `as` TypeNoBounds
```

r[expr.as.intro]
类型转换表达式由二元运算符 `as` 表示。

r[expr.as.result]
执行 `as` 表达式会将左侧的值转换为右侧的类型。

 `as` 表达式的一个示例：

```rust
# fn sum(values: &[f64]) -> f64 { 0.0 }
# fn len(values: &[f64]) -> i32 { 0 }
fn average(values: &[f64]) -> f64 {
    let sum: f64 = sum(values);
    let size: f64 = len(values) as f64;
    sum / size
}
```

r[expr.as.coercions]
 `as` 可用于显式执行 [类型强制转换](../type-coercions.md) ，以及以下额外的转换。
任何既不符合强制转换规则也不符合表中条目的转换都是编译器错误。
这里 `*T` 表示 `*const T` 或 `*mut T` 。 `m` 代表引用类型中可选的 `mut` ，以及指针类型中的 `mut` 或 `const` 。

| `e` 的类型           | `U`                   | `e as U` 执行的转换                            |
|-----------------------|-----------------------|-------------------------------------------------------|
| 整数或浮点数类型 | 整数或浮点数类型 | [数值转换][expr.as.numeric]                       |
| 枚举           | 整数类型          | [枚举转换][expr.as.enum]                             |
| `bool` 或 `char`      | 整数类型          | [原始类型到整数转换][expr.as.bool-char-as-int] |
| `u8`                  | `char`                | [`u8` 到 `char` 转换][expr.as.u8-as-char]             |
| `*T`                  | `*V` [^meta-compat]   | [指针到指针转换][expr.as.pointer]            |
| `*T` 当 `T: Sized` | 整数类型          | [指针到地址转换][expr.as.pointer-as-int]     |
| 整数类型          | `*V` 当 `V: Sized` | [地址到指针转换][expr.as.int-as-pointer]     |
| `&m₁ [T; n]`          | `*m₂ T` [^lessmut]    | 数组到指针转换                                 |
| `*m₁ [T; n]`          | `*m₂ T` [^lessmut]    | 数组到指针转换                                 |
| [函数项][Function item]       | [函数指针][Function pointer]    | 函数项到函数指针转换                |
| [函数项][Function item]       | `*V` 当 `V: Sized` | 函数项到指针转换                         |
| [函数项][Function item]       | 整数               | 函数项到地址转换                         |
| [函数指针][Function pointer]    | `*V` 当 `V: Sized` | 函数指针到指针转换                      |
| [函数指针][Function pointer]    | 整数               | 函数指针到地址转换                      |
| 闭包 [^no-capture] | 函数指针      | 闭包到函数指针转换                      |

[^meta-compat]: 其中 `T` 和 `V` 具有兼容的元数据：
      * `V: Sized` ，或
      * 都是切片元数据（ `*[u16]` -> `*[u8]` ， `*str` -> `*(u8, [u32])` ），或
      * 都是相同的特型对象元数据，除了删除自动 特型 （ `*dyn Debug` -> `*(u16, dyn Debug)` ， `*dyn Debug + Send` -> `*dyn Debug` ）
          * **注意**：只有当主 特型 将自动 特型 作为超级 特型 时，才允许 *增加* 自动 特型 （给定 `trait T: Send {}` ， `*dyn T` -> `*dyn T + Send` 是有效的，但 `*dyn Debug` -> `*dyn Debug + Send` 则无效）
          * **注意**：泛型（包括生命周期）必须匹配（ `*dyn T<'a, A>` -> `*dyn T<'b, B>` 要求 `'a = 'b` 且 `A = B` ）

[^lessmut]: 仅当 `m₁` 为 `mut` 或 `m₂` 为 `const` 时。允许将 `mut` 引用/指针转换为 `const` 指针。

[^no-capture]: 只有不捕获（封闭）任何局部变量的闭包才能转换为函数指针。

### 语义

r[expr.as.numeric]
#### 数值转换

r[expr.as.numeric.int-same-size]
* 在两个大小相同的整数之间转换（例如 i32 -> u32）是无操作的（Rust 对固定整数的负值使用补码）

  ```rust
  assert_eq!(42i8 as u8, 42u8);
  assert_eq!(-1i8 as u8, 255u8);
  assert_eq!(255u8 as i8, -1i8);
  assert_eq!(-1i16 as u16, 65535u16);
  ```

r[expr.as.numeric.int-truncation]
* 从较大的整数转换为较小的整数（例如 u32 -> u8）将发生截断

  ```rust
  assert_eq!(42u16 as u8, 42u8);
  assert_eq!(1234u16 as u8, 210u8);
  assert_eq!(0xabcdu16 as u8, 0xcdu8);

  assert_eq!(-42i16 as i8, -42i8);
  assert_eq!(1234u16 as i8, -46i8);
  assert_eq!(0xabcdi32 as i8, -51i8);
  ```

r[expr.as.numeric.int-extension]
* 从较小的整数转换为较大的整数（例如 u8 -> u32）将
    * 如果源是无符号的，则进行零扩展
    * 如果源是有符号的，则进行符号扩展

  ```rust
  assert_eq!(42i8 as i16, 42i16);
  assert_eq!(-17i8 as i16, -17i16);
  assert_eq!(0b1000_1010u8 as u16, 0b0000_0000_1000_1010u16, "零扩展");
  assert_eq!(0b0000_1010i8 as i16, 0b0000_0000_0000_1010i16, "符号扩展 0");
  assert_eq!(0b1000_1010u8 as i8 as i16, 0b1111_1111_1000_1010u16 as i16, "符号扩展 1");
  ```

r[expr.as.numeric.float-as-int]
* 从浮点数转换为整数将向零舍入
    * `NaN` 将返回 `0`
    * 大于最大整数值的值（包括 `INFINITY` ），将饱和为整数类型的最大值。
    * 小于最小整数值的值（包括 `NEG_INFINITY` ），将饱和为整数类型的最小值。

  ```rust
  assert_eq!(42.9f32 as i32, 42);
  assert_eq!(-42.9f32 as i32, -42);
  assert_eq!(42_000_000f32 as i32, 42_000_000);
  assert_eq!(std::f32::NAN as i32, 0);
  assert_eq!(1_000_000_000_000_000f32 as i32, 0x7fffffffi32);
  assert_eq!(std::f32::NEG_INFINITY as i32, -0x80000000i32);
  ```

r[expr.as.numeric.int-as-float]
* 从整数转换为浮点数将产生最接近的浮点数 \*
    * 如有必要，根据 `roundTiesToEven` 模式进行舍入 \*\*\*
    * 溢出时，产生（与输入符号相同的）无穷大
    * 注意：在当前的数值类型集中，溢出只能发生在 `u128 as f32` 且值大于或等于 `f32::MAX + (0.5 ULP)` 时

  ```rust
  assert_eq!(1337i32 as f32, 1337f32);
  assert_eq!(123_456_789i32 as f32, 123_456_790f32, "已舍入");
  assert_eq!(0xffffffff_ffffffff_ffffffff_ffffffff_u128 as f32, std::f32::INFINITY);
  ```

r[expr.as.numeric.float-widening]
* 从 f32 转换为 f64 是完美的且无损的

  ```rust
  assert_eq!(1_234.5f32 as f64, 1_234.5f64);
  assert_eq!(std::f32::INFINITY as f64, std::f64::INFINITY);
  assert!((std::f32::NAN as f64).is_nan());
  ```

r[expr.as.numeric.float-narrowing]
* 从 f64 转换为 f32 将产生最接近的 f32 \*\*
    * 如有必要，根据 `roundTiesToEven` 模式进行舍入 \*\*\*
    * 溢出时，产生（与输入符号相同的）无穷大

  ```rust
  assert_eq!(1_234.5f64 as f32, 1_234.5f32);
  assert_eq!(1_234_567_891.123f64 as f32, 1_234_567_890f32, "已舍入");
  assert_eq!(std::f64::INFINITY as f32, std::f32::INFINITY);
  assert!((std::f64::NAN as f32).is_nan());
  ```

\* 如果硬件不原生支持具有这种舍入模式和溢出行为的整数到浮点数转换，则这些转换可能会比预期的慢。

\*\* 如果硬件不原生支持具有这种舍入模式和溢出行为的 f64 到 f32 转换，则这些转换可能会比预期的慢。

\*\*\* 按照 IEEE 754-2008 §4.3.1 的定义：选择最接近的浮点数，如果在两个浮点数中间，则优先选择最低有效位为偶数的那个。

r[expr.as.enum]
#### 枚举转换

r[expr.as.enum.discriminant]
将枚举转换为其判别值，然后根据需要使用数值转换。
转换仅限于以下几种枚举：

* [仅单元项枚举][Unit-only enums]
* [无字段枚举][Field-less enums] 且没有 [显式判别值][explicit discriminants] ，或者只有单元项变体具有显式判别值

```rust
enum Enum { A, B, C }
assert_eq!(Enum::A as i32, 0);
assert_eq!(Enum::B as i32, 1);
assert_eq!(Enum::C as i32, 2);
```

r[expr.as.enum.no-drop]
如果枚举实现了 [`Drop`] ，则不允许进行转换。

r[expr.as.bool-char-as-int]
#### 原始类型到整数转换

* `false` 转换为 `0` ， `true` 转换为 `1`
* `char` 转换为码位的值，然后根据需要使用数值转换。

```rust
assert_eq!(false as i32, 0);
assert_eq!(true as i32, 1);
assert_eq!('A' as i32, 65);
assert_eq!('Ö' as i32, 214);
```

r[expr.as.u8-as-char]
#### `u8` 到 `char` 转换

转换为具有相应码位的 `char` 。

```rust
assert_eq!(65u8 as char, 'A');
assert_eq!(214u8 as char, 'Ö');
```

r[expr.as.pointer-as-int]
#### 指针到地址转换

从原始指针转换为整数会产生引用内存的机器地址。
如果整数类型小于指针类型，则地址可能会被截断；使用 `usize` 可以避免这种情况。

r[expr.as.int-as-pointer]
#### 地址到指针转换

从整数转换为原始指针会将整数解释为内存地址，并产生一个引用该内存的指针。

> [!WARNING]
> 这与仍处于开发阶段的 Rust 内存模型有关。
> 从此转换获得的指针即使在位上等于有效指针，也可能会受到额外的限制。
> 如果不遵循别名规则，对这种指针进行解引用可能会导致 [未定义行为][undefined behavior] 。

一个简单的健全地址算术示例：

```rust
let mut values: [i32; 2] = [1, 2];
let p1: *mut i32 = values.as_mut_ptr();
let first_address = p1 as usize;
let second_address = first_address + 4; // 4 == size_of::<i32>()
let p2 = second_address as *mut i32;
unsafe {
    *p2 += 1;
}
assert_eq!(values[1], 3);
```

r[expr.as.pointer]
#### 指针到指针转换

r[expr.as.pointer.behavior]
 `*const T` / `*mut T` 可以转换为 `*const U` / `*mut U` ，具有以下行为：

r[expr.as.pointer.sized]
- 如果 `T` 和 `U` 都是有大小的，则指针保持不变并返回。

r[expr.as.pointer.unsized]
- 如果 `T` 和 `U` 都是无大小的，则指针也保持不变并返回。
  特别地，元数据会被精确保留。

  例如，从 `*const [T]` 到 `*const [U]` 的转换会保留元素的数量。
  请注意，因此此类转换不一定保留指针引用对象的大小
  （例如，将 `*const [u16]` 转换为 `*const [u8]` 将产生一个原始指针，它指向的对象大小是原来的一半）。
  对于 `str` 和任何其无大小尾部是切片类型的复合类型（如 `struct Foo(i32, [u8])` 或 `(u64, Foo)` ），情况也是如此。

r[expr.as.pointer.discard-metadata]
- 如果 `T` 是无大小的且 `U` 是有大小的，则转换会丢弃完成宽指针 `T` 的所有元数据，并产生一个由无大小指针的数据部分组成的细指针 `U` 。

r[expr.assign]
## 赋值表达式

r[expr.assign.syntax]
```grammar,表达式
AssignmentExpression -> Expression `=` Expression
```

r[expr.assign.intro]
一个 _赋值表达式_ 将一个值移动到一个指定的位置。

r[expr.assign.assignee]
赋值表达式由一个 [可变][mutable] [赋值目标表达式][assignee expression] （即 _赋值目标操作数_ ），后跟一个等号（ `=` ）和一个 [值表达式][value expression] （即 _被赋值操作数_ ）组成。

r[expr.assign.behavior-basic]
在其最基本的形式中，赋值目标表达式是一个 [位置表达式][place expression] ，我们首先讨论这种情况。

r[expr.assign.behavior-destructuring]
解构赋值的更一般情况在下面讨论，但这种情况总是分解为对位置表达式的顺序赋值，这可以被视为更基本的情况。

r[expr.assign.basic]
### 基本赋值

r[expr.assign.evaluation-order]
求值赋值表达式始于对其操作数求值。
首先对被赋值操作数求值，然后对赋值目标表达式求值。

r[expr.assign.destructuring-order]
对于解构赋值，赋值目标表达式的子表达式按从左到右的顺序求值。

> [!NOTE]
> 这与其他表达式不同，因为右侧操作数在左侧操作数之前求值。

r[expr.assign.drop-target]
然后它的效果是首先 [释放][dropping] 赋值位置的值，除非该位置是未初始化的局部变量或局部变量的未初始化字段。

r[expr.assign.behavior]
接着，它要么将分配的值 [复制或移动][copies or moves] 到赋值位置。

r[expr.assign.result]
赋值表达式总是产生 [单元值][unit] 。

示例：

```rust
let mut x = 0;
let y = 0;
x = y;
```

r[expr.assign.destructure]
### 解构赋值

r[expr.assign.destructure.intro]
解构赋值是变量声明中解构模式匹配的对应物，允许对复杂值（如元组或结构体）进行赋值。
例如，我们可以交换两个可变变量：

```rust
let (mut a, mut b) = (0, 1);
// 使用解构赋值交换 `a` 和 `b`。
(b, a) = (a, b);
```

r[expr.assign.destructure.assignee]
与使用 `let` 的解构声明相反，由于语法歧义，模式不能出现在赋值语句的左侧。
相反，对应于模式的一组表达式被指定为 [赋值目标表达式][assignee expression] ，并允许出现在赋值语句的左侧。
赋值目标表达式随后被脱糖为模式匹配，后跟顺序赋值。

r[expr.assign.destructure.irrefutable]
脱糖后的模式必须是不可驳回的：特别地，这意味着只有在编译时长度已知的切片模式，以及平凡切片 `[..]` ，才允许用于解构赋值。

脱糖方法非常直接，通过示例可以最好地说明。

```rust
# struct Struct { x: u32, y: u32 }
# let (mut a, mut b) = (0, 0);
(a, b) = (3, 4);

[a, b] = [3, 4];

Struct { x: a, y: b } = Struct { x: 3, y: 4};

// 脱糖为：

{
    let (_a, _b) = (3, 4);
    a = _a;
    b = _b;
}

{
    let [_a, _b] = [3, 4];
    a = _a;
    b = _b;
}

{
    let Struct { x: _a, y: _b } = Struct { x: 3, y: 4};
    a = _a;
    b = _b;
}
```

r[expr.assign.destructure.repeat-ident]
并不禁止在单个赋值目标表达式中多次使用标识符。

r[expr.assign.destructure.discard-value]
 [下划线表达式][Underscore expressions] 和空 [范围表达式][range expressions] 可用于忽略某些值，而不绑定它们。

r[expr.assign.destructure.default-binding]
请注意，默认绑定模式不适用于脱糖表达式。

r[expr.assign.destructure.tmp-scopes]
> [!NOTE]
> 脱糖限制了解构赋值中被赋值操作数（右侧）的 [临时作用域][temporary scope] 。
>
> 在基本赋值中， [临时变量][temporary] 在封闭的临时作用域结束时被释放。在下面，那就是语句。因此，赋值和使用是被允许的。
>
> ```rust
> # fn temp() {}
> fn f<T>(x: T) -> T { x }
> let x;
> (x = f(&temp()), x); // OK
> ```
>
> 相反，在解构赋值中，临时变量在脱糖后的 `let` 语句结束时被释放。由于这发生在尝试给 `x` 赋值之前，如下所示，它将失败。
>
> ```rust,compile_fail,E0716
> # fn temp() {}
> # fn f<T>(x: T) -> T { x }
> # let x;
> [x] = [f(&temp())]; // 错误
> ```
>
> 这被脱糖为：
>
> ```rust,compile_fail,E0716
> # fn temp() {}
> # fn f<T>(x: T) -> T { x }
> # let x;
> {
>     let [_x] = [f(&temp())];
>     //                     ^
>     //      临时变量在这里被释放。
>     x = _x; // 错误
> }
> ```

r[expr.assign.destructure.tmp-ext]
> [!NOTE]
> 由于脱糖，解构赋值的被赋值操作数（右侧）是新引入块内的 [扩展表达式][extending expression] 。
>
> 在下面，因为 [临时作用域][temporary scope] 被扩展到了此引入块的末尾，所以赋值是被允许的。
>
> ```rust
> # fn temp() {}
> # let x;
> [x] = [&temp()]; // OK
> ```
>
> 这被脱糖为：
>
> ```rust
> # fn temp() {}
> # let x;
> { let [_x] = [&temp()]; x = _x; } // OK
> ```
>
> 然而，如果我们尝试使用 `x` ，即使在同一语句中，也会得到一个错误，因为 [临时变量][temporary] 在此引入块的末尾被释放。
>
> ```rust,compile_fail,E0716
> # fn temp() {}
> # let x;
> ([x] = [&temp()], x); // 错误
> ```
>
> 这被脱糖为：
>
> ```rust,compile_fail,E0716
> # fn temp() {}
> # let x;
> (
>     {
>         let [_x] = [&temp()];
>         x = _x;
>     }, // <-- 临时变量在这里被释放。
>     x, // 错误
> );
> ```

r[expr.compound-assign]
## 复合赋值表达式

r[expr.compound-assign.syntax]
```grammar,表达式
CompoundAssignmentExpression ->
      Expression `+=` Expression
    | Expression `-=` Expression
    | Expression `*=` Expression
    | Expression `/=` Expression
    | Expression `%=` Expression
    | Expression `&=` Expression
    | Expression `|=` Expression
    | Expression `^=` Expression
    | Expression `<<=` Expression
    | Expression `>>=` Expression
```

r[expr.compound-assign.intro]
 *复合赋值表达式* 将算术和逻辑二元运算符与赋值表达式结合在一起。

例如：

```rust
let mut x = 5;
x += 1;
assert!(x == 6);
```

复合赋值的语法是一个 [可变][mutable] [位置表达式][place expression] （即 _被赋值操作数_ ），然后是一个由运算符后跟 `=` 组成的单个 词法单元 （无空格），最后是一个 [值表达式][value expression] （即 _修改操作数_ ）。

r[expr.compound-assign.place]
与其他位置操作数不同，被赋值的操作数必须是一个位置表达式。

r[expr.compound-assign.no-value]
尝试使用值表达式是编译器错误，而不是将其提升为临时变量。

r[expr.compound-assign.operand-order]
复合赋值表达式的求值得取决于操作数的类型。

r[expr.compound-assign.primitives]
如果在单态化之前已知两个操作数的类型都是原始类型，则首先对右侧求值，接着对左侧求值，然后通过将运算符应用于两侧的值来修改由左侧求值得出的位置。

```rust
# use core::{num::Wrapping, ops::AddAssign};
#
trait Equate {}
impl<T> Equate for (T, T) {}

fn f1(x: (u8,)) {
    let mut order = vec![];
    // 首先对右侧求值，因为两个操作数都是原始类型。
    { order.push(2); x }.0 += { order.push(1); x }.0;
    assert!(order.is_sorted());
}

fn f2(x: (Wrapping<u8>,)) {
    let mut order = vec![];
    // 首先对左侧求值，因为 `Wrapping<_>` 不是原始类型。
    { order.push(1); x }.0 += { order.push(2); (0u8,) }.0;
    assert!(order.is_sorted());
}

fn f3<T: AddAssign<u8> + Copy>(x: (T,)) where (T, u8): Equate {
    let mut order = vec![];
    // 首先对左侧求值，因为其中一个操作数是泛型参数，
    // 即使该泛型参数由于 where 子句界限可以与原始类型统一。
    { order.push(1); x }.0 += { order.push(2); (0u8,) }.0;
    assert!(order.is_sorted());
}

fn main() {
    f1((0u8,));
    f2((Wrapping(0u8),));
    // 我们提供一个原始类型作为泛型参数，但这不会影响单态化后 `f3` 中的求值顺序。
    f3::<u8>((0u8,));
}
```

> [!NOTE]
> 这是不寻常的。在其他地方，从左到右求值是常规。
>
> 更多示例请参阅 [求值顺序测试][eval order test] 。

r[expr.compound-assign.trait]
否则，此表达式是使用该运算符对应的 特型 （见 [expr.arith-logic.behavior] ）并以左侧作为 [接收者][receiver] 、右侧作为下一个参数调用其方法的语法糖。

例如，以下两个语句是等价的：

```rust
# use std::ops::AddAssign;
fn f<T: AddAssign + Copy>(mut x: T, y: T) {
    x += y; // 语句 1。
    x.add_assign(y); // 语句 2。
}
```

> [!NOTE]
> 令人惊讶的是，将其进一步脱糖为完全限定的方法调用并不等价，因为当通过 [自动引用][autoref] 获取指向第一个操作数的可变引用时，存在特殊的借用检查器行为。
>
> ```rust
> # use std::ops::AddAssign;
> fn f<T: AddAssign + Copy>(mut x: T) {
>     // 这里我们将 `x` 同时用作左侧和右侧。因为调用特型方法所需的
>     // 左侧可变借用是通过自动引用隐式获取的，所以这是 OK 的。
>     x += x; //~ OK
>     x.add_assign(x); //~ OK
> }
> ```
>
> ```rust,compile_fail,E0503
> # use std::ops::AddAssign;
> fn f<T: AddAssign + Copy>(mut x: T) {
>     // 我们不能将上面的代码脱糖为下面的形式，因为一旦我们为了传递第一个参数
>     // 而获取了 `x` 的可变借用，我们就不能在第二个参数中按值传递 `x` ，
>     // 因为可变引用仍然有效。
>     <T as AddAssign>::add_assign(&mut x, x);
>     //~^ 错误：不能使用 `x` 因为它已被可变借用
> }
> ```
>
> ```rust,compile_fail,E0503
> # use std::ops::AddAssign;
> fn f<T: AddAssign + Copy>(mut x: T) {
>     // 同上。
>     (&mut x).add_assign(x);
>     //~^ 错误：不能使用 `x` 因为它已被可变借用
> }
> ```

r[expr.compound-assign.result]
与普通赋值表达式一样，复合赋值表达式总是产生 [单元值][unit] 。

> [!WARNING]
> 避免编写依赖于复合赋值中操作数求值顺序的代码，因为它可能是不寻常且令人惊讶的。

[`Try`]: core::ops::Try
[autoref]: expr.method.candidate-receivers-refs
[copies or moves]: ../expressions.md#moved-and-copied-types
[dropping]: ../destructors.md
[eval order test]: https://github.com/rust-lang/rust/blob/1.58.0/src/test/ui/expr/compound-assignment/eval-order.rs
[explicit discriminants]: ../items/enumerations.md#explicit-discriminants
[extending expression]: destructors.scope.lifetime-extension.exprs
[field-less enums]: ../items/enumerations.md#field-less-enum
[grouped expression]: grouped-expr.md
[literal expression]: literal-expr.md#integer-literal-expressions
[logical and]: ../types/boolean.md#logical-and
[logical not]: ../types/boolean.md#logical-not
[logical or]: ../types/boolean.md#logical-or
[logical xor]: ../types/boolean.md#logical-xor
[mutable]: ../expressions.md#mutability
[place expression]: ../expressions.md#位置表达式和值表达式
[assignee expression]: ../expressions.md#位置表达式和值表达式
[undefined behavior]: ../behavior-considered-undefined.md
[unit]: ../types/tuple.md
[Unit-only enums]: ../items/enumerations.md#unit-only-enum
[value expression]: ../expressions.md#位置表达式和值表达式
[temporary lifetime extension]: destructors.scope.lifetime-extension
[temporary scope]: destructors.scope.temporary
[temporary value]: ../expressions.md#temporaries
[float-float]: https://github.com/rust-lang/rust/issues/15536
[Function pointer]: ../types/function-pointer.md
[Function item]: ../types/function-item.md
[receiver]: expr.method.intro
[temporary]: expr.temporary
[undefined behavior]: ../behavior-considered-undefined.md
[Underscore expressions]: ./underscore-expr.md
[range expressions]: ./range-expr.md
