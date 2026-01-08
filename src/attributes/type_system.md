r[attributes.type-system]
# 类型系统属性

以下 [属性][attributes] 用于更改类型的使用方式。

r[attributes.type-system.non_exhaustive]
## `non_exhaustive`属性

r[attributes.type-system.non_exhaustive.intro]
 *non_exhaustive 属性* 表示以后可能会向类型或变体中添加更多字段或变体。

r[attributes.type-system.non_exhaustive.allowed-positions]
它可以应用于 [结构体][struct]、[枚举][enum] 和 enum 变体。

r[attributes.type-system.non_exhaustive.syntax]
 `non_exhaustive` 属性使用 [MetaWord] 语法格式，因此不接受任何输入。

r[attributes.type-system.non_exhaustive.same-crate]
在定义它的 crate 内部， `non_exhaustive` 没有效果。

```rust
#[non_exhaustive]
pub struct Config {
    pub window_width: u16,
    pub window_height: u16,
}

#[non_exhaustive]
pub struct Token;

#[non_exhaustive]
pub struct Id(pub u64);

#[non_exhaustive]
pub enum Error {
    Message(String),
    Other,
}

pub enum Message {
    #[non_exhaustive] Send { from: u32, to: u32, contents: String },
    #[non_exhaustive] Reaction(u32),
    #[non_exhaustive] Quit,
}

// 在定义它的 crate 内部，可以像往常一样构造非详尽结构体。
let config = Config { window_width: 640, window_height: 480 };
let token = Token;
let id = Id(4);

// 在定义它的 crate 内部，可以对非详尽结构体进行详尽匹配。
let Config { window_width, window_height } = config;
let Token = token;
let Id(id_number) = id;

let error = Error::Other;
let message = Message::Reaction(3);

// 在定义它的 crate 内部，可以对非详尽枚举进行详尽匹配。
match error {
    Error::Message(ref s) => { },
    Error::Other => { },
}

match message {
    // 在定义它的 crate 内部，可以对非详尽变体进行详尽匹配。
    Message::Send { from, to, contents } => { },
    Message::Reaction(id) => { },
    Message::Quit => { },
}
```

r[attributes.type-system.non_exhaustive.external-crate]
在定义它的 crate 之外，带有 `non_exhaustive` 标注的类型具有一些限制，以便在添加新字段或变体时保持向后兼容性。

r[attributes.type-system.non_exhaustive.construction]
在定义它的 crate 之外，无法构造非详尽类型：

- 非详尽变体（ [结构体][struct] 或 [枚举变体][enum] ）不能使用 [结构体表达式][StructExpression] 构造（包括使用 [功能性更新语法格式][functional update syntax] ）。
- [单元结构体][struct] 隐式定义的同名常量，或 [元组结构体][struct] 的同名构造函数，其 [可见性][visibility] 不大于 `pub(crate)` 。
  也就是说，如果结构体的可见性是 `pub` ，那么常量或构造函数的可见性就是 `pub(crate)` ，否则这两个 [项][item] 的可见性相同（就像没有 `#[non_exhaustive]` 的情况一样）。
- [枚举][enum] 实例可以被构造。

以下构造示例在定义它的 crate 之外时无法编译：

<!-- ignore: requires external crates -->
```rust,ignore
// 这些是在上游 crate 中定义并标注为 `#[non_exhaustive]` 的类型。
use upstream::{Config, Token, Id, Error, Message};

// 无法构造 Config 的实例；如果在 upstream 的新版本中添加了新字段，
// 这将导致编译失败，因此是不允许的。
let config = Config { window_width: 640, window_height: 480 };

// 无法构造 Token 的实例；如果添加了新字段，那么
// 它将不再是一个单元结构体，因此由其作为单元结构体创建的
// 同名常量在 crate 外部是不公开的；这段代码无法编译。
let token = Token;

// 无法构造 Id 的实例；如果添加了新字段，那么
// 其构造函数签名将会改变，因此其构造函数
// 在 crate 外部是不公开的；这段代码无法编译。
let id = Id(5);

// 可以构造 Error 的实例；引入新变体
// 不会导致编译失败。
let error = Error::Message("foo".to_string());

// 无法构造 Message::Send 或 Message::Reaction 的实例；
// 如果在 upstream 的新版本中添加了新字段，那么这将会
// 导致编译失败，因此是不允许的。
let message = Message::Send { from: 0, to: 1, contents: "foo".to_string(), };
let message = Message::Reaction(0);

// 无法构造 Message::Quit 的实例；如果这在 upstream 中被转换为
// 元组枚举变体，这将导致编译失败。
let message = Message::Quit;
```

r[attributes.type-system.non_exhaustive.match]
在定义它的 crate 外部匹配非详尽类型时存在一些限制：

- 在对非详尽变体（ [结构体][struct] 或 [枚举变体][enum] ）进行模式匹配时，必须使用包含 `..` 的 [结构体模式][StructPattern] 。元组枚举变体的构造函数的 [可见性][visibility] 被降低到不大于 `pub(crate)` 。
- 在对非详尽 [枚举][enum] 进行模式匹配时，匹配某个变体不会对匹配臂的详尽性做出贡献。当在定义它的 crate 之外时，以下匹配示例无法编译：

<!-- ignore: requires external crates -->
```rust, ignore
// 这些是在上游 crate 中定义并标注为 `#[non_exhaustive]` 的类型。
use upstream::{Config, Token, Id, Error, Message};

// 在不包含通配符臂的情况下，无法匹配非详尽枚举。
match error {
  Error::Message(ref s) => { },
  Error::Other => { },
  // 使用 `_ => {},` 可以编译
}

// 在没有通配符的情况下，无法匹配非详尽结构体。
if let Ok(Config { window_width, window_height }) = config {
    // 使用 `..` 可以编译
}

// 除非使用带有通配符的大括号结构体语法，否则
// 无法匹配非详尽单元结构体或元组结构体。
// 这将编译为 `let Token { .. } = token;`
let Token = token;
// 这将编译为 `let Id { 0: id_number, .. } = id;`
let Id(id_number) = id;

match message {
  // 在不包含通配符的情况下，无法匹配非详尽结构体枚举变体。
  Message::Send { from, to, contents } => { },
  // 无法匹配非详尽元组或单元枚举变体。
  Message::Reaction(type) => { },
  Message::Quit => { },
}
```

也不允许在包含任何非详尽变体的枚举上使用数值转换（ `as` ）。

例如，可以转换以下枚举，因为它不包含任何非详尽变体：

```rust
#[non_exhaustive]
pub enum Example {
    First,
    Second
}
```

但是，如果枚举包含哪怕一个非详尽变体，转换也会导致错误。考虑同一枚举的这个修改版本：

```rust
#[non_exhaustive]
pub enum EnumWithNonExhaustiveVariants {
    First,
    #[non_exhaustive]
    Second
}
```

<!-- ignore: needs multiple crates -->
```rust,ignore
use othercrate::EnumWithNonExhaustiveVariants;

// 错误：无法转换在另一个 crate 中定义且带有非详尽变体的枚举
let _ = EnumWithNonExhaustiveVariants::First as u8;
```

在下游 crate 中，非详尽类型始终被视为 inhabited 。

[`match`]: ../expressions/match-expr.md
[attributes]: ../attributes.md
[enum]: ../items/enumerations.md
[functional update syntax]: ../expressions/struct-expr.md#functional-update-syntax
[struct]: ../items/structs.md
[visibility]: ../visibility-and-privacy.md
