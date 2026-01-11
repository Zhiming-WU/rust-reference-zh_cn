<!-- https://github.com/rust-lang/reference/blob/master/src/whitespace.md -->
<!-- commit 68bdfd15fa9feebfbf94a06cf574de598e239198 -->

r[lex.whitespace]
# 空白字符

r[whitespace.syntax]
```grammar,词法分析器
@root WHITESPACE ->
      U+0009 // 水平制表符，`'\t'`
    | U+000A // 换行符，`'\n'`
    | U+000B // 垂直制表符
    | U+000C // 换页符
    | U+000D // 回车符，`'\r'`
    | U+0020 // 空格，`' '`
    | U+0085 // 下一行
    | U+200E // 从左到右标记
    | U+200F // 从右到左标记
    | U+2028 // 行分隔符
    | U+2029 // 段落分隔符

TAB -> U+0009 // 水平制表符，`'\t'`

LF -> U+000A  // 换行符，`'\n'`

CR -> U+000D  // 回车符，`'\r'`
```

r[lex.whitespace.intro]
空白字符是指任何非空字符串，它只包含具有[`Pattern_White_Space`] Unicode属性的字符。

r[lex.whitespace.token-sep]
Rust是一种自由格式语言，这意味着所有形式的空白字符仅用于在语法格式中分隔_词法单元_，并且没有语义上的意义。

r[lex.whitespace.replacement]
如果Rust程序中的每个空白字符元素被任何其他合法的空白字符元素（例如一个空格字符）替换，其含义保持不变。

[`Pattern_White_Space`]: https://www.unicode.org/reports/tr31/