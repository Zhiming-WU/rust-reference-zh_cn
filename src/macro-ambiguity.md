r[macro.ambiguity]
# 附录：宏后继集歧义形式化规范

本页记录了 [声明宏][Macros By Example] 后继规则的形式化规范。它们最初在 [RFC 550] 中指定，本文的大部分内容即复制自该 RFC，并在后续的 RFC 中进行了扩展。

r[macro.ambiguity.convention]
## 定义和约定

r[macro.ambiguity.convention.defs]
  - `macro`: 源代码中可调用为 `foo!(...)` 的任何内容。
  - `MBE`: 声明宏，由 `macro_rules` 定义的宏。
  - `matcher`: `macro_rules` 调用中规则的左侧，或其子部分。
  - `macro 解析器`: Rust 解析器中用于使用所有匹配器派生出的语法格式来解析输入的代码片段。
  - `fragment`: 给定 匹配器 将接受（或“匹配”）的 Rust 语法类别。
  - `repetition`: 遵循常规重复模式的片段。
  - `NT`: 非终结符，出现在 匹配器 中的各种“元变量”或重复匹配器，在 MBE 语法格式中以 `$` 字符开头。
  - `simple NT`: “元变量”非终结符（下文将进一步讨论）。
  - `complex NT`: 重复匹配非终结符，通过重复运算符（`*`、`+`、`?`）指定。
  - `token`: 匹配器的原子元素；即 标识符、运算符、开/闭分隔符，*以及* 简单 NT。
  - `token tree`: 由 词法单元（叶子）、复杂 NT 和有限 词法单元树 序列组成的树结构。
  - `delimiter token`: 用于分隔一个片段的结尾和下一个片段的开头的 词法单元。
  - `separator token`: 复杂 NT 中可选的 分隔符 词法单元，用于分隔匹配重复中的每对元素。
  - `separated complex NT`: 具有自己 分隔符 词法单元 的复杂 NT。
  - `delimited sequence`: 以适当的开闭分隔符开始和结束的 词法单元树 序列。
  - `empty fragment`: 分隔 词法单元 的不可见 Rust 语法格式类别，即 空白符，或（在某些词法上下文中）空 词法单元 序列。
  - `fragment specifier`: 简单 NT 中指定 NT 接受哪个片段的 标识符。
  - `language`: 上下文无关语言。

示例：

```rust,compile_fail
macro_rules! i_am_an_mbe {
    (start $foo:expr $($i:ident),* end) => ($foo)
}
```

r[macro.ambiguity.convention.matcher]
`(start $foo:expr $($i:ident),* end)` 是一个 匹配器。整个 匹配器 是一个 分隔序列（带有开闭分隔符 `(` 和 `)`），`$foo` 和 `$i` 是带有 `expr` 和 `ident` 作为其各自 片段指定符 的 简单 NT。

r[macro.ambiguity.convention.complex-nt]
`$(i:ident),*` *也是* 一个 NT；它是一个 复杂 NT，匹配逗号分隔的 标识符 重复。`,` 是 复杂 NT 的 分隔符 词法单元；它出现在匹配片段的每对元素（如果存在）之间。

复杂 NT 的另一个例子是 `$(hi $e:expr ;)+`，它匹配 `hi <expr>; hi <expr>; ...` 形式的任何片段，其中 `hi <expr>;` 至少出现一次。请注意，此 复杂 NT 没有专用的 分隔符 词法单元。

（请注意，Rust 的 解析器 确保 分隔序列 总是以适当嵌套的 词法单元树 结构和正确的开闭分隔符匹配出现。）

r[macro.ambiguity.convention.vars]
我们将倾向于使用变量 “M” 代表 匹配器，变量 “t” 和 “u” 代表任意单个 词法单元，变量 “tt” 和 “uu” 代表任意 词法单元树。（“tt” 的使用确实可能与它作为 片段指定符 的额外角色产生歧义；但从上下文中可以清楚地看出其意图。）

r[macro.ambiguity.convention.set]
“SEP” 将表示 分隔符 词法单元，“OP” 表示重复运算符 `*`、`+` 和 `?`，“OPEN”/“CLOSE” 表示围绕 分隔序列 的匹配 词法单元 对（例如 `[` 和 `]`）。

r[macro.ambiguity.convention.sequence-vars]
希腊字母 “α”、“β”、“γ”、“δ” 代表可能为空的 词法单元树 序列。（但是，希腊字母 “ε”（epsilon）在表达中具有特殊作用，不代表 词法单元树 序列。）

  * 这种希腊字母约定通常仅在序列的存在是技术细节时使用；特别是，当我们希望 *强调* 我们正在对 词法单元树 序列进行操作时，我们将使用符号 “tt ...” 代表该序列，而不是希腊字母。

请注意，匹配器 仅仅是 词法单元树。如上所述，“简单 NT” 是一个元变量 NT；因此它不是一个 重复。例如，`$foo:ty` 是一个 简单 NT，但 `$($foo:ty)+` 是一个 复杂 NT。

另请注意，在此形式化上下文中，“词法单元” 一词通常 *包含* 简单 NT。

最后，读者需要记住，根据此形式化的定义，没有 简单 NT 匹配 空片段，同样也没有 词法单元 匹配 Rust 语法格式的 空片段。（因此，*唯一* 可以匹配 空片段 的 NT 是 复杂 NT。）这实际上不是真的，因为 `vis` 匹配器 可以匹配一个 空片段。因此，为了形式化的目的，我们将把 `$v:vis` 视为 `$($v:vis)?`，并要求 匹配器 匹配一个 空片段。

r[macro.ambiguity.invariant]
### 匹配器不变式

r[macro.ambiguity.invariant.list]
要有效，匹配器 必须满足以下三个 不变式。FIRST 和 FOLLOW 的定义将在后面描述。

1.  在 匹配器 `M` 中，对于任意两个连续的 词法单元树 序列（即 `M = ... tt uu ...`），且 `uu ...` 非空，我们必须有 FOLLOW(`... tt`) ∪ {ε} ⊇ FIRST(`uu ...`)。
1.  在 匹配器 中，对于任何 分隔复杂 NT，`M = ... $(tt ...) SEP OP ...`，我们必须有 `SEP` ∈ FOLLOW(`tt ...`)。
1.  在 匹配器 中，对于未分隔的 复杂 NT，`M = ... $(tt ...) OP ...`，如果 OP = `*` 或 `+`，我们必须有 FOLLOW(`tt ...`) ⊇ FIRST(`tt ...`)。

r[macro.ambiguity.invariant.follow-matcher]
第一个 不变式 表示，无论 匹配器 之后实际出现什么 词法单元（如果存在），它都必须在预定的 后继集 中。这确保了合法的宏定义将继续分配相同的确定，即 `... tt` 在哪里结束，`uu ...` 在哪里开始，即使在语言中添加了新的 语法格式 形式。

r[macro.ambiguity.invariant.separated-complex-nt]
第二个 不变式 表示，分隔复杂 NT 必须使用一个 分隔符 词法单元，该 词法单元 是 NT 内部内容的预定 后继集 的一部分。这确保了合法的宏定义将继续将输入片段解析为相同的 `tt ...` 分隔序列，即使在语言中添加了新的 语法格式 形式。

r[macro.ambiguity.invariant.unseparated-complex-nt]
第三个 不变式 表示，当我们有一个 复杂 NT 可以匹配两个或多个相同且没有分隔符 的实体时，必须允许它们根据第一个 不变式 放置在一起。这个 不变式 还要求它们非空，这消除了可能的歧义。

**注意：由于历史遗留问题和对行为的严重依赖，第三个 不变式 目前未强制执行。目前尚未决定下一步如何处理。不遵守该行为的宏在未来的 Rust 版次 中可能会变得无效。参见 [跟踪 issue][tracking issue]。**

r[macro.ambiguity.sets]
### FIRST和FOLLOW，非正式地

r[macro.ambiguity.sets.intro]
给定 匹配器 M 映射到三个集合：FIRST(M)、LAST(M) 和 FOLLOW(M)。

这三个集合中的每一个都由 词法单元 组成。FIRST(M) 和 LAST(M) 还可以包含一个特殊的非 词法单元 元素 ε (“epsilon”)，它表示 M 可以匹配 空片段。（但 FOLLOW(M) 始终只是 词法单元 集合。）

非正式地：

r[macro.ambiguity.sets.first]
  * FIRST(M): 收集在将片段匹配到 M 时可能首先使用的 词法单元。

r[macro.ambiguity.sets.last]
  * LAST(M): 收集在将片段匹配到 M 时可能最后使用的 词法单元。

r[macro.ambiguity.sets.follow]
  * FOLLOW(M): 允许紧跟在 M 匹配的某个片段之后的 词法单元 集合。

    换句话说：当且仅当存在（可能为空） 词法单元 序列 α、β、γ、δ，其中：

      * M 匹配 β，

      * t 匹配 γ，并且

      * 拼接 α β γ δ 是一个可解析的 Rust 程序，

    则 t ∈ FOLLOW(M)。

r[macro.ambiguity.sets.universe]
我们使用简写 ANYTOKEN 表示所有 词法单元（包括 简单 NT）的集合。例如，如果任何 词法单元 在 匹配器 M 之后都是合法的，那么 FOLLOW(M) = ANYTOKEN。

（为了回顾对上述非正式描述的理解，读者此时可能希望跳到 [FIRST/LAST 示例](#examples-of-first-and-last) 之前阅读它们的正式定义。）

r[macro.ambiguity.sets.def]
### FIRST，LAST

r[macro.ambiguity.sets.def.intro]
以下是 FIRST 和 LAST 的形式化归纳定义。

r[macro.ambiguity.sets.def.notation]
“A ∪ B” 表示集合并集，“A ∩ B” 表示集合交集，“A \ B” 表示集合差集（即 A 中所有不在 B 中的元素）。

r[macro.ambiguity.sets.def.first]
#### FIRST

r[macro.ambiguity.sets.def.first.intro]
FIRST(M) 根据序列 M 及其第一个 词法单元树（如果存在）的结构进行案例分析定义：

r[macro.ambiguity.sets.def.first.epsilon]
  * 如果 M 是空序列，则 FIRST(M) = { ε }，

r[macro.ambiguity.sets.def.first.token]
  * 如果 M 以 词法单元 t 开头，则 FIRST(M) = { t }，

    （注意：这涵盖了 M 以 分隔词法单元树 序列 `M = OPEN tt ... CLOSE ...` 开头的情况，在这种情况下 `t = OPEN`，因此 FIRST(M) = { `OPEN` }。）

    （注意：这严重依赖于没有 简单 NT 匹配 空片段 的属性。）

r[macro.ambiguity.sets.def.first.complex]
  * 否则，M 是以 复杂 NT 开头的 词法单元树 序列：`M = $( tt ... ) OP α`，或 `M = $( tt ... ) SEP OP α`（其中 `α` 是 匹配器 剩余部分的（可能为空） 词法单元树 序列）。

      * 如果 SEP 存在且 ε ∈ FIRST(`tt ...`)，则令 SEP\_SET(M) = { SEP }；否则 SEP\_SET(M) = {}。

  * 如果 OP = `*` 或 `?`，则令 ALPHA\_SET(M) = FIRST(`α`)；如果 OP = `+`，则 ALPHA\_SET(M) = {}。
  * FIRST(M) = (FIRST(`tt ...`) \ {ε}) ∪ SEP\_SET(M) ∪ ALPHA\_SET(M)。

复杂 NT 的定义值得一些解释。SEP\_SET(M) 定义了分隔符可能作为 M 的有效第一个 词法单元 的可能性，当存在分隔符且重复片段可能为空时，就会发生这种情况。ALPHA\_SET(M) 定义了 复杂 NT 可能为空的可能性，这意味着 M 的有效第一个 词法单元 是紧随其后的 词法单元树 序列 `α` 的。这发生在使用了 `*` 或 `?` 的情况下，此时可能存在零次重复。理论上，如果使用 `+` 且重复片段可能为空，也可能发生这种情况，但这被第三个 不变式 禁止。

从那里，显然 FIRST(M) 可以包含来自 SEP\_SET(M) 或 ALPHA\_SET(M) 的任何 词法单元，如果 复杂 NT 匹配非空，那么任何以 FIRST(`tt ...`) 开头的 词法单元 也可以。最后要考虑的是 ε。SEP\_SET(M) 和 FIRST(`tt ...`) \ {ε} 不能包含 ε，但 ALPHA\_SET(M) 可以。因此，此定义允许 M 接受 ε 当且仅当 ε ∈ ALPHA\_SET(M) 时。这是正确的，因为对于 复杂 NT 情况下的 M 要接受 ε，复杂 NT 和 α 都必须接受它。如果 OP = `+`，这意味着 复杂 NT 不能为空，那么根据定义 ε ∉ ALPHA\_SET(M)。否则，复杂 NT 可以接受零次重复，然后 ALPHA\_SET(M) = FOLLOW(`α`)。因此，此定义对于 \varepsilon 也是正确的。

r[macro.ambiguity.sets.def.last]
#### LAST

r[macro.ambiguity.sets.def.last.intro]
LAST(M) 根据 M 本身（ 词法单元树 序列）进行案例分析定义：

r[macro.ambiguity.sets.def.last.empty]
  * 如果 M 是空序列，则 LAST(M) = { ε }

r[macro.ambiguity.sets.def.last.token]
  * 如果 M 是单例 词法单元 t，则 LAST(M) = { t }

r[macro.ambiguity.sets.def.last.rep-star]
  * 如果 M 是重复零次或多次的单例 复杂 NT，`M = $( tt ... ) *`，或 `M = $( tt ... ) SEP *`

      * 如果 SEP 存在，则令 sep\_set = { SEP }；否则 sep\_set = {}。

      * 如果 ε ∈ LAST(`tt ...`)，则 LAST(M) = LAST(`tt ...`) ∪ sep\_set

      * 否则，序列 `tt ...` 必须非空；LAST(M) = LAST(`tt ...`) ∪ {ε}。

r[macro.ambiguity.sets.def.last.rep-plus]
  * 如果 M 是重复一次或多次的单例 复杂 NT，`M = $( tt ... ) +`，或 `M = $( tt ... ) SEP +`

      * 如果 SEP 存在，则令 sep\_set = { SEP }；否则 sep\_set = {}。

      * 如果 ε ∈ LAST(`tt ...`)，则 LAST(M) = LAST(`tt ...`) ∪ sep\_set

      * 否则，序列 `tt ...` 必须非空；LAST(M) = LAST(`tt ...`)

r[macro.ambiguity.sets.def.last.rep-question]
  * 如果 M 是重复零次或一次的单例 复杂 NT，`M = $( tt ...) ?`，则 LAST(M) = LAST(`tt ...`) ∪ {ε}。

r[macro.ambiguity.sets.def.last.delim]
  * 如果 M 是 分隔词法单元树 序列 `OPEN tt ... CLOSE`，则 LAST(M) = { `CLOSE` }。

r[macro.ambiguity.sets.def.last.sequence]
  * 如果 M 是非空 词法单元树 序列 `tt uu ...`，

      * 如果 ε ∈ LAST(`uu ...`)，则 LAST(M) = LAST(`tt`) ∪ (LAST(`uu ...`) \ { ε })。

      * 否则，序列 `uu ...` 必须非空；则 LAST(M) = LAST(`uu ...`)。

### FIRST和LAST的示例

以下是 FIRST 和 LAST 的一些示例。
（请特别注意 ε 元素是如何根据输入片段之间的交互引入和消除的。）

我们的第一个示例以树形结构呈现，以详细说明 匹配器 的分析如何组合。（一些更简单的子树已被省略。）

```text
INPUT:  $(  $d:ident   $e:expr   );*    $( $( h )* );*    $( f ; )+   g
            ~~~~~~~~   ~~~~~~~                ~
                |         |                   |
FIRST:   { $d:ident }  { $e:expr }          { h }


INPUT:  $(  $d:ident   $e:expr   );*    $( $( h )* );*    $( f ; )+
            ~~~~~~~~~~~~~~~~~~             ~~~~~~~           ~~~
                        |                      |               |
FIRST:          { $d:ident }               { h, ε }         { f }

INPUT:  $(  $d:ident   $e:expr   );*    $( $( h )* );*    $( f ; )+   g
        ~~~~~~~~~~~~~~~~~~~~~~~~~~~~    ~~~~~~~~~~~~~~    ~~~~~~~~~   ~
                        |                       |              |       |
FIRST:        { $d:ident, ε }            {  h, ε, ;  }      { f }   { g }


INPUT:  $(  $d:ident   $e:expr   );*    $( $( h )* );*    $( f ; )+   g
        ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
                                        |
FIRST:                       { $d:ident, h, ;,  f }
```

因此：

 * FIRST(`$($d:ident $e:expr );* $( $(h)* );* $( f ;)+ g`) = { `$d:ident`, `h`, `;`, `f` }

但请注意：

 * FIRST(`$($d:ident $e:expr );* $( $(h)* );* $($( f ;)+ g)*`) = { `$d:ident`, `h`, `;`, `f`, ε }

以下是类似的示例，但现在针对 LAST。

 * LAST(`$d:ident $e:expr`) = { `$e:expr` }
 * LAST(`$( $d:ident $e:expr );*`) = { `$e:expr`, ε }
 * LAST(`$( $d:ident $e:expr );* $(h)*`) = { `$e:expr`, ε, `h` }
 * LAST(`$( $d:ident $e:expr );* $(h)* $( f ;)+`) = { `;` }
 * LAST(`$( $d:ident $e:expr );* $(h)* $( f ;)+ g`) = { `g` }

r[macro.ambiguity.sets.def.follow]
### FOLLOW(M)

r[macro.ambiguity.sets.def.follow.intro]
最后，FOLLOW(M) 的定义如下构建。pat, expr 等表示具有给定 片段指定符 的 简单 非终结符。

r[macro.ambiguity.sets.def.follow.pat]
  * FOLLOW(pat) = {`=>`, `,`, `=`, `|`, `if`, `in`}。

r[macro.ambiguity.sets.def.follow.expr-stmt]
  * FOLLOW(expr) = FOLLOW(expr_2021) = FOLLOW(stmt) = {`=>`, `,`, `;`}。

r[macro.ambiguity.sets.def.follow.ty-path]
  * FOLLOW(ty) = FOLLOW(path) = {`{`, `[`, `,`, `=>`, `:`, `=`, `>`, `>>`, `;`,
    `|`, `as`, `where`, 块 非终结符}。

r[macro.ambiguity.sets.def.follow.vis]
  * FOLLOW(vis) = {`,`l 任何 关键字 或 标识符，除了非原始 `priv`；任何可以开始一个 类型 的 词法单元；标识符， 类型 和 路径 非终结符}。

r[macro.ambiguity.sets.def.follow.simple]
  * FOLLOW(t) = ANYTOKEN 对于任何其他 简单 词法单元，包括 块，标识符，
    tt， 项，生命周期，字面量 和 元 简单 非终结符，以及所有 终结符。

r[macro.ambiguity.sets.def.follow.other-matcher]
  * FOLLOW(M)，对于任何其他 M，定义为 FOLLOW(t) 的交集，其中 t 遍历 (LAST(M) \ {ε})。

r[macro.ambiguity.sets.def.follow.type-first]
可以开始一个 类型 的 词法单元，截至本文撰写之时，有 {`(`, `[`, `!`, `*`,
`&`, `&&`, `?`, 生命周期，`>`, `>>`, `::`, 任何非 关键字 标识符，`super`,
`self`, `Self`, `extern`, `crate`, `$crate`, `_`, `for`, `impl`, `fn`, `unsafe`,
`typeof`, `dyn`}，尽管此列表可能不完整，因为人们可能不会总是记得在添加新 词法单元 时更新附录。

复杂 M 的 FOLLOW 示例：

 * FOLLOW(`$( $d:ident $e:expr )*`) = FOLLOW(`$e:expr`)
 * FOLLOW(`$( $d:ident $e:expr )* $(;)*`) = FOLLOW(`$e:expr`) ∩ ANYTOKEN = FOLLOW(`$e:expr`)
 * FOLLOW(`$( $d:ident $e:expr )* $(;)* $( f |)+`) = ANYTOKEN

### 有效和无效匹配器的示例

有了上述规范，我们可以给出为什么特定 匹配器 合法而其他不合法的论证。

 * `($ty:ty < foo ,)`：非法，因为 FIRST(`< foo ,`) = { `<` } ⊈ FOLLOW(`ty`)

 * `($ty:ty , foo <)`：合法，因为 FIRST(`, foo <`) = { `,` } ⊆ FOLLOW(`ty`)。

 * `($pa:pat $pb:pat $ty:ty ,)`：非法，因为 FIRST(`$pb:pat $ty:ty ,`) = { `$pb:pat` } ⊈ FOLLOW(`pat`)，并且 FIRST(`$ty:ty ,`) = { `$ty:ty` } ⊈ FOLLOW(`pat`)。

 * `( $($a:tt $b:tt)* ; )`：合法，因为 FIRST(`$b:tt`) = { `$b:tt` } ⊆ FOLLOW(`tt`) = ANYTOKEN，FIRST(`;`) = { `;` } 也是。

 * `( $($t:tt),* , $(t:tt),* )`：合法（尽管任何实际使用此宏的尝试都会在展开期间发出本地歧义错误）。

 * `($ty:ty $(; not sep)* -)`：非法，因为 FIRST(`$(; not sep)* -`) = { `;`, `-` } 不在 FOLLOW(`ty`) 中。

 * `($($ty:ty)-+)`：非法，因为分隔符 `-` 不在 FOLLOW(`ty`) 中。

 * `($($e:expr)*)`：非法，因为 expr NT 不在 FOLLOW(expr NT) 中。

[Macros by Example]: macros-by-example.md
[RFC 550]: https://github.com/rust-lang/rfcs/blob/master/text/0550-macro-future-proofing.md
[tracking issue]: https://github.com/rust-lang/rust/issues/56575
