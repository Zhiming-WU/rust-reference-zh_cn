<!-- https://github.com/rust-lang/reference/blob/master/src/comments.md -->
<!-- commit 68bdfd15fa9feebfbf94a06cf574de598e239198 -->

r[comments]
# 注释

r[comments.syntax]
```grammar,词法分析器
@root LINE_COMMENT ->
      `//` (~[`/` `!` LF] | `//`) ~LF*
    | `//`

BLOCK_COMMENT ->
      `/*`
        ( ~[`*` `!`] | `**` | BLOCK_COMMENT_OR_DOC )
        ( BLOCK_COMMENT_OR_DOC | ~`*/` )*
      `*/`
    | `/**/`
    | `/***/`

@root INNER_LINE_DOC ->
    `//!` ~[LF CR]*

INNER_BLOCK_DOC ->
    `/*!` ( BLOCK_COMMENT_OR_DOC | ~[`*/` CR] )* `*/`

@root OUTER_LINE_DOC ->
    `///` (~`/` ~[LF CR]*)?

OUTER_BLOCK_DOC ->
    `/**`
      ( ~`*` | BLOCK_COMMENT_OR_DOC )
      ( BLOCK_COMMENT_OR_DOC | ~[`*/` CR] )*
    `*/`

@root BLOCK_COMMENT_OR_DOC ->
      BLOCK_COMMENT
    | OUTER_BLOCK_DOC
    | INNER_BLOCK_DOC
```

r[comments.normal]
## 非文档注释

注释遵循C++风格的单行注释（`//`）和块注释（`/* ... */`）形式。支持嵌套的块注释。

r[comments.normal.tokenization]
非文档注释被解释为一种空白字符。

r[comments.doc]
## 文档注释

r[comments.doc.syntax]
以恰好_三_个斜杠（`///`）开头的行文档注释，以及块文档注释（`/** ... */`），这两种都是外部文档注释，被解释为[`doc`属性][`doc` attributes]的一种特殊语法格式。

r[comments.doc.attributes]
也就是说，它们等价于在注释主体周围编写`#[doc="..."]`，例如，`/// Foo` 变为`#[doc="Foo"]`，`/** Bar */` 变为`#[doc="Bar"]`。因此，它们必须出现在接受外部属性的项之前。

r[comments.doc.inner-syntax]
以`//!`开头的行注释和`/*! ... */`块注释是文档注释，它们应用于注释的父级，而不是紧随其后的项。

r[comments.doc.inner-attributes]
也就是说，它们等价于在注释主体周围编写`#![doc="..."]`。`//!`注释通常用于文档化占据一个源文件的模块。

r[comments.doc.bare-crs]
字符`U+000D`(CR)不允许出现在文档注释中。

> [!NOTE]
> 按照惯例，文档注释包含Markdown，这是`rustdoc`所期望的。然而，注释语法格式不识别任何内部Markdown。`/** \`glob = "*/*.rs";\` */` 会在第一个`*/`处终止注释，其余代码将导致语法格式错误。与行文档注释相比，这稍微限制了块文档注释的内容。

> [!NOTE]
> 序列`U+000D`(CR)紧跟`U+000A`(LF)在之前会被转换为单个`U+000A`(LF)。

## 示例

```rust
//! 适用于此crate的隐式匿名模块的文档注释

pub mod outer_module {

    //!  - 内部行级文档注释
    //!! - 仍然是内部行级文档注释（但开头带感叹号）

    /*!  - 内部块级文档注释 */
    /*!! - 仍然是内部块级文档注释（但开头带感叹号） */

    //   - 仅是普通注释
    ///  - 外部行级文档注释（恰好三个斜杠）
    //// - 仅是普通注释

    /*   - 仅是普通注释 */
    /**  - 外部块级文档注释（恰好两个星号） */
    /*** - 仅是普通注释 */

    pub mod inner_module {}

    pub mod nested_comments {
        /* 在 Rust 中 /* 我们可以 /* 嵌套注释 */ */ */

        // 所有三种块级注释都可以包含或嵌套在
        // 任何其他类型的注释中：

        /*   /* */  /** */  /*! */  */
        /*!  /* */  /** */  /*! */  */
        /**  /* */  /** */  /*! */  */
        pub mod dummy_item {}
    }

    pub mod degenerate_cases {
        // 空的内部行级文档注释
        //!

        // 空的内部块级文档注释
        /*!*/

        // 空的行级注释
        //

        // 空的外部行级文档注释
        ///

        // 空的块级注释
        /**/

        pub mod dummy_item {}

        // 空的两个星号的块注释不是文档块，它是普通的块注释
        /***/

    }

    /* 下一个是不允许的，因为外部文档注释
       需要一个接收文档的条目 */

    /// 我的条目在哪里？
#   mod boo {}
}
```

[`doc` attributes]: ../rustdoc/the-doc-attribute.html
