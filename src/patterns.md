r[patterns]
# 模式

r[patterns.syntax]
```grammar,模式
Pattern -> `|`? PatternNoTopAlt  ( `|` PatternNoTopAlt )*

PatternNoTopAlt ->
      PatternWithoutRange
    | RangePattern

PatternWithoutRange ->
      LiteralPattern
    | IdentifierPattern
    | WildcardPattern
    | RestPattern
    | ReferencePattern
    | StructPattern
    | TupleStructPattern
    | TuplePattern
    | GroupedPattern
    | SlicePattern
    | PathPattern
    | MacroInvocation
```

r[patterns.intro]
模式用于根据结构匹配值，并可选地将变量绑定到这些结构内部的值。
它们也用于变量声明以及函数和闭包的参数。

下面示例中的模式做了四件事：

* 测试 `person` 是否有名为 `car` 的字段且填充了内容。
* 测试该人的 `age` 字段是否在 13 到 19 之间，并将其值绑定到 `person_age` 变量。
* 将指向 `name` 字段的引用绑定到变量 `person_name`。
* 忽略 `person` 的其余字段。
  其余字段可以具有任何值，且不绑定到任何变量。

```rust
# struct Car;
# struct Computer;
# struct Person {
#     name: String,
#     car: Option<Car>,
#     computer: Option<Computer>,
#     age: u8,
# }
# let person = Person {
#     name: String::from("John"),
#     car: Some(Car),
#     computer: None,
#     age: 15,
# };
if let
    Person {
        car: Some(_),
        age: person_age @ 13..=19,
        name: ref person_name,
        ..
    } = person
{
    println!("{} has a car and is {} years old.", person_name, person_age);
}
```

r[patterns.usage]
模式用于：

r[patterns.let]
* [`let` 声明](statements.md#let-statements)

r[patterns.param]
* [函数](items/functions.md) 和 [闭包](expressions/closure-expr.md) 参数

r[patterns.match]
* [`match` 表达式](expressions/match-expr.md)

r[patterns.if-let]
* [`if let` 表达式](expressions/if-expr.md)

r[patterns.while-let]
* [`while let` 表达式](expressions/loop-expr.md#while-let-patterns)

r[patterns.for]
* [`for` 表达式](expressions/loop-expr.md#iterator-loops)

r[patterns.destructure]
## 解构

r[patterns.destructure.intro]
模式可以用来 *解构* [结构体][structs] 、 [枚举][enums] 和 [元组][tuples] 。
解构将一个值分解为其组成部分。
使用的 语法格式 与创建此类值时几乎相同。

r[patterns.destructure.wildcard]
在 [被匹配][scrutinee] 表达式具有 结构体 、 枚举 或 元组 类型的模式中， [通配符模式](#wildcard-pattern) (`_`) 代表 *单个* 数据字段，而 [等等](#grammar-StructPatternEtCetera) 或 [剩余模式][patterns.rest] (`..`) 代表特定变体的 *所有* 剩余字段。

r[patterns.destructure.named-field-shorthand]
当解构具有命名（但未编号）字段的数据结构时，允许将 `fieldname` 简写为 `fieldname: fieldname` 。

```rust
# enum Message {
#     Quit,
#     WriteString(String),
#     Move { x: i32, y: i32 },
#     ChangeColor(u8, u8, u8),
# }
# let message = Message::Quit;
match message {
    Message::Quit => println!("Quit"),
    Message::WriteString(write) => println!("{}", &write),
    Message::Move{ x, y: 0 } => println!("move {} horizontally", x),
    Message::Move{ .. } => println!("other move"),
    Message::ChangeColor { 0: red, 1: green, 2: _ } => {
        println!("color change, red: {}, green: {}", red, green);
    }
};
```

r[patterns.refutable]
## 可反驳性

当一个模式有可能不与其匹配的值相匹配时，该模式被称为是 *可反驳的* 。
另一方面， *不可反驳的* 模式总是匹配它们所针对的值。
示例：

```rust
let (x, y) = (1, 2);               // "(x, y)" 是一个不可反驳的模式

if let (a, 3) = (1, 2) {           // "(a, 3)" 是可反驳的，且不会匹配
    panic!("Shouldn't reach here");
} else if let (a, 4) = (3, 4) {    // "(a, 4)" 是可反驳的，且会匹配
    println!("Matched ({}, 4)", a);
}
```

r[patterns.literal]
## 字面量模式

r[patterns.literal.syntax]
```grammar,模式
LiteralPattern -> `-`? LiteralExpression
```

r[patterns.literal.intro]
 _字面量模式_ 匹配的值与字面量创建的值完全相同。由于负数不是 [字面量][literals] ，模式中的字面量可以前缀一个可选的减号，其作用类似于取反运算符。

> [!WARNING]
> C string and raw C string literals are accepted in literal patterns, but `&CStr` doesn't implement structural equality (`#[derive(Eq, PartialEq)]`) and therefore any such `match` on a `&CStr` will be rejected with a type error.

r[patterns.literal.refutable]
字面量模式始终是可反驳的。

示例：

```rust
for i in -2..5 {
    match i {
        -1 => println!("It's minus one"),
        1 => println!("It's a one"),
        2|4 => println!("It's either a two or a four"),
        _ => println!("Matched none of the arms"),
    }
}
```

r[patterns.ident]
## 标识符模式

r[patterns.ident.syntax]
```grammar,模式
IdentifierPattern -> `ref`? `mut`? IDENTIFIER ( `@` PatternNoTopAlt )?
```

r[patterns.ident.intro]
标识符模式将它们匹配的值绑定到 [值命名空间][value namespace] 中的变量。

r[patterns.ident.unique]
标识符在模式中必须是唯一的。

r[patterns.ident.scope]
该变量将遮蔽作用域中同名的任何变量。
新绑定的 [作用域][scope] 取决于使用模式的上下文（例如 `let` 绑定或 `match` 臂）。

r[patterns.ident.bare]
仅由标识符（可能带有 `mut` ）组成的模式匹配任何值并将其绑定到该标识符。
这是变量声明以及函数和闭包参数中最常用的模式。

```rust
let mut variable = 10;
fn sum(x: i32, y: i32) -> i32 {
#    x + y
# }
```

r[patterns.ident.scrutinized]
要将模式匹配的值绑定到变量，请使用 `variable @ subpattern` 语法格式。
例如，下面将值 2 绑定到 `e` （不是整个范围：这里的范围是一个范围子模式）。

```rust
let x = 2;

match x {
    e @ 1 ..= 5 => println!("got a range element {}", e),
    _ => println!("anything"),
}
```

r[patterns.ident.move]
默认情况下，标识符模式根据被匹配值是否实现了 [Copy] ，将变量绑定到被匹配值的副本或从中移动。

r[patterns.ident.ref]
这可以通过使用 `ref` 关键字更改为绑定到引用，或者使用 `ref mut` 更改为可变引用。例如：

```rust
# let a = Some(10);
match a {
    None => (),
    Some(value) => (),
}

match a {
    None => (),
    Some(ref value) => (),
}
```

在第一个 match 表达式中，值被复制（或移动）。
在第二个 match 中，指向相同内存位置的引用被绑定到变量值。
需要此语法格式是因为在解构子模式中， `&` 运算符不能应用于值的字段。
例如，以下内容无效：

```rust,compile_fail
# struct Person {
#    name: String,
#    age: u8,
# }
# let value = Person { name: String::from("John"), age: 23 };
if let Person { name: &person_name, age: 18..=150 } = value { }
```

为了使其有效，请编写以下内容：

```rust
# struct Person {
#    name: String,
#    age: u8,
# }
# let value = Person { name: String::from("John"), age: 23 };
if let Person { name: ref person_name, age: 18..=150 } = value { }
```

r[patterns.ident.ref-ignored]
因此， `ref` 不是被匹配的对象。
它的目的完全是为了使匹配的绑定成为引用，而不是潜在地复制或移动匹配的内容。

r[patterns.ident.precedent]
[路径模式](#path-patterns) 优先于标识符模式。

> [!NOTE]
> When a pattern is a single-segment identifier, the grammar is ambiguous whether it means an [IdentifierPattern] or a [PathPattern]. This ambiguity can only be resolved after [name resolution].
>
> ```rust
> const EXPECTED_VALUE: u8 = 42;
> //    ^^^^^^^^^^^^^^ 此常量在作用域内会影响下面模式的处理方式。
> //                   patterns below are treated.
>
> fn check_value(x: u8) -> Result<u8, u8> {
>     match x {
>         EXPECTED_VALUE => Ok(x),
>     //  ^^^^^^^^^^^^^^ 被解析为解析为常量 `42` 的 `PathPattern` 。
>     //                 the constant `42`.
>         other_value => Err(x),
>     //  ^^^^^^^^^^^ 被解析为 `IdentifierPattern` 。
>     }
> }
>
> // If `EXPECTED_VALUE` were treated as an `IdentifierPattern` above,
> // that pattern would always match, making the function always return
> // `Ok(_) regardless of the input.
> assert_eq!(check_value(42), Ok(42));
> assert_eq!(check_value(43), Err(43));
> ```

r[patterns.ident.constraint]
如果指定了 `ref` 或 `ref mut` 且标识符遮蔽了常量，则这是一个错误。

r[patterns.ident.refutable]
如果 `@` 子模式是不可反驳的或者没有指定子模式，则标识符模式是不可反驳的。

r[patterns.ident.binding]
### 绑定模式

r[patterns.ident.binding.intro]
为了提供更好的易用性，模式以不同的 *绑定模式* 运行，以便更轻松地将引用绑定到值。
当非引用模式匹配引用值时，它将自动被视为 `ref` 或 `ref mut` 绑定。
示例：

```rust
let x: &Option<i32> = &Some(3);
if let Some(y) = x {
    // y 被转换为 `ref y` 且其类型为 &i32
}
```

r[patterns.ident.binding.non-reference]
 *非引用模式* 包括除绑定、 [通配符模式](#wildcard-pattern) (`_`) 、引用类型的 [`const` 模式](#path-patterns) 以及 [引用模式](#reference-patterns) 之外的所有模式。

r[patterns.ident.binding.default-mode]
如果绑定模式没有显式具有 `ref` 、 `ref mut` 或 `mut` ，那么它使用 *默认绑定模式* 来确定变量如何绑定。

r[patterns.ident.binding.move]
默认绑定模式以使用移动语义的 "move" 模式开始。

r[patterns.ident.binding.top-down]
匹配模式时，编译器从模式外部开始并向内工作。

r[patterns.ident.binding.auto-deref]
每当使用非引用模式匹配引用时，它会自动解引用该值并更新默认绑定模式。

r[patterns.ident.binding.ref]
引用将默认绑定模式设置为 `ref` 。

r[patterns.ident.binding.ref-mut]
可变引用将模式设置为 `ref mut` ，除非模式已经是 `ref` ，在这种情况下它保持为 `ref` 。

r[patterns.ident.binding.nested-references]
如果自动解引用的值仍然是引用，则对其进行解引用并重复此过程。

r[patterns.ident.binding.mode-limitations-binding]
只有当默认绑定模式为 "move" 时，绑定模式才可以显式指定 `ref` 或 `ref mut` 绑定模式，或者使用 `mut` 指定可变性。例如，这些是不被接受的：

```rust,edition2024,compile_fail
let [mut x] = &[()]; //~ ERROR
let [ref x] = &[()]; //~ ERROR
let [ref mut x] = &mut [()]; //~ ERROR
```

r[patterns.ident.binding.mode-limitations.edition2024]
> [!EDITION-2024]
> Before the 2024 edition, bindings could explicitly specify a `ref` or `ref mut` binding mode even when the default binding mode was not "move", and they could specify mutability on such bindings with `mut`. In these editions, specifying `mut` on a binding set the binding mode to "move" regardless of the current default binding mode.

r[patterns.ident.binding.mode-limitations-reference]
类似地，引用模式仅在默认绑定模式为 "move" 时出现。例如，这是不被接受的：

```rust,edition2024,compile_fail
let [&x] = &[&()]; //~ ERROR
```

r[patterns.ident.binding.mode-limitations-reference.edition2024]
> [!EDITION-2024]
> Before the 2024 edition, reference patterns could appear even when the default binding mode was not "move", and had both the effect of matching against the scrutinee and of causing the default binding mode to be reset to "move".

r[patterns.ident.binding.mixed]
移动绑定和引用绑定可以混合在同一个模式中。
这样做会导致绑定到的对象发生部分移动，之后该对象将无法使用。
这仅在类型不可复制时适用。

在下面的示例中， `name` 从 `person` 中移出。
尝试将 `person` 作为一个整体使用或使用 `person.name` 会因为 *部分移动* 而导致错误。

示例：

```rust
# struct Person {
#    name: String,
#    age: u8,
# }
# let person = Person{ name: String::from("John"), age: 23 };
// `name` 从 person 中移出且 `age` 被引用
let Person { name, ref age } = person;
```

r[patterns.wildcard]
## 通配符模式

r[patterns.wildcard.syntax]
```grammar,模式
WildcardPattern -> `_`
```

r[patterns.wildcard.intro]
 _通配符模式_ （下划线符号）匹配任何值。
它用于在值无关紧要时忽略它们。

r[patterns.wildcard.struct-matcher]
在其他模式内部，它匹配单个数据字段（与匹配剩余字段的 `..` 相反）。

r[patterns.wildcard.no-binding]
与标识符模式不同，它不会复制、移动或借用它匹配的值。

示例：

```rust
# let x = 20;
let (a, _) = (10, x);   // x 始终被 _ 匹配
# assert_eq!(a, 10);

// 忽略一个函数/闭包参数
let real_part = |a: f64, _: f64| { a };

// 忽略 结构体 中的一个字段
# struct RGBA {
#    r: f32,
#    g: f32,
#    b: f32,
#    a: f32,
# }
# let color = RGBA{r: 0.4, g: 0.1, b: 0.9, a: 0.5};
let RGBA{r: red, g: green, b: blue, a: _} = color;
# assert_eq!(color.r, red);
# assert_eq!(color.g, green);
# assert_eq!(color.b, blue);

// 接受任何 Some，无论其值为何
# let x = Some(10);
if let Some(_) = x {}
```

r[patterns.wildcard.refutable]
通配符模式始终是不可反驳的。

r[patterns.rest]
## 剩余模式

r[patterns.rest.syntax]
```grammar,模式
RestPattern -> `..`
```

r[patterns.rest.intro]
 _剩余模式_ （ `..` 词法单元）充当变长模式，匹配前后尚未匹配的零个或多个元素。

r[patterns.rest.allowed-patterns]
它只能用于 [元组](#tuple-patterns) 、 [元组结构体](#tuple-struct-patterns) 和 [切片](#slice-patterns) 模式，并且只能作为这些模式中的元素之一出现一次。
仅对于 [切片模式](#slice-patterns) ，它也允许在 [标识符模式](#identifier-patterns) 中使用。

r[patterns.rest.refutable]
剩余模式始终是不可反驳的。

示例：

```rust
# let words = vec!["a", "b", "c"];
# let slice = &words[..];
match slice {
    [] => println!("slice is empty"),
    [one] => println!("single element {}", one),
    [head, tail @ ..] => println!("head={} tail={:?}", head, tail),
}

match slice {
    // 忽略除最后一个元素（必须是 "!"）之外的所有内容。
    [.., "!"] => println!("!!!"),

    // `start` 是除最后一个元素（必须是 "z"）之外的所有内容的切片。
    [start @ .., "z"] => println!("starts with: {:?}", start),

    // `end` 是除第一个元素（必须是 "a"）之外的所有内容的切片。
    ["a", end @ ..] => println!("ends with: {:?}", end),

    // 'whole' 是整个切片且 `last` 是最后一个元素
    whole @ [.., last] => println!("the last element of {:?} is {}", whole, last),

    rest => println!("{:?}", rest),
}

if let [.., penultimate, _] = slice {
    println!("next to last is {}", penultimate);
}

# let tuple = (1, 2, 3, 4, 5);
// 剩余模式也可用于元组和元组
// 结构体模式。
match tuple {
    (1, .., y, z) => println!("y={} z={}", y, z),
    (.., 5) => println!("tail must be 5"),
    (..) => println!("matches everything else"),
}
```

r[patterns.range]
## 范围模式

r[patterns.range.syntax]
```grammar,模式
RangePattern ->
      RangeExclusivePattern
    | RangeInclusivePattern
    | RangeFromPattern
    | RangeToExclusivePattern
    | RangeToInclusivePattern
    | ObsoleteRangePattern[^obsolete-range-edition]

RangeExclusivePattern ->
      RangePatternBound `..` RangePatternBound

RangeInclusivePattern ->
      RangePatternBound `..=` RangePatternBound

RangeFromPattern ->
      RangePatternBound `..`

RangeToExclusivePattern ->
      `..` RangePatternBound

RangeToInclusivePattern ->
      `..=` RangePatternBound

ObsoleteRangePattern ->
    RangePatternBound `...` RangePatternBound

RangePatternBound ->
      LiteralPattern
    | PathExpression
```

[^obsolete-range-edition]:  [ObsoleteRangePattern]  语法格式 在 2021 版次中已被移除。

r[patterns.range.intro]
 *范围模式* 匹配由其边界定义的范围内的标量值。
它们包含一个 *符号* （ `..` 或 `..=` ）和一侧或两侧的边界。

符号左侧的边界称为 *下边界* 。
右侧的边界称为 *上边界* 。

r[patterns.range.exclusive]
 *排他性范围模式* 匹配从下边界到（但不包括）上边界的所有值。
它写为下边界，后跟 `..` ，再后跟上边界。

例如，模式 `'m'..'p'` 将仅匹配 `'m'` 、 `'n'` 和 `'o'` ，特别 **不** 包括 `'p'` 。

r[patterns.range.inclusive]
 *包含性范围模式* 匹配从下边界到并包括上边界的所有值。
它写为下边界，后跟 `..=` ，再后跟上边界。

例如，模式 `'m'..='p'` 将仅匹配值 `'m'` 、 `'n'` 、 `'o'` 和 `'p'` 。

r[patterns.range.from]
 *起点范围模式* 匹配大于或等于下边界的所有值。
它写为下边界，后跟 `..` 。

例如， `1..` 将匹配任何大于或等于 1 的整数，如 1、9 或 9001，或者 9007199254740991（如果它具有适当的大小），但不匹配 0，对于有符号整数也不匹配负数。

r[patterns.range.to-exclusive]
 *终点排他性范围模式* 匹配所有小于上边界的值。
它写为 `..` 后跟上边界。

例如， `..10` 将匹配任何小于 10 的整数，如 9、1、0，对于有符号整数类型，还匹配所有负值。

r[patterns.range.to-inclusive]
 *终点包含性范围模式* 匹配所有小于或等于上边界的值。
它写为 `..=` 后跟上边界。

例如， `..=10` 将匹配任何小于或等于 10 的整数，如 10、1、0，对于有符号整数类型，还匹配所有负值。

r[patterns.range.constraint-nonempty]
范围模式必须非空；它必须至少跨越其类型可能值集中的一个值。换句话说：

* 在 `a..=b` 中，必须满足 a &le; b。例如，出现范围模式 `10..=0` 是一个错误，但允许 `10..=10` 。
* 在 `a..b` 中，必须满足 a &lt; b。例如，出现范围模式 `10..0` 或 `10..10` 是一个错误。
* 在 `..b` 中，b 必须不是其类型的最小值。例如，出现范围模式 `..-128i8` 或 `..f64::NEG_INFINITY` 是一个错误。

r[patterns.range.bound]
边界写为以下之一：

* 字符、字节、整数或浮点数字面量。
* 一个 `-` 后跟整数或浮点数字面量。
* 一个 [路径][path] 。

> [!NOTE]
>
> We syntactically accept more than this for a *[RangePatternBound]*. We later reject the other things semantically.

r[patterns.range.constraint-bound-path]
如果边界写为路径，则在宏解析后，该路径必须解析为 `char` 类型、整数类型或浮点类型的常量项。

r[patterns.range.type]
范围模式匹配其上边界和下边界的类型，这两者必须是同一类型。

r[patterns.range.path-value]
如果边界是一个 [路径][path] ，则边界匹配该类型，并具有路径解析到的 [常量][constant] 的值。

r[patterns.range.literal-value]
如果边界是一个字面量，则边界匹配该类型，并具有相应 [字面量表达式][literal expression] 的值。

r[patterns.range.negation]
如果边界是一个前面带有 `-` 的字面量，则边界匹配与相应 [字面量表达式][literal expression] 相同的类型，并具有对相应字面量表达式的值进行 [取反][negating] 后的值。

r[patterns.range.float-restriction]
对于浮点范围模式，常量不能是 `NaN` 。

示例：

```rust
# let c = 'f';
let valid_variable = match c {
    'a'..='z' => true,
    'A'..='Z' => true,
    'α'..='ω' => true,
    _ => false,
};

# let ph = 10;
println!("{}", match ph {
    0..7 => "acid",
    7 => "neutral",
    8..=14 => "base",
    _ => unreachable!(),
});

# let uint: u32 = 5;
match uint {
    0 => "zero!",
    1.. => "positive number!",
};

// 使用指向常量的路径：
# const TROPOSPHERE_MIN : u8 = 6;
# const TROPOSPHERE_MAX : u8 = 20;
#
# const STRATOSPHERE_MIN : u8 = TROPOSPHERE_MAX + 1;
# const STRATOSPHERE_MAX : u8 = 50;
#
# const MESOSPHERE_MIN : u8 = STRATOSPHERE_MAX + 1;
# const MESOSPHERE_MAX : u8 = 85;
#
# let altitude = 70;
#
println!("{}", match altitude {
    TROPOSPHERE_MIN..=TROPOSPHERE_MAX => "troposphere",
    STRATOSPHERE_MIN..=STRATOSPHERE_MAX => "stratosphere",
    MESOSPHERE_MIN..=MESOSPHERE_MAX => "mesosphere",
    _ => "outer space, maybe",
});

# pub mod binary {
#     pub const MEGA : u64 = 1024*1024;
#     pub const GIGA : u64 = 1024*1024*1024;
# }
# let n_items = 20_832_425;
# let bytes_per_item = 12;
if let size @ binary::MEGA..=binary::GIGA = n_items * bytes_per_item {
    println!("It fits and occupies {} bytes", size);
}

# trait MaxValue {
#     const MAX: u64;
# }
# impl MaxValue for u8 {
#     const MAX: u64 = (1 << 8) - 1;
# }
# impl MaxValue for u16 {
#     const MAX: u64 = (1 << 16) - 1;
# }
# impl MaxValue for u32 {
#     const MAX: u64 = (1 << 32) - 1;
# }
// 使用限定路径：
println!("{}", match 0xfacade {
    0 ..= <u8 as MaxValue>::MAX => "fits in a u8",
    0 ..= <u16 as MaxValue>::MAX => "fits in a u16",
    0 ..= <u32 as MaxValue>::MAX => "fits in a u32",
    _ => "too big",
});
```

r[patterns.range.refutable]
当固定宽度整数和 `char` 类型的范围模式跨越一个类型的所有可能值集时，它们是不可反驳的。
例如， `0u8..=255u8` 是不可反驳的。

r[patterns.range.refutable-integer]
整数类型的值范围是从其最小值到最大值的闭区间。

r[patterns.range.refutable-char]
 `char` 类型的值范围恰好是那些包含所有 Unicode 标量值的范围： `'\u{0000}'..='\u{D7FF}'` 和 `'\u{E000}'..='\u{10FFFF}'` 。

r[patterns.range.constraint-slice]
 [RangeFromPattern] 不能用作 [切片模式](#slice-patterns) 中子模式的顶层模式。
例如，模式 `[1.., _]` 不是一个有效的模式。

r[patterns.range.edition2021]
> [!EDITION-2021]
> Before the 2021 edition, range patterns with both a lower and upper bound may also be written using `...` in place of `..=`, with the same meaning.

r[patterns.ref]
## 引用模式

r[patterns.ref.syntax]
```grammar,模式
ReferencePattern -> (`&`|`&&`) `mut`? PatternWithoutRange
```

r[patterns.ref.intro]
引用模式对正在匹配的指针进行解引用，从而借用它们。

例如，在 `x: &i32` 上的这两个 match 是等价的：

```rust
let int_reference = &3;

let a = match *int_reference { 0 => "zero", _ => "some" };
let b = match int_reference { &0 => "zero", _ => "some" };

assert_eq!(a, b);
```

r[patterns.ref.ref-ref]
引用模式的语法产生式必须匹配 词法单元 `&&` 才能匹配引用的引用，因为它本身就是一个 词法单元 ，而不是两个 `&` 词法单元 。

r[patterns.ref.mut]
添加 `mut` 关键字会解引用一个可变引用。可变性必须与引用的可变性匹配。

r[patterns.ref.refutable]
引用模式始终是不可反驳的。

r[patterns.struct]
## 结构体模式

r[patterns.struct.syntax]
```grammar,模式
StructPattern ->
    PathInExpression `{`
        StructPatternElements?
    `}`

StructPatternElements ->
      StructPatternFields (`,` | `,` StructPatternEtCetera)?
    | StructPatternEtCetera

StructPatternFields ->
    StructPatternField (`,` StructPatternField)*

StructPatternField ->
    OuterAttribute*
    (
        TUPLE_INDEX `:` Pattern
      | IDENTIFIER `:` Pattern
      | `ref`? `mut`? IDENTIFIER
    )

StructPatternEtCetera -> `..`
```

r[patterns.struct.intro]
结构体模式匹配 结构体 、 枚举 和 联合体 值，这些值需满足其子模式定义的所有准则。
它们也用于 [解构](#destructuring) 一个 结构体 、 枚举 或 联合体 值。

r[patterns.struct.ignore-rest]
在 结构体 模式中，字段通过名称、索引（在元组结构体的情况下）引用，或者使用 `..` 忽略：

```rust
# struct Point {
#     x: u32,
#     y: u32,
# }
# let s = Point {x: 1, y: 1};
#
match s {
    Point {x: 10, y: 20} => (),
    Point {y: 10, x: 20} => (),    // 顺序无关紧要
    Point {x: 10, ..} => (),
    Point {..} => (),
}

# struct PointTuple (
#     u32,
#     u32,
# );
# let t = PointTuple(1, 2);
#
match t {
    PointTuple {0: 10, 1: 20} => (),
    PointTuple {1: 10, 0: 20} => (),   // 顺序无关紧要
    PointTuple {0: 10, ..} => (),
    PointTuple {..} => (),
}

# enum Message {
#     Quit,
#     Move { x: i32, y: i32 },
# }
# let m = Message::Quit;
#
match m {
    Message::Quit => (),
    Message::Move {x: 10, y: 20} => (),
    Message::Move {..} => (),
}
```

r[patterns.struct.constraint-struct]
如果不使用 `..` ，则用于匹配 结构体 的 结构体 模式需要指定所有字段：

```rust
# struct Struct {
#    a: i32,
#    b: char,
#    c: bool,
# }
# let mut struct_value = Struct{a: 10, b: 'X', c: false};
#
match struct_value {
    Struct{a: 10, b: 'X', c: false} => (),
    Struct{a: 10, b: 'X', ref c} => (),
    Struct{a: 10, b: 'X', ref mut c} => (),
    Struct{a: 10, b: 'X', c: _} => (),
    Struct{a: _, b: _, c: _} => (),
}
```

r[patterns.struct.constraint-union]
用于匹配 联合体 的 结构体 模式必须恰好指定一个字段（参见 [在联合体上进行模式匹配][Pattern matching on unions] ）。

r[patterns.struct.binding-shorthand]
 [IDENTIFIER] 语法格式匹配任何值，并将其绑定到与给定字段同名的变量。它是 `fieldname: fieldname` 的简写。 `ref` 和 `mut` 限定符可以包含在内，其行为如 [patterns.ident.ref] 中所述。

```rust
# struct Struct {
#    a: i32,
#    b: char,
#    c: bool,
# }
# let struct_value = Struct{a: 10, b: 'X', c: false};
#
let Struct { a, b, c } = struct_value;
```

r[patterns.struct.refutable]
如果 [PathInExpression] 解析为具有多个变体的枚举构造函数，或者其子模式之一是可反驳的，则 结构体 模式是可反驳的。

r[patterns.struct.namespace]
 结构体 模式根据从 [类型命名空间][type namespace] 中的 [PathInExpression] 解析出的 结构体 、 联合体 或枚举变体进行匹配。有关更多详细信息，请参见 [patterns.tuple-struct.namespace] 。

r[patterns.tuple-struct]
## 元组结构体模式

r[patterns.tuple-struct.syntax]
```grammar,模式
TupleStructPattern -> PathInExpression `(` TupleStructItems? `)`

TupleStructItems -> Pattern ( `,` Pattern )* `,`?
```

r[patterns.tuple-struct.intro]
元组结构体模式匹配元组结构体和枚举值，这些值需满足其子模式定义的所有准则。
它们也用于 [解构](#destructuring) 元组结构体或枚举值。

r[patterns.tuple-struct.refutable]
如果 [PathInExpression] 解析为具有多个变体的枚举构造函数，或者其子模式之一是可反驳的，则元组结构体模式是可反驳的。

r[patterns.tuple-struct.namespace]
元组结构体模式根据从 [值命名空间][value namespace] 中的 [PathInExpression] 解析出的元组结构体或 [类元组枚举变体][tuple-like enum variant] 进行匹配。

> [!NOTE]
> Conversely, a struct pattern for a tuple struct or [tuple-like enum variant], e.g. `S { 0: _ }`, matches against the tuple struct or variant whose constructor is resolved in the [type namespace].
>
> ```rust,no_run
> enum E1 { V(u16) }
> enum E2 { V(u32) }
>
> // 只从类型命名空间导入 `E1::V`。
> mod _0 {
>     const V: () = (); // 用于命名空间掩蔽。
>     pub(super) use super::E1::*;
> }
> use _0::*;
>
> // 只从值命名空间导入 `E2::V`。
> mod _1 {
>     struct V {} // 用于命名空间掩蔽。
>     pub(super) use super::E2::*;
> }
> use _1::*;
>
> fn f() {
>     // 此结构体模式根据在类型命名空间中
>     // 找到其构造函数的类元组枚举变体
>     // 进行匹配。
>     let V { 0: ..=u16::MAX } = (loop {}) else { loop {} };
>     // 此元组结构体模式根据在值命名空间中
>     // 找到其构造函数的类元组枚举变体
>     // 进行匹配。
>     let V(..=u32::MAX) = (loop {}) else { loop {} };
> }
> # // 在函数中使用 super 的奇怪行为所必需的。
> # fn main() {}
> ```
>
> The Lang team has made certain decisions, such as in [PR #138458], that raise questions about the desirability of using the value namespace in this way for patterns, as described in [PR #140593]. It might be prudent to not intentionally rely on this nuance in your code.

r[patterns.tuple]
## 元组模式

r[patterns.tuple.syntax]
```grammar,模式
TuplePattern -> `(` TuplePatternItems? `)`

TuplePatternItems ->
      Pattern `,`
    | RestPattern
    | Pattern (`,` Pattern)+ `,`?
```

r[patterns.tuple.intro]
元组模式匹配满足其子模式定义的所有准则的元组值。
它们也用于 [解构](#destructuring) 元组。

r[patterns.tuple.rest-syntax]
带有单个 [RestPattern] 的形式 `(..)` 是一种特殊形式，不需要逗号，并且匹配任何大小的元组。

r[patterns.tuple.refutable]
当元组模式的一个子模式是可反驳的时，该元组模式是可反驳的。

使用元组模式的示例：

```rust
let pair = (10, "ten");
let (a, b) = pair;

assert_eq!(a, 10);
assert_eq!(b, "ten");
```

r[patterns.paren]
## 分组模式

r[patterns.paren.syntax]
```grammar,模式
GroupedPattern -> `(` Pattern `)`
```

r[patterns.paren.intro]
将模式括在括号中可用于显式控制复合模式的优先级。
例如，紧邻范围模式的引用模式（如 `&0..=5` ）具有歧义且不被允许，但可以用括号表示。

```rust
let int_reference = &3;
match int_reference {
    &(0..=5) => (),
    _ => (),
}
```

r[patterns.slice]
## 切片模式

r[patterns.slice.syntax]
```grammar,模式
SlicePattern -> `[` SlicePatternItems? `]`

SlicePatternItems -> Pattern (`,` Pattern)* `,`?
```

r[patterns.slice.intro]
切片模式既可以匹配固定大小的数组，也可以匹配动态大小的切片。

```rust
// 固定大小
let arr = [1, 2, 3];
match arr {
    [1, _, _] => "starts with one",
    [a, b, c] => "starts with something else",
};
```
```rust
// 动态大小
let v = vec![1, 2, 3];
match v[..] {
    [a, b] => { /* 此臂不适用，因为长度不匹配 */ }
    [a, b, c] => { /* 此臂将适用 */ }
    _ => { /* 此通配符是必需的，因为长度在静态时是未知的 */ }
};
```

r[patterns.slice.refutable-array]
匹配数组时，只要每个元素都是不可反驳的，切片模式就是不可反驳的。

r[patterns.slice.refutable-slice]
匹配切片时，只有在具有单个 `..` [剩余模式][patterns.rest] 或以 `..` 剩余模式作为子模式的 [标识符模式](#identifier-patterns) 的形式下，它才是不可反驳的。

r[patterns.slice.restriction]
在切片内，没有下边界和上边界的范围模式必须括在括号中（如 `(a..)` ），以澄清其目的是匹配单个切片元素。
具有下边界和上边界的范围模式（如 `a..=b` ）不需要括在括号中。

r[patterns.path]
## 路径模式

r[patterns.path.syntax]
```grammar,模式
PathPattern -> PathExpression
```

r[patterns.path.intro]
 _路径模式_ 是指常量值、或者没有字段的 结构体 或枚举变体的模式。

r[patterns.path.unqualified]
未限定路径模式可以引用：

* 枚举变体
* 结构体
* 常量
* 关联常量

r[patterns.path.qualified]
限定路径模式只能引用关联常量。

r[patterns.path.refutable]
当路径模式引用 结构体 或只有一个变体的枚举变体，或者引用其类型不可反驳的常量时，它们是不可反驳的。
当它们引用可反驳的常量或具有多个变体的枚举变体时，它们是可反驳的。

r[patterns.const]
### 常量模式

r[patterns.const.partial-eq]
当 `T` 类型的常量 `C` 用作模式时，我们首先检查 `T: PartialEq` 。

r[patterns.const.structural-equality]
此外，我们要求 `C` 的值 *具有（递归）结构相等性* ，其递归定义如下：

r[patterns.const.primitive]
- 整数以及 `str` 、 `bool` 和 `char` 值始终具有结构相等性。

r[patterns.const.builtin-aggregate]
- 如果元组、数组和切片的所有字段/元素都具有结构相等性，则它们具有结构相等性。
  （特别是， `()` 和 `[]` 始终具有结构相等性。）

r[patterns.const.ref]
- 如果引用指向的值具有结构相等性，则该引用具有结构相等性。

r[patterns.const.aggregate]
- 如果 结构体 或 枚举 类型的值其 `PartialEq` 实例是通过 `#[derive(PartialEq)]` 派生的，并且所有字段（对于枚举：活跃变体的字段）都具有结构相等性，则该值具有结构相等性。

r[patterns.const.pointer]
- 如果原始指针被定义为常量整数（然后进行转换/变形），则它具有结构相等性。

r[patterns.const.float]
- 如果浮点值不是 `NaN` ，则它具有结构相等性。

r[patterns.const.exhaustive]
- 其他任何东西都不具有结构相等性。

r[patterns.const.generic]
特别地， `C` 的值必须在模式构建时（即预单态化阶段）已知。
这意味着涉及 泛型 参数的关联常量不能用作模式。

r[patterns.const.immutable]
 `C` 的值不能包含对可变静态变量（ `static mut` 项 或内部可变 `static` 项）或 `extern` 静态变量的任何引用。

r[patterns.const.translation]
在确保满足所有条件后，常量值将被转换为模式，其行为现在完全就像直接编写了该模式一样。
特别是，它完全参与穷尽性检查。
（对于原始指针，常量是编写此类模式的唯一方法。对于这些类型，只有 `_` 被认为是穷尽的。）

r[patterns.or]
## 或模式

 _或模式_ 是匹配两个或多个子模式之一的模式（例如 `A | B | C` ）。
它们可以任意嵌套。
从 语法格式 上讲，或模式允许出现在允许其他模式出现的任何地方（由 [Pattern] 产生式表示），但 `let` 绑定以及函数和闭包参数（由 [PatternNoTopAlt] 产生式表示）除外。

r[patterns.constraints]
### 静态语义

r[patterns.constraints.pattern]
1. 给定某种深度的模式 `p | q` （对于任意模式 `p` 和 `q` ），如果满足以下条件，则认为该模式是格式不良的：

   + 为 `p` 推断出的类型与为 `q` 推断出的类型不统一，或者
   + `p` 和 `q` 中没有引入相同的绑定集，或者
   + `p` 和 `q` 中具有相同名称的任何两个绑定的类型在类型或绑定模式方面不统一。

   在上述所有实例中，类型的统一都是精确的，且隐式 [类型强制转换][type coercions] 不适用。

r[patterns.constraints.match-type-check]
2. 在对表达式 `match e_s { a_1 => e_1, ... a_n => e_n }` 进行类型检查时，对于包含 `p_i | q_i` 形式模式的每个 match 臂 `a_i` ，如果在其存在的深度 `d` 处， `e_s` 在深度 `d` 处的片段类型与 `p_i | q_i` 不统一，则模式 `p_i | q_i` 被认为是格式不良的。

r[patterns.constraints.exhaustiveness-or-pattern]
3. 关于穷尽性检查，模式 `p | q` 被视为同时覆盖了 `p` 和 `q` 。
   对于某些构造函数 `c(x, ..)` ，分配律适用，使得 `c(p | q, ..rest)` 覆盖的值集与 `c(p, ..rest) | c(q, ..rest)` 覆盖的值集相同。
   这可以递归应用，直到除了存在于顶层的模式之外，不再有 `p | q` 形式的嵌套模式。

   请注意，这里的 "构造函数" 指的不是元组结构体模式，而是指任何乘积类型的模式。
   这包括枚举变体、元组结构体、具有命名字段的 结构体 、数组、元组和切片。

r[patterns.behavior]
### 动态语义

r[patterns.behavior.nested-or-patterns]
1. 在深度 `d` 处，将 [被匹配][scrutinee] 表达式 `e_s` 与模式 `c(p | q, ..rest)` 进行模式匹配的动态语义（其中 `c` 是某种构造函数， `p` 和 `q` 是任意模式， `rest` 是 `c` 中可选的任何剩余潜在因子）被定义为与 `c(p, ..rest) | c(q, ..rest)` 的动态语义相同。

r[patterns.precedence]
### 与其他无定界模式的优先级

正如本章其他地方所示，有几种类型的模式在 语法格式 上是无定界的，包括标识符模式、引用模式和或模式。
或模式始终具有最低优先级。
这允许我们为将来可能出现的类型注记（type ascription）特性预留 语法格式 空间，并减少歧义。
例如， `x @ A(..) | B(..)` 将导致错误，即 `x` 未在所有模式中绑定。
 `&A(x) | B(x)` 将导致不同子模式中 `x` 之间的类型不匹配。

[PR #138458]: https://github.com/rust-lang/rust/pull/138458
[PR #140593]: https://github.com/rust-lang/rust/pull/140593#issuecomment-2972338457
[`Copy`]: special-types-and-traits.md#copy
[constant]: items/constant-items.md
[enums]: items/enumerations.md
[literals]: expressions/literal-expr.md
[literal expression]: expressions/literal-expr.md
[name resolution]: names/name-resolution.md
[negating]: expressions/operator-expr.md#negation-operators
[path]: expressions/path-expr.md
[pattern matching on unions]: items/unions.md#pattern-matching-on-unions
[range expressions]: expressions/range-expr.md
[scope]: names/scopes.md
[structs]: items/structs.md
[tuples]: types/tuple.md
[scrutinee]: glossary.md#scrutinee
[tuple-like enum variant]: items.enum.tuple-expr
[type coercions]: type-coercions.md
[type namespace]: names.namespaces.kinds
[value namespace]: names.namespaces.kinds
