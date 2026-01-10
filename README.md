<!-- https://github.com/rust-lang/reference/blob/master/README.md -->
<!-- commit 68bdfd15fa9feebfbf94a06cf574de598e239198 -->

# Rust 语言参考

本文档是 Rust 编程语言的主要参考资料。

## 翻译说明

本项目在尝试把 `Rust 语言参考手册` 文档翻译成简体中文。主要翻译方法是机器翻译加人工确认/编辑。术语对照表等请参考<a href="中文翻译说明.md">`中文翻译说明`</a>。目前处在尝试阶段。结果网页请参考 https://zhiming-wu.github.io/rust-reference-zh_cn/ 。
> 注：本节及"中文翻译说明.md"为译者添加内容，其它部分为原始英文内容或其译文。

## 贡献者文档

有几个页面是为那些在 参考手册 上工作的人准备的：

* [撰写指南](https://github.com/rust-lang/reference/blob/master/docs/authoring.md): 编写内容的指导原则。
* [审查策略](https://github.com/rust-lang/reference/blob/master/docs/review-policy.md): 审查人员的指导原则。
* [语法格式](https://github.com/rust-lang/reference/blob/master/docs/grammar.md): 语法格式 的工作原理。
* [属性 模板](https://github.com/rust-lang/reference/blob/master/docs/attribute-template.md): 文档化 属性 的标准 模板。

## 构建

要构建 参考手册，首先克隆项目：

```sh
git clone https://github.com/rust-lang/reference.git
cd reference
```

（或者，如果你不想使用 `git`，[下载][download] 项目的 ZIP 文件，使用你喜欢的工具解压，并将顶级目录重命名为 `reference`。）

[download]: https://github.com/rust-lang/reference/archive/refs/heads/master.zip

### 安装mdbook

参考手册 使用 [mdbook] 构建。

首先，确保你安装了最新版的 nightly Rust 编译器，因为运行测试需要它：

```sh
rustup toolchain install nightly
rustup override set nightly
```

现在，确保你安装了 `mdbook`，因为构建 参考手册 需要它：

```sh
cargo install --locked mdbook
```

[mdbook]: https://rust-lang.github.io/mdBook/

### 运行 mdbook

`mdbook` 提供了各种不同的命令和选项来帮助你编写此书：

* `mdbook build --open`: 构建此书并在 Web 浏览器中打开它。
* `mdbook serve --open`: 在 localhost 上启动 Web 服务器。它还会在任何文件更改时自动重建此书，并自动重新加载你的 Web 浏览器。

此书的内容由 `SUMMARY.md` 文件驱动，每个文件都必须链接到那里。请参阅 <https://rust-lang.github.io/mdBook/> 了解其用法。

### `SPEC_RELATIVE`

`SPEC_RELATIVE=0` 环境变量使指向标准库的链接指向 <https://doc.rust-lang.org/> 而不是相对链接，这在本地查看时很有用，因为你通常没有标准库的副本。

```sh
SPEC_RELATIVE=0 mdbook serve --open
```

发布到 <https://doc.rust-lang.org/reference/> 的网站（或使用 `rustup doc` 的本地 文档）没有设置此项，这意味着它将使用相对链接，这支持离线查看并链接到正确的 版本（例如，<https://doc.rust-lang.org/1.81.0/reference/> 中的链接将保留在 1.81.0 目录中）。

### `SPEC_DENY_WARNINGS`

`SPEC_DENY_WARNINGS=1` 环境变量会将 `mdbook-spec` 生成的所有警告转换为错误。这在 CI 中使用，以确保此书内容没有任何问题。

### `SPEC_RUST_ROOT`

`SPEC_RUST_ROOT` 可用于指向 <https://github.com/rust-lang/rust> 的签出目录。这由测试链接 特性 使用，以便它可以找到链接到 参考手册 规则的测试。如果未设置此项，则不会链接测试。

## 运行测试

有几种不同类型的测试可以运行（这些在 CI 上强制执行）：

* `mdbook test`: 这将运行内联 Rust 代码块（它内部使用 `rustdoc` 来执行此操作）。
* `cargo xtask style-check`: 这将验证一些样式检查（参见 [撰写指南](docs/authoring.md)）。
* `cargo xtask linkcheck`: 这将验证 markdown 链接是否未损坏。
* `cargo xtask test-all`: 运行所有测试。

## 这是如何发布的？

将 参考手册 内容发布到 [Rust 发布版](https://doc.rust-lang.org/reference/#rust-releases) 和网站的过程是：

1. 更改合并到此仓库。
2. [Triagebot](https://forge.rust-lang.org/triagebot/doc-updates.html) 将自动将此仓库同步到 [rust-lang/rust]。这每隔一周发生一次。参考手册 在 [rust-lang/rust] 中作为 [子模块](https://github.com/rust-lang/rust/tree/master/src/doc) 进行跟踪。
  - 这将在 [rust-lang/rust] 上打开一个 PR，需要合并，这可能需要几天时间。
3. UTC 午夜时分，[rust-lang/rust] 默认分支上的内容将成为该 nightly 发布版 的一部分，并将在几个小时后发布到 <https://doc.rust-lang.org/nightly/reference/>。
4. 遵循 Rust 的 [发布流程](https://doc.rust-lang.org/book/appendix-07-nightly-rust.html)，每 6 周，nightly 将升级为 beta（<https://doc.rust-lang.org/beta/reference/>），然后再过 6 周，它将升级为 stable（<https://doc.rust-lang.org/stable/reference/>）。

[rust-lang/rust]: https://github.com/rust-lang/rust/
