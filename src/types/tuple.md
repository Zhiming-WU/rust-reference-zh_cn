r[type.tuple]
# 元组类型

r[type.tuple.syntax]
```grammar,types
TupleType ->
      `(` `)`
    | `(` ( Type `,` )+ Type? `)`
```

r[type.tuple.intro]
 *元组类型* 是一系列用于存放其它类型的异构列表的 结构化类型 [^1]。

元组类型的语法是一个括号括起来的、逗号分隔的类型列表。

r[type.tuple.restriction]
一元元组要求在其元素类型后加一个逗号，以便与 [括号括起来的类型][parenthesized type] 区分开来。

r[type.tuple.field-number]
元组类型的字段数量等于类型列表的长度。此字段数量决定了元组的 *元数* 。具有 `n` 个字段的元组被称为 *n 元元组* 。例如，具有 2 个字段的元组是 2 元元组。

r[type.tuple.field-name]
元组的字段使用与其在类型列表中的位置匹配的递增数字名称命名。第一个字段是 `0` 。第二个字段是 `1` 。依此类推。每个字段的类型是元组类型列表中相同位置的类型。

r[type.tuple.unit]
由于便利和历史原因，没有字段的元组类型 ( `()` ) 通常被称为 *unit* 或 *unit 类型* 。它的唯一值也被称为 *unit* 或 *unit 值* 。

一些元组类型的示例：

* `()` (unit)
* `(i32,)` (一元元组)
* `(f64, f64)`
* `(String, i32)`
* `(i32, String)` (与上一个示例不同的类型)
* `(i32, f64, Vec<String>, Option<bool>)`

r[type.tuple.constructor]
此类型的值是使用 [元组表达式][tuple expression] 构造的。此外，如果没有其他有意义的值可以求值，各种表达式都将产生 unit 值。

r[type.tuple.access]
元组字段可以通过 [元组索引表达式][tuple index expression] 或 [模式匹配][pattern matching] 来访问。

[^1]: 如果 结构化类型 的内部类型等效，则它们始终等效。对于元组的标称版本，请参阅 [元组结构体][tuple structs] 。

[parenthesized type]: ../types.md#parenthesized-types
[pattern matching]: ../patterns.md#tuple-patterns
[tuple expression]: ../expressions/tuple-expr.md#tuple-expressions
[tuple index expression]: ../expressions/tuple-expr.md#tuple-indexing-expressions
[tuple structs]: ./struct.md
