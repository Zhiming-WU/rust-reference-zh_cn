<!-- https://github.com/rust-lang/reference/blob/master/src/macros-by-example.md -->
<!-- commit 68bdfd15fa9feebfbf94a06cf574de598e239198 -->

r[macro.decl]
# 声明宏

r[macro.decl.syntax]
```grammar,宏
MacroRulesDefinition ->
    `macro_rules` `!` IDENTIFIER MacroRulesDef

MacroRulesDef ->
      `(` MacroRules `)` `;`
    | `[` MacroRules `]` `;`
    | `{` MacroRules `}`

MacroRules ->
    MacroRule ( `;` MacroRule )* `;`?

MacroRule ->
    MacroMatcher `=>` MacroTranscriber

MacroMatcher ->
      `(` MacroMatch* `)`
    | `[` MacroMatch* `]`
    | `{` MacroMatch* `}`

MacroMatch ->
      Token _except `$` and [delimiters][lex.token.delim]_
    | MacroMatcher
    | `$` ( IDENTIFIER_OR_KEYWORD _except `crate`_ | RAW_IDENTIFIER ) `:` MacroFragSpec
    | `$` `(` MacroMatch+ `)` MacroRepSep? MacroRepOp

MacroFragSpec ->
      `block` | `expr` | `expr_2021` | `ident` | `item` | `lifetime` | `literal`
    | `meta` | `pat` | `pat_param` | `path` | `stmt` | `tt` | `ty` | `vis`

MacroRepSep -> Token _except [delimiters][lex.token.delim] and [MacroRepOp]_

MacroRepOp -> `*` | `+` | `?`

MacroTranscriber -> DelimTokenTree
```

r[macro.decl.intro]
`macro_rules` 允许用户以声明式的方式定义语法扩展。我们称这种扩展为“声明宏”或简称“宏”。

每个声明宏都有一个名称和一条或多条 _规则_ 。每条规则包含两部分：一个 _匹配器_ ，描述它匹配的语法；一个 _转录器_ ，描述成功匹配的调用将替换成的语法。匹配器和转录器都必须由分隔符包围。宏可以展开为表达式、语句、项（包括特型、impl 和外部项）、类型或模式。

r[macro.decl.transcription]
## 转录

r[macro.decl.transcription.intro]
当宏被调用时，宏扩展器会按名称查找宏调用，并依次尝试每条宏规则。它转录第一个成功匹配的规则；如果此操作导致错误，则不会尝试后续匹配。

r[macro.decl.transcription.lookahead]
匹配时，不执行前瞻(lookahead)；如果编译器无法逐个词法单元明确地确定如何解析宏调用，则会报错。在下面的示例中，编译器不会在前瞻到标识符之后，去检查后面的词法单元是否是 `)`，即使那样可以使其明确地解析调用：

```rust,compile_fail
macro_rules! ambiguity {
    ($($i:ident)* $j:ident) => { };
}

ambiguity!(error); // 错误：局部歧义
```

r[macro.decl.transcription.syntax]
在匹配器和转录器中，`$` 词法单元用于调用宏引擎的特殊行为（在下面的[元变量][Metavariables]和[重复][Repetitions]中描述）。不属于此类调用的词法单元会被字面匹配和转录，但有一个例外。这个例外是，匹配器的外部分隔符将匹配任意一对分隔符。因此，例如，匹配器 `(())` 将匹配 `{()}` 但不匹配 `{{}}`。字符 `$` 不能被字面匹配或转录。

r[macro.decl.transcription.fragment]
### 转发匹配到的片段

当将一个匹配到的片段转发给另一个声明宏时，第二个宏中的匹配器将看到该片段类型的不透明 AST。第二个宏不能使用字面词法单元来匹配匹配器中的片段，只能使用相同类型的片段说明符。`ident`、`lifetime` 和 `tt` 片段类型是一个例外，它们*可以*通过字面词法单元进行匹配。下面示例说明了此限制：

```rust,compile_fail
macro_rules! foo {
    ($l:expr) => { bar!($l); }
// 错误：               ^^ 宏调用中不期望此词法单元
}

macro_rules! bar {
    (3) => {}
}

foo!(3);
```

下面示例说明了在匹配 `tt` 片段后如何直接匹配词法单元：

```rust
// 编译通过
macro_rules! foo {
    ($l:tt) => { bar!($l); }
}

macro_rules! bar {
    (3) => {}
}

foo!(3);
```

r[macro.decl.meta]
## 元变量

r[macro.decl.meta.intro]
在匹配器中，`$` _名称_ `:` _片段说明符_ 匹配指定种类的Rust语法片段，并将其绑定到元变量 `$`_名称_。

r[macro.decl.meta.specifier]
有效的片段说明符有：

  * `block`：一个[块表达式][BlockExpression]
  * `expr`：一个[表达式][Expression]
  * `expr_2021`：一个[表达式][Expression]，除了[下划线表达式][UnderscoreExpression]和[常量块表达式][ConstBlockExpression]（参见[macro.decl.meta.edition2024]）
  * `ident`：一个[IDENTIFIER_OR_KEYWORD]，除了 `_`、[RAW_IDENTIFIER]或[`$crate`]
  * `item`：一个[项][Item]
  * `lifetime`：一个[LIFETIME_TOKEN]
  * `literal`：匹配 `-`<sup>?</sup>[字面量表达式][LiteralExpression]
  * `meta`：一个[Attr]，一个属性的内容
  * `pat`：一个[模式][Pattern]（参见[macro.decl.meta.edition2021]）
  * `pat_param`：一个[PatternNoTopAlt]
  * `path`：一个[TypePath]样式的路径
  * `stmt`：一个[语句][grammar-Statement]，不带末尾分号（需要分号的项语句除外）
  * `tt`：一个[TokenTree]&nbsp;（一个单独的[词法单元][token]或匹配分隔符 `()`、`[]` 或 `{}` 中的词法单元）
  * `ty`：一个[类型][grammar-Type]
  * `vis`：一个可能为空的[可见性][Visibility]限定符

r[macro.decl.meta.transcription]
在转录器中，元变量仅通过 `$`_名称_ 引用，因为片段种类已在匹配器中指定。元变量会被替换为匹配它们的语法元素。元变量可以被转录多次或根本不转录。

r[macro.decl.meta.dollar-crate]
关键字元变量 [`$crate`] 可用于指代当前 crate。

r[macro.decl.meta.edition2021]
> [!EDITION-2021]
> 从2021版次开始，`pat` 片段说明符匹配顶层或模式（即，它们接受[模式][Pattern]）。
>
> 在2021版次之前，它们匹配的片段与 `pat_param` 完全相同（即，它们接受[PatternNoTopAlt]）。
>
> 相关的版次是 `macro_rules!` 定义生效的版次。

r[macro.decl.meta.edition2024]
> [!EDITION-2024]
> 在2024版次之前，`expr`片段说明符不匹配顶层的[下划线表达式][UnderscoreExpression]或[常量块表达式][ConstBlockExpression]。它们在子表达式中是允许的。
>
> 存在`expr_2021`片段说明符是为了保持与2024之前的版次的向后兼容性。

r[macro.decl.repetition]
## 重复

r[macro.decl.repetition.intro]
在匹配器和转录器中，重复是通过将要重复的词法单元放在 `$(`…`)` 内部，然后跟随一个重复操作符来表示的，可选地在两者之间放置一个分隔词法单元。

r[macro.decl.repetition.separator]
分隔词法单元可以是除分隔符或重复操作符之外的任何词法单元，但 `;` 和 `,` 最为常见。例如，`$( $i:ident ),*` 表示任意数量的用逗号分隔的标识符。允许嵌套重复。

r[macro.decl.repetition.operators]
重复操作符有：

- `*` — 表示任意数量的重复。
- `+` — 表示任意数量，但至少一个。
- `?` — 表示一个可选片段，出现零次或一次。

r[macro.decl.repetition.optional-restriction]
由于 `?` 最多表示一次出现，它不能与分隔符一起使用。

r[macro.decl.repetition.fragment]
重复片段既匹配又转录为指定数量的片段，并由分隔词法单元分隔。元变量匹配其相应片段的每次重复。例如，上面 `$( $i:ident ),*` 示例将 `$i` 匹配到列表中的所有标识符。

在转录期间，对重复施加了额外的限制，以便编译器知道如何正确地展开它们：

1. 元变量在转录器中出现的重复次数、种类和嵌套顺序必须与在匹配器中完全相同。因此，对于匹配器 `$( $i:ident ),*`，转录器 `=> { $i }`、`=> { $( $( $i )* )* }` 和 `=> { $( $i )+ }` 都是非法的，但 `=> { $( $i );* }` 是正确的，它将逗号分隔的标识符列表替换为分号分隔的列表。
2. 转录器中的每个重复必须至少包含一个元变量，以决定要展开多少次。如果在同一个重复中出现多个元变量，它们必须绑定到相同数量的片段。例如，`( $( $i:ident ),* ; $( $j:ident ),* ) => (( $( ($i,$j) ),* ))` 必须绑定与 `$j` 片段相同数量的 `$i` 片段。这意味着用 `(a, b, c; d, e, f)` 调用宏是合法的，并展开为 `((a,d), (b,e), (c,f))`，但 `(a, b, c; d, e)` 是非法的，因为它没有相同数量的片段。此要求适用于每个嵌套重复层。

r[macro.decl.scope]
## 作用域、导出和导入

r[macro.decl.scope.intro]
由于历史原因，声明宏的作用域不完全像项那样工作。宏有两种形式的作用域：文本作用域和基于路径的作用域。文本作用域基于事物在源文件中出现的顺序，甚至可以跨多个文件，它是默认的作用域。这将在下面进一步解释。基于路径的作用域与项作用域的工作方式完全相同。宏的作用域、导出和导入主要由属性控制。

r[macro.decl.scope.unqualified]
当宏通过非限定标识符（不属于多部分路径）调用时，它首先在文本作用域中查找。如果没有结果，则在基于路径的作用域中查找。如果宏的名称使用路径进行限定，则它只在基于路径的作用域中查找。

<!-- ignore: requires external crates -->
```rust,ignore
use lazy_static::lazy_static; // 基于路径的导入。

macro_rules! lazy_static { // 文本定义。
    (lazy) => {};
}

lazy_static!{lazy} // 文本查找首先找到我们的宏。
self::lazy_static!{} // 基于路径的查找忽略我们的宏，找到导入的宏。
```

r[macro.decl.scope.textual]
### 文本作用域

r[macro.decl.scope.textual.intro]
文本作用域主要基于事物在源文件中出现的顺序，其工作方式类似于用 `let` 声明的局部变量的作用域，只不过它也适用于模块级别。当使用 `macro_rules!` 定义宏时，宏在定义之后（请注意，它仍然可以递归使用，因为名称是从调用站点查找的）进入作用域，直到其周围的作用域（通常是模块）关闭。这可以进入子模块，甚至跨越多个文件：

<!-- ignore: requires external modules -->
```rust,ignore
//// src/lib.rs
mod has_macro {
    // m!{} // 错误：m 不在作用域中。

    macro_rules! m {
        () => {};
    }
    m!{} // 正常：出现在 m 的声明之后。

    mod uses_macro;
}

// m!{} // 错误：m 不在作用域中。

//// src/has_macro/uses_macro.rs

m!{} // 正常：出现在 src/lib.rs 中 m 的声明之后。
```

r[macro.decl.scope.textual.shadow]
多次定义宏不是错误；除非前一个声明已超出作用域，否则最近的声明将遮蔽前一个。

```rust
macro_rules! m {
    (1) => {};
}

m!(1);

mod inner {
    m!(1);

    macro_rules! m {
        (2) => {};
    }
    // m!(1); // 错误：没有规则匹配 '1'
    m!(2);

    macro_rules! m {
        (3) => {};
    }
    m!(3);
}

m!(1);
```

宏也可以在函数内部局部声明和使用，工作方式类似：

```rust
fn foo() {
    // m!(); // 错误：m 不在作用域中。
    macro_rules! m {
        () => {};
    }
    m!();
}

// m!(); // 错误：m 不在作用域中。
```

r[macro.decl.scope.textual.shadow.path-based]
宏的文本作用域名称绑定会遮蔽宏的基于路径的作用域绑定。

```rust
macro_rules! m2 {
    () => {
        println!("m2");
    };
}

// 解析为下方 use 声明中的基于路径的候选。
m!(); // 打印 "m2\n"

// 引入第二个具有文本作用域的 `m` 候选。
//
// 这将遮蔽本示例其余部分中来自下方的基于路径的候选。
macro_rules! m {
    () => {
        println!("m");
    };
}

// 引入 `m2` 宏作为基于路径的候选。
//
// 此项在此整个示例中均在作用域内，而不仅仅在 use 声明下方。
use m2 as m;

// 解析为 use 声明上方具有文本作用域的宏候选。
m!(); // 打印 "m\n"
```

> [!NOTE]
> 有关不允许遮蔽的区域，请参见[名称解析歧义][name resolution ambiguities]。

r[macro.decl.scope.path-based]
### 基于路径的作用域

r[macro.decl.scope.path-based.intro]
默认情况下，宏没有基于路径的作用域。宏可以通过两种方式获得基于路径的作用域：

- [use声明重导出][Use declaration re-export]
- [`macro_export`][macro.decl.scope.macro_export]

r[macro.decl.scope.path.reexport]
宏可以被重导出，以使其获得来自除 crate 根模块之外的模块的基于路径的作用域。

```rust
mac::m!(); // 正常：基于路径的查找在 mac 模块中找到 `m`。

mod mac {
    // 引入具有文本作用域的宏 `m`。
    macro_rules! m {
        () => {};
    }

    // 从 `m` 的文本作用域内重导出具有基于路径的作用域。
    pub(crate) use m;
}
```

r[macro.decl.scope.path-based.visibility]
宏具有隐式的 `pub(crate)` 可见性。`#[macro_export]` 将隐式可见性更改为 `pub`。

```rust
// 隐式可见性为 `pub(crate)`。
macro_rules! private_m {
    () => {};
}

// 隐式可见性为 `pub`。
#[macro_export]
macro_rules! pub_m {
    () => {};
}

pub(crate) use private_m as private_macro; // 正常。
pub use pub_m as pub_macro; // 正常。
```

```rust,compile_fail,E0364
# // 隐式可见性为 `pub(crate)`。
# macro_rules! private_m {
#     () => {};
# }
#
# // 隐式可见性为 `pub`。
# #[macro_export]
# macro_rules! pub_m {
#     () => {};
# }
#
# pub(crate) use private_m as private_macro; // 正常。
# pub use pub_m as pub_macro; // 正常。
#
pub use private_m; // 错误：`private_m` 仅在该 crate 内可见
                   // 且不能在外部重导出。
```

<!-- template:attributes -->
r[macro.decl.scope.macro_use]
### `macro_use`属性

r[macro.decl.scope.macro_use.intro]
*`macro_use` [属性][attributes]* 有两个目的：它可以用于模块，以扩展其中定义的宏的作用域；它也可以用于 [`extern crate`][items.extern-crate]，将其他 crate 中的宏导入到[`macro_use`预导入][`macro_use` prelude]中。

> [!EXAMPLE]
> ```rust
> #[macro_use]
> mod inner {
>     macro_rules! m {
>         () => {};
>     }
> }
> m!(); // 正常
> ```
>
> ```rust,ignore
> #[macro_use]
> extern crate log;
> ```

r[macro.decl.scope.macro_use.syntax]
当用于模块时，`macro_use` 属性使用 [MetaWord] 语法。

当用于 `extern crate` 时，它使用 [MetaWord] 和 [MetaListIdents] 语法。有关这些语法如何使用的更多信息，请参见[macro.decl.scope.macro_use.prelude]。

r[macro.decl.scope.macro_use.allowed-positions]
`macro_use` 属性可以应用于模块或 `extern crate`。

> [!NOTE]
> `rustc` 会忽略在其他位置的使用，但会对其进行 lint。这在将来可能会成为错误。

r[macro.decl.scope.macro_use.extern-crate-self]
`macro_use` 属性不能用于[`extern crate self`][items.extern-crate.self]。

r[macro.decl.scope.macro_use.duplicates]
`macro_use` 属性可以在一个形式上使用任意次数。

可以指定 [MetaListIdents] 语法中的 `macro_use` 多个实例。所有指定的宏的并集将被导入。

> [!NOTE]
> 在模块上，`rustc` 会对第一个 [MetaWord] `macro_use` 属性之后的任何 `macro_use` 属性进行 lint。
>
> 在 `extern crate` 上，`rustc` 会对任何由于未导入已被另一个 `macro_use` 属性导入的宏而没有效果的 `macro_use` 属性进行 lint。如果两个或更多 [MetaListIdents] `macro_use` 属性导入相同的宏，则会对第一个进行 lint。如果存在任何 [MetaWord] `macro_use` 属性，则会对所有 [MetaListIdents] `macro_use` 属性进行 lint。如果存在两个或更多 [MetaWord] `macro_use` 属性，则会对第一个之后的属性进行 lint。

r[macro.decl.scope.macro_use.mod-decl]
当 `macro_use` 用于模块时，模块的宏作用域会超出模块的词法作用域。

> [!EXAMPLE]
> ```rust
> #[macro_use]
> mod inner {
>     macro_rules! m {
>         () => {};
>     }
> }
> m!(); // 正常
> ```

r[macro.decl.scope.macro_use.prelude]
在 crate 根目录中的 `extern crate` 声明上指定 `macro_use` 会从该 crate 导入导出的宏。

以这种方式导入的宏被导入到[`macro_use`预导入][`macro_use` prelude]中，而不是文本式地，这意味着它们可以被任何其他名称遮蔽。`macro_use` 导入的宏可以在 import 语句之前使用。

> [!NOTE]
> `rustc` 目前在冲突时会优先选择最后导入的宏。请不要依赖此行为。这种行为很不寻常，因为 Rust 中的导入通常与顺序无关。`macro_use` 的这种行为在将来可能会改变。
>
> 有关详细信息，请参见 [Rust问题单#148025](https://github.com/rust-lang/rust/issues/148025)。

当使用 [MetaWord] 语法时，所有导出的宏都会被导入。当使用 [MetaListIdents] 语法时，只有指定的宏会被导入。

> [!EXAMPLE]
> <!-- ignore: requires external crates -->
> ```rust,ignore
> #[macro_use(lazy_static)] // 或者 `#[macro_use]` 导入所有宏。
> extern crate lazy_static;
>
> lazy_static!{}
> // self::lazy_static!{} // 错误：`lazy_static` 未在 `self` 中定义。
> ```

r[macro.decl.scope.macro_use.export]
要使用 `macro_use` 导入的宏必须使用 [`macro_export`][macro.decl.scope.macro_export] 导出。

<!-- template:attributes -->
r[macro.decl.scope.macro_export]
### `macro_export`属性

r[macro.decl.scope.macro_export.intro]
*`macro_export` [属性][attributes]* 将宏从 crate 导出，并使其在 crate 根目录中可用，以进行基于路径的解析。

> [!EXAMPLE]
> ```rust
> self::m!();
> //  ^^^^ 正常：基于路径的查找在当前模块中找到 `m`。
> m!(); // 同上。
>
> mod inner {
>     super::m!();
>     crate::m!();
> }
>
> mod mac {
>     #[macro_export]
>     macro_rules! m {
>         () => {};
>     }
> }
> ```

r[macro.decl.scope.macro_export.syntax]
`macro_export` 属性使用 [MetaWord] 和 [MetaListIdents] 语法。使用 [MetaListIdents] 语法时，它接受单个 [`local_inner_macros`][macro.decl.scope.macro_export.local_inner_macros] 值。

r[macro.decl.scope.macro_export.allowed-positions]
`macro_export` 属性可以应用于 `macro_rules` 定义。

> [!NOTE]
> `rustc` 会忽略在其他位置的使用，但会对其进行 lint。这在将来可能会成为错误。

r[macro.decl.scope.macro_export.duplicates]
宏上只有第一次使用 `macro_export` 有效。

> [!NOTE]
> `rustc` 会对第一次使用之后的任何使用进行 lint。

r[macro.decl.scope.macro_export.path-based]
默认情况下，宏只有[文本作用域][macro.decl.scope.textual]，不能通过路径解析。当使用 `macro_export` 属性时，该宏会在 crate 根目录中可用，并可以通过其路径引用。

> [!EXAMPLE]
> 没有 `macro_export` 时，宏只有文本作用域，因此宏的基于路径的解析会失败。
>
> ```rust,compile_fail,E0433
> macro_rules! m {
>     () => {};
> }
> self::m!(); // 错误
> crate::m!(); // 错误
> # fn main() {}
> ```
>
> 有了 `macro_export`，基于路径的解析就有效了。
>
> ```rust
> #[macro_export]
> macro_rules! m {
>     () => {};
> }
> self::m!(); // 正常
> crate::m!(); // 正常
> # fn main() {}
> ```

r[macro.decl.scope.macro_export.export]
`macro_export` 属性导致宏从 crate 根目录导出，以便其他 crate 可以通过路径引用它。

> [!EXAMPLE]
> 给定 `log` crate 中的以下内容：
>
> ```rust
> #[macro_export]
> macro_rules! warn {
>     ($message:expr) => { eprintln!("WARN: {}", $message) };
> }
> ```
>
> 从另一个crate中，你可以通过路径引用宏：
>
> <!-- ignore: requires external crates -->
> ```rust,ignore
> fn main() {
>     log::warn!("example warning");
> }
> ```

r[macro.decl.scope.macro_export.macro_use]
`macro_export` 允许在 `extern crate` 上使用 [`macro_use`][macro.decl.scope.macro_use]，将宏导入到 [`macro_use`预导入][`macro_use` prelude]中。

> [!EXAMPLE]
> 给定 `log` crate 中的以下内容：
>
> ```rust
> #[macro_export]
> macro_rules! warn {
>     ($message:expr) => { eprintln!("WARN: {}", $message) };
> }
> ```
>
> 在依赖 crate 中使用 `macro_use` 允许你从预导入中使用宏：
>
> <!-- ignore: requires external crates -->
> ```rust,ignore
> #[macro_use]
> extern crate log;
>
> pub mod util {
>     pub fn do_thing() {
>         // 通过宏预导入解析。
>         warn!("example warning");
>     }
> }
> ```

r[macro.decl.scope.macro_export.local_inner_macros]
将 `local_inner_macros` 添加到 `macro_export` 属性会使宏定义中的所有单段宏调用具有隐式的 `$crate::` 前缀。

> [!NOTE]
> 这主要旨在作为一种工具，用于迁移在语言中添加 [`$crate`] 之前编写的代码，以使其与 Rust 2018 基于路径的宏导入协同工作。不建议在新代码中使用它。

> [!EXAMPLE]
> ```rust
> #[macro_export(local_inner_macros)]
> macro_rules! helped {
>     () => { helper!() } // 自动转换为 $crate::helper!()。
> }
>
> #[macro_export]
> macro_rules! helper {
>     () => { () }
> }
> ```

r[macro.decl.hygiene]
## 卫生

r[macro.decl.hygiene.intro]
声明宏具有 _混合站点卫生_ 。这意味着[循环标签][loop labels]、[块标签][block labels]和局部变量在宏定义站点查找，而其他符号在宏调用站点查找。例如：

```rust
let x = 1;
fn func() {
    unreachable!("this is never called")
}

macro_rules! check {
    () => {
        assert_eq!(x, 1); // 使用定义站点的 `x`。
        func();           // 使用调用站点的 `func`。
    };
}

{
    let x = 2;
    fn func() { /* 不会恐慌 */ }
    check!();
}
```

在宏展开中定义的标签和局部变量不共享于不同的调用之间，因此此代码无法编译：

```rust,compile_fail,E0425
macro_rules! m {
    (define) => {
        let x = 1;
    };
    (refer) => {
        dbg!(x);
    };
}

m!(define);
m!(refer);
```

r[macro.decl.hygiene.crate]
一个特殊情况是 `$crate` 元变量。它指代定义宏的 crate，可以用于路径的开头，以查找在调用站点不在作用域内的项或宏。

<!-- ignore: requires external crates -->
```rust,ignore
//// `helper_macro` crate 中的定义。
#[macro_export]
macro_rules! helped {
    // () => { helper!() } // 这可能会因 'helper' 不在作用域中而导致错误。
    () => { $crate::helper!() }
}

#[macro_export]
macro_rules! helper {
    () => { () }
}

//// 在另一个 crate 中的使用。
// 请注意，`helper_macro::helper` 并未导入！
use helper_macro::helped;

fn unit() {
    helped!();
}
```

请注意，由于 `$crate` 指代当前 crate，因此在引用非宏项时必须与完全限定的模块路径一起使用：

```rust
pub mod inner {
    #[macro_export]
    macro_rules! call_foo {
        () => { $crate::inner::foo() };
    }

    pub fn foo() {}
}
```

r[macro.decl.hygiene.vis]
此外，即使 `$crate` 允许宏在展开时引用其自身 crate 内的项，其使用对可见性也没有影响。被引用的项或宏仍必须从调用站点可见。在下面的示例中，任何尝试从其 crate 外部调用 `call_foo!()` 的操作都会失败，因为 `foo()` 不是公共的。

```rust
#[macro_export]
macro_rules! call_foo {
    () => { $crate::foo() };
}

fn foo() {}
```

> [!NOTE]
> 在 Rust 1.30 之前，不支持 `$crate` 和 [`local_inner_macros`][macro.decl.scope.macro_export.local_inner_macros]。它们是与[基于路径的宏导入][macro.decl.scope.macro_export]一起添加的，以确保辅助宏不需要由导出宏的 crate 用户手动导入。为早期 Rust 版本编写的、使用辅助宏的 crate 需要修改以使用 `$crate` 或 `local_inner_macros`，才能与基于路径的导入良好协作。

r[macro.decl.follow-set]
## 后续集歧义限制

r[macro.decl.follow-set.intro]
宏系统使用的解析器功能相当强大，但它受到限制，以防止在语言的当前或未来版本中出现歧义。

具体来说，除了关于歧义展开的规则之外，元变量匹配的非终结符之后必须跟随一个已被确定可以安全地用于该类匹配的词法单元。

例如，像 `$i:expr [ , ]` 这样的宏匹配器理论上今天可以在 Rust 中接受，因为 `[,]` 不能是合法表达式的一部分，因此解析将始终是明确的。然而，由于 `[` 可以开始尾随表达式，`[` 不是一个可以安全地排除在表达式之后出现的字符。如果在后续的 Rust 版本中接受 `[,]`，这个匹配器将变得模糊或解析错误，从而破坏现有代码。然而，像 `$i:expr,` 或 `$i:expr;` 这样的匹配器是合法的，因为 `,` 和 `;` 是合法的表达式分隔符。具体规则是：

r[macro.decl.follow-set.token-expr-stmt]
  * `expr` 和 `stmt` 只能跟随以下之一：`=>`、`,` 或 `;`。

r[macro.decl.follow-set.token-pat_param]
  * `pat_param` 只能跟随以下之一：`=>`、`,`、`=`、`|`、`if` 或 `in`。

r[macro.decl.follow-set.token-pat]
  * `pat` 只能跟随以下之一：`=>`、`,`、`=`、`if` 或 `in`。

r[macro.decl.follow-set.token-path-ty]
  * `path` 和 `ty` 只能跟随以下之一：`=>`、`,`、`=`、`|`、`;`、`:`、`>`、`>>`、`[`、`{`、`as`、`where` 或 `block` 片段说明符的宏变量。

r[macro.decl.follow-set.token-vis]
  * `vis` 只能跟随以下之一：`,`，非原始 `priv` 之外的标识符，任何可以开始类型的词法单元，或具有 `ident`、`ty` 或 `path` 片段说明符的元变量。

r[macro.decl.follow-set.token-other]
  * 所有其他片段说明符都没有限制。

r[macro.decl.follow-set.edition2021]
> [!EDITION-2021]
> 在 2021 版次之前，`pat` 后面也可以跟随 `|`。

r[macro.decl.follow-set.repetition]
当涉及重复时，规则适用于所有可能的展开次数，并考虑分隔符。这意味着：

  * 如果重复包含分隔符，则该分隔符必须能够跟随重复的内容。
  * 如果重复可以多次重复（`*` 或 `+`），则内容必须能够跟随自身。
  * 重复的内容必须能够跟随其之前的内容，并且其之后的内容必须能够跟随重复的内容。
  * 如果重复可以匹配零次（`*` 或 `?`），则其之后的内容必须能够跟随其之前的内容。

有关更多详细信息，请参见[形式化规范][formal specification]。

[Metavariables]: #元变量
[Repetitions]: #重复
[`macro_export`]: #`macro_export`属性
[`$crate`]: macro.decl.hygiene.crate
[`extern crate self`]: items.extern-crate.self
[`macro_use` prelude]: names/preludes.md#macro_use-prelude
[block labels]: expr.loop.block-labels
[delimiters]: tokens.md#分隔符
[formal specification]: macro-ambiguity.md
[loop labels]: expressions/loop-expr.md#loop-labels
[name resolution ambiguities]: names/name-resolution.md#r-names.resolution.expansion.imports.ambiguity
[token]: tokens.md
[use declaration re-export]: items/use-declarations.md#use可见性
