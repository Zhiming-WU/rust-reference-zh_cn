<!-- https://github.com/rust-lang/reference/blob/master/src/items.md -->
<!-- commit 68bdfd15fa9feebfbf94a06cf574de598e239198 -->

r[items]
# 项

r[items.syntax]
```grammar,项
Item ->
    OuterAttribute* ( VisItem | MacroItem )

VisItem ->
    Visibility?
    (
        Module
      | ExternCrate
      | UseDeclaration
      | Function
      | TypeAlias
      | Struct
      | Enumeration
      | Union
      | ConstantItem
      | StaticItem
      | Trait
      | Implementation
      | ExternBlock
    )

MacroItem ->
      MacroInvocationSemi
    | MacroRulesDefinition
```

r[items.intro]
一个 _项_ 是 crate 的一个组件。 项 在 crate 中通过一组嵌套的 [模块][modules] 来组织。每个 crate 都有一个唯一的“最外层”匿名模块；crate 内的所有后续 项 在 crate 的模块树中都有 [路径][paths]。

r[items.static-def]
项 完全在编译时确定，通常在执行期间保持固定，并可能驻留在只读内存中。

r[items.kinds]
有几种类型的 项 ：

* [模块][modules]
* [`extern crate` 声明][`extern crate` declarations]
* [`use` 声明][`use` declarations]
* [函数定义][function definitions]
* [类型别名定义][type alias definitions]
* [结构体定义][struct definitions]
* [枚举定义][enumeration definitions]
* [联合体定义][union definitions]
* [常量项][constant items]
* [静态项][static items]
* [特型定义][trait definitions]
* [实现][implementations]
* [`extern` 块][`extern` blocks]

r[items.locations]
项 可以在 [crate 根部][root of the crate]、[模块][modules] 或 [块表达式][block expression] 中声明。

r[items.associated-locations]
项 的一个子集，称为 [关联项][associated items]，可以在 [特型][traits] 和 [实现][implementations] 中声明。

r[items.extern-locations]
项 的一个子集，称为外部项，可以在 [`extern` 块][`extern` blocks] 中声明。

r[items.decl-order]
项 可以按任何顺序定义，唯一的例外是 [`macro_rules`]，它有自己的作用域行为。

r[items.name-resolution]
项 名称的 [名称解析][Name resolution] 允许在模块或块中引用该 项 的位置之前或之后定义它。

有关 项 的作用域规则，请参见 [项作用域][item scopes]。

[`extern crate` declarations]: items/extern-crates.md
[`extern` blocks]: items/external-blocks.md
[`macro_rules`]: macros-by-example.md
[`use` declarations]: items/use-declarations.md
[associated items]: items/associated-items.md
[block expression]: expressions/block-expr.md
[constant items]: items/constant-items.md
[enumeration definitions]: items/enumerations.md
[function definitions]: items/functions.md
[implementations]: items/implementations.md
[item scopes]: names/scopes.md#项作用域
[modules]: items/modules.md
[name resolution]: names/name-resolution.md
[paths]: paths.md
[root of the crate]: crates-and-source-files.md
[statement]: statements.md
[static items]: items/static-items.md
[struct definitions]: items/structs.md
[trait definitions]: items/traits.md
[traits]: items/traits.md
[type alias definitions]: items/type-aliases.md
[union definitions]: items/unions.md
