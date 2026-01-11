<!-- https://github.com/rust-lang/reference/blob/master/src/types/enum.md -->
<!-- commit 68bdfd15fa9feebfbf94a06cf574de598e239198 -->

r[type.enum]
# 枚举类型

r[type.enum.intro]
一个 *枚举类型* 是一种标称的、异构的离散联合类型，由 [`enum` 项][`enum` item] 的名称表示。[^enumtype]

r[type.enum.declaration]
一个 [`enum` 项][`enum` item] 声明了该类型以及若干 *变体* ，每个变体都有独立的名称，并且具有 结构体 、元组结构体或类单元结构体的 语法格式 。

r[type.enum.constructor]
`enum` 的新实例可以通过 [结构体表达式][struct expression] 构造。

r[type.enum.value]
任何 `enum` 值消耗的内存与其对应的 `enum` 类型中最大的 变体 一样多，外加存储判别式所需的大小。

r[type.enum.name]
枚举类型不能作为类型在 *结构上* 被表示，而必须通过对 [`enum` 项][`enum` item] 的命名引用来表示。

[^enumtype]: `enum` 类型类似于 Haskell 中的 `data` 构造函数声明，或 Limbo 中的 *pick ADT* 。

[`enum` item]: ../items/enumerations.md
[struct expression]: ../expressions/struct-expr.md
