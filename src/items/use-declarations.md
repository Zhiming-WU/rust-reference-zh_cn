<!-- https://github.com/rust-lang/reference/blob/master/src/items/use-declarations.md -->
<!-- commit 68bdfd15fa9feebfbf94a06cf574de598e239198 -->

r[items.use]
# use声明

r[items.use.syntax]
```grammar,项
UseDeclaration -> `use` UseTree `;`

UseTree ->
      (SimplePath? `::`)? `*`
    | (SimplePath? `::`)? `{` (UseTree ( `,`  UseTree )* `,`?)? `}`
    | SimplePath ( `as` ( IDENTIFIER | `_` ) )?
```

r[items.use.intro]
一个  _use 声明_  创建一个或多个局部名称绑定，作为某个其他 [路径][path] 的同义词。通常， `use` 声明用于缩短引用模块 项 所需的路径。这些声明可以出现在 [模块][modules] 和 [块][blocks] 中，通常在顶部。
一个 `use` 声明有时也称为  _导入 (import)_ ，如果它是公共的，则称为  _重新导出 (re-export)_ 。

[path]: ../paths.md
[modules]: modules.md
[blocks]: ../expressions/block-expr.md

r[items.use.forms]
`use` 声明支持许多方便的快捷方式：

r[items.use.forms.multiple]
* 使用大括号语法 `use a::b::{c, d, e::f, g::h::i};` 同时绑定具有共同前缀的路径列表。

r[items.use.forms.self]
* 使用 `self` 关键字同时绑定具有共同前缀的路径列表及其共同的父模块，例如 `use a::b::{self, c, d::e};`。

r[items.use.forms.as]
* 使用 `use p::q::r as x;` 语法将目标名称重新绑定为新的局部名称。这也可以与最后两个特性一起使用： `use a::b::{self as ab, c as abc}`。

r[items.use.forms.glob]
* 使用星号通配符语法 `use a::b::*;` 绑定所有匹配给定前缀的路径。

r[items.use.forms.nesting]
* 多次嵌套前述特性的组，例如 `use a::b::{self as ab, c, d::{*, e::f}};`。

`use` 声明的示例：

```rust
use std::collections::hash_map::{self, HashMap};

fn foo<T>(_: T){}
fn bar(map1: HashMap<String, usize>, map2: hash_map::HashMap<String, usize>){}

fn main() {
    // use 声明也可以存在于函数内部
    use std::option::Option::{Some, None};

    // 等价于 'foo(vec![std::option::Option::Some(1.0f64),
    // std::option::Option::None]);'
    foo(vec![Some(1.0f64), None]);

    // `hash_map` 和 `HashMap` 都在作用域内。
    let map1 = HashMap::new();
    let map2 = hash_map::HashMap::new();
    bar(map1, map2);
}
```

r[items.use.visibility]
## `use`可见性

r[items.use.visibility.intro]
默认情况下，与 项 一样， `use` 声明对其包含的模块是私有的。与 项 一样，如果用 `pub` 关键字限定， `use` 声明也可以是公共的。这样的 `use` 声明用于  _重新导出 (re-export)_  一个名称。因此，公共 `use` 声明可以  _重定向 (redirect)_  某个公共名称到不同的目标定义：即使是位于不同模块内具有私有规范路径的定义。

r[items.use.visibility.unambiguous]
如果一系列此类重定向形成循环或无法明确解析，则它们会引发编译时错误。

重新导出的示例：

```rust
mod quux {
    pub use self::foo::{bar, baz};
    pub mod foo {
        pub fn bar() {}
        pub fn baz() {}
    }
}

fn main() {
    quux::bar();
    quux::baz();
}
```

在此示例中，模块 `quux` 重新导出了在 `foo` 中定义的两个公共名称。

r[items.use.path]
## `use`路径

r[items.use.path.intro]
`use` 项中允许的 [路径][path] 遵循 [简单路径][SimplePath] 语法格式，并且与表达式中可用的路径相似。
它们可以为以下内容创建绑定：

* 可命名的 [项][items]
* [枚举变体][Enum variants]
* [内置类型][Built-in types]
* [属性][Attributes]
* [派生宏][Derive macros]
* [`macro_rules`]

r[items.use.path.disallowed]
它们不能导入 [关联项][associated items]、 [泛型参数][generic parameters]、 [局部变量][local variables]、包含 [`Self`] 的路径或 [工具属性][tool attributes]。更多限制如下所述。

r[items.use.path.namespace]
`use` 将为所有导入实体的 [命名空间][namespaces] 创建绑定，但 `self` 导入仅从类型命名空间导入（如下所述）。
例如，以下示例说明了在两个命名空间中为相同名称创建绑定：

```rust
mod stuff {
    pub struct Foo(pub i32);
}

// 同时导入 `Foo` 类型和 `Foo` 构造器。
use stuff::Foo;

fn example() {
    let ctor = Foo; // 使用值命名空间中的 `Foo`。
    let x: Foo = ctor(123); // 使用类型命名空间中的 `Foo`。
}
```

r[items.use.path.edition2018]
> [!EDITION-2018]
> 在 2015 版次中， `use` 路径相对于 crate 根。例如：
>
> ```rust,edition2015
> mod foo {
>     pub mod example { pub mod iter {} }
>     pub mod baz { pub fn foobaz() {} }
> }
> mod bar {
>     // 从 crate 根解析 `foo`。
>     use foo::example::iter;
>     // `::` 前缀显式地从 crate 根解析 `foo`。
>     use ::foo::baz::foobaz;
> }
>
> # fn main() {}
> ```
>
> 2015 版次不允许 use 声明引用 [外部预导入][extern prelude]。
> 因此，在 2015 版次中仍然需要 [`extern crate`] 声明才能在 use 声明中引用外部 crate。
> 从 2018 版次开始， `use` 声明可以像 `extern crate` 一样指定外部 crate 依赖项。

r[items.use.as]
## `as`重命名

`as` 关键字可用于更改导入实体的名称。
例如：

```rust
// 为函数 `foo` 创建一个非公共别名 `bar`。
use inner::foo as bar;

mod inner {
    pub fn foo() {}
}
```

r[items.use.multiple-syntax]
## 大括号语法

r[items.use.multiple-syntax.intro]
大括号可用于路径的最后一个段中，以从前一个段导入多个实体，或者，如果前一个段不存在，则从当前作用域导入。
大括号可以嵌套，创建路径树，其中每个段组都与其父级逻辑组合以创建完整路径。

```rust
// 创建以下绑定：
// - `std::collections::BTreeSet`
// - `std::collections::hash_map`
// - `std::collections::hash_map::HashMap`
use std::collections::{BTreeSet, hash_map::{self, HashMap}};
```

r[items.use.multiple-syntax.empty]
一个空大括号不导入任何内容，尽管会验证其前导路径是否可访问。

r[items.use.multiple-syntax.edition2018]
> [!EDITION-2018]
> 在 2015 版次中，路径是相对于 crate 根的，因此像 `use {foo, bar};` 这样的导入会从 crate 根导入名称 `foo` 和 `bar`，而从 2018 版次开始，这些名称是相对于当前作用域的。

r[items.use.self]
## `self`导入

r[items.use.self.intro]
关键字 `self` 可在 [大括号语法](#大括号语法) 中使用，以其自身名称创建父实体的绑定。

```rust
mod stuff {
    pub fn foo() {}
    pub fn bar() {}
}
mod example {
    // 为 `stuff` 和 `foo` 创建绑定。
    use crate::stuff::{self, foo};
    pub fn baz() {
        foo();
        stuff::bar();
    }
}
# fn main() {}
```

r[items.use.self.namespace]
`self` 只从父实体的 [类型命名空间][type namespace] 创建绑定。
例如，在以下代码中，只导入了 `foo` 模块：

```rust,compile_fail
mod bar {
    pub mod foo {}
    pub fn foo() {}
}

// 这仅导入模块 `foo`。函数 `foo` 存在于
// 值命名空间中，且未被导入。
use bar::foo::{self};

fn main() {
    foo(); //~ 错误 `foo` 是一个模块
}
```

> [!Note]
> `self` 也可以用作路径的第一个段。 `self` 作为第一个段的用法与在 `use` 大括号内的用法在逻辑上是相同的；它表示父段的当前模块，或者如果没有父段，则表示当前模块。有关前导 `self` 的含义的更多信息，请参见路径章节中的 [`self`]。

r[items.use.glob]
## 通配符导入

r[items.use.glob.intro]
`*` 字符可用作 `use` 路径的最后一个段，以从前一个段的实体中导入所有可导入的实体。
例如：

```rust
// 为 `bar` 创建一个非公共别名。
use foo::*;

mod foo {
    fn i_am_private() {}
    enum Example {
        V1,
        V2,
    }
    pub fn bar() {
        // 为 `Example` 枚举的 `V1` 和 `V2` 创建局部别名。
        use Example::*;
        let x = V1;
    }
}
```

r[items.use.glob.shadowing]
允许 项 和命名导入遮蔽来自同一 [命名空间][namespaces] 中通配符导入的名称。
也就是说，如果同一命名空间中已由另一个 项 定义了某个名称，则通配符导入将被遮蔽。
例如：

```rust
// 这为 `clashing::Foo` 元组结构体构造器创建了一个绑定，
// 但没有导入其类型，因为那会与此处定义的 `Foo` 结构体冲突。
//
// 注意，此处的定义顺序并不重要。
use clashing::*;
struct Foo {
    field: f32,
}

fn do_stuff() {
    // 使用 `clashing::Foo` 的构造器。
    let f1 = Foo(123);
    // 结构体表达式使用上面定义的 `Foo` 结构体的类型。
    let f2 = Foo { field: 1.0 };
    // 由于通配符导入， `Bar` 也在作用域内。
    let z = Bar {};
}

mod clashing {
    pub struct Foo(pub i32);
    pub struct Bar {}
}
```

> [!NOTE]
> 对于不允许遮蔽的区域，请参见 [名称解析歧义][name resolution ambiguities]。

r[items.use.glob.last-segment-only]
`*` 不能用作第一个或中间段。

r[items.use.glob.self-import]
`*` 不能用于将模块的内容导入自身（例如 `use self::*;` ）。

r[items.use.glob.edition2018]
> [!EDITION-2018]
> 在 2015 版次中，路径是相对于 crate 根的，因此像 `use *;` 这样的导入是有效的，它意味着从 crate 根导入所有内容。这不能在 crate 根本身中使用。

r[items.use.as-underscore]
## 下划线导入

r[items.use.as-underscore.intro]
可以通过使用下划线形式 `use path as _` 导入 项 而不绑定到名称。这对于导入一个 特型 特别有用，以便可以使用其方法而无需导入该 特型 的符号，例如，如果该 特型 的符号可能与其他符号冲突。另一个例子是链接外部 crate 而不导入其名称。

r[items.use.as-underscore.glob]
星号通配符导入将以不可命名的形式导入带有 `_` 的 项。

```rust
mod foo {
    pub trait Zoo {
        fn zoo(&self) {}
    }

    impl<T> Zoo for T {}
}

use self::foo::Zoo as _;
struct Zoo;  // 下划线导入避免与此项发生命名冲突。

fn main() {
    let z = Zoo;
    z.zoo();
}
```

r[items.use.as-underscore.macro]
独特的、不可命名的符号在宏展开后创建，这样宏可以安全地发出对 `_` 导入的多个引用。例如，以下代码不应产生错误：

```rust
macro_rules! m {
    ($item: item) => { $item $item }
}

m!(use std as _;);
// 这将展开为：
// use std as _;
// use std as _;
```

r[items.use.restrictions]
## 限制

以下是有效 `use` 声明的限制：

r[items.use.restrictions.crate]
* `use crate;` 必须使用 `as` 来定义要绑定 crate 根的名称。

r[items.use.restrictions.self]
* `use {self};` 是一个错误；使用 `self` 时必须有一个前导段。

r[items.use.restrictions.duplicate-name]
* 与任何 项 定义一样， `use` 导入不能在模块或块中的同一命名空间内创建相同名称的重复绑定。

r[items.use.restrictions.macro-crate]
* 包含 `$crate` 的 `use` 路径不允许在 [`macro_rules`] 展开中使用。

r[items.use.restrictions.variant]
* `use` 路径不能通过 [类型别名][type alias] 引用枚举变体。例如：
  ```rust,compile_fail
  enum MyEnum {
      MyVariant
  }
  type TypeAlias = MyEnum;

  use MyEnum::MyVariant; //~ 正常
  use TypeAlias::MyVariant; //~ 错误
  ```

[Attributes]: ../attributes.md
[Built-in types]: ../types.md
[Derive macros]: macro.proc.derive
[Enum variants]: enumerations.md
[`extern crate`]: extern-crates.md
[`macro_rules`]: ../macros-by-example.md
[`self`]: ../paths.md#self
[associated items]: associated-items.md
[extern prelude]: ../names/preludes.md#extern-prelude
[generic parameters]: generics.md
[items]: ../items.md
[local variables]: ../variables.md
[name resolution ambiguities]: names.resolution.expansion.imports.ambiguity
[namespace]: ../names/namespaces.md
[namespaces]: ../names/namespaces.md
[paths]: ../paths.md
[tool attributes]: ../attributes.md#tool-attributes
[type alias]: type-aliases.md
[type namespace]: ../names/namespaces.md
