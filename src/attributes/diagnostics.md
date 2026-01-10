r[attributes.diagnostics]
# 诊断属性

以下 [属性][attributes] 用于在编译期间控制或生成诊断消息。

r[attributes.diagnostics.lint]
## Lint检查属性

Lint 检查命名了一种可能不理想的编码模式，例如不可达代码或遗漏的文档。

r[attributes.diagnostics.lint.level]
lint 属性 `allow`、
`expect`、`warn`、`deny` 和 `forbid` 使用 [MetaListPaths] 语法格式
来指定 lint 名称列表，以更改属性所应用实体的 lint 级别。

对于任何 lint 检查 `C`：

r[attributes.diagnostics.lint.allow]
* `#[allow(C)]` 覆盖对 `C` 的检查，以便违规行为将不被报告。

r[attributes.diagnostics.lint.expect]
* `#[expect(C)]` 表示预期会发出 lint `C`。如果预期未实现，则该属性将抑制 `C` 的发出或发出警告。

r[attributes.diagnostics.lint.warn]
* `#[warn(C)]` 警告 `C` 的违规行为，但继续编译。

r[attributes.diagnostics.lint.deny]
* `#[deny(C)]` 在遇到 `C` 的违规行为后发出错误信号，

r[attributes.diagnostics.lint.forbid]
* `#[forbid(C)]` 与 `deny(C)` 相同，但事后也禁止更改 lint 级别，

> [!NOTE]
> 通过 `rustc -W help` 可以找到 `rustc` 支持的 lint 检查，以及它们的默认设置，并记录在 [rustc book] 中。

```rust
pub mod m1 {
    // 此处忽略缺少文档
    #[allow(missing_docs)]
    pub fn undocumented_one() -> i32 { 1 }

    // 此处缺少文档会发出警告
    #[warn(missing_docs)]
    pub fn undocumented_too() -> i32 { 2 }

    // 此处缺少文档会发出错误
    #[deny(missing_docs)]
    pub fn undocumented_end() -> i32 { 3 }
}
```

r[attributes.diagnostics.lint.override]
lint 属性可以覆盖先前属性指定的级别，只要该级别不尝试更改被禁止的 lint
（除了 `deny`，它在 `forbid` 上下文中是允许的，但会被忽略）。
先前的属性是指在语法树中更高级别的属性，或在同一实体上按从左到右源代码顺序列出的先前属性。

此示例展示了如何使用 `allow` 和 `warn` 来开启和关闭特定检查：

```rust
#[warn(missing_docs)]
pub mod m2 {
    #[allow(missing_docs)]
    pub mod nested {
        // 此处忽略缺少文档
        pub fn undocumented_one() -> i32 { 1 }

        // 此处缺少文档会发出警告，
        // 尽管上面有 allow。
        #[warn(missing_docs)]
        pub fn undocumented_two() -> i32 { 2 }
    }

    // 此处缺少文档会发出警告
    pub fn undocumented_too() -> i32 { 3 }
}
```

此示例展示了如何使用 `forbid` 来禁止对该 lint 检查使用 `allow` 或 `expect`：

```rust,compile_fail
#[forbid(missing_docs)]
pub mod m3 {
    // 尝试切换警告在此处发出错误
    #[allow(missing_docs)]
    /// 返回 2。
    pub fn undocumented_too() -> i32 { 2 }
}
```

> [!NOTE]
> `rustc` 允许在 [命令行][rustc-lint-cli] 上设置 lint 级别，并且还支持对报告的 lint [设置上限][rustc-lint-caps]。

r[attributes.diagnostics.lint.reason]
### Lint原因

所有 lint 属性都支持一个额外的 `reason` 参数，以提供添加某个属性的上下文。如果 lint 在定义的级别发出，此原因将作为 lint 消息的一部分显示。

```rust,edition2015,compile_fail
// `keyword_idents` 默认是允许的。此处我们禁止它以
// 避免在更新 版次 时标识符的迁移。
#![deny(
    keyword_idents,
    reason = "we want to avoid these idents to be future compatible"
)]

// 此名称在 Rust 的 2015 版次 中是允许的。我们仍然旨在避免
// 这以便未来兼容并且不混淆最终用户。
fn dyn() {}
```

这是另一个示例，其中 lint 被允许并带有原因：

```rust
use std::path::PathBuf;

pub fn get_path() -> PathBuf {
    // `allow` 属性上的 `reason` 参数作为读者的文档。
    #[allow(unused_mut, reason = "this is only modified on some platforms")]
    let mut file_name = PathBuf::from("git");

    #[cfg(target_os = "windows")]
    file_name.set_extension("exe");

    file_name
}
```

r[attributes.diagnostics.expect]
### `#[expect]`属性

r[attributes.diagnostics.expect.intro]
`#[expect(C)]` 属性为 lint `C` 创建一个 lint 预期。如果同一位置的 `#[warn(C)]` 属性会导致 lint 发出，则该预期将被实现。如果预期未实现，因为 lint `C` 不会发出，则将在该属性处发出 `unfulfilled_lint_expectations` lint。

```rust
fn main() {
    // 这个 `#[expect]` 属性创建了一个 lint 预期，即 `unused_variables`
    // lint 将由以下语句发出。由于 `question` 变量被 `println!` 宏使用，
    // 因此此预期未实现。
    // 因此，`unfulfilled_lint_expectations` lint 将在此属性处发出。
    #[expect(unused_variables)]
    let question = "who lives in a pineapple under the sea?";
    println!("{question}");

    // 这个 `#[expect]` 属性创建了一个 lint 预期，它将被实现，因为
    // `answer` 变量从未被使用。通常会发出的 `unused_variables` lint 被抑制。
    // 不会为语句或属性发出警告。
    #[expect(unused_variables)]
    let answer = "SpongeBob SquarePants!";
}
```

r[attributes.diagnostics.expect.fulfillment]
lint 预期仅由被 `expect` 属性抑制的 lint 发出所实现。如果在作用域中使用 `allow` 或 `warn` 等其他级别属性修改了 lint 级别，则 lint 发出将相应处理，并且预期将保持未实现状态。

```rust
#[expect(unused_variables)]
fn select_song() {
    // 这将按 `warn` 属性定义发出 `unused_variables` lint 的警告级别。
    // 这不会实现函数上方的预期。
    #[warn(unused_variables)]
    let song_name = "Crab Rave";

    // `allow` 属性抑制了 lint 发出。这不会实现预期，
    // 因为它已被 `allow` 属性抑制，而不是函数上方的 `expect` 属性。
    #[allow(unused_variables)]
    let song_creator = "Noisestorm";

    // 这个 `expect` 属性将抑制变量处的 `unused_variables` lint 发出。
    // 函数上方的 `expect` 属性仍然不会被实现，因为此 lint 发出已被本地
    // expect 属性抑制。
    #[expect(unused_variables)]
    let song_version = "Monstercat Release";
}
```

r[attributes.diagnostics.expect.independent]
如果 `expect` 属性包含多个 lint，则每个 lint 都单独预期。对于 lint 组，只要组内有一个 lint 被发出就足够了：

```rust
// 这个预期将被函数内未使用的值实现，
// 因为发出的 `unused_variables` lint 在 `unused` lint 组内。
#[expect(unused)]
pub fn thoughts() {
    let unused = "I'm running out of examples";
}

pub fn another_example() {
    // 此属性创建两个 lint 预期。`unused_mut` lint 将被抑制，
    // 从而实现第一个预期。`unused_variables` 不会发出，因为变量已被使用。
    // 因此，该预期将未实现，并会发出警告。
    #[expect(unused_mut, unused_variables)]
    let mut link = "https://www.rust-lang.org/";

    println!("Welcome to our community: {link}");
}
```

> [!NOTE]
> `#[expect(unfulfilled_lint_expectations)]` 的行为目前定义为始终生成 `unfulfilled_lint_expectations` lint。

r[attributes.diagnostics.lint.group]
### Lint组

Lint 可以组织成命名组，以便可以一起调整相关 lint 的级别。使用命名组等同于列出该组内的 lint。

```rust,compile_fail
// 这允许“unused”组中的所有 lint。
#[allow(unused)]
// 这将“unused”组中的“unused_must_use”lint 覆盖为 deny。
#[deny(unused_must_use)]
fn example() {
    // 这不会生成警告，因为“unused_variables”
    // lint 在“unused”组中。
    let x = 1;
    // 这会生成错误，因为结果未使用，并且
    // “unused_must_use”被标记为“deny”。
    std::fs::remove_file("some_file"); // 错误：必须使用的未使用的 `Result`
}
```

r[attributes.diagnostics.lint.group.warnings]
有一个名为“warnings”的特殊组，它包含所有警告级别的 lint。 “warnings”组忽略属性顺序，并应用于实体内所有本应发出警告的 lint。

```rust,compile_fail
# unsafe fn an_unsafe_fn() {}
// 这两个属性的顺序无关紧要。
#[deny(warnings)]
// `unsafe_code` lint  normally "allow" by default.
#[warn(unsafe_code)]
fn example_err() {
    // 这是一个错误，因为 `unsafe_code` 警告已
    // 提升为“deny”。
    unsafe { an_unsafe_fn() } // 错误：使用了 `unsafe` 块
}
```

r[attributes.diagnostics.lint.tool]
### 工具lint属性

r[attributes.diagnostics.lint.tool.intro]
工具 lint 允许使用作用域 lint，以 `allow`、`warn`、`deny` 或 `forbid`
某些工具的 lint。

r[attributes.diagnostics.lint.tool.activation]
工具 lint 仅在相关工具处于活动状态时才会被检查。如果像 `allow` 这样的 lint 属性引用了不存在的工具 lint，编译器在您使用该工具之前不会警告不存在的 lint。

否则，它们的工作方式与常规 lint 属性相同：

```rust
// set the entire `pedantic` clippy lint group to warn
#![warn(clippy::pedantic)]
// silence warnings from the `filter_map` clippy lint
#![allow(clippy::filter_map)]

fn main() {
    // ...
}

// silence the `cmp_nan` clippy lint just for this function
#[allow(clippy::cmp_nan)]
fn foo() {
    // ...
}
```

> [!NOTE]
> `rustc` 目前识别 "[clippy]" 和 "[rustdoc]" 的工具 lint。

r[attributes.diagnostics.deprecated]
## `deprecated`属性

r[attributes.diagnostics.deprecated.intro]
*`deprecated` 属性* 将一个 项 标记为已弃用。`rustc` 将在使用 `#[deprecated]` 项 时发出警告。`rustdoc` 将显示 项 弃用信息，包括 `since` 版本和 `note`（如果可用）。

r[attributes.diagnostics.deprecated.syntax]
`deprecated` 属性有几种形式：

- `deprecated` --- 发出通用消息。
- `deprecated = "message"` --- 在弃用消息中包含给定的字符串。
- [MetaListNameValueStr] 语法格式，带有两个可选字段：
  - `since` --- 指定 项 被弃用时的版本号。`rustc` 目前不解释该字符串，但 [Clippy] 等外部工具可能会检查该值的有效性。
  - `note` --- 指定应包含在弃用消息中的字符串。这通常用于提供有关弃用的解释和首选替代方案。

r[attributes.diagnostic.deprecated.allowed-positions]
`deprecated` 属性可以应用于任何 [项][item]、[特型项][trait item]、[枚举变体][enum variant]、[结构体字段][struct field]、[外部块项][external block item] 或 [宏定义][macro definition]。它不能应用于 [特型实现项][trait-impl]。当应用于包含其他 项 的 项 （例如 [模块][module] 或 [实现][implementation]）时，所有子 项 都继承弃用属性。

这里有一个例子：

```rust
#[deprecated(since = "5.2.0", note = "foo was rarely used. Users should instead use bar")]
pub fn foo() {}

pub fn bar() {}
```

[RFC][1270-deprecation.md] 包含了动机和更多细节。

[1270-deprecation.md]: https://github.com/rust-lang/rfcs/blob/master/text/1270-deprecation.md

r[attributes.diagnostics.must_use]
## `must_use`属性

r[attributes.diagnostics.must_use.intro]
*`must_use` 属性* 用于在值未“使用”时发出诊断警告。

r[attributes.diagnostics.must_use.allowed-positions]
`must_use` 属性可以应用于用户定义的复合类型（[`结构体`][struct]、[`枚举`][enum] 和 [`联合体`][union]）、[函数][functions] 和 [特型][traits]。

r[attributes.diagnostics.must_use.message]
`must_use` 属性可以使用 [MetaNameValueStr] 语法格式 包含一条消息，例如 `#[must_use = "example message"]`。该消息将与警告一起给出。

r[attributes.diagnostics.must_use.type]
当用于用户定义的复合类型时，如果 [表达式语句][expression statement] 的 [表达式][expression] 具有该类型，则 `unused_must_use` lint 被违反。

```rust
#[must_use]
struct MustUse {
    // 一些字段
}

# impl MustUse {
#   fn new() -> MustUse { MustUse {} }
# }
#
// 违反 `unused_must_use` lint。
MustUse::new();
```

r[attributes.diagnostics.must_use.fn]
当用于函数时，如果 [表达式语句][expression statement] 的 [表达式][expression] 是对该函数的 [调用表达式][call expression]，则 `unused_must_use` lint 被违反。

```rust
#[must_use]
fn five() -> i32 { 5i32 }

// 违反 `unused_must_use` lint。
five();
```

r[attributes.diagnostics.must_use.trait]
当用于 [特型声明][trait declaration] 时，如果 [表达式语句][expression statement] 对一个返回该 特型 的 [impl trait] 或 [dyn trait] 的函数的 [调用表达式][call expression] 被未使用，则违反 `unused_must_use` lint。

```rust
#[must_use]
trait Critical {}
impl Critical for i32 {}

fn get_critical() -> impl Critical {
    4i32
}

// 违反 `unused_must_use` lint。
get_critical();
```

r[attributes.diagnostics.must_use.trait-function]
当用于 特型 声明中的函数时，当调用表达式是 特型 实现中的函数时，该行为也适用。

```rust
trait Trait {
    #[must_use]
    fn use_me(&self) -> i32;
}

impl Trait for i32 {
    fn use_me(&self) -> i32 { 0i32 }
}

// 违反 `unused_must_use` lint。
5i32.use_me();
```

r[attributes.diagnostics.must_use.trait-impl-function]
当用于 特型 实现中的函数时，该属性不执行任何操作。

> [!NOTE]
> 包含该值的普通无操作表达式不会违反 lint。示例包括将值包装在一个不实现 [`Drop`] 的类型中然后不使用该类型，以及作为未使用的 [块表达式][block expression] 的最终表达式。
>
> ```rust
> #[must_use]
> fn five() -> i32 { 5i32 }
>
> // 这些都不会违反 `unused_must_use` lint。
> (five(),);
> Some(five());
> { five() };
> if true { five() } else { 0i32 };
> match true {
>     _ => five()
> };
> ```

> [!NOTE]
> 当有意丢弃一个必须使用的值时，习惯上使用带有 `_` 模式的 [let 语句][let statement]。
>
> ```rust
> #[must_use]
> fn five() -> i32 { 5i32 }
>
> // 不违反 `unused_must_use` lint。
> let _ = five();
> ```

r[attributes.diagnostic.namespace]
## `diagnostic`工具属性命名空间

r[attributes.diagnostic.namespace.intro]
`#[diagnostic]` 属性命名空间是用于影响编译时错误消息的属性的归属。这些属性提供的提示不保证会被使用。

r[attributes.diagnostic.namespace.unknown-invalid-syntax]
此命名空间中未知的属性会被接受，尽管它们可能会为未使用的属性发出警告。此外，已知属性的无效输入通常会是警告（详见属性定义）。这旨在允许将来添加或丢弃属性和更改输入，从而允许更改而无需保持无意义的属性或选项继续工作。

r[attributes.diagnostic.on_unimplemented]
### `diagnostic::on_unimplemented`属性

r[attributes.diagnostic.on_unimplemented.intro]
`#[diagnostic::on_unimplemented]` 属性是给编译器的一个提示，用于补充在需要 特型 但类型上未实现 特型 的情况下通常会生成的错误消息。

r[attributes.diagnostic.on_unimplemented.allowed-positions]
该属性应放置在 [特型声明][trait declaration] 上，尽管放置在其他位置不是错误。

r[attributes.diagnostic.on_unimplemented.syntax]
该属性使用 [MetaListNameValueStr] 语法格式 来指定其输入，尽管任何格式错误的属性输入都不被视为错误，以提供向前和向后兼容性。

r[attributes.diagnostic.on_unimplemented.keys]
以下键具有给定含义：
* `message` --- 顶级错误消息的文本。
* `label` --- 错误消息中损坏代码内联显示的标签文本。
* `note` --- 提供额外备注。

r[attributes.diagnostic.on_unimplemented.note-repetition]
`note` 选项可以出现多次，这将导致发出多条备注消息。

r[attributes.diagnostic.on_unimplemented.repetition]
如果其他选项中的任何一个出现多次，则相关选项的第一次出现指定实际使用的值。随后的出现会生成警告。

r[attributes.diagnostic.on_unimplemented.unknown-keys]
任何未知键都会生成警告。

r[attributes.diagnostic.on_unimplemented.format-string]
所有三个选项都接受一个字符串作为参数，使用与 [`std::fmt`] 字符串相同的格式进行解释。

r[attributes.diagnostic.on_unimplemented.format-parameters]
带有给定命名参数的格式参数将被以下文本替换：
* `{Self}` --- 实现 特型 的类型名称。
* `{` *GenericParameterName* `}` --- 给定 泛型 参数的 泛型 参数的类型名称。

r[attributes.diagnostic.on_unimplemented.invalid-formats]
任何其他格式参数都会生成警告，但除此之外，将按原样包含在字符串中。

r[attributes.diagnostic.on_unimplemented.invalid-string]
无效格式字符串可能会生成警告，但除此之外是允许的，但可能不会按预期显示。格式说明符可能会生成警告，但除此之外会被忽略。

在此示例中：

```rust,compile_fail,E0277
#[diagnostic::on_unimplemented(
    message = "My Message for `ImportantTrait<{A}>` implemented for `{Self}`",
    label = "My Label",
    note = "Note 1",
    note = "Note 2"
)]
trait ImportantTrait<A> {}

fn use_my_trait(_: impl ImportantTrait<i32>) {}

fn main() {
    use_my_trait(String::new());
}
```

编译器可能会生成一个看起来像这样的错误消息：

```text
error[E0277]: My Message for `ImportantTrait<i32>` implemented for `String`
  --> src/main.rs:14:18
   |
14 |     use_my_trait(String::new());
   |     ------------ ^^^^^^^^^^^^^ My Label
   |     |
   |     required by a bound introduced by this call
   |
   = help: the trait `ImportantTrait<i32>` is not implemented for `String`
   = note: Note 1
   = note: Note 2
```

r[attributes.diagnostic.do_not_recommend]
### `diagnostic::do_not_recommend`属性

r[attributes.diagnostic.do_not_recommend.intro]
`#[diagnostic::do_not_recommend]` 属性是给编译器的一个提示，指示它不要在诊断消息中显示带注解的 特型 实现。

> [!NOTE]
> 如果你知道该推荐通常对程序员没有用处，那么抑制该推荐可能会很有用。这种情况经常发生在广泛的、包罗万象的 impl 上。该推荐可能会将程序员引入歧途，或者 特型 实现可能是你不希望公开的内部细节，或者程序员可能无法满足其边界。
>
> 例如，在关于某个类型未实现所需 特型 的错误消息中，编译器可能会找到一个 特型 实现，如果不是因为 特型 实现中的特定边界，该实现将满足要求。编译器可能会告诉用户存在一个 impl，但问题在于 特型 实现中的边界。`#[diagnostic::do_not_recommend]` 属性可以用来告诉编译器 *不要* 告诉用户该 特型 实现，而是简单地告诉用户该类型未实现所需的 特型。

r[attributes.diagnostic.do_not_recommend.allowed-positions]
该属性应放置在 [特型实现项][trait-impl] 上，尽管放置在其他位置不是错误。

r[attributes.diagnostic.do_not_recommend.syntax]
该属性不接受任何参数，尽管意外参数不被视为错误。

在以下示例中，有一个名为 `AsExpression` 的 特型，它用于将任意类型转换为 SQL 库中使用的 `Expression` 类型。有一个名为 `check` 的方法，它接受一个 `AsExpression`。

```rust,compile_fail,E0277
# pub trait Expression {
#     type SqlType;
# }
#
# pub trait AsExpression<ST> {
#     type Expression: Expression<SqlType = ST>;
# }
#
# pub struct Text;
# pub struct Integer;
#
# pub struct Bound<T>(T);
# pub struct SelectInt;
#
# impl Expression for SelectInt {
#     type SqlType = Integer;
# }
#
# impl<T> Expression for Bound<T> {
#     type SqlType = T;
# }
#
# impl AsExpression<Integer> for i32 {
#     type Expression = Bound<Integer>;
# }
#
# impl AsExpression<Text> for &'_ str {
#     type Expression = Bound<Text>;
# }
#
# impl<T> Foo for T where T: Expression {}

// 取消注释此行以更改推荐。
// #[diagnostic::do_not_recommend]
impl<T, ST> AsExpression<ST> for T
where
    T: Expression<SqlType = ST>,
{
    type Expression = T;
}

trait Foo: Expression + Sized {
    fn check<T>(&self, _: T) -> <T as AsExpression<<Self as Expression>::SqlType>>::Expression
    where
        T: AsExpression<Self::SqlType>,
    {
        todo!()
    }
}

fn main() {
    SelectInt.check("bar");
}
```

`SelectInt` 类型的 `check` 方法期望 `Integer` 类型。使用 i32 类型调用它会成功，因为它通过 `AsExpression` 特型 转换为 `Integer`。但是，使用字符串调用它则不会成功，并会生成一个可能看起来像这样的错误：

```text
error[E0277]: 特型 边界 `&str: Expression` 未满足
  --> src/main.rs:53:15
   |
53 |     SelectInt.check("bar");
   |               ^^^^^ 特型 `Expression` 未为 `&str` 实现
   |
   = help: 以下其他类型实现了 特型 `Expression`：
             Bound<T>
             SelectInt
note: `&str` 实现 `AsExpression<Integer>` 所需
  --> src/main.rs:45:13
   |
45 | impl<T, ST> AsExpression<ST> for T
   |             ^^^^^^^^^^^^^^^^     ^
46 | where
47 |     T: Expression<SqlType = ST>,
   |        ------------------------ 此处引入的未满足的 特型 边界
```

通过将 `#[diagnostic::do_not_recommend]` 属性添加到 `AsExpression` 的通用 `impl` 中，消息更改为：

```text
error[E0277]: 特型 边界 `&str: AsExpression<Integer>` 未满足
  --> src/main.rs:53:15
   |
53 |     SelectInt.check("bar");
   |               ^^^^^ 特型 `AsExpression<Integer>` 未为 `&str` 实现
   |
   = help: 特型 `AsExpression<Integer>` 未为 `&str` 实现
           但 特型 `AsExpression<Text>` 已为其实现
   = help: 对于该 特型 实现，预期为 `Text`，但找到 `Integer`
```

第一个错误消息包含了一个关于 `&str` 和 `Expression` 关系的有些令人困惑的错误消息，以及通用 impl 中未满足的 特型 边界。添加 `#[diagnostic::do_not_recommend]` 后，它不再考虑通用 impl 作为推荐。消息应该更清晰一些，表明字符串无法转换为 `Integer`。

[Clippy]: https://github.com/rust-lang/rust-clippy
[`Drop`]: ../special-types-and-traits.md#drop
[attributes]: ../attributes.md
[block expression]: ../expressions/block-expr.md
[call expression]: ../expressions/call-expr.md
[dyn trait]: ../types/trait-object.md
[enum variant]: ../items/enumerations.md
[enum]: ../items/enumerations.md
[expression statement]: ../statements.md#表达式语句
[expression]: ../expressions.md
[external block item]: ../items/external-blocks.md
[functions]: ../items/functions.md
[impl trait]: ../types/impl-trait.md
[implementation]: ../items/implementations.md
[item]: ../items.md
[let statement]: ../statements.md#let语句
[macro definition]: ../macros-by-example.md
[module]: ../items/modules.md
[rustc book]: https://doc.rust-lang.org/rustc/lints/index.html
[rustc-lint-caps]: https://doc.rust-lang.org/rustc/lints/levels.html#capping-lints
[rustc-lint-cli]: https://doc.rust-lang.org/rustc/lints/levels.html#via-compiler-flag
[rustdoc]: ../../rustdoc/lints.html
[struct field]: ../items/structs.md
[struct]: ../items/structs.md
[trait declaration]: ../items/traits.md
[trait item]: ../items/traits.md
[trait-impl]: ../items/implementations.md#特型实现
[traits]: ../items/traits.md
[union]: ../items/unions.md
