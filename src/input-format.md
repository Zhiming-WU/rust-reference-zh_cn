<!-- https://github.com/rust-lang/reference/blob/master/src/input-format.md -->
<!-- commit 68bdfd15fa9feebfbf94a06cf574de598e239198 -->

r[input]
# 输入格式

r[input.syntax]
```grammar,词法分析器
@root CHAR -> <a Unicode scalar value>

NUL -> U+0000
```

r[input.intro]
本章描述了源文件如何被解释为词法单元序列。

有关程序如何组织成文件的描述，请参阅[crate和源文件][Crates and source files]。

r[input.encoding]
## 源文件编码

r[input.encoding.utf8]
每个源文件都被解释为以UTF-8编码的Unicode字符序列。

r[input.encoding.invalid]
如果文件不是有效的UTF-8，则会报错。

r[input.byte-order-mark]
## 字节序标记移除

如果序列中的第一个字符是`U+FEFF`（[字节序标记][BYTE ORDER MARK]），则将其移除。

r[input.crlf]
## CRLF规范化

每对紧跟着`U+000A` (LF) 的`U+000D` (CR) 字符会被单个`U+000A` (LF) 替换。此操作只执行一次，不会重复，因此在规范化之后，输入中仍然可能存在紧跟着`U+000A` (LF) 的`U+000D` (CR)（例如，如果原始输入包含 "CR CR LF LF"）。

字符`U+000D` (CR) 的其他出现位置则保留不变（它们被视为[空白符][whitespace]）。

r[input.shebang]
## Shebang移除

r[input.shebang.intro]
如果剩余序列以字符`#!`开头，则从序列中移除直到（并包括）第一个`U+000A` (LF) 的所有字符。

例如，以下文件的第一行将被忽略：

<!-- ignore: tests don't like shebang -->
```rust,ignore
#!/usr/bin/env rustx

fn main() {
    println!("Hello!");
}
```

r[input.shebang.inner-attribute]
作为例外，如果`#!`字符后面紧跟着（忽略中间的[注释][comments]或[空白符][whitespace]）一个`[`词法单元，则不进行任何移除。这可以防止源文件开头的[内部属性][inner attribute]被移除。

> [!NOTE]
> 标准库的[`include!`]宏会对其读取的文件进行字节序标记移除、CRLF规范化和Shebang移除。[`include_str!`]和[`include_bytes!`]宏则不会。

r[input.tokenization]
## 词法单元化

然后，所得的字符序列将转换为词法单元，具体描述见本章的其余部分。

[inner attribute]: attributes.md
[BYTE ORDER MARK]: https://en.wikipedia.org/wiki/Byte_order_mark#UTF-8
[comments]: comments.md
[Crates and source files]: crates-and-source-files.md
[_shebang_]: https://en.wikipedia.org/wiki/Shebang_(Unix)
[whitespace]: whitespace.md
