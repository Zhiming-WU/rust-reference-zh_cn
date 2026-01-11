r[expr.tuple]
# 元组和元组索引表达式

## 元组表达式

r[expr.tuple.syntax]
```grammar,表达式
TupleExpression -> `(` TupleElements? `)`

TupleElements -> ( Expression `,` )+ Expression?
```

r[expr.tuple.result]
一个  *元组表达式*  构造 [元组值][tuple type]。

r[expr.tuple.intro]
元组表达式 的 语法格式 是括号括起来的、逗号分隔的表达式列表，称为  *元组初始化操作数* 。

r[expr.tuple.unary-tuple-restriction]
一元 元组表达式 要求在其 元组初始化操作数 之后加一个逗号，以便与 [括号表达式][parenthetical expression] 区分开。

r[expr.tuple.value]
元组表达式 是 [值表达式][value expression]，它求值为新构造的 元组类型 的值。

r[expr.tuple.type]
元组初始化操作数 的数量是构造出的元组的元数。

r[expr.tuple.unit]
没有任何 元组初始化操作数 的 元组表达式 会产生单元元组。

r[expr.tuple.fields]
对于其他 元组表达式，第一个写入的 元组初始化操作数 初始化字段 `0`，随后的操作数初始化下一个更高的字段。
例如，在 元组表达式 `('a', 'b', 'c')` 中，`'a'` 初始化字段 `0` 的值，`'b'` 字段 `1`，而 `'c'` 字段 `2`。

元组表达式 及其类型的示例：

| 表达式           | 类型         |
| -------------------- | ------------ |
| `()`                 | `()` (单元)  |
| `(0.0, 4.5)`         | `(f64, f64)` |
| `("x".to_string(), )` | `(String, )`  |
| `("a", 4usize, true)`| `(&'static str, usize, bool)` |

r[expr.tuple-index]
## 元组索引表达式

r[expr.tuple-index.syntax]
```grammar,表达式
TupleIndexingExpression -> Expression `.` TUPLE_INDEX
```

r[expr.tuple-index.intro]
一个  *元组索引表达式*  访问 [元组][tuple type] 和 [元组结构体][tuple struct] 的字段。

元组索引表达式 的 语法格式 是一个表达式（称为  *元组操作数* ），然后是一个 `.`，最后是一个元组索引。

r[expr.tuple-index.index-syntax]
 *元组索引*  的 语法格式 是一个没有前导零、下划线或后缀的 [十进制字面量][decimal literal]。
例如 `0` 和 `2` 是有效的元组索引，但 `01`、`0_` 和 `0i32` 不是。

r[expr.tuple-index.required-type]
元组操作数 的类型必须是 [元组类型][tuple type] 或 [元组结构体][tuple struct]。

r[expr.tuple-index.index-name-operand]
元组索引必须是 元组操作数 类型的一个字段名。

r[expr.tuple-index.result]
元组索引表达式 的求值除了其 元组操作数 的求值之外没有副作用。
作为 [位置表达式][place expression]，它求值为 元组操作数 中与元组索引同名的字段的位置。

元组索引表达式 示例：

```rust
// 索引元组
let pair = ("a string", 2);
assert_eq!(pair.1, 2);

// 索引元组结构体
# struct Point(f32, f32);
let point = Point(1.0, 0.0);
assert_eq!(point.0, 1.0);
assert_eq!(point.1, 0.0);
```

> [!NOTE]
> 与 [字段访问表达式][field access expressions] 不同，元组索引表达式 可以是 [调用表达式][call expression] 的函数操作数，因为它不会与方法调用混淆，因为方法名不能是数字。

> [!NOTE]
> 尽管数组和切片也有元素，你必须使用 [数组或切片索引表达式][array or slice indexing expression] 或 [切片模式][slice pattern] 来访问它们的元素。

[array or slice indexing expression]: array-expr.md#array-and-slice-indexing-expressions
[call expression]: ./call-expr.md
[decimal literal]: ../tokens.md#整数字面量
[field access expressions]: ./field-expr.html#field-access-expressions
[operands]: ../expressions.md
[parenthetical expression]: grouped-expr.md
[place expression]: ../expressions.md#位置表达式和值表达式
[slice pattern]: ../patterns.md#slice-patterns
[tuple type]: ../types/tuple.md
[tuple struct]: ../types/struct.md
[value expression]: ../expressions.md#位置表达式和值表达式
