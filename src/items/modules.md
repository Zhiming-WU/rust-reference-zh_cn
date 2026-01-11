r[items.mod]
# 模块

r[items.mod.syntax]
```grammar,项
Module ->
      `unsafe`? `mod` IDENTIFIER `;`
    | `unsafe`? `mod` IDENTIFIER `{`
        InnerAttribute*
        Item*
      `}`
```

r[items.mod.intro]
模块是零个或多个 [项][items] 的容器。

r[items.mod.def]
一个  _模块项 (module item)_  是一个模块，由大括号包围、命名，并以前缀关键字 `mod` 开头。 模块项 向构成 crate 的模块树中引入一个新的、具名的模块。

r[items.mod.nesting]
模块可以任意嵌套。

一个模块示例：

```rust
mod math {
    type Complex = (f64, f64);
    fn sin(f: f64) -> f64 {
        /* ... */
#       unimplemented!();
    }
    fn cos(f: f64) -> f64 {
        /* ... */
#       unimplemented!();
    }
    fn tan(f: f64) -> f64 {
        /* ... */
#       unimplemented!();
    }
}
```

r[items.mod.namespace]
模块定义在其所在的模块或块的 [类型命名空间][type namespace] 中。

r[items.mod.multiple-items]
在模块内的同一命名空间中定义多个同名 项 是错误的。有关限制和遮蔽行为的更多细节，请参阅 [作用域章节][scopes chapter]。

r[items.mod.unsafe]
`unsafe` 关键字在语法上允许出现在 `mod` 关键字之前，但在语义层面会被拒绝。这允许宏消费该语法并利用 `unsafe` 关键字，然后将其从 词法单元 流中移除。

r[items.mod.outlined]
## 外部源文件模块

r[items.mod.outlined.intro]
不带主体的模块从外部文件加载。当模块没有 `path` 属性时，文件的路径反映了逻辑 [模块路径][module path]。

r[items.mod.outlined.search]
祖先模块路径组件是目录，而模块的内容位于一个文件名与模块名相同并带有 `.rs` 扩展名的文件中。例如，以下模块结构可以具有对应如下的文件系统结构：

模块路径 | 文件系统路径 | 文件内容
------------------------- | ---------------  | -------------
`crate`                   | `lib.rs`         | `mod util;`
`crate::util`             | `util.rs`        | `mod config;`
`crate::util::config`     | `util/config.rs` |

r[items.mod.outlined.search-mod]
模块文件名也可以是以模块命名的目录，其内容位于该目录内名为 `mod.rs` 的文件中。上述示例也可以交替表示为将 `crate::util` 的内容放在名为 `util/mod.rs` 的文件中。不允许同时存在 `util.rs` 和 `util/mod.rs`。

> [!NOTE]
> 在 `rustc` 1.30 之前，使用 `mod.rs` 文件是加载带有嵌套子项模块的唯一方式。鼓励使用新的命名约定，因为它更一致，并能避免在项目中出现许多名为 `mod.rs` 的文件。

r[items.mod.outlined.path]
### `path`属性

r[items.mod.outlined.path.intro]
用于加载外部文件模块的目录和文件可以通过 `path` 属性来影响。

r[items.mod.outlined.path.search]
对于不在内联模块块内部的模块上的 `path` 属性，文件路径相对于源文件所在的目录。例如，以下代码片段将根据其所在位置使用显示的路径：

<!-- ignore: requires external files -->
```rust,ignore
#[path = "foo.rs"]
mod c;
```

源文件 | `c` 的文件位置 | `c` 的模块路径
-------------- | ------------------- | ----------------------
`src/a/b.rs`   | `src/a/foo.rs`      | `crate::a::b::c`
`src/a/mod.rs` | `src/a/foo.rs`      | `crate::a::c`

r[items.mod.outlined.path.search-nested]
对于内联模块块内部的 `path` 属性，文件路径的相对位置取决于 `path` 属性所在的源文件类型。“mod-rs” 源文件是根模块（如 `lib.rs` 或 `main.rs`）和文件名为 `mod.rs` 的模块。“non-mod-rs” 源文件是所有其它模块文件。位于 mod-rs 文件中内联模块块内部的 `path` 属性的路径相对于 mod-rs 文件所在的目录，其中包含作为目录的内联模块组件。对于 non-mod-rs 文件，情况相同，不同之处在于路径起始于一个以 non-mod-rs 模块命名的目录。例如，以下代码片段将根据其所在位置使用显示的路径：

<!-- ignore: requires external files -->
```rust,ignore
mod inline {
    #[path = "other.rs"]
    mod inner;
}
```

源文件 | `inner` 的文件位置 | `inner` 的模块路径
-------------- | --------------------------| ----------------------------
`src/a/b.rs`   | `src/a/b/inline/other.rs` | `crate::a::b::inline::inner`
`src/a/mod.rs` | `src/a/inline/other.rs`   | `crate::a::inline::inner`

结合上述关于内联模块及其内部嵌套模块的 `path` 属性规则的示例（适用于 mod-rs 和 non-mod-rs 文件）：

<!-- ignore: requires external files -->
```rust,ignore
#[path = "thread_files"]
mod thread {
    // 相对于此源文件的目录，从 `thread_files/tls.rs` 加载 `local_data` 模块。
    #[path = "tls.rs"]
    mod local_data;
}
```

r[items.mod.attributes]
## 模块上的属性

r[items.mod.attributes.intro]
模块与所有 项 一样，接受外部属性。它们也接受内部属性：对于带有主体的模块是在 `{` 之后，或者在源文件的开头，在可选的 BOM 和 shebang 之后。

r[items.mod.attributes.supported]
对模块有意义的内置属性有 [`cfg`][`cfg`]、[`deprecated`][`deprecated`]、[`doc`][`doc`]、[lint 检查属性][the lint check attributes]、[`path`][`path`] 以及 [`no_implicit_prelude`][`no_implicit_prelude`]。模块也接受宏属性。

[`cfg`]: ../conditional-compilation.md
[`deprecated`]: ../attributes/diagnostics.md#the-deprecated-attribute
[`doc`]: https://doc.rust-lang.org/rustdoc/the-doc-attribute.html
[`no_implicit_prelude`]: ../names/preludes.md#the-no_implicit_prelude-attribute
[`path`]: #path属性
[attribute]: ../attributes.md
[items]: ../items.md
[module path]: ../paths.md
[scopes chapter]: ../names/scopes.md
[the lint check attributes]: ../attributes/diagnostics.md#lint-check-attributes
[type namespace]: ../names/namespaces.md
