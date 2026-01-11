<!-- https://github.com/rust-lang/reference/blob/master/src/visibility-and-privacy.md -->
<!-- commit 68bdfd15fa9feebfbf94a06cf574de598e239198 -->

r[vis]
# 可见性和私有性

r[vis.syntax]
```grammar,项
Visibility ->
      `pub`
    | `pub` `(` `crate` `)`
    | `pub` `(` `self` `)`
    | `pub` `(` `super` `)`
    | `pub` `(` `in` SimplePath `)`
```

r[vis.intro]
这两个术语经常互换使用，它们试图传达的是对 "这个 项 是否可以在此位置使用？" 这一问题的回答。

r[vis.name-hierarchy]
Rust 的名称解析在命名空间的全局层级结构上运行。层级结构中的每一层都可以被看作是某个 项 。这些 项 是上述提到的那些，但也包括外部 crate 。声明或定义一个新模块可以被看作是在定义所在的位置向层级结构中插入一棵新树。

r[vis.privacy]
为了控制接口是否可以跨模块使用，Rust 会检查每个 项 的使用，看它是否应该被允许。这就是生成私有性警告的地方，或者说是 "你使用了另一个模块的私有 项 ，这是不被允许的。"

r[vis.default]
默认情况下，一切都是 *private* (私有的)，但有两个例外： `pub` 特型 中的关联 项 默认是公有的； `pub` 枚举中的枚举 变体 默认也是公有的。当一个 项 被声明为 `pub` 时，可以认为它是可以被外界访问的。例如：

```rust
# fn main() {}
// 声明一个私有的结构体
struct Foo;

// 声明一个带有一个私有字段的公有结构体
pub struct Bar {
    field: i32,
}

// 声明一个带有两个公有变体的公有枚举
pub enum State {
    PubliclyAccessibleState,
    PubliclyAccessibleState2,
}
```

r[vis.access]
有了 项 是公有或私有这一概念，Rust 在两种情况下允许 项 访问：

1. 如果一个 项 是公有的，那么如果你可以从模块 `m` 访问该 项 的所有祖先模块，那么它就可以从模块 `m` 外部访问。你还可能能够通过重导出为该 项 命名。见下文。
2. 如果一个 项 是私有的，它可以被当前模块及其后代访问。

这两条规则在创建暴露公有 API 同时隐藏内部实现细节的模块层级结构方面出奇地强大。为了帮助解释，这里有几个用例及其含义：

* 库开发人员需要向链接到其库的 crate 暴露功能。作为第一种情况的结果，这意味着任何可以从外部使用的东西，从根部到目标 项 都必须是 `pub` 的。链条中任何私有的 项 都会禁止外部访问。

* 一个 crate 需要为其自身提供一个全局可用的 "辅助模块"，但它不想将该辅助模块作为公有 API 暴露。为了实现这一点，该 crate 层级结构的根部会有一个私有模块，然后该模块内部有一个 "公有 API"。因为整个 crate 都是根部的后代，所以整个本地 crate 都可以通过第二种情况访问这个私有模块。

* 在为模块编写单元测试时，一个常见的惯例是拥有一个名为 `mod test` 的待测模块的直接子模块。该模块可以通过第二种情况访问父模块的任何 项 ，这意味着内部实现细节也可以从子模块中无缝测试。

在第二种情况中提到，一个私有的 项 "可以被" 当前模块及其后代访问，但访问一个 项 的确切含义取决于该 项 是什么。

r[vis.usage]
例如，访问一个模块意味着查看其内部（以导入更多 项 ）。另一方面，访问一个函数意味着它被调用。此外，路径表达式和导入语句被认为是访问一个 项 ，因为只有当目标在当前的可见性作用域内时，导入/表达式才有效。

这里有一个体现了上述三种情况的程序示例：

```rust
// 这个模块是私有的，意味着没有外部 crate 可以访问这个模块。
// 然而，由于它在当前 crate 的根部是私有的，crate 中的任何模块
// 都可以访问这个模块中任何公开可见的项。
mod crate_helper_module {

    // 这个函数可以被当前 crate 中的任何东西使用
    pub fn crate_helper() {}

    // 这个函数 *不能* 被 crate 中的其他任何东西使用。它在
    // `crate_helper_module` 之外不是公开可见的，所以只有
    // 当前模块及其后代可以访问它。
    fn implementation_detail() {}
}

// 这个函数是 "对根部公有" 的，意味着它对链接到此 crate 的
// 外部 crate 可用。
pub fn public_api() {}

// 与 'public_api' 类似，这个模块是公有的，因此外部 crate 可以查看其内部。
pub mod submodule {
    use crate::crate_helper_module;

    pub fn my_method() {
        // 本地 crate 中的任何项都可以通过上述两条规则的组合
        // 调用辅助模块的公有接口。
        crate_helper_module::crate_helper();
    }

    // 这个函数对任何不是 `submodule` 后代的模块都是隐藏的
    fn my_implementation() {}

    #[cfg(test)]
    mod test {

        #[test]
        fn test_my_implementation() {
            // 因为这个模块是 `submodule` 的后代，所以允许
            // 访问 `submodule` 内部的私有项而不会违反私有性。
            super::my_implementation();
        }
    }
}

# fn main() {}
```

为了使 Rust 程序通过私有性检查，基于上述两条规则，所有路径都必须是有效的访问。这包括所有的 `use` 语句、表达式、类型等。

r[vis.scoped]
## `pub(in path)`，`pub(crate)`，`pub(super)`，和`pub(self)`

r[vis.scoped.intro]
除了公有和私有，Rust 还允许用户声明一个 项 仅在给定的作用域内可见。 `pub` 限制的规则如下：

r[vis.scoped.in]
- `pub(in path)` 使一个 项 在提供的 `path` 内可见。 `path` 必须是一个简单路径，解析为声明其可见性的 项 的祖先模块。 `path` 中的每个标识符必须直接引用一个模块（而不是由 `use` 语句引入的名称）。

r[vis.scoped.crate]
- `pub(crate)` 使一个 项 在当前 crate 内可见。

r[vis.scoped.super]
- `pub(super)` 使一个 项 对父模块可见。这等同于 `pub(in super)` 。

r[vis.scoped.self]
- `pub(self)` 使一个 项 对当前模块可见。这等同于 `pub(in self)` 或根本不使用 `pub` 。

r[vis.scoped.edition2018]
> [!EDITION-2018]
> 从 2018 版次开始， `pub(in path)` 的路径必须以 `crate` ， `self` ，或 `super` 开头。2015 版次还可以使用以 `::` 开头的路径或来自 crate 根部的模块。

这里有一个例子：

```rust,edition2015
pub mod outer_mod {
    pub mod inner_mod {
        // 这个函数在 `outer_mod` 内可见
        pub(in crate::outer_mod) fn outer_mod_visible_fn() {}
        // 与上面相同，这仅在 2015 版次中有效。
        pub(in outer_mod) fn outer_mod_visible_fn_2015() {}

        // 这个函数对整个 crate 可见
        pub(crate) fn crate_visible_fn() {}

        // 这个函数在 `outer_mod` 内可见
        pub(super) fn super_mod_visible_fn() {
            // 这个函数是可见的，因为我们在同一个 `mod` 中
            inner_mod_visible_fn();
        }

        // 这个函数仅在 `inner_mod` 内可见，
        // 这与保持其私有是一样的。
        pub(self) fn inner_mod_visible_fn() {}
    }
    pub fn foo() {
        inner_mod::outer_mod_visible_fn();
        inner_mod::crate_visible_fn();
        inner_mod::super_mod_visible_fn();

        // 这个函数不再可见，因为我们在 `inner_mod` 之外
        // 错误！`inner_mod_visible_fn` 是私有的
        //inner_mod::inner_mod_visible_fn();
    }
}

fn bar() {
    // 这个函数仍然可见，因为我们在同一个 crate 中
    outer_mod::inner_mod::crate_visible_fn();

    // 这个函数不再可见，因为我们在 `outer_mod` 之外
    // 错误！`super_mod_visible_fn` 是私有的
    //outer_mod::inner_mod::super_mod_visible_fn();

    // 这个函数不再可见，因为我们在 `outer_mod` 之外
    // 错误！`outer_mod_visible_fn` 是私有的
    //outer_mod::inner_mod::outer_mod_visible_fn();

    outer_mod::foo();
}

fn main() { bar() }
```

> [!NOTE]
> 此语法格式仅为 项 的可见性添加了另一个限制。它并不保证该 项 在指定作用域的所有部分都可见。要访问一个 项 ，直到当前作用域的所有父 项 必须仍然也是可见的。

r[vis.reexports]
## 重导出和可见性

r[vis.reexports.intro]
Rust 允许通过 `pub use` 指令公开重导出 项 。因为这是一个公开指令，这允许根据上述规则在当前模块中使用该 项 。它本质上允许对重导出的 项 进行公有访问。例如，这个程序是有效的：

```rust
pub use self::implementation::api;

mod implementation {
    pub mod api {
        pub fn f() {}
    }
}

# fn main() {}
```

这意味着任何引用 `implementation::api::f` 的外部 crate 都会收到一个私有性违规，而路径 `api::f` 将是被允许的。

r[vis.reexports.private-item]
当重导出私有 项 时，可以将其看作是允许 "私有链" 通过重导出短路，而不是像通常那样通过命名空间层级结构。
