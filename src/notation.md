<!-- https://github.com/rust-lang/reference/blob/master/src/notation.md -->
<!-- commit 68bdfd15fa9feebfbf94a06cf574de598e239198 -->

r[notation]
# 记法

r[notation.grammar]
## 语法

r[notation.grammar.syntax]

以下记法用于*词法分析器*和*语法格式*文法片段：

| 记法          | 示例                      | 含义                                   | 译者注 |
|-------------------|-------------------------------|-------------------------------------------|----|
| CAPITAL           | KW_IF, INTEGER_LITERAL        | 词法分析器生成的词法单元             | 大写 |
| _ItalicCamelCase_ | _LetStatement_, _Item_        | 一个语法产生式                  | 斜体、驼峰方式大小写(单词首字母大写) |
| `string`          | `x`, `while`, `*`             | 确切的字符(或字符串)                  | |
| x<sup>?</sup>     | `pub`<sup>?</sup>             | 一个可选的项                          | |
| x<sup>\*</sup>    | _OuterAttribute_<sup>\*</sup> | 0个或多个x                            | |
| x<sup>+</sup>     |  _MacroMatch_<sup>+</sup>     | 1个或多个x                            | |
| x<sup>a..b</sup>  | HEX_DIGIT<sup>1..6</sup>      | a到b次重复x                   | |
| Rule1 Rule2       | `fn` _Name_ _Parameters_      | 规则的顺序序列                | |
| \|                | `u8` \| `u16`, Block \| Item  | 二者之一                     | |
| \[ ]               | \[`b` `B`]                     | 列出的任意字符               | |
| \[ - ]             | \[`a`-`z`]                     | 范围内的任意字符             | |
| ~\[ ]              | ~\[`b` `B`]                    | 任意字符，除了列出的       | |
| ~`string`         | ~`\n`, ~`*/`                  | 任意字符，除了此序列      | |
| ( )               | (`,` _Parameter_)<sup>?</sup> | 分组项                             | |
| U+xxxx            | U+0060                        | 一个Unicode字符                | |
| \<text\>          | \<any ASCII char except CR\>  | 一个匹配内容的英文描述 | |
| Rule <sub>suffix</sub> | IDENTIFIER_OR_KEYWORD <sub>_except `crate`_</sub> | 对前一个规则的修改 | |
| // Comment. | // Single line comment. | 延伸到行尾的注释。 | |

序列的优先级高于`|`选项。

r[notation.grammar.string-tables]
### 字符串表产生式

语法中的某些规则 —— 特别是[一元运算符][unary operators]、[二元运算符][binary operators]和[关键字][keywords] —— 以简化形式给出：作为可打印字符串的列表。这些情况构成了关于[词法单元][tokens]规则的一个子集，并被认为是词法分析阶段的结果，该阶段由一个<abbr title="确定性有限自动机">DFA</abbr>驱动，作用于所有此类字符串表条目的析取，从而为解析器提供输入。

当语法中出现`等宽字体`的字符串时，它隐式引用了此类字符串表产生式的一个成员。有关更多信息，请参阅[词法单元][tokens]。

r[notation.grammar.visualizations]
### 语法可视化

在每个语法块下方都有一个按钮，用于切换[语法格式图][syntax diagram]的显示。一个方形元素是一个非终结规则，一个圆角矩形是一个终结符。

[binary operators]: expressions/operator-expr.md#算术和逻辑二元运算符
[keywords]: keywords.md
[syntax diagram]: https://en.wikipedia.org/wiki/Syntax_diagram
[tokens]: tokens.md
[unary operators]: expressions/operator-expr.md#借用运算符