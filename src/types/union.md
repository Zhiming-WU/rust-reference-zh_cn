r[type.union]
# 联合体类型

r[type.union.intro]
一个 *联合体 类型* 是一种标称的、异构的类 C 联合体，由 [`union` 项][item] 的名称表示。

r[type.union.access]
联合体 没有 "active field" 的概念。相反，每次对 联合体 的访问都会将 联合体 内容的一部分转录为被访问字段的类型。

r[type.union.safety]
由于转录可能导致意外或未定义的行为，因此从 联合体 字段中读取数据需要 `unsafe` 。

r[type.union.constraint]
联合体 字段类型也被限制为类型的一个子集，以确保它们永远不需要丢弃。有关更多详细信息，请参阅 [项][item] 文档。

r[type.union.layout]
默认情况下， `union` 的内存布局是未定义的（特别是，字段 *不* 一定要在偏移量 0 处），但 `#[repr(...)]` 属性可以用来固定布局。

[`Copy`]: ../special-types-and-traits.md#copy
[item]: ../items/unions.md
