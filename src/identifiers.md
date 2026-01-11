<!-- https://github.com/rust-lang/reference/blob/master/src/identifiers.md -->
<!-- commit 68bdfd15fa9feebfbf94a06cf574de598e239198 -->

r[ident]
# 标识符

r[ident.syntax]
```grammar,词法分析器
IDENTIFIER_OR_KEYWORD -> ( XID_Start | `_` ) XID_Continue*

XID_Start -> <`XID_Start` defined by Unicode>

XID_Continue -> <`XID_Continue` defined by Unicode>

RAW_IDENTIFIER -> `r#` IDENTIFIER_OR_KEYWORD

NON_KEYWORD_IDENTIFIER -> IDENTIFIER_OR_KEYWORD _except a [strict][lex.keywords.strict] or [reserved][lex.keywords.reserved] keyword_

IDENTIFIER -> NON_KEYWORD_IDENTIFIER | RAW_IDENTIFIER

RESERVED_RAW_IDENTIFIER ->
    `r#` (`_` | `crate` | `self` | `Self` | `super`) _not immediately followed by XID_Continue_
```

<!-- When updating the version, update the UAX links, too. -->
r[ident.unicode]
标识符遵循[Unicode标准附件#31][UAX31]中针对Unicode 16.0版的规范，并增加了下述内容。一些标识符的例子：

* `foo`
* `_identifier`
* `r#true`
* `Москва`
* `東京`

r[ident.profile]
UAX #31中使用的概要是：

* Start := [`XID_Start`]，加上下划线字符(U+005F)
* Continue := [`XID_Continue`]
* Medial := 空

> [!NOTE]
> 以`_`开头的标识符通常用于表明一个有意不使用的标识符，并且会消除`rustc`中的未使用警告。

r[ident.keyword]
标识符不能是[严格][strict]或[保留][reserved]关键字，除非带上下述[原始标识符](#raw-identifiers)中描述的`r#`前缀。

r[ident.zero-width-chars]
零宽不连接符(ZWNJ U+200C)和零宽连接符(ZWJ U+200D)字符不允许出现在标识符中。

r[ident.ascii-limitations]
在以下情况，标识符被限制为[`XID_Start`]和[`XID_Continue`]的ASCII子集：

* [`extern crate`][`extern crate`]声明（除了[AsClause]标识符）
* 在[路径][path]中引用的外部crate名
* 从文件系统加载而不带[`path`属性][`path` attribute]的[模块][module]名
* 带有[`no_mangle`][`no_mangle`]属性的项
* [外部块][external blocks]中的项名

r[ident.normalization]
## 规范化

标识符使用规范化形式 C (NFC) 进行规范化，如[Unicode标准附件#15][UAX15]中所定义。两个标识符在它们的 NFC 形式相等时才相等。

[过程][proc-macro]宏和[声明式][mbe]宏在它们的输入中接收规范化的标识符。

r[ident.raw]
## 原始标识符

r[ident.raw.intro]
原始标识符类似于普通标识符，但带有`r#`前缀。（请注意，`r#`前缀不作为实际标识符的一部分。）

r[ident.raw.allowed]
与普通标识符不同，原始标识符可以是任何[严格][strict]或[保留][reserved]关键字，除了上面为`RAW_IDENTIFIER`列出的那些。

r[ident.raw.reserved]
使用[RESERVED_RAW_IDENTIFIER]词法单元是错误的。

[`extern crate`]: items/extern-crates.md
[`no_mangle`]: abi.md#the-no_mangle-attribute
[`path` attribute]: items/modules.md#path属性
[`XID_Continue`]: http://unicode.org/cldr/utility/list-unicodeset.jsp?a=%5B%3AXID_Continue%3A%5D&abb=on&g=&i=
[`XID_Start`]:  http://unicode.org/cldr/utility/list-unicodeset.jsp?a=%5B%3AXID_Start%3A%5D&abb=on&g=&i=
[external blocks]: items/external-blocks.md
[mbe]: macros-by-example.md
[module]: items/modules.md
[path]: paths.md
[proc-macro]: procedural-macros.md
[reserved]: keywords.md#保留关键字
[strict]: keywords.md#严格关键字
[UAX15]: https://www.unicode.org/reports/tr15/tr15-56.html
[UAX31]: https://www.unicode.org/reports/tr31/tr31-41.html