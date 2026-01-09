r[const-eval]
# 常量求值

r[const-eval.general]
常量求值是在编译过程中计算 [表达式][expressions] 结果的过程。所有表达式中只有一部分可以在编译时求值。

r[const-eval.const-expr]
## 常量表达式

r[const-eval.const-expr.general]
某些形式的表达式，称为常量表达式，可以在编译时求值。

r[const-eval.const-expr.const-context]
在 [常量上下文][const context] 中的表达式必须是常量表达式。

r[const-eval.const-expr.evaluation]
常量上下文中的表达式总是在编译时求值。

r[const-eval.const-expr.runtime-context]
在常量上下文之外，常量表达式 *可能* 会被求值，但不保证一定在编译时求值。

r[const-eval.const-expr.error]
如果一个值必须在编译时（即在常量上下文中）求值，那么诸如越界 [数组索引][array indexing] 或 [溢出][overflow] 之类的行为将是编译器错误。否则，这些行为只是警告，但在运行时很可能会产生 恐慌。

r[const-eval.const-expr.list]
只要所有操作数也都是常量表达式，且不会导致任何 [`Drop::drop`][destructors] 调用被运行，以下表达式就是常量表达式。

r[const-eval.const-expr.literal]
* [字面量][Literals]。

r[const-eval.const-expr.parameter]
* [常量参数][Const parameters]。

r[const-eval.const-expr.path-item]
* 指向 [函数][functions] 和 [常量][constants] 的 [路径][paths]。不允许递归地定义常量。

r[const-eval.const-expr.path-static]
* 指向 [静态项][statics] 的路径，具有以下限制：
  * 在任何常量求值上下文中都不允许对 `static` 项进行写入。
  * 在任何常量求值上下文中都不允许从 `extern` 静态项中读取。
  * 如果求值 *不是* 在 `static` 项的初始化器中进行的，那么就不允许从任何可变的 `static` 中读取。可变的 `static` 是指 `static mut` 项，或者具有内部可变类型的 `static` 项。

  这些要求仅在常量求值时检查。换句话说，只要这些访问从未被执行，它们在语法上出现在常量上下文中是允许的。

r[const-eval.const-expr.tuple]
* [元组表达式][tuple expressions]。

r[const-eval.const-expr.array]
* [数组表达式][array expressions]。

r[const-eval.const-expr.constructor]
* [结构体表达式][Struct expressions]。

r[const-eval.const-expr.block]
* [块表达式][block expressions]，包括 `unsafe` 和 `const` 块。
    * [let 语句][let statements] 及其不可驳 [模式][patterns]，包括可变绑定
    * [赋值表达式][assignment expressions]
    * [复合赋值表达式][compound assignment expressions]
    * [表达式语句][expression statements]

r[const-eval.const-expr.field]
* [字段][field] 表达式。

r[const-eval.const-expr.index]
* 索引表达式，使用 `usize` 进行 [数组索引][array indexing] 或 [切片][slice]。

r[const-eval.const-expr.range]
* [范围表达式][range expressions]。

r[const-eval.const-expr.closure]
* 不从环境中捕获变量的 [闭包表达式][closure expressions]。

r[const-eval.const-expr.builtin-arith-logic]
* 用于整数和浮点类型、`bool` 和 `char` 的内建 [取反][negation]、[算术][arithmetic]、[逻辑][logical]、[比较][comparison] 或 [惰性布尔][lazy boolean] 运算符。

r[const-eval.const-expr.borrows]
* 所有形式的 [借用][borrow]，包括原始借用，但以下表达式的借用除外（这些表达式的临时作用域会被延长（见 [临时变量生命周期延长][temporary lifetime extension]）至程序结束）：
  * 可变借用。
  * 对产生具有 [内部可变性][interior mutability] 的值的表达式的共享借用。

  ```rust,compile_fail,E0764
  // 由于处于尾部位置，此借用将临时变量的作用域延长至程序结束。
  // 由于借用是可变的，这在常量表达式中是不允许的。
  const C: &u8 = &mut 0; // ERROR not allowed
  ```

  ```rust,compile_fail,E0764
  // 常量块类似于常量项的初始化器。
  let _: &u8 = const { &mut 0 }; // ERROR not allowed
  ```

  ```rust,compile_fail,E0492
  # use core::sync::atomic::AtomicU8;
  // 这是不允许的，因为 1) 临时作用域延长到了程序结束，且 2) 临时变量具有内部可变性。
  const C: &AtomicU8 = &AtomicU8::new(0); // ERROR not allowed
  ```

  ```rust,compile_fail,E0492
  # use core::sync::atomic::AtomicU8;
  // 同上。
  let _: &_ = const { &AtomicU8::new(0) }; // ERROR not allowed
  ```

  ```rust
  # #![allow(static_mut_refs)]
  // 尽管此借用是可变的，但它不是对临时变量的借用，因此这是允许的。
  const C: &u8 = unsafe { static mut S: u8 = 0; &mut S }; // OK
  ```

  ```rust
  # use core::sync::atomic::AtomicU8;
  // 尽管此借用是对具有内部可变性的值的借用，但它不是对临时变量的借用，因此这是允许的。
  const C: &AtomicU8 = {
      static S: AtomicU8 = AtomicU8::new(0); &S // OK
  };
  ```

  ```rust
  # use core::sync::atomic::AtomicU8;
  // 这种对内部可变临时变量的共享借用是允许的，因为其作用域没有被延长。
  const C: () = { _ = &AtomicU8::new(0); }; // OK
  ```

  ```rust
  // 尽管借用是可变的，且临时变量因提升而存续到程序结束，但这是允许的，因为
  // 借用不在尾部位置，因此临时变量的作用域不会通过临时变量生命周期延长来延长。
  const C: () = { let _: &'static mut [u8] = &mut []; }; // OK
  //                                              ~~
  //                                     提升的临时变量。
  ```

  > [!NOTE]
  > 换句话说 —— 为了关注什么是允许的而不是什么是不允许的 —— 只有当被借用的 [位置表达式][place expression] 是 *瞬态的* 、 *间接的* 或 *静态的* 时，才允许在 [常量上下文][const context] 中对内部可变数据进行共享借用和可变借用。
  >
  > 如果位置表达式是当前常量上下文的局部变量，或者是临时作用域包含在当前常量上下文中的表达式，则该位置表达式是 *瞬态的* 。
  >
  > ```rust
  > // 借用是对初始化器局部变量的借用，因此此位置表达式是瞬态的。
  > const C: () = { let mut x = 0; _ = &mut x; };
  > ```
  >
  > ```rust
  > // 借用是对作用域未延长的临时变量的借用，因此此位置表达式是瞬态的。
  > const C: () = { _ = &mut 0u8; };
  > ```
  >
  > ```rust
  > // 当临时变量被提升但没有延长生命周期时，其位置表达式仍被视为瞬态的。
  > const C: () = { let _: &'static mut [u8] = &mut []; };
  > ```
  >
  > 如果位置表达式是 [解引用表达式][dereference expression]，则该位置表达式是 *间接的* 。
  >
  > ```rust
  > const C: () = { _ = &mut *(&mut 0); };
  > ```
  >
  > 如果位置表达式是一个 `static` 项，则该位置表达式是 *静态的* 。
  >
  > ```rust
  > # #![allow(static_mut_refs)]
  > const C: &u8 = unsafe { static mut S: u8 = 0; &mut S };
  > ```

  > [!NOTE]
  > 这些规则的一个令人惊讶的后果是我们允许这样做：
  >
  > ```rust
  > const C: &[u8] = { let x: &mut [u8] = &mut []; x }; // OK
  > //                                    ~~~~~~~
  > // 即使在可变借用之后，空数组也会被提升。
  > ```
  >
  > 但我们不允许类似的这段代码：
  >
  > ```rust,compile_fail,E0764
  > const C: &[u8] = &mut []; // ERROR
  > //               ~~~~~~~
  > //           尾部表达式。
  > ```
  >
  > 它们之间的区别在于，在第一种情况中，空数组被 [提升][promoted] 了，但它的作用域没有经历 [临时变量生命周期延长][temporary lifetime extension]，所以我们认为 [位置表达式][place expression] 是瞬态的（即使在提升之后该位置确实存续到程序结束）。在第二种情况中，空数组临时变量的作用域确实经历了生命周期延长，因此它因为是对生命周期延长的临时变量的可变借用（从而借用了非瞬态的位置表达式）而被拒绝。
  >
  > 这种效果令人惊讶，因为在这种情况下，临时变量生命周期延长导致可编译的代码比没有它时更少。
  >
  > 见 [issue #143129](https://github.com/rust-lang/rust/issues/143129) 了解更多细节。

r[const-eval.const-expr.deref]
* [解引用表达式][dereference expressions]。

  ```rust,no_run
  # use core::cell::UnsafeCell;
  const _: u8 = unsafe {
      let x: *mut u8 = &raw mut *&mut 0;
      //                        ^^^^^^^
      //             对可变引用的解引用。
      *x = 1; // 对可变指针的解引用。
      *(x as *const u8) // 对常量指针的解引用。
  };
  const _: u8 = unsafe {
      let x = &UnsafeCell::new(0);
      *x.get() = 1; // 对内部可变值的修改。
      *x.get()
  };
  ```

r[const-eval.const-expr.group]

* [分组][grouped] 表达式。

r[const-eval.const-expr.cast]
* [转换][cast] 表达式，除了
  * 指针到地址转换以及
  * 函数指针到地址转换。

r[const-eval.const-expr.const-fn]
* 调用 [常量函数][const functions] 和常量方法。

r[const-eval.const-expr.loop]
* [loop] 和 [while] 表达式。

r[const-eval.const-expr.if-match]
* [if] 和 [match] 表达式。

r[const-eval.const-context]
## 常量上下文
[const context]: #常量上下文

r[const-eval.const-context.general]
*常量上下文* 是以下之一：

r[const-eval.const-context.array-length]
* [数组类型长度表达式][Array type length expressions]

r[const-eval.const-context.repeat-length]
* [数组重复长度表达式][array expressions]

r[const-eval.const-context.init]
* 以下项的初始化器
  * [常量][constants]
  * [静态项][statics]
  * [枚举判别值][enum discriminants]

r[const-eval.const-context.generic]
* [常量泛型参数][const generic argument]

r[const-eval.const-context.block]
* [常量块][const block]

r[const-eval.const-context.outer-generics]
作为类型一部分使用的常量上下文（数组类型和重复长度表达式以及常量泛型参数）只能限制性地使用周围的泛型参数：此类表达式必须要么是单个裸常量泛型参数，要么是不使用任何泛型的任意表达式。

r[const-eval.const-fn]
## 常量函数

r[const-eval.const-fn.intro]
*常量函数* 是可以从常量上下文中调用的函数。它使用 `const` 限定符定义，并且还包括 [元组结构体][tuple struct] 和 [元组枚举变体][tuple enum variant] 构造函数。

> [!EXAMPLE]
> ```rust
> const fn square(x: i32) -> i32 { x * x }
>
> const VALUE: i32 = square(12);
> square(12);
> ```

r[const-eval.const-fn.const-context]
当从常量上下文中调用时，常量函数由编译器在编译时解释。这种解释发生在编译 target 的环境中，而不是宿主环境。因此，如果你针对 `32` 位系统进行编译，`usize` 就是 `32` 位，而不管你是在 `64` 位还是 `32` 位系统上进行构建。

r[const-eval.const-fn.outside-context]
当在常量上下文之外调用常量函数时，它的行为与没有 `const` 限定符时相同。

r[const-eval.const-fn.body-restriction]
常量函数的主体只能使用 [常量表达式][constant expressions]。

r[const-eval.const-fn.async]
常量函数不允许是 [异步][async] 的。

r[const-eval.const-fn.type-restrictions]
常量函数的参数类型和返回类型仅限于那些与常量上下文兼容的类型。
<!-- TODO: 定义类型限制。 -->

[arithmetic]:           expressions/operator-expr.md#arithmetic-and-logical-binary-operators
[array expressions]:    expressions/array-expr.md
[array indexing]:       expressions/array-expr.md#array-and-slice-indexing-expressions
[array indexing]:       expressions/array-expr.md#array-and-slice-indexing-expressions
[array type length expressions]: types/array.md
[assignment expressions]: expressions/operator-expr.md#assignment-expressions
[async]:                items/functions.md#async-functions
[compound assignment expressions]: expressions/operator-expr.md#compound-assignment-expressions
[block expressions]:    expressions/block-expr.md
[borrow]:               expressions/operator-expr.md#borrow-operators
[cast]:                 expressions/operator-expr.md#type-cast-expressions
[closure expressions]:  expressions/closure-expr.md
[comparison]:           expressions/operator-expr.md#comparison-operators
[const block]:          expressions/block-expr.md#const-blocks
[const functions]:      items/functions.md#const-functions
[const generic argument]: items/generics.md#const-generics
[const generic parameters]: items/generics.md#const-generics
[constant expressions]: #constant-expressions
[constants]:            items/constant-items.md
[Const parameters]:     items/generics.md
[dereference expression]: expr.deref
[dereference expressions]: expr.deref
[destructors]:          destructors.md
[enum discriminants]:   items/enumerations.md#discriminants
[expression statements]: statements.md#expression-statements
[expressions]:          expressions.md
[`extern` statics]:     items/external-blocks.md#statics
[field]:                expressions/field-expr.md
[functions]:            items/functions.md
[grouped]:              expressions/grouped-expr.md
[interior mutability]:  interior-mutability.md
[if]:                   expressions/if-expr.md#if-expressions
[lazy boolean]:         expressions/operator-expr.md#lazy-boolean-operators
[let statements]:       statements.md#let-statements
[literals]:             expressions/literal-expr.md
[logical]:              expressions/operator-expr.md#arithmetic-and-logical-binary-operators
[loop]:                 expressions/loop-expr.md#infinite-loops
[match]:                expressions/match-expr.md
[negation]:             expressions/operator-expr.md#negation-operators
[overflow]:             expressions/operator-expr.md#overflow
[paths]:                expressions/path-expr.md
[patterns]:             patterns.md
[place expression]:     expr.place-value.place-memory-location
[promoted expression]:  destructors.md#constant-promotion
[promoted]:             destructors.md#constant-promotion
[range expressions]:    expressions/range-expr.md
[slice]:                types/slice.md
[statics]:              items/static-items.md
[Struct expressions]:   expressions/struct-expr.md
[temporary lifetime extension]: destructors.scope.lifetime-extension
[tuple enum variant]:   items/enumerations.md
[tuple expressions]:    expressions/tuple-expr.md
[tuple struct]:         items/structs.md
[while]:                expressions/loop-expr.md#predicate-loops
