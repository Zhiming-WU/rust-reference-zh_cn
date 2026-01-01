r[lex.token]
# 词法单元

r[lex.token.syntax]
```grammar,lexer
Token ->
      RESERVED_TOKEN
    | RAW_IDENTIFIER
    | CHAR_LITERAL
    | STRING_LITERAL
    | RAW_STRING_LITERAL
    | BYTE_LITERAL
    | BYTE_STRING_LITERAL
    | RAW_BYTE_STRING_LITERAL
    | C_STRING_LITERAL
    | RAW_C_STRING_LITERAL
    | FLOAT_LITERAL
    | INTEGER_LITERAL
    | LIFETIME_TOKEN
    | PUNCTUATION
    | IDENTIFIER_OR_KEYWORD
```

r[lex.token.intro]
词法单元是语法格式中由正则（非递归）语言定义的原始生成式。Rust源输入可以分解为以下几种词法单元：

* [关键字][Keywords]
* [标识符][identifier]
* [字面量](#字面量)
* [生命周期和循环标签](#生命周期和循环标签)
* [标点符号](#标点符号)
* [分隔符](#分隔符)

在此文档的语法中，“简单”词法单元以[字符串表生成式][string table production]形式给出，并以`等宽字体`显示。

[string table production]: notation.md#字符串表产生式

r[lex.token.literal]
## 字面量

字面量是[字面量表达式][literal expressions]中使用的词法单元。

### 示例

#### 字符和字符串

| | 示例 | `#`&nbsp;数量[^nsets] | 字符 | 转义 |
|---|---|---|---|---|
| [字符字面量](#字符字面量) | `'H'` | 0 | 所有 Unicode | [引号](#引号转义) & [ASCII](#ascii转义) & [Unicode](#unicode转义) |
| [字符串字面量](#字符串字面量) | `"hello"` | 0 | 所有 Unicode | [引号](#引号转义) & [ASCII](#ascii转义) & [Unicode](#unicode转义) |
| [原始字符串字面量](#原始字符串字面量) | `r#"hello"#` | <256 | 所有Unicode | `N/A` |
| [字节字面量](#字节字面量) | `b'H'` | 0 | 所有 ASCII | [引号](#引号转义) & [字节](#字节转义) |
| [字节字符串字面量](#字节字符串字面量) | `b"hello"` | 0 | 所有 ASCII | [引号](#引号转义) & [字节](#字节转义) |
| [原始字节字符串字面量](#原始字节字符串字面量) | `br#"hello"#` | <256 | 所有 ASCII | `N/A` |
| [C字符串字面量](#c字符串字面量) | `c"hello"` | 0 | 所有 Unicode | [引号](#引号转义) & [字节](#字节转义) & [Unicode](#unicode转义) |
| [原始C字符串字面量](#原始c字符串字面量) | `cr#"hello"#` | <256 | 所有 Unicode | `N/A` |

[^nsets]: 相同字面量两侧`#`的数量必须相等。

#### ASCII转义

| | 名称 |
|---|---|
| `\x41` | 7位字符码（恰好2个十六进制数字，最大0x7F） |
| `\n` | 换行 |
| `\r` | 回车 |
| `\t` | Tab键 |
| `\\` | 反斜杠 |
| `\0` | 空字符 |

#### 字节转义

| | 名称 |
|---|---|
| `\x7F` | 8位字符码（恰好2个十六进制数字） |
| `\n` | 换行 |
| `\r` | 回车 |
| `\t` | Tab键 |
| `\\` | 反斜杠 |
| `\0` | 空字符 |

#### Unicode转义

| | 名称 |
|---|---|
| `\u{7FFF}` | 24位Unicode字符码（最多6个十六进制数字） |

#### 引号转义

| | 名称 |
|---|---|
| `\'` | 单引号 |
| `\"` | 双引号 |

#### 数字

| [数字字面量](#数字字面量)[^nl] | 示例 | 幂运算 |
|---|---|---|
| 十进制整数 | `98_222` | `N/A` |
| 十六进制整数 | `0xff` | `N/A` |
| 八进制整数 | `0o77` | `N/A` |
| 二进制整数 | `0b1111_0000` | `N/A` |
| 浮点数 | `123.0E+77` | `可选` |

[^nl]: 所有数字字面量都允许使用`_`作为视觉分隔符：`1_234.0E+18f64`

r[lex.token.literal.suffix]
#### 后缀

r[lex.token.literal.literal.suffix.intro]
后缀是紧跟在字面量主要部分（无中间空白）后面的一串字符，其形式与非原始[标识符][identifier]或[关键字][Keywords]相同。

r[lex.token.literal.suffix.syntax]
```grammar,lexer
SUFFIX -> IDENTIFIER_OR_KEYWORD _except `_`_

SUFFIX_NO_E -> SUFFIX _not beginning with `e` or `E`_
```

r[lex.token.literal.suffix.validity]
任何类型的字面量（字符串、整数等）与任何后缀组合，都可作为有效的词法单元。

带有任何后缀的字面量词法单元可以传递给宏而不会产生错误。宏本身将决定如何解释此类词法单元以及是否产生错误。特别是，声明宏的`literal` fragment specifier匹配带有任意后缀的字面量词法单元。

```rust
macro_rules! blackhole { ($tt:tt) => () }
macro_rules! blackhole_lit { ($l:literal) => () }

blackhole!("string"suffix); // OK
blackhole_lit!(1suffix); // OK
```

r[lex.token.literal.suffix.parse]
然而，在被解释为字面量表达式或模式的字面量词法单元上，后缀是受限制的。非数字字面量词法单元的任何后缀都会被拒绝，而数字字面量词法单元只接受以下列表中的后缀。

| 整数 | 浮点数 |
|---|---|
| `u8`, `i8`, `u16`, `i16`, `u32`, `i32`, `u64`, `i64`, `u128`, `i128`, `usize`, `isize` | `f32`, `f64` |

### 字符和字符串字面量

r[lex.token.literal.char]
#### 字符字面量

r[lex.token.literal.char.syntax]
```grammar,lexer
CHAR_LITERAL ->
    `'`
        ( ~[`'` `\` LF CR TAB] | QUOTE_ESCAPE | ASCII_ESCAPE | UNICODE_ESCAPE )
    `'` SUFFIX?

QUOTE_ESCAPE -> `\'` | `\"`

ASCII_ESCAPE ->
      `\x` OCT_DIGIT HEX_DIGIT
    | `\n` | `\r` | `\t` | `\\` | `\0`

UNICODE_ESCAPE ->
    `\u{` ( HEX_DIGIT `_`* ){1..6} _valid hex char value_ `}`[^valid-hex-char]
```

[^valid-hex-char]: 参阅[lex.token.literal.char-escape.unicode][lex.token.literal.char-escape.unicode]。

r[lex.token.literal.char.intro]
字符字面量是单个Unicode字符，用两个`U+0027`（单引号）字符括起来，但`U+0027`本身除外，它必须通过前面加一个`U+005C`字符（`\`）进行转义。

r[lex.token.literal.str]
#### 字符串字面量

r[lex.token.literal.str.syntax]
```grammar,lexer
STRING_LITERAL ->
    `"` (
        ~[`"` `\` CR]
      | QUOTE_ESCAPE
      | ASCII_ESCAPE
      | UNICODE_ESCAPE
      | STRING_CONTINUE
    )* `"` SUFFIX?

STRING_CONTINUE -> `\` LF
```

r[lex.token.literal.str.intro]
字符串字面量是任何Unicode字符序列，用两个`U+0022`（双引号）字符括起来，但`U+0022`本身除外，它必须通过前面加一个`U+005C`字符（`\`）进行转义。

r[lex.token.literal.str.linefeed]
换行符（由字符`U+000A` (LF)表示）在字符串字面量中是允许的。字符`U+000D` (CR)不得出现在字符串字面量中。当未转义的`U+005C`字符（`\`）紧接在换行符之前时，换行符不会出现在词法单元所表示的字符串中。有关详细信息，请参阅[字符串续行转义][String continuation escapes]。

r[lex.token.literal.char-escape]
#### 字符转义

r[lex.token.literal.char-escape.intro]
在字符或非原始字符串字面量中，还有一些额外的_转义_可用。转义以`U+005C`（`\`）开头，并接以下形式之一：

r[lex.token.literal.char-escape.ascii]
* 7位码点转义以`U+0078`（`x`）开头，后跟恰好两个十六进制数字，值最大为`0x7F`。它表示ASCII字符，其值等于提供的十六进制值。不允许更高的值，因为它们是Unicode码点还是字节值存在歧义。

r[lex.token.literal.char-escape.unicode]
* 24位码点转义以`U+0075`（`u`）开头，后跟最多六个十六进制数字，并用花括号`U+007B`（`{`）和`U+007D`（`}`）括起来。它表示等于所提供十六进制值的Unicode码点。该值必须是有效的Unicode标量值。

r[lex.token.literal.char-escape.whitespace]
* 空白转义是字符`U+006E`（`n`）、`U+0072`（`r`）或`U+0074`（`t`）之一，分别表示Unicode值`U+000A` (LF)、`U+000D` (CR)或`U+0009` (HT)。

r[lex.token.literal.char-escape.null]
* 空转义是字符`U+0030`（`0`），表示Unicode值`U+0000` (NUL)。

r[lex.token.literal.char-escape.slash]
* 反斜杠转义是字符`U+005C`（`\`），它必须被转义才能表示自身。

r[lex.token.literal.str-raw]
#### 原始字符串字面量

r[lex.token.literal.str-raw.syntax]
```grammar,lexer
RAW_STRING_LITERAL -> `r` RAW_STRING_CONTENT SUFFIX?

RAW_STRING_CONTENT ->
      `"` ( ~CR )*? `"`
    | `#` RAW_STRING_CONTENT `#`
```

r[lex.token.literal.str-raw.intro]
原始字符串字面量不处理任何转义。它们以字符`U+0072`（`r`）开头，后跟少于256个`U+0023`（`#`）字符和一个`U+0022`（双引号）字符。

r[lex.token.literal.str-raw.body]
原始字符串体可以包含除`U+000D` (CR)以外的任何Unicode字符序列。它只由另一个`U+0022`（双引号）字符终止，后跟与起始`U+0022`（双引号）字符前相同数量的`U+0023`（`#`）字符。

r[lex.token.literal.str-raw.content]
原始字符串体中包含的所有Unicode字符都表示其本身，字符`U+0022`（双引号）（除非后面紧跟的`U+0023`（`#`）字符数量与用于开始原始字符串字面量的字符数量相同或更多）或`U+005C`（`\`）不具有任何特殊含义。

字符串字面量示例：

```rust
"foo"; r"foo";                     // foo
"\"foo\""; r#""foo""#;             // "foo"

"foo #\"# bar";
r##"foo #"# bar"##;                // foo #"# bar

"\x52"; "R"; r"R";                 // R
"\\x52"; r"\x52";                  // \x52
```

### 字节和字节字符串字面量

r[lex.token.byte]
#### 字节字面量

r[lex.token.byte.syntax]
```grammar,lexer
BYTE_LITERAL ->
    `b'` ( ASCII_FOR_CHAR | BYTE_ESCAPE )  `'` SUFFIX?

ASCII_FOR_CHAR ->
    <any ASCII (i.e. 0x00 to 0x7F) except `'`, `\`, LF, CR, or TAB>

BYTE_ESCAPE ->
      `\x` HEX_DIGIT HEX_DIGIT
    | `\n` | `\r` | `\t` | `\\` | `\0` | `\'` | `\"`
```

r[lex.token.byte.intro]
字节字面量是一个单个ASCII字符（在`U+0000`到`U+007F`范围内）或一个单个_转义_字符，前面带有字符`U+0062`（`b`）和`U+0027`（单引号），后面带有字符`U+0027`。如果`U+0027`字符出现在字面量中，它必须通过前面加一个`U+005C`（`\`）字符进行转义。它等同于一个`u8`无符号8位整数数字字面量。

r[lex.token.str-byte]
#### 字节字符串字面量

r[lex.token.str-byte.syntax]
```grammar,lexer
BYTE_STRING_LITERAL ->
    `b"` ( ASCII_FOR_STRING | BYTE_ESCAPE | STRING_CONTINUE )* `"` SUFFIX?

ASCII_FOR_STRING ->
    <any ASCII (i.e 0x00 to 0x7F) except `"`, `\`, or CR>
```

r[lex.token.str-byte.intro]
非原始字节字符串字面量是一个ASCII字符和_转义_序列，前面带有字符`U+0062`（`b`）和`U+0022`（双引号），后面带有字符`U+0022`。如果`U+0022`字符出现在字面量中，它必须通过前面加一个`U+005C`（`\`）字符进行转义。另外，字节字符串字面量可以是下面定义的原始字节字符串字面量。

r[lex.token.str-byte.linefeed]
换行符（由字符`U+000A` (LF)表示）在字节字符串字面量中是允许的。字符`U+000D` (CR)不得出现在字节字符串字面量中。当未转义的`U+005C`字符（`\`）紧接在换行符之前时，换行符不会出现在词法单元所表示的字符串中。有关详细信息，请参阅[字符串续行转义][String continuation escapes]。

r[lex.token.str-byte.escape]
在字节或非原始字节字符串字面量中，还有一些额外的_转义_可用。转义以`U+005C`（`\`）开头，并接以下形式之一：

r[lex.token.str-byte.escape-byte]
* 字节转义以`U+0078`（`x`）开头，后跟恰好两个十六进制数字。它表示等于所提供十六进制值的字节。

r[lex.token.str-byte.escape-whitespace]
* 空白转义是字符`U+006E`（`n`）、`U+0072`（`r`）或`U+0074`（`t`）之一，分别表示字节值`0x0A` (ASCII LF)、`0x0D` (ASCII CR)或`0x09` (ASCII HT)。

r[lex.token.str-byte.escape-null]
* 空转义是字符`U+0030`（`0`），表示字节值`0x00` (ASCII NUL)。

r[lex.token.str-byte.escape-slash]
* 反斜杠转义是字符`U+005C`（`\`），它必须被转义才能表示其ASCII编码`0x5C`。

r[lex.token.str-byte-raw]
#### 原始字节字符串字面量

r[lex.token.str-byte-raw.syntax]
```grammar,lexer
RAW_BYTE_STRING_LITERAL ->
    `br` RAW_BYTE_STRING_CONTENT SUFFIX?

RAW_BYTE_STRING_CONTENT ->
      `"` ASCII_FOR_RAW*? `"`
    | `#` RAW_BYTE_STRING_CONTENT `#`

ASCII_FOR_RAW ->
    <any ASCII (i.e. 0x00 to 0x7F) except CR>
```

r[lex.token.str-byte-raw.intro]
原始字节字符串字面量不处理任何转义。它们以字符`U+0062`（`b`）开头，后跟`U+0072`（`r`），再后跟少于256个`U+0023`（`#`）字符和一个`U+0022`（双引号）字符。

r[lex.token.str-byte-raw.body]
原始字符串体可以包含除`U+000D` (CR)以外的任何ASCII字符序列。它只由另一个`U+0022`（双引号）字符终止，后跟与起始`U+0022`（双引号）字符前相同数量的`U+0023`（`#`）字符。原始字节字符串字面量不能包含任何非ASCII字节。

r[lex.token.literal.str-byte-raw.content]
原始字符串体中包含的所有字符都表示其ASCII编码，字符`U+0022`（双引号）（除非后面紧跟的`U+0023`（`#`）字符数量与用于开始原始字符串字面量的字符数量相同或更多）或`U+005C`（`\`）不具有任何特殊含义。

字节字符串字面量示例：

```rust
b"foo"; br"foo";                     // foo
b"\"foo\""; br#""foo""#;             // "foo"

b"foo #\"# bar";
br##"foo #"# bar"##;                 // foo #"# bar

b"\x52"; b"R"; br"R";                // R
b"\\x52"; br"\x52";                  // \x52
```

### C字符串和原始C字符串字面量

r[lex.token.str-c]
#### C字符串字面量

r[lex.token.str-c.syntax]
```grammar,lexer
C_STRING_LITERAL ->
    `c"` (
        ~[`"` `\` CR NUL]
      | BYTE_ESCAPE _except `\0` or `\x00`_
      | UNICODE_ESCAPE _except `\u{0}`, `\u{00}`, …, `\u{000000}`_
      | STRING_CONTINUE
    )* `"` SUFFIX?

```

r[lex.token.str-c.intro]
C字符串字面量是一个Unicode字符和_转义_序列，前面带有字符`U+0063`（`c`）和`U+0022`（双引号），后面带有字符`U+0022`。如果`U+0022`字符出现在字面量中，它必须通过前面加一个`U+005C`（`\`）字符进行转义。另外，C字符串字面量可以是下面定义的原始C字符串字面量。

[CStr]: core::ffi::CStr

r[lex.token.str-c.null]
C字符串隐式以字节`0x00`终止，因此C字符串字面量`c""`等同于手动从字节字符串字面量`b"\x00"`构造一个`&CStr`。除了隐式终止符，字节`0x00`不允许出现在C字符串中。

r[lex.token.str-c.linefeed]
换行符（由字符`U+000A` (LF)表示）在C字符串字面量中是允许的。字符`U+000D` (CR)不得出现在C字符串字面量中。当未转义的`U+005C`字符（`\`）紧接在换行符之前时，换行符不会出现在词法单元所表示的字符串中。有关详细信息，请参阅[字符串续行转义][String continuation escapes]。

r[lex.token.str-c.escape]
在非原始C字符串字面量中，还有一些额外的_转义_可用。转义以`U+005C`（`\`）开头，并接以下形式之一：

r[lex.token.str-c.escape-byte]
* 字节转义以`U+0078`（`x`）开头，后跟恰好两个十六进制数字。它表示等于所提供十六进制值的字节。

r[lex.token.str-c.escape-unicode]
* 24位码点转义以`U+0075`（`u`）开头，后跟最多六个十六进制数字，并用花括号`U+007B`（`{`）和`U+007D`（`}`）括起来。它表示等于所提供十六进制值的Unicode码点，以UTF-8编码。

r[lex.token.str-c.escape-whitespace]
* 空白转义是字符`U+006E`（`n`）、`U+0072`（`r`）或`U+0074`（`t`）之一，分别表示字节值`0x0A` (ASCII LF)、`0x0D` (ASCII CR)或`0x09` (ASCII HT)。

r[lex.token.str-c.escape-slash]
* 反斜杠转义是字符`U+005C`（`\`），它必须被转义才能表示其ASCII编码`0x5C`。

r[lex.token.str-c.char-unicode]
C字符串表示没有定义编码的字节，但C字符串字面量可以包含高于`U+007F`的Unicode字符。这些字符将被替换为该字符的UTF-8表示字节。

以下C字符串字面量是等效的：

```rust
c"æ";        // LATIN SMALL LETTER AE (U+00E6)
c"\u{00E6}";
c"\xC3\xA6";
```

r[lex.token.str-c.edition2021]
> [!EDITION-2021]
> C字符串字面量在2021或更高版次中被接受。在更早的版次中，词法单元`c""`被解析为`c ""`。

r[lex.token.str-c-raw]
#### 原始C字符串字面量

r[lex.token.str-c-raw.syntax]
```grammar,lexer
RAW_C_STRING_LITERAL ->
    `cr` RAW_C_STRING_CONTENT SUFFIX?

RAW_C_STRING_CONTENT ->
      `"` ( ~[CR NUL] )*? `"`
    | `#` RAW_C_STRING_CONTENT `#`
```

r[lex.token.str-c-raw.intro]
原始C字符串字面量不处理任何转义。它们以字符`U+0063`（`c`）开头，后跟`U+0072`（`r`），再后跟少于256个`U+0023`（`#`）字符和一个`U+0022`（双引号）字符。

r[lex.token.str-c-raw.body]
原始C字符串体可以包含除`U+0000` (NUL)和`U+000D` (CR)以外的任何Unicode字符序列。它只由另一个`U+0022`（双引号）字符终止，后跟与起始`U+0022`（双引号）字符前相同数量的`U+0023`（`#`）字符。

r[lex.token.str-c-raw.content]
原始C字符串体中包含的所有字符都以UTF-8编码表示其本身。字符`U+0022`（双引号）（除非后面紧跟的`U+0023`（`#`）字符数量与用于开始原始C字符串字面量的字符数量相同或更多）或`U+005C`（`\`）不具有任何特殊含义。

r[lex.token.str-c-raw.edition2021]
> [!EDITION-2021]
> 原始C字符串字面量在2021或更高版次中被接受。在更早的版次中，词法单元`cr""`被解析为`cr ""`，而`cr#""#`被解析为`cr #""#`（这不符合语法格式）。

#### C字符串和原始C字符串字面量示例

```rust
c"foo"; cr"foo";                     // foo
c"\"foo\""; cr#""foo""#;             // "foo"

c"foo #\"# bar";
cr##"foo #"# bar"##;                 // foo #"# bar

c"\x52"; c"R"; cr"R";                // R
c"\\x52"; cr"\x52";                  // \x52
```

r[lex.token.literal.num]
### 数字字面量

数字字面量可以是整数字面量，也可以是浮点数字面量。识别这两种字面量的语法格式是混合的。

r[lex.token.literal.int]
#### 整数字面量

r[lex.token.literal.int.syntax]
```grammar,lexer
INTEGER_LITERAL ->
    ( BIN_LITERAL | OCT_LITERAL | HEX_LITERAL | DEC_LITERAL ) SUFFIX_NO_E?

DEC_LITERAL -> DEC_DIGIT (DEC_DIGIT|`_`)*

BIN_LITERAL -> `0b` `_`* BIN_DIGIT (BIN_DIGIT|`_`)*

OCT_LITERAL -> `0o` `_`* OCT_DIGIT (OCT_DIGIT|`_`)*

HEX_LITERAL -> `0x` `_`* HEX_DIGIT (HEX_DIGIT|`_`)*

BIN_DIGIT -> [`0`-`1`]

OCT_DIGIT -> [`0`-`7`]

DEC_DIGIT -> [`0`-`9`]

HEX_DIGIT -> [`0`-`9` `a`-`f` `A`-`F`]
```

r[lex.token.literal.int.kind]
整数字面量有以下四种形式：

r[lex.token.literal.int.kind-dec]
* 十进制字面量以十进制数字开头，并可包含十进制数字和下划线的任意组合。

r[lex.token.literal.int.kind-hex]
* 十六进制字面量以字符序列`U+0030` `U+0078`（`0x`）开头，并可包含十六进制数字和下划线的任意组合（至少包含一个数字）。

r[lex.token.literal.int.kind-oct]
* 八进制字面量以字符序列`U+0030` `U+006F`（`0o`）开头，并可包含八进制数字和下划线的任意组合（至少包含一个数字）。

r[lex.token.literal.int.kind-bin]
* 二进制字面量以字符序列`U+0030` `U+0062`（`0b`）开头，并可包含二进制数字和下划线的任意组合（至少包含一个数字）。

r[lex.token.literal.int.restriction]
像任何字面量一样，整数字面量可以紧跟（不带任何空格）一个如上所述的后缀。后缀不能以`e`或`E`开头，因为这将被解释为浮点字面量的指数。有关这些后缀的效果，请参阅[整数字面量表达式][Integer literal expressions]。

被接受为字面量表达式的整数字面量示例：

```rust
# #![allow(overflowing_literals)]
123;
123i32;
123u32;
123_u32;

0xff;
0xff_u8;
0x01_f32; // integer 7986, not floating-point 1.0
0x01_e3;  // integer 483, not floating-point 1000.0

0o70;
0o70_i16;

0b1111_1111_1001_0000;
0b1111_1111_1001_0000i64;
0b________1;

0usize;

// These are too big for their type, but are accepted as literal expressions.
128_i8;
256_u8;

// This is an integer literal, accepted as a floating-point literal expression.
5f32;
```

请注意，例如`-1i8`被解析为两个词法单元：`-`后跟`1i8`。

不被接受为字面量表达式的整数字面量示例：

```rust
# #[cfg(false)] {
0invalidSuffix;
123AFB43;
0b010a;
0xAB_CD_EF_GH;
0b1111_f32;
# }
```

r[lex.token.literal.int.tuple-field]
#### 元组[项][item]索引

r[lex.token.literal.int.tuple-field.syntax]
```grammar,lexer
TUPLE_INDEX -> DEC_LITERAL | BIN_LITERAL | OCT_LITERAL | HEX_LITERAL
```

r[lex.token.literal.int.tuple-field.intro]
元组[项][item]索引用于引用[元组][tuples]、[元组结构体][tuple structs]和[元组枚举变体][tuple enum variants]的字段。

r[lex.token.literal.int.tuple-field.eq]
元组[项][item]索引与字面量词法单元直接比较。元组[项][item]索引从`0`开始，每个后续索引的十进制值递增`1`。因此，只有十进制值会匹配，并且该值不能有任何额外的`0`前缀字符。

元组[项][item]索引不能包含任何后缀（例如`usize`）。

```rust,compile_fail
let example = ("dog", "cat", "horse");
let dog = example.0;
let cat = example.1;
// The following examples are invalid.
let cat = example.01;  // ERROR no field named `01`
let horse = example.0b10;  // ERROR no field named `0b10`
let unicorn = example.0usize; // ERROR suffixes on a tuple index are invalid
let underscore = example.0_0; // ERROR no field `0_0` on type `(&str, &str, &str)`
```

r[lex.token.literal.float]
#### 浮点数字面量

r[lex.token.literal.float.syntax]
```grammar,lexer
FLOAT_LITERAL ->
      DEC_LITERAL (`.` DEC_LITERAL)? FLOAT_EXPONENT SUFFIX?
    | DEC_LITERAL `.` DEC_LITERAL SUFFIX_NO_E?
    | DEC_LITERAL `.` _not immediately followed by `.`, `_` or an XID_Start character_

FLOAT_EXPONENT ->
    (`e`|`E`) (`+`|`-`)? `_`* DEC_DIGIT (DEC_DIGIT|`_`)*
```

r[lex.token.literal.float.form]
浮点数字面量有两种形式：

* 十进制字面量，后跟句点字符`U+002E`（`.`）。这后面可选地跟另一个十进制字面量，带有可选的指数。
* 单个十进制字面量，后跟指数。

r[lex.token.literal.float.suffix]
像整数字面量一样，浮点数字面量可以后跟一个后缀，只要后缀前的部分不以`U+002E`（`.`）结尾。如果字面量不包含指数，则后缀不能以`e`或`E`开头。有关这些后缀的效果，请参阅[浮点数字面量表达式][Floating-point literal expressions]。

被接受为字面量表达式的浮点数字面量示例：

```rust
123.0f64;
0.1f64;
0.1f32;
12E+99_f64;
let x: f64 = 2.;
```

最后一个示例有所不同，因为浮点数字面量以句点结尾时，无法使用后缀语法格式。`2.f64`会尝试在`2`上调用名为`f64`的方法。

请注意，例如`-1.0`被解析为两个词法单元：`-`后跟`1.0`。

不被接受为字面量表达式的浮点数字面量示例：

```rust
# #[cfg(false)] {
2.0f80;
2e5f80;
2e5e6;
2.0e5e6;
1.3e10u64;
# }
```

r[lex.token.literal.reserved]
#### 类似于数字字面量的保留形式

r[lex.token.literal.reserved.syntax]
```grammar,lexer
RESERVED_NUMBER ->
      BIN_LITERAL [`2`-`9`]
    | OCT_LITERAL [`8`-`9`]
    | ( BIN_LITERAL | OCT_LITERAL | HEX_LITERAL ) `.` _not immediately followed by `.`, `_` or an XID_Start character_
    | ( BIN_LITERAL | OCT_LITERAL ) (`e`|`E`)
    | `0b` `_`* <end of input or not BIN_DIGIT>
    | `0o` `_`* <end of input or not OCT_DIGIT>
    | `0x` `_`* <end of input or not HEX_DIGIT>
    | DEC_LITERAL ( `.` DEC_LITERAL )? (`e` | `E`) (`+` | `-`)? <end of input or not DEC_DIGIT>

```

r[lex.token.literal.reserved.intro]
以下类似于数字字面量的词法形式是_保留形式_。由于可能存在的歧义，词法分析器会拒绝这些形式，而不是将其解释为单独的词法单元。

r[lex.token.literal.reserved.out-of-range]
* 未加后缀的二进制或八进制字面量，紧接着（无中间空白）一个超出其基数范围的十进制数字。

r[lex.token.literal.reserved.period]
* 未加后缀的二进制、八进制或十六进制字面量，紧接着（无中间空白）一个句点字符（对句点后面的内容有与浮点数字面量相同的限制）。

r[lex.token.literal.reserved.exp]
* 未加后缀的二进制或八进制字面量，紧接着（无中间空白）字符`e`或`E`。

r[lex.token.literal.reserved.empty-with-radix]
* 以基数前缀之一开头但不是有效的二进制、八进制或十六进制字面量（因为它不包含任何数字）的输入。

r[lex.token.literal.reserved.empty-exp]
* 具有浮点数字面量形式，但指数中没有数字的输入。

保留形式的示例：

```rust,compile_fail
0b0102;  // 这不是`0b010`后跟`2`
0o1279;  // 这不是`0o127`后跟`9`
0x80.0;  // 这不是`0x80`后跟`.`和`0`
0b101e;  // 这不是一个带后缀的字面量，也不是`0b101`后跟`e`
0b;      // 这不是一个整数字面量，也不是`0`后跟`b`
0b_;     // 这不是一个整数字面量，也不是`0`后跟`b_`
2e;      // 这不是一个浮点数字面量，也不是`2`后跟`e`
2.0e;    // 这不是一个浮点数字面量，也不是`2.0`后跟`e`
2em;     // 这不是一个带后缀的字面量，也不是`2`后跟`em`
2.0em;   // 这不是一个带后缀的字面量，也不是`2.0`后跟`em`
```

r[lex.token.life]
## 生命周期和循环标签

r[lex.token.life.syntax]
```grammar,lexer
LIFETIME_TOKEN ->
      RAW_LIFETIME
    | `'` IDENTIFIER_OR_KEYWORD _not immediately followed by `'`_

LIFETIME_OR_LABEL ->
      RAW_LIFETIME
    | `'` NON_KEYWORD_IDENTIFIER _not immediately followed by `'`_

RAW_LIFETIME ->
    `'r#` IDENTIFIER_OR_KEYWORD _not immediately followed by `'`_

RESERVED_RAW_LIFETIME -> `'r#` (`_` | `crate` | `self` | `Self` | `super`) _not immediately followed by `'`_
```

r[lex.token.life.intro]
生命周期参数和[循环标签][loop labels]使用LIFETIME_OR_LABEL词法单元。任何LIFETIME_TOKEN都会被词法分析器接受，例如，可以在宏中使用。

r[lex.token.life.raw.intro]
原始生命周期类似于普通生命周期，但其[标识符][identifier]带有`r#`前缀。（请注意，`r#`前缀不作为实际生命周期的一部分。）

r[lex.token.life.raw.allowed]
与普通生命周期不同，原始生命周期可以是除上述`RAW_LIFETIME`列出的[关键字][Keywords]之外的任何严格或保留[关键字][Keywords]。

r[lex.token.life.raw.reserved]
使用[RESERVED_RAW_LIFETIME]词法单元是错误的。

r[lex.token.life.raw.edition2021]
> [!EDITION-2021]
> 原始生命周期在2021或更高版次中被接受。在更早的版次中，词法单元`'r#lt`被解析为`'r # lt`。

r[lex.token.punct]
## 标点符号

r[lex.token.punct.intro]
标点符号词法单元用作运算符、分隔符和语法格式的其他部分。

r[lex.token.punct.syntax]
```grammar,lexer
PUNCTUATION ->
      `...`
    | `..=`
    | `<<=`
    | `>>=`
    | `!=`
    | `%=`
    | `&&`
    | `&=`
    | `*=`
    | `+=`
    | `-=`
    | `->`
    | `..`
    | `/=`
    | `::`
    | `<-`
    | `<<`
    | `<=`
    | `==`
    | `=>`
    | `>=`
    | `>>`
    | `>`
    | `^=`
    | `|=`
    | `||`
    | `!`
    | `#`
    | `$`
    | `%`
    | `&`
    | `(`
    | `)`
    | `*`
    | `+`
    | `,`
    | `-`
    | `.`
    | `/`
    | `:`
    | `;`
    | `<`
    | `=`
    | `?`
    | `@`
    | `[`
    | `]`
    | `^`
    | `{`
    | `|`
    | `}`
    | `~`
```

> [!NOTE]
> 有关标点符号字符如何使用的链接，请参阅[语法格式索引][syntax index]。

r[lex.token.delim]
## 分隔符

括号标点符号用于语法格式的各个部分。开括号必须始终与闭括号配对。括号及其内部的词法单元在[宏][macros]中被称为“词法单元树”。括号有三种类型：

| 括号 | 类型 |
|---|---|
| `{` `}` | 花括号 |
| `[` `]` | 方括号 |
| `(` `)` | 小括号 |

r[lex.token.reserved]
## 保留词法单元

r[lex.token.reserved.intro]
几种词法单元形式被保留以备将来使用或避免混淆。源输入匹配其中一种形式是错误的。

r[lex.token.reserved.syntax]
```grammar,lexer
RESERVED_TOKEN ->
      RESERVED_GUARDED_STRING_LITERAL
    | RESERVED_NUMBER
    | RESERVED_POUNDS
    | RESERVED_RAW_IDENTIFIER
    | RESERVED_RAW_LIFETIME
    | RESERVED_TOKEN_DOUBLE_QUOTE
    | RESERVED_TOKEN_LIFETIME
    | RESERVED_TOKEN_POUND
    | RESERVED_TOKEN_SINGLE_QUOTE
```

r[lex.token.reserved-prefix]
## 保留前缀

r[lex.token.reserved-prefix.syntax]
```grammar,lexer
RESERVED_TOKEN_DOUBLE_QUOTE ->
    IDENTIFIER_OR_KEYWORD _except `b` or `c` or `r` or `br` or `cr`_ `"`

RESERVED_TOKEN_SINGLE_QUOTE ->
    IDENTIFIER_OR_KEYWORD _except `b`_ `'`

RESERVED_TOKEN_POUND ->
    IDENTIFIER_OR_KEYWORD _except `r` or `br` or `cr`_ `#`

RESERVED_TOKEN_LIFETIME ->
    `'` IDENTIFIER_OR_KEYWORD _except `r`_ `#`
```

r[lex.token.reserved-prefix.intro]
一些被称为_保留前缀_的词法形式被保留以备将来使用。

r[lex.token.reserved-prefix.id]
源输入如果被词法解释为非原始[标识符][identifier]（或[关键字][Keywords]），且紧跟`#`、`'`或`"`字符（无中间空白），则被识别为保留前缀。

r[lex.token.reserved-prefix.raw-token]
请注意，原始[标识符][identifier]、原始字符串字面量和原始字节字符串字面量可能包含`#`字符，但不会被解释为包含保留前缀。

r[lex.token.reserved-prefix.strings]
同样，用于原始字符串字面量、字节字面量、字节字符串字面量、原始字节字符串字面量、C字符串字面量和原始C字符串字面量的`r`、`b`、`br`、`c`和`cr`前缀不被解释为保留前缀。

r[lex.token.reserved-prefix.life]
源输入如果被词法解释为非原始生命周期（或[关键字][Keywords]），且紧跟`#`字符（无中间空白），则被识别为保留生命周期前缀。

r[lex.token.reserved-prefix.edition2021]
> [!EDITION-2021]
> 从2021版次开始，词法分析器会将保留前缀报告为错误（特别是，它们不能传递给宏）。
>
> 在2021版次之前，词法分析器会接受保留前缀，并将其解释为多个词法单元（例如，[标识符][identifier]或[关键字][Keywords]的一个词法单元，后跟一个`#`词法单元）。
>
> 所有版次都接受的示例：
> ```rust
> macro_rules! lexes {($($_:tt)*) => {}}
> lexes!{a #foo}
> lexes!{continue 'foo}
> lexes!{match "..." {}}
> lexes!{r#let#foo}         // 三个词法单元: r#let # foo
> lexes!{'prefix #lt}
> ```
>
> 在2021版次之前接受但之后被拒绝的示例：
> ```rust,edition2018
> macro_rules! lexes {($($_:tt)*) => {}}
> lexes!{a#foo}
> lexes!{continue'foo}
> lexes!{match"..." {}}
> lexes!{'prefix#lt}
> ```

r[lex.token.reserved-guards]
## 保留的守卫

r[lex.token.reserved-guards.syntax]
```grammar,lexer
RESERVED_GUARDED_STRING_LITERAL -> `#`+ STRING_LITERAL

RESERVED_POUNDS -> `#`{2..}
```

r[lex.token.reserved-guards.intro]
保留的守卫是为将来使用而保留的语法格式，如果使用，将生成编译错误。

r[lex.token.reserved-guards.string-literal]
*保留的带守卫字符串字面量*是一个词法单元，由一个或多个`U+0023`（`#`）紧接着一个[STRING_LITERAL]组成。

r[lex.token.reserved-guards.pounds]
*保留的井号*是一个由两个或更多`U+0023`（`#`）组成的词法单元。

r[lex.token.reserved-guards.edition2024]
> [!EDITION-2024]
> 在2024版次之前，词法分析器会接受保留的守卫，并将其解释为多个词法单元。例如，`#"foo"#`形式被解释为三个词法单元。`##`被解释为两个词法单元。

[Floating-point literal expressions]: expressions/literal-expr.md#floating-point-literal-expressions
[identifier]: identifiers.md
[Integer literal expressions]: expressions/literal-expr.md#integer-literal-expressions
[keywords]: keywords.md
[literal expressions]: expressions/literal-expr.md
[loop labels]: expressions/loop-expr.md
[macros]: macros-by-example.md
[String continuation escapes]: expressions/literal-expr.md#string-continuation-escapes
[syntax index]: syntax-index.md#operators-and-punctuation
[tuple structs]: items/structs.md
[tuple enum variants]: items/enumerations.md
[tuples]: types/tuple.md