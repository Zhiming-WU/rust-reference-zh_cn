r[expr.match]
# `match`表达式

r[expr.match.syntax]
```grammar,expressions
MatchExpression ->
    `match` Scrutinee `{`
        InnerAttribute*
        MatchArms?
    `}`

Scrutinee -> Expression _except [StructExpression]_

MatchArms ->
    ( MatchArm `=>` ( ExpressionWithoutBlock `,` | ExpressionWithBlock `,`? ) )*
    MatchArm `=>` Expression `,`?

MatchArm -> OuterAttribute* Pattern MatchArmGuard?

MatchArmGuard -> `if` Expression
```
<!-- TODO: The exception above isn't accurate, see https://github.com/rust-lang/reference/issues/569 -->

r[expr.match.intro]
一个  *`match` 表达式* 根据模式进行分支。
发生的精确匹配形式取决于 [模式][pattern]。

r[expr.match.scrutinee]
一个 `match` 表达式有一个  *[待匹配值][scrutinee] 表达式* ，它是要与模式进行比较的值。

r[expr.match.scrutinee-constraint]
待匹配值表达式和模式必须具有相同的类型。

r[expr.match.scrutinee-behavior]
`match` 的行为根据待匹配值表达式是 [位置表达式或值表达式][place expression] 而有所不同。

r[expr.match.scrutinee-value]
如果待匹配值表达式是一个 [值表达式][value expression]，它首先被求值为一个临时位置，并将结果值按顺序与各个臂中的模式进行比较，直到找到匹配项。
第一个具有匹配模式的臂被选为 `match` 的分支目标，由该模式绑定的任何变量都会赋值给该臂所在块中的局部变量，然后控制流进入该块。

r[expr.match.scrutinee-place]
当待匹配值表达式是一个 [位置表达式][place expression] 时，匹配不会分配临时位置；
然而，按值绑定可能会从内存位置进行复制或移动。
在可能的情况下，首选对位置表达式进行匹配，因为这些匹配的生命周期继承自位置表达式的生命周期，而不是被限制在匹配内部。

`match` 表达式的一个示例：

```rust
let x = 1;

match x {
    1 => println!("one"),
    2 => println!("two"),
    3 => println!("three"),
    4 => println!("four"),
    5 => println!("five"),
    _ => println!("something else"),
}
```

r[expr.match.pattern-vars]
在模式内绑定的变量的作用域仅限于匹配守卫和该臂的表达式。

r[expr.match.pattern-var-binding]
 [绑定模式][binding mode] （移动、复制或引用）取决于模式。

r[expr.match.or-pattern]
多个匹配模式可以用 `|` 运算符连接。
每个模式将按从左到右的顺序进行测试，直到找到成功的匹配。

```rust
let x = 9;
let message = match x {
    0 | 1  => "not many",
    2 ..= 9 => "a few",
    _      => "lots"
};

assert_eq!(message, "a few");

// 演示模式匹配顺序。
struct S(i32, i32);

match S(1, 2) {
    S(z @ 1, _) | S(_, z @ 2) => assert_eq!(z, 1),
    _ => panic!(),
}
```

> [!NOTE]
> `2..=9` 是一个 [范围模式][Range Pattern] ，而不是一个 [范围表达式][Range Expression]。因此，只有范围模式支持的那些类型的范围才能在匹配臂中使用。

r[expr.match.or-patterns-restriction]
在每个由 `|` 分隔的模式中的每个绑定必须出现在该臂的所有模式中。

r[expr.match.binding-restriction]
具有相同名称的每个绑定必须具有相同的类型，并具有相同的绑定模式。

r[expr.match.guard]
## 匹配臂守卫

r[expr.match.guard.intro]
匹配臂可以接受  *匹配臂守卫*  以进一步细化匹配情况的准则。

r[expr.match.guard.type]
模式守卫出现在模式之后，由 `if` 关键字后跟一个 `bool` 类型的表达式组成。

r[expr.match.guard.behavior]
当模式匹配成功时，将执行模式守卫表达式。
如果表达式求值为真，则模式匹配成功。

r[expr.match.guard.next]
否则，将测试下一个模式，包括同一臂中由 `|` 运算符连接的其他匹配。

```rust
# let maybe_digit = Some(0);
# fn process_digit(i: i32) { }
# fn process_other(i: i32) { }
let message = match maybe_digit {
    Some(x) if x < 10 => process_digit(x),
    Some(x) => process_other(x),
    None => panic!(),
};
```

> [!NOTE]
> 使用 `|` 运算符进行多次匹配可能会导致模式守卫及其产生的副作用执行多次。例如：
>
> ```rust
> # use std::cell::Cell;
> let i : Cell<i32> = Cell::new(0);
> match 1 {
>     1 | _ if { i.set(i.get() + 1); false } => {}
>     _ => {}
> }
> assert_eq!(i.get(), 2);
> ```

r[expr.match.guard.bound-variables]
模式守卫可以引用它们所跟随的模式中绑定的变量。

r[expr.match.guard.shared-ref]
在求值守卫之前，会对变量匹配的待匹配值部分进行共享引用。
在求值守卫期间，访问该变量时将使用此共享引用。

r[expr.match.guard.value]
只有当守卫求值为真时，值才会从待匹配值移动或复制到变量中。
这允许在守卫内部使用共享借用，而不会在守卫匹配失败的情况下从待匹配值中移出。

r[expr.match.guard.no-mutation]
此外，通过在求值守卫时持有共享引用，还可以防止在守卫内部进行修改。

r[expr.match.attributes]
## 匹配臂上的属性

r[expr.match.attributes.outer]
匹配臂上允许使用外部属性。
在匹配臂上有意义的属性只有 [`cfg`] 和 [lint 检查属性][lint check attributes]。

r[expr.match.attributes.inner]
 [内部属性][Inner attributes] 被允许直接放在匹配表达式的左大括号之后，其所在的表达式上下文与 [块表达式上的属性][attributes on block expressions] 相同。

[`cfg`]: ../conditional-compilation.md
[attributes on block expressions]: block-expr.md#attributes-on-block-expressions
[binding mode]: ../patterns.md#binding-modes
[Inner attributes]: ../attributes.md
[lint check attributes]: ../attributes/diagnostics.md#lint-check-attributes
[pattern]: ../patterns.md
[place expression]: ../expressions.md#位置表达式和值表达式
[Range Expression]: range-expr.md
[Range Pattern]: ../patterns.md#range-patterns
[scrutinee]: ../glossary.md#scrutinee
[value expression]: ../expressions.md#位置表达式和值表达式
