<!-- https://github.com/rust-lang/reference/blob/master/src/types/struct.md -->
<!-- commit 68bdfd15fa9feebfbf94a06cf574de598e239198 -->

r[type.struct]
# 结构体类型

r[type.struct.intro]
一个 结构体  *类型* 是其他类型的异构乘积，被称为该类型的 *字段* 。[^structtype]

r[type.struct.constructor]
 结构体 的新实例可以通过 [结构体表达式][struct expression] 构造。

r[type.struct.layout]
默认情况下， 结构体 的内存布局是未定义的，以便允许编译器进行诸如字段重排之类的优化，但它可以通过 [`repr` 属性][`repr` attribute] 固定。在任何一种情况下，字段都可以在相应的 结构体  *表达式* 中以任何顺序给出；生成的 结构体 值将始终具有相同的内存布局。

r[type.struct.field-visibility]
 结构体 的字段可以用 [可见性修饰符][visibility modifiers] 限定，以允许在模块外部访问 结构体 中的数据。

r[type.struct.tuple]
一个  *元组结构体* 类型就像 结构体 类型一样，除了字段是匿名的。

r[type.struct.unit]
一个  *类单元结构体* 类型就像 结构体 类型一样，除了它没有字段。通过关联的 [结构体表达式][struct expression] 构造的一个值是这种类型中唯一的值。

[^structtype]: 结构体 类型类似于 C 中的 结构体 类型、ML 系列的 *记录* 类型或 Lisp 系列的 *结构体* 类型。

[`repr` attribute]: ../type-layout.md#表示法
[struct expression]: ../expressions/struct-expr.md
[visibility modifiers]: ../visibility-and-privacy.md
