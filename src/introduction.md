<!-- https://github.com/rust-lang/reference/blob/master/src/introduction.md -->
<!-- commit 68bdfd15fa9feebfbf94a06cf574de598e239198 -->

# 引言

本书是 Rust 编程语言的主要参考资料。
> [!NOTE]
> 有关本书中已知错误和遗漏，请参阅我们的 [问题单][github issues]。如果您发现编译器行为与本文描述不符的情况，请提交问题，以便我们讨论哪个是正确的。

## Rust 发布版本

Rust 每六周发布一个新的语言版本。
该语言的第一个稳定版本是 Rust 1.0.0，随后是 Rust 1.1.0 等。
工具（`rustc`、`cargo` 等）和文档（[标准库][standard library]、本书等）随语言版本一起发布。

本书的最新版本，与最新的 Rust 版本相匹配，始终可在 <https://doc.rust-lang.org/reference/> 找到。
可以通过在 "reference" 目录前添加 Rust 版本号来查找以前的版本。
例如，Rust 1.49.0 的参考资料位于 <https://doc.rust-lang.org/1.49.0/reference/>。

## 《参考手册》不是什么

本书并非语言入门指南。
阅读本书需假定您已具备语言基础知识。
另有一本 [书籍][book] 可帮助您获取此类基础知识。

本书也不是语言发行版中包含的 [标准库][standard library] 的参考资料。
这些库通过从其源代码中提取文档属性来单独记录。
许多您可能期望是语言特性的功能在 Rust 中是库特性，因此您要查找的内容可能在那里，而不是在这里。

同样，本书通常不记录 `rustc` 作为工具或 Cargo 的具体细节。
`rustc` 有自己的 [书籍][rustc book]。
Cargo 有一本 [书籍][cargo book]，其中包含一份 [参考手册][cargo reference]。
还有一些页面，例如 [链接][linkage]，也是描述 `rustc` 的工作方式。

本书也仅作为稳定版 Rust 中可用功能的参考。
对于正在开发的不稳定功能，请参阅 [不稳定手册][Unstable Book]。

Rust 编译器，包括 `rustc`，会执行优化。
本参考手册没有指定允许或不允许哪些优化。
相反，请将编译后的程序视为一个黑盒子。
您只能通过运行它，提供输入并观察其输出来进行探测。
所有以这种方式发生的事情都必须符合参考手册的规定。

## 如何使用本书

本书不假设您是按顺序阅读本书的。
每个章节通常可以独立阅读，但会交叉链接到其他章节，以提及它们所涉及但不讨论的语言方面。

阅读本文档主要有两种方式。

第一种是回答一个具体问题。
如果您知道哪个章节回答了该问题，您可以直接跳转到目录中的该章节。
否则，您可以按 `s` 键或点击顶部栏的放大镜图标来搜索与您问题相关的关键词。
例如，假设您想知道在 let 语句中创建的临时值何时被丢弃。
如果您不知道 [临时值的生命周期][lifetime of temporaries] 定义在 [表达式章节][expressions chapter] 中，您可以搜索“临时 let”，第一个搜索结果将带您到该部分。

第二种是普遍提高您对语言某方面的知识。
在这种情况下，只需浏览目录，直到看到您想了解更多内容的部分，然后开始阅读即可。
如果某个链接看起来很有趣，请点击它，然后阅读该部分。

话虽如此，阅读本书没有错误的方式。请以您认为最有帮助的方式阅读。

### 约定

像所有技术书籍一样，本书在信息展示方面也有某些约定。
这些约定在此处记录。

* 定义术语的语句包含 *斜体* 的该术语。
  每当该术语在该章节之外使用时，它通常是链接到包含此定义的章节。

  *示例术语* 是一个正在定义的术语的示例。

* 主要文本描述最新的稳定版本。与以前版本的差异在版本块中分离：

  > [!EDITION-2018]
  > 在 2018 版次之前，行为是这样的。从 2018 版次开始，行为是那样的。

* 包含有关本书状态的有用信息或指出有用但大多超出范围的信息的注释位于注释块中。

  > [!NOTE]
  > 这是一个示例注释。

* 示例块显示一个示例，演示某个规则或指出某个有趣的方面。一些示例可能有隐藏行，可以通过将鼠标悬停或点击示例时出现的眼睛图标来查看。

  > [!EXAMPLE]
  > 这是一个代码示例。
  > ```rust
  > println!("hello world");
  > ```

* 显示语言中不健全行为或语言功能之间可能令人困惑的交互的警告位于特殊的警告框中。

  > [!WARNING]
  > 这是一个示例警告。

* 文本中的内联代码片段位于 `<code>` 标签内。

  较长的代码示例位于一个带有语法高亮显示的框中，其右上角有复制、执行和显示隐藏行的控件。

  ```rust
  # // 这是一行隐藏行。
  fn main() {
      println!("这是一个代码示例");
  }
  ```

  除非另有说明，所有示例均针对最新版本编写。

* 语法和词法产生式在 [符号][Notation] 章节中描述。

r[example.rule.label]
* 规则标识符出现在每个语言规则之前，用方括号括起来。这些标识符提供了一种引用和链接到语言中特定规则的方法（例如，[示例规则][example rule]）。规则标识符使用句点将部分从最一般到最具体地分开（例如，[解构器.作用域.嵌套.函数体][destructors.scope.nesting.function-body]）。在窄屏幕上，规则名称将折叠显示 `[*]`。

  可以点击规则名称以链接到该规则。

  > [!WARNING]
  > 规则的组织目前处于变动中。暂时而言，这些标识符名称在不同版本之间不稳定，如果它们发生更改，指向这些规则的链接可能会失效。我们打算在组织稳定后将其稳定化，以便指向规则名称的链接在不同版本之间不会中断。

* 具有相关测试的规则将在其下方包含一个“测试”链接（在窄屏幕上，该链接为 `[T]`）。单击该链接将弹出一个测试列表，可以单击以查看测试。例如，请参见 [输入.编码.utf8][input.encoding.utf8]。

  将规则链接到测试是一项持续进行的工作。有关概述，请参阅 [测试摘要](test-summary.md) 章节。

## 贡献

我们欢迎各种形式的贡献。

您可以通过在 [Rust语言参考仓库][the Rust Reference repository] 中提出问题或发送拉取请求来为本书做出贡献。
如果本书没有回答您的问题，并且您认为它的答案属于其范围，请不要犹豫 [提交问题][github issues] 或在 [Zulip] 上的 `t-lang/doc` 流中提问。
了解人们最常使用本书做什么有助于我们将注意力集中在使这些部分尽善尽美上。
当然，如果您发现任何错误或非规范性但未明确指出的内容，也请 [提交问题][github issues]。

[book]: ../book/index.html
[github issues]: https://github.com/rust-lang/reference/issues
[standard library]: std
[the Rust Reference repository]: https://github.com/rust-lang/reference/
[Unstable Book]: https://doc.rust-lang.org/nightly/unstable-book/
[cargo book]: ../cargo/index.html
[cargo reference]: ../cargo/reference/index.html
[example rule]: example.rule.label
[expressions chapter]: expressions.html
[file an issue]: https://github.com/rust-lang/reference/issues
[lifetime of temporaries]: expressions.html#temporaries
[linkage]: linkage.html
[rustc book]: ../rustc/index.html
[Notation]: notation.md
[Zulip]: https://rust-lang.zulipchat.com/#narrow/stream/237824-t-lang.2Fdoc
