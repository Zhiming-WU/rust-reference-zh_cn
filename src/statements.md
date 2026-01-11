r[statement]
# 语句

r[statement.syntax]
```grammar,语句
Statement ->
      `;`
    | Item
    | LetStatement
    | ExpressionStatement
    | OuterAttribute* MacroInvocationSemi
```

r[statement.intro]
*语句 (statement)* 是 [块][block] 的组成部分，而块又是外部 [表达式][expression] 或 [函数][function] 的组成部分。

r[statement.kind]
Rust 有两种语句： [声明语句](#declaration-statements) 和 [表达式语句](#expression-statements)。

r[statement.decl]
## 声明语句

*声明语句* 是指在封闭的语句块中引入一个或多个 *名称* 的语句。声明的名称可能表示新变量或新 [项][item]。

两种声明语句分别是项声明和 `let` 语句。

r[statement.item]
### 项声明

r[statement.item.intro]
*项声明语句* 的 语法格式 与 [模块][module] 中的 [项声明][item] 相同。

r[statement.item.scope]
在语句块中声明项会将其 [作用域][scope] 限制在包含该语句的块中。该项及其可能声明的任何子项都不会被赋予 [规范路径][canonical path]。

r[statement.item.associated-scope]
此规则的例外是，只要项以及（如果适用） 特型 是可访问的，由 [实现][implementations] 定义的关联项在外部作用域中仍然是可访问的。在其他方面的含义与在模块内部声明该项相同。

r[statement.item.outer-generics]
不会隐式捕获包含函数的 泛型 参数、参数和局部变量。例如， `inner` 无法访问 `outer_var`。

```rust
fn outer() {
  let outer_var = true;

  fn inner() { /* outer_var 在此处不在作用域内 */ }

  inner();
}
```

r[statement.let]
### `let`语句

r[statement.let.syntax]
```grammar,语句
LetStatement ->
    OuterAttribute* `let` PatternNoTopAlt ( `:` Type )?
    (
          `=` Expression
        | `=` Expression _except [LazyBooleanExpression] or end with a `}`_ `else` BlockExpression
    )? `;`
```

r[statement.let.intro]
* `let` 语句* 通过 [模式][pattern] 引入一组新 [变量][variables]。模式后面可以跟一个可选的类型标注，然后要么结束，要么后面跟一个初始化表达式以及一个可选的 `else` 块。

r[statement.let.inference]
如果未给出类型标注，编译器将推断类型；如果没有足够的类型信息进行明确推断，则报错。

r[statement.let.scope]
由变量声明引入的任何变量从声明点到封闭块作用域结束都是可见的，除非它们被另一个变量声明遮蔽。

r[statement.let.constraint]
如果没有 `else` 块，模式必须是不可驳回的。如果有 `else` 块，模式可以是可驳回的。

r[statement.let.behavior]
如果模式不匹配（这要求它是可驳回的），则执行 `else` 块。 `else` 块必须始终发散（求值为 [never 类型][never type]）。

```rust
let (mut v, w) = (vec![1, 2, 3], 42); // 绑定可以是 mut 或 const
let Some(t) = v.pop() else { // 可驳回模式需要 else 块
    panic!(); // else 块必须发散
};
let [u, v] = [v[0], v[1]] else { // 此模式是不可驳回的，因此编译器
                                 // 会发出 lint，因为 else 块是多余的。
    panic!();
};
```

r[statement.expr]
## 表达式语句

r[statement.expr.syntax]
```grammar,语句
ExpressionStatement ->
      ExpressionWithoutBlock `;`
    | ExpressionWithBlock `;`?
```

r[statement.expr.intro]
*表达式语句* 是指对 [表达式][expression] 求值并忽略其结果的语句。通常，表达式语句的目的是触发其表达式求值的副作用。

r[statement.expr.restriction-semicolon]
在允许使用语句的上下文中，仅由 [块表达式][block] 或控制流表达式组成的表达式可以省略末尾的分号。这可能会导致将其解析为独立语句还是解析为另一个表达式的一部分之间产生歧义；在这种情况下，它被解析为语句。

r[statement.expr.constraint-block]
当 [ExpressionWithBlock] 表达式用作语句时，其类型必须是单元类型。

```rust
# let mut v = vec![1, 2, 3];
v.pop();          // 忽略 pop 返回的元素
if v.is_empty() {
    v.push(5);
} else {
    v.remove(0);
}                 // 分号可以省略。
[1];              // 独立的表达式语句，不是索引表达式。
```

当省略末尾分号时，结果必须是类型 `()`。

```rust
// 错误：块的类型是 i32，而不是 ()
// Error: expected `()` because of default return type
// if true {
//   1
// }

// 正确：块的类型是 i32
if true {
  1
} else {
  2
};
```

r[statement.attribute]
## 语句上的属性

语句接受 [外部属性][outer attributes]。在语句上有意义的属性是 [`cfg`] 和 [lint 检查属性][the lint check attributes]。

[block]: expressions/block-expr.md
[expression]: expressions.md
[function]: items/functions.md
[item]: items.md
[module]: items/modules.md
[never type]: types/never.md
[canonical path]: paths.md#canonical-paths
[implementations]: items/implementations.md
[variables]: variables.md
[outer attributes]: attributes.md
[`cfg`]: conditional-compilation.md
[the lint check attributes]: attributes/diagnostics.md#lint-check-attributes
[pattern]: patterns.md
[scope]: names/scopes.md
