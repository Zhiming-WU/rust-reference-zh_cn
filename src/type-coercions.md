r[coerce]
# 隐式类型转换

r[coerce.intro]
 **隐式类型转换** 是改变值类型的隐式操作。它们在特定位置自动发生，并且对实际进行转换的类型有严格限制。

r[coerce.as]
任何允许隐式转换的转换也可以通过 [类型转换运算符][type cast operator] `as` 显式执行。

隐式转换最初在 [RFC 401] 中定义，并在 [RFC 1558] 中得到扩展。

r[coerce.site]
## 转换点

r[coerce.site.intro]
隐式转换只能发生在程序中的某些转换点；这些位置通常是所需类型明确或可以通过从明确类型传播（无需类型推断）导出的地方。可能的转换点有：

r[coerce.site.let]
* 给出显式类型的 `let` 语句。

   例如，在以下代码中， `&mut 42` 被隐式转换为 `&i8` 类型：

   ```rust
   let _: &i8 = &mut 42;
   ```

r[coerce.site.value]
* `static` 和 `const` 项 声明（类似于 `let` 语句）。

r[coerce.site.argument]
* 函数调用的参数

  被转换的值是实际参数，它被转换为正式参数的类型。

  例如，在以下代码中， `&mut 42` 被隐式转换为 `&i8` 类型：

  ```rust
  fn bar(_: &i8) { }

  fn main() {
      bar(&mut 42);
  }
  ```

  对于方法调用，接收者（ `self` 参数）类型的转换方式不同，详情请参阅 [方法调用表达式][method-call expressions] 的文档。

r[coerce.site.constructor]
* 结构体、 联合体 或 枚举 变体 字段的实例化

  例如，在以下代码中， `&mut 42` 被隐式转换为 `&i8` 类型：

  ```rust
  struct Foo<'a> { x: &'a i8 }

  fn main() {
      Foo { x: &mut 42 };
  }
  ```

r[coerce.site.return]
* 函数结果&mdash;&mdash;如果代码块不是以分号结尾，则为代码块的最后一行，或者是 `return` 语句中的任何表达式

  例如，在以下代码中， `x` 被隐式转换为 `&dyn Display` 类型：

  ```rust
  use std::fmt::Display;
  fn foo(x: &u32) -> &dyn Display {
      x
  }
  ```

r[coerce.site.subexpr]
如果这些转换点中的表达式是一个转换传播表达式，那么该表达式中相关的子表达式也是转换点。传播从这些新的转换点递归进行。传播表达式及其相关的子表达式包括：

r[coerce.site.array]
* 数组字面量，其中数组类型为 `[U; n]` 。数组字面量中的每个子表达式都是转换为 `U` 类型的转换点。

r[coerce.site.repeat]
* 具有重复语法格式的数组字面量，其中数组类型为 `[U; n]` 。重复的子表达式是转换为 `U` 类型的转换点。

r[coerce.site.tuple]
* 元组，元组本身是转换为类型 `(U_0, U_1, ..., U_n)` 的转换点。每个子表达式都是对应类型的转换点，例如，第 0 个子表达式是转换为 `U_0` 类型的转换点。

r[coerce.site.parenthesis]
* 括号子表达式（ `(e)` ）：如果表达式具有类型 `U` ，则子表达式是转换为 `U` 的转换点。

r[coerce.site.block]
* 代码块：如果代码块具有类型 `U` ，则代码块中的最后一个表达式（如果不是以分号结尾）是转换为 `U` 的转换点。这包括属于控制流语句（如 `if` / `else` ）一部分的代码块，前提是该代码块具有已知类型。

r[coerce.types]
## 转换类型

允许在以下类型之间进行隐式转换：

r[coerce.types.reflexive]
* 如果 `T` 是 `U` 的 [子类型][subtype] ，则 `T` 到 `U` （ *自反情况* ）

r[coerce.types.transitive]
* `T_1` 到 `T_3` ，其中 `T_1` 可隐式转换为 `T_2` 且 `T_2` 可隐式转换为 `T_3` （ *传递情况* ）

    请注意，这尚未得到完全支持。

r[coerce.types.mut-reborrow]
* `&mut T` 到 `&T` 

r[coerce.types.mut-pointer]
* `*mut T` 到 `*const T` 

r[coerce.types.ref-to-pointer]
* `&T` 到 `*const T` 

r[coerce.types.mut-to-pointer]
* `&mut T` 到 `*mut T` 

r[coerce.types.deref]
* 如果 `T` 实现了 `Deref<Target = U>` ，则 `&T` 或 `&mut T` 到 `&U` 。例如：

  ```rust
  use std::ops::Deref;

  struct CharContainer {
      value: char,
  }

  impl Deref for CharContainer {
      type Target = char;

      fn deref<'a>(&'a self) -> &'a char {
          &self.value
      }
  }

  fn foo(arg: &char) {}

  fn main() {
      let x = &mut CharContainer { value: 'y' };
      foo(x); // &mut CharContainer 被隐式转换为 &char。
  }
  ```

r[coerce.types.deref-mut]
* 如果 `T` 实现了 `DerefMut<Target = U>` ，则 `&mut T` 到 `&mut U` 。

r[coerce.types.unsize]
* TyCtor( `T` ) 到 TyCtor( `U` )，其中 TyCtor( `T` ) 是以下之一
    - `&T`
    - `&mut T`
    - `*const T`
    - `*mut T`
    - `Box<T>`

    并且可以通过 [非定长转换](#unsized-coercions) 从 `T` 获得 `U` 。

    <!--In the future, coerce_inner will be recursively extended to tuples and
    structs. In addition, coercions from subtraits to supertraits will be
    added. See [RFC 401] for more details.-->

r[coerce.types.fn]
* 函数 项 类型到 `fn` 指针

r[coerce.types.closure]
* 非捕获闭包到 `fn` 指针

r[coerce.types.never]
* `!` 到任何 `T` 

r[coerce.unsize]
### 非定长转换

r[coerce.unsize.intro]
以下转换被称为 `非定长转换` ，因为它们与将类型转换为非定长类型有关，并且在上述其他转换不允许的少数情况下是允许的。它们仍然可以发生在隐式转换可以发生的任何其他地方。

r[coerce.unsize.trait]
两个 特型 [`Unsize`] 和 [`CoerceUnsized`] 被用于协助此过程并将其公开给库使用。以下转换是内置的，如果 `T` 可以通过其中之一隐式转换为 `U` ，则将提供 `T` 对 `Unsize<U>` 的实现：

r[coerce.unsize.slice]
* `[T; n]` 到 `[T]` 。

r[coerce.unsize.trait-object]
* `T` 到 `dyn U` ，当 `T` 实现 `U + Sized` ，且 `U` 是 [dyn 兼容][dyn compatible] 的。

r[coerce.unsize.trait-upcast]
* `dyn T` 到 `dyn U` ，当 `U` 是 `T` 的 [父特型][supertraits] 之一时。
    * 这允许丢弃自动特型，即允许 `dyn T + Auto` 到 `dyn U` 。
    * 如果主特型将自动特型作为父特型，则允许添加自动特型，即给定 `trait T: U + Send {}` ，允许 `dyn T` 到 `dyn T + Send` 或到 `dyn U + Send` 的隐式转换。

r[coerce.unsized.composite]
* `Foo<..., T, ...>` 到 `Foo<..., U, ...>` ，当：
    * `Foo` 是一个 结构体 。
    * `T` 实现了 `Unsize<U>` 。
    * `Foo` 的最后一个字段具有涉及 `T` 的类型。
    * 如果该字段的类型为 `Bar<T>` ，则 `Bar<T>` 实现了 `Unsize<Bar<U>>` 。
    * `T` 不是任何其他字段类型的一部分。

r[coerce.unsized.pointer]
此外，当 `T` 实现了 `Unsize<U>` 或 `CoerceUnsized<Foo<U>>` 时，类型 `Foo<T>` 可以实现 `CoerceUnsized<Foo<U>>` 。这允许它提供到 `Foo<U>` 的非定长转换。

> [!NOTE]
> 虽然非定长转换的定义及其实现已经稳定，但特型本身尚未稳定，因此不能在稳定版 Rust 中直接使用。

r[coerce.least-upper-bound]
## 最小上界转换

r[coerce.least-upper-bound.intro]
在某些语境中，编译器必须将多个类型一起进行隐式转换，以尝试找到最通用的类型。这被称为 “最小上界” （LUB）转换。LUB 转换仅在以下情况下使用：

+ 寻找一系列 `if` 分支的共同类型。
+ 寻找一系列 `match` 臂的共同类型。
+ 寻找数组元素的共同类型。
+ 寻找具有多个返回语句的闭包的返回类型。
+ 检查具有多个返回语句的函数的返回类型。

r[coerce.least-upper-bound.target]
在每种情况下，都有一组类型 `T0..Tn` 需要相互转换为某个目标类型 `T_t` ，该类型在开始时是未知的。

r[coerce.least-upper-bound.computation]
计算 LUB 转换是迭代进行的。目标类型 `T_t` 最初为类型 `T0` 。对于每个新类型 `Ti` ，我们考虑：

r[coerce.least-upper-bound.computation-identity]
+ 如果 `Ti` 可以隐式转换为当前目标类型 `T_t` ，则不作更改。

r[coerce.least-upper-bound.computation-replace]
+ 否则，检查 `T_t` 是否可以隐式转换为 `Ti` ；如果是，则将 `T_t` 更改为 `Ti` 。（此检查还取决于迄今为止考虑的所有源表达式是否都具有隐式转换。）

r[coerce.least-upper-bound.computation-unify]
+ 如果都不是，尝试计算 `T_t` 和 `Ti` 的共同超类型，这将成为新的目标类型。

### 示例：

```rust
# let (a, b, c) = (0, 1, 2);
// 对于 if 分支
let bar = if true {
    a
} else if false {
    b
} else {
    c
};

// 对于 match 臂
let baw = match 42 {
    0 => a,
    1 => b,
    _ => c,
};

// 对于数组元素
let bax = [a, b, c];

// 对于具有多个返回语句的闭包
let clo = || {
    if true {
        a
    } else if false {
        b
    } else {
        c
    }
};
let baz = clo();

// 对于具有多个返回语句的函数的类型检查
fn foo() -> i32 {
    let (a, b, c) = (0, 1, 2);
    match 42 {
        0 => a,
        1 => b,
        _ => c,
    }
}
```

在这些示例中， `ba*` 的类型是通过 LUB 转换找到的。并且编译器在处理函数 `foo` 时会检查 `a` 、 `b` 、 `c` 的 LUB 转换结果是否为 `i32` 。

### 警告

这种描述显然是非正式的。更精确的描述预计将作为更精确地规范 Rust 类型检查器总体工作的一部分来推进。

[RFC 401]: https://github.com/rust-lang/rfcs/blob/master/text/0401-coercions.md
[RFC 1558]: https://github.com/rust-lang/rfcs/blob/master/text/1558-closure-to-fn-coercion.md
[subtype]: subtyping.md
[dyn compatible]: items/traits.md#dyn兼容性
[type cast operator]: expressions/operator-expr.md#type-cast-expressions
[`Unsize`]: std::marker::Unsize
[`CoerceUnsized`]: std::ops::CoerceUnsized
[method-call expressions]: expressions/method-call-expr.md
[supertraits]: items/traits.md#supertraits
