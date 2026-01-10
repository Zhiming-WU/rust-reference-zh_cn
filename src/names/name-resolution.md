r[names.resolution]
# 名称解析

r[names.resolution.intro]
_名称解析_ 是将路径和其他标识符绑定到这些实体的声明的过程。名称被分隔到不同的 [命名空间][namespaces] 中，允许不同命名空间中的实体在没有冲突的情况下共享相同的名称。每个名称在 [作用域][scope]（即可以引用该名称的源代码区域）内有效。对名称的访问可能会根据其 [可见性][visibility] 受到限制。

在整个编译过程中，名称解析分为三个阶段。第一个阶段， * 展开时解析 *，解析所有的 [`use` 声明][`use` declarations] 和 [宏调用][macro invocations]。第二个阶段， * 初级解析 *，解析所有尚未解析且不依赖于类型信息来解析的名称。最后一个阶段， * 类型相关解析 *，一旦类型信息可用，就解析剩余的名称。

> [!NOTE]
> 展开时解析 也被称为 * 早期解析 *。 初级解析 也被称为 * 晚期解析 *。

r[names.resolution.general]
## 通用

r[names.resolution.general.intro]
本节中的规则适用于名称解析的所有阶段。

r[names.resolution.general.scopes]
### 作用域

r[names.resolution.general.scopes.intro]
> [!NOTE]
> 这是未来关于各种作用域内名称解析扩展的占位符。

r[names.resolution.expansion]
## 展开时名称解析

r[names.resolution.expansion.intro]
展开时名称解析是完成宏展开并完全生成 crate 的 [AST] 所必需的名称解析阶段。此阶段需要解析宏调用和 `use` 声明。解析 `use` 声明是解析通过 [基于路径的作用域][path-based scope] 的宏调用所必需的。为了展开宏调用，必须先解析它们。

r[names.resolution.expansion.unresolved-invocations]
在展开时名称解析之后， [AST] 不得包含任何未展开的宏调用。每个宏调用都必须解析为存在于最终 [AST] 中或外部 crate 中的有效定义。

```rust,compile_fail
m!(); // 错误：在此作用域内找不到宏 `m`。
```

r[names.resolution.expansion.expansion-order-stability]
名称的解析必须是稳定的。展开后，完全展开的 [AST] 中的名称必须解析为相同的定义，无论宏展开和导入解析的顺序如何。

r[names.resolution.expansion.speculation]
宏展开期间选择的所有名称解析候选者都被视为推测性的。一旦 crate 被完全展开，所有的推测性导入解析都会被验证，以确保宏展开没有引入任何新的歧义。

> [!NOTE]
> 由于宏展开的迭代性质，这会导致所谓的时空穿越歧义，例如当宏或通配符导入引入了一个与其自身基路径歧义的项时。
>
> ```rust,compile_fail,E0659
> # fn main() {}
> macro_rules! f {
>     () => {
>         mod m {
>             pub(crate) use f;
>         }
>     }
> }
> f!();
>
> const _: () = {
>     // 最初，我们将 `m` 推测性地解析为 crate 根中的模块。
>     //
>     // `f` 的展开在此体内部引入了第二个 `m` 模块。
>     //
>     // 展开时解析通过重新解析所有导入和宏调用来完成解析，
>     // 发现引入的歧义并将其作为错误报告。
>     m::f!(); // 错误：`m` 存在歧义。
> };
> ```

r[names.resolution.expansion.imports]
### 导入
r[names.resolution.expansion.imports.intro]
所有的 `use` 声明都在此解析阶段被完全解析。 [类型相关路径][type-relative paths] 在此阶段无法解析，并将产生错误。

```rust,no_run
mod m {
    pub const C: () = ();
    pub enum E { V }
    pub type A = E;
    impl E {
        pub const C: () = ();
    }
}

// 在展开时解析的有效导入：
use m::C; // OK。
use m::E; // OK。
use m::A; // OK。
use m::E::V; // OK。

// 在类型相关解析期间解析的有效表达式：
let _ = m::A::V; // OK。
let _ = m::E::C; // OK。
```

```rust,compile_fail,E0432
# mod m {
#     pub const C: () = ();
#     pub enum E { V }
#     pub type A = E;
#     impl E {
#         pub const C: () = ();
#     }
# }
// 在展开时无法解析的无效类型相关导入：
use m::A::V; // 错误：未解析的导入 `m::A::V`。
use m::E::C; // 错误：未解析的导入 `m::E::C`。
```

r[names.resolution.expansion.imports.shadowing]
除了受 [名称解析歧义][name resolution ambiguities] 限制的情况外，通过 [外部作用域][outer scope] 中的 `use` 声明引入的名称会被内部作用域中同一命名空间内同名的候选者遮蔽。

```rust,no_run
pub mod m1 {
    pub mod ambig {
        pub const C: u8 = 1;
    }
}

pub mod m2 {
    pub mod ambig {
        pub const C: u8 = 2;
    }
}

// 这在外部作用域中引入了名称 `ambig`。
use m1::ambig;
const _: () = {
    // 这在内部作用域中遮蔽了 `ambig`。
    use m2::ambig;
    // 这里的 `ambig` 解析选择内部候选者。
    use ambig::C;
    assert!(C == 2);
};
```

r[names.resolution.expansion.imports.shadowing.shared-scope]
在单个作用域内，允许对通过 `use` 声明引入的名称进行遮蔽的情况如下：

- [`use` 通配符遮蔽][`use` glob shadowing]
- [宏文本作用域遮蔽][macro textual scope shadowing]

r[names.resolution.expansion.imports.ambiguity]
#### 歧义

r[names.resolution.expansion.imports.ambiguity.intro]
在展开时解析期间，某些情况下存在多个宏定义、 `use` 声明或模块，导入或宏调用的名称可能指向这些定义、声明或模块，而编译器无法始终如一地确定哪个候选者应该遮蔽另一个。在这些情况下不允许遮蔽，编译器会发出歧义错误。

r[names.resolution.expansion.imports.ambiguity.glob-vs-glob]
名称不能通过有歧义的通配符导入来解析。通配符导入允许在同一命名空间中导入冲突的名称，只要不使用该名称即可。具有来自歧义通配符导入的冲突候选者的名称仍然可以被非通配符导入遮蔽，并在不产生错误的情况下使用。错误发生在使用的时刻，而不是导入的时刻。

```rust,compile_fail,E0659
mod m1 {
    pub struct Ambig;
}

mod m2 {
    pub struct Ambig;
}

// OK：这将同一命名空间中冲突的名称引入作用域，但尚未被使用。
use m1::*;
use m2::*;

const _: () = {
    // 当使用具有冲突候选者的名称时发生错误。
    let x = Ambig; // 错误：`Ambig` 存在歧义。
}
```

```rust,no_run
# mod m1 {
#     pub struct Ambig;
# }
#
# mod m2 {
#     pub struct Ambig;
# }
#
# use m1::*;
# use m2::*; // OK：没有名称冲突。
const _: () = {
    // 这是允许的，因为解析不是通过歧义通配符进行的。
    struct Ambig;
    let x = Ambig; // OK。
};
```

允许通过多个通配符导入导入相同的名称，如果导入的是同一个项（遵循重导出），则允许使用该名称。该名称的可见性是这些导入中的最大可见性。

```rust,no_run
mod m1 {
    pub struct Ambig;
}

mod m2 {
    // 这从第二个模块重导出同一个 `Ambig` 项。
    pub use super::m1::Ambig;
}

mod m3 {
    // 这两者都导入同一个 `Ambig`。
    //
    // `Ambig` 的可见性是 `pub`，因为这是这两个 `use` 声明之间的最大可见性。
    pub use super::m1::*;
    use super::m2::*;
}

mod m4 {
    // `Ambig` 可以通过 `m3` 的通配符使用，且仍然具有 `pub` 可见性。
    pub use crate::m3::Ambig;
}

const _: () = {
    // 因此，我们可以在这里使用它。
    let _ = m4::Ambig; // OK。
};
# fn main() {}
```

r[names.resolution.expansion.imports.ambiguity.glob-vs-outer]
当在 [外部作用域][outer scope] 中有另一个可用候选者时，导入和宏调用中的名称不能通过通配符导入来解析。

```rust,compile_fail,E0659
mod glob {
    pub mod ambig {
        pub struct Name;
    }
}

// 外部 `ambig` 候选者。
pub mod ambig {
    pub struct Name;
}

const _: () = {
    // 无法通过此通配符解析 `ambig`，
    // 因为上方有外部 `ambig` 候选者。
    use glob::*;
    use ambig::Name; // 错误：`ambig` 存在歧义。
};
```

```rust,compile_fail,E0659
// 如上，但使用宏。
pub mod m {
    macro_rules! f {
        () => {};
    }
    pub(crate) use f;
}
pub mod glob {
    macro_rules! f {
        () => {};
    }
    pub(crate) use f as ambig;
}

use m::f as ambig;

const _: () = {
    use glob::*;
    ambig!(); // 错误：`ambig` 存在歧义。
};
```

> [!NOTE]
> 这些歧义错误特定于展开时解析。在解析的后续阶段，为一个给定的名称提供多个候选者不被视为错误。只要导入本身没有歧义，就总是会有一个唯一的、无歧义的最近解析。
>
> ```rust,no_run
> mod glob {
>     pub const AMBIG: u8 = 1;
> }
>
> mod outer {
>     pub const AMBIG: u8 = 2;
> }
>
> use outer::AMBIG;
>
> const C: () = {
>     use glob::*;
>     assert!(AMBIG == 1);
>     //      ^---- 这个 `AMBIG` 在初级解析期间解析。
> };
> ```

r[names.resolution.expansion.imports.ambiguity.path-vs-textual-macro]
名称不能通过有歧义的宏重导出进行解析。当宏重导出会遮蔽 [外部作用域][outer scope] 中同名的文本宏候选者时，宏重导出即具有歧义。

```rust,compile_fail,E0659
// 文本宏候选者。
macro_rules! ambig {
    () => {}
}

// 基于路径的宏候选者。
macro_rules! path_based {
    () => {}
}

pub fn f() {
    // 将 `path_based` 宏定义重导出为 `ambig` 
    // 不得遮蔽通过文本宏作用域解析的 `ambig` 宏定义。
    use path_based as ambig;
    ambig!(); // 错误：`ambig` 存在歧义。
}
```

> [!NOTE]
> 这种限制是由于编译器中的实现细节而需要的，特别是当前的作用域访问逻辑和支持此行为的复杂性。此歧义错误可能会在未来被移除。

r[names.resolution.expansion.macros]
### 宏

r[names.resolution.expansion.macros.intro]
宏通过遍历可用作用域来查找可用候选者进行解析。宏被分为两个子命名空间，一个用于类函数宏，另一个用于属性和派生宏。来自错误子命名空间的解析候选者将被忽略。

r[names.resolution.expansion.macros.visitation-order]
可用作用域种类的访问顺序如下。这些作用域种类中的每一个都代表一个或多个作用域。

* [派生辅助属性][Derive helpers]
* [文本作用域宏][Textual scope macros]
* [基于路径的作用域宏][Path-based scope macros]
* [`macro_use` 预导入][`macro_use` prelude]
* [标准库预导入][Standard library prelude]
* [内置属性][Builtin attributes]

> [!NOTE]
> 编译器将尝试解析在其关联宏将其引入作用域之前使用的派生辅助属性。此作用域在访问已正确处于作用域内的派生辅助候选者的作用域之后被访问。此行为计划被移除。
>
> 更多信息请参见 [派生辅助属性作用域][derive helper scope]。

> [!NOTE]
> 此访问顺序将来可能会改变，例如根据文本和基于路径的作用域候选者的词法作用域来交替访问它们。

> [!EDITION-2018]
> 从 2018 版次开始，当存在 [`#[no_implicit_prelude]`][names.preludes.no_implicit_prelude] 时，不会访问 `#[macro_use]` 预导入。

r[names.resolution.expansion.macros.reserved-names]
名称 `cfg` 和 `cfg_attr` 在宏属性 [子命名空间][sub-namespace] 中被保留。

r[names.resolution.expansion.macros.ambiguity]
#### 歧义

r[names.resolution.expansion.macros.ambiguity.more-expanded-vs-outer]
名称不能通过宏展开内部的歧义候选者进行解析。当宏展开内部的候选者会遮蔽来自第一个候选者宏展开外部的同名候选者，且正在解析的名称调用也来自第一个候选者宏展开外部时，宏展开内部的候选者即具有歧义。

```rust,compile_fail,E0659
macro_rules! define_ambig {
    () => {
        macro_rules! ambig {
            () => {}
        }
    }
}

// 为 `ambig` 宏调用引入外部候选者定义。
macro_rules! ambig {
    () => {}
}

// 在宏展开内部引入第二个 `ambig` 候选者定义。
define_ambig!();

// 来自 `define_ambig` 第二次调用的 `ambig` 定义是最内部的候选者。
//
// 来自 `define_ambig` 第一次调用的 `ambig` 定义是第二个候选者。
//
// 编译器检查第一个候选者是否在宏展开内部，
// 第二个候选者是否不在同一个宏展开内部，
// 以及正在解析的名称是否不在同一个宏展开内部。
ambig!(); // 错误：`ambig` 存在歧义。
```

反过来则不被视为歧义。

```rust,no_run
# macro_rules! define_ambig {
#     () => {
#         macro_rules! ambig {
#             () => {}
#         }
#     }
# }
// 交换定义顺序。
define_ambig!();
macro_rules! ambig {
    () => {}
}
// 最内部的候选者现在展开程度较低，因此它可以遮蔽其上方的宏展开定义。
ambig!();
```

如果正在解析的调用在最内部候选者的展开范围内，也不是歧义。

```rust,no_run
macro_rules! ambig {
    () => {}
}

macro_rules! define_and_invoke_ambig {
    () => {
        // 定义最内部候选者。
        macro_rules! ambig {
            () => {}
        }

        // `ambig` 的调用与最内部候选者在同一个展开中。
        ambig!(); // OK
    }
}

define_and_invoke_ambig!();
```

即使两个定义都来自同一个宏的调用，这也没有关系；最外部的候选者仍然被视为 “展开较少”，因为它不在包含最内部候选者定义的展开范围内。

```rust,compile_fail,E0659
# macro_rules! define_ambig {
#     () => {
#         macro_rules! ambig {
#             () => {}
#         }
#     }
# }
define_ambig!();
define_ambig!();
ambig!(); // 错误：`ambig` 存在歧义。
```

这也适用于导入，只要名称的最内部候选者来自宏展开。

```rust,compile_fail,E0659
macro_rules! define_ambig {
    () => {
        mod ambig {
            pub struct Name;
        }
    }
}

mod ambig {
    pub struct Name;
}

const _: () = {
    // 在此宏展开中引入 `ambig` 模块的最内部候选者。
    define_ambig!();
    use ambig::Name; // 错误：`ambig` 存在歧义。
};
```

r[names.resolution.expansion.macros.ambiguity.built-in-attr]
用户定义的属性或派生宏不得遮蔽内置的非宏属性（例如 inline）。

<!-- ignore: test doesn't support proc-macro -->
```rust,ignore
// with-helper/src/lib.rs
# use proc_macro::TokenStream;
#[proc_macro_derive(WithHelperAttr, attributes(non_exhaustive))]
//                                             ^^^^^^^^^^^^^^
//                                   用户定义的属性候选者。
// ...
# pub fn derive_with_helper_attr(_item: TokenStream) -> TokenStream {
#     TokenStream::new()
# }
```

<!-- ignore: requires external crates -->
```rust,ignore
// src/lib.rs
#[derive(with_helper::WithHelperAttr)]
#[non_exhaustive] // 错误：`non_exhaustive` 存在歧义。
struct S;
```

> [!NOTE]
> 无论内置属性是哪个名称的候选者，这都适用：
>
> <!-- ignore: test doesn't support proc-macro -->
> ```rust,ignore
> // with-helper/src/lib.rs
> # use proc_macro::TokenStream;
> #
> #[proc_macro_derive(WithHelperAttr, attributes(helper))]
> //                                             ^^^^^^
> //                                 用户定义的属性候选者。
> // ...
> # pub fn derive_with_helper_attr(_item: TokenStream) -> TokenStream {
> #     TokenStream::new()
> # }
> ```
>
> <!-- ignore: requires external crates -->
> ```rust,ignore
> // src/lib.rs
> use inline as helper;
> //            ^----- 通过重导出的内置属性候选者。
>
> #[derive(with_helper::WithHelperAttr)]
> #[helper] // 错误：`helper` 存在歧义。
> struct S;
> ```

r[names.resolution.primary]
## 初级名称解析
> [!NOTE]
> 这是未来关于初级名称解析扩展的占位符。

r[names.resolution.type-relative]
## 类型相关解析
> [!NOTE]
> 这是未来关于类型依赖解析扩展的占位符。

[AST]: glossary.ast
[Builtin attributes]: ./preludes.md#r-names.preludes.lang
[Derive helpers]: ../procedural-macros.md#r-macro.proc.derive.attributes
[Macros]: ../macros.md
[Path-based scope macros]: ../macros.md#r-macro.invocation.name-resolution
[Standard library prelude]: ./preludes.md#r-names.preludes.std
[Textual scope macros]: ../macros-by-example.md#r-macro.decl.scope.textual
[`let` bindings]: ../statements.md#let-statements
[`macro_use` prelude]: ./preludes.md#r-names.preludes.macro_use
[`use` declarations]: ../items/use-declarations.md
[`use` glob shadowing]: ../items/use-declarations.md#r-items.use.glob.shadowing
[derive helper scope]: ../procedural-macros.md#r-macro.proc.derive.attributes.scope
[item definitions]: ../items.md
[macro invocations]: ../macros.md#宏调用
[macro textual scope shadowing]: ../macros-by-example.md#r-macro.decl.scope.textual.shadow
[name resolution ambiguities]: #r-names.resolution.expansion.imports.ambiguity
[namespaces]: ../names/namespaces.md
[outer scope]: #r-names.resolution.general.scopes
[path-based scope]: ../macros.md#r-macro.invocation.name-resolution
[scope]: ../names/scopes.md
[sub-namespace]: ../names/namespaces.md#r-names.namespaces.sub-namespaces
[type-relative paths]: names.resolution.type-relative
[visibility]: ../visibility-and-privacy.md
