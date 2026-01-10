r[expr.array]
# 数组和数组索引表达式

## 数组表达式

r[expr.array.syntax]
```grammar,expressions
ArrayExpression -> `[` ArrayElements? `]`

ArrayElements ->
      Expression ( `,` Expression )* `,`?
    | Expression `;` Expression
```

r[expr.array.constructor]
 *数组表达式* 构造 [数组][array]。
数组表达式有两种形式。

r[expr.array.array]
第一种形式列出数组中的每一个值。

r[expr.array.array-syntax]
这种形式的 语法格式 是在方括号中包含一个由逗号分隔的具有统一类型的表达式列表。

r[expr.array.array-behavior]
这会产生一个数组，按编写顺序包含这些值。

r[expr.array.repeat]
第二种形式的 语法格式 是在方括号中包含两个由分号 (`;`) 分隔的表达式。

r[expr.array.repeat-operand]
`;` 之前的表达式称为 *重复操作数*。

r[expr.array.length-operand]
`;` 之后的表达式称为 *长度操作数*。

r[expr.array.length-restriction]
长度操作数必须是一个 [推断常量][inferred const] 或者是一个 `usize` 类型的 [常量表达式][constant expression]（例如 [字面量][literal] 或 [常量 项][constant item]）。

```rust
const C: usize = 1;
let _: [u8; C] = [0; 1]; // 字面量。
let _: [u8; C] = [0; C]; // 常量 项。
let _: [u8; C] = [0; _]; // 推断常量。
let _: [u8; C] = [0; (((_)))]; // 推断常量。
```

> [!NOTE]
> 在数组表达式中，一个 [推断常量][inferred const] 被解析为一个 [表达式][Expression]，但在语义上被视为一种单独的 [常量 泛型 参数][const generic argument]。

r[expr.array.repeat-behavior]
这种形式的数组表达式创建一个长度为长度操作数值的数组，其中每个元素都是重复操作数的副本。
也就是说，`[a; b]` 创建一个包含 `b` 个 `a` 的值的副本的数组。

r[expr.array.repeat-copy]
如果长度操作数的值大于 1，则要求重复操作数的类型实现 [`Copy`]、或者是一个 [常量块表达式][const block expression]、或者是一个指向常量 项 的 [路径][path]。

r[expr.array.repeat-const-item]
当重复操作数是一个常量块或指向常量 项 的路径时，它将被评估长度操作数指定的次数。

r[expr.array.repeat-evaluation-zero]
如果该值为 `0`，则常量块或常量 项 根本不会被评估。

r[expr.array.repeat-non-const]
对于既不是常量块也不是指向常量 项 的路径的表达式，它仅被评估一次，然后结果被复制长度操作数的值次。

```rust
[1, 2, 3, 4];
["a", "b", "c", "d"];
[0; 128];              // 包含 128 个零的数组
[0u8, 0u8, 0u8, 0u8,];
[[1, 0, 0], [0, 1, 0], [0, 0, 1]]; // 二维数组
const EMPTY: Vec<i32> = Vec::new();
[EMPTY; 2];
```

r[expr.array.index]
## 数组和切片索引表达式

r[expr.array.index.syntax]
```grammar,expressions
IndexExpression -> Expression `[` Expression `]`
```

r[expr.array.index.array]
[数组][array] 和 [切片][slice] 类型的值可以通过在它们后面编写一个由方括号括起来的 `usize` 类型表达式（索引）来进行索引。
当数组是可变的时，生成的 [内存位置][memory location] 可以被赋值。

r[expr.array.index.trait]
对于其他类型，索引表达式 `a[b]` 等同于 `*std::ops::Index::index(&a, b)`，或者在可变位置表达式上下文中等同于 `*std::ops::IndexMut::index_mut(&mut a, b)`。
与方法一样，Rust 也会在 `a` 上重复插入解引用操作以寻找实现。

r[expr.array.index.zero-index]
数组和切片的索引是从零开始的。

r[expr.array.index.const]
数组访问是一个 [常量表达式][constant expression]，因此可以使用常量索引值在编译时检查边界。
否则将在运行时执行检查，如果失败，将使线程处于 [ _恐慌_ 状态][panic]。

```rust,should_panic
// lint 默认是 deny。
#![warn(unconditional_panic)]

([1, 2, 3, 4])[2];        // 求值为 3

let b = [[1, 0, 0], [0, 1, 0], [0, 0, 1]];
b[1][2];                  // 多维数组索引

let x = (["a", "b"])[10]; // 警告：索引越界

let n = 10;
let y = (["a", "b"])[n];  // 恐慌

let arr = ["a", "b"];
arr[10];                  // 警告：索引越界
```

r[expr.array.index.trait-impl]
数组索引表达式可以通过实现 [Index] 和 [IndexMut] 特型 来为数组和切片以外的类型实现。

[`Copy`]: ../special-types-and-traits.md#copy
[IndexMut]: std::ops::IndexMut
[Index]: std::ops::Index
[array]: ../types/array.md
[const generic argument]: items.generics.const.argument
[const block expression]: expr.block.const
[constant expression]: ../const_eval.md#常量表达式
[constant item]: ../items/constant-items.md
[inferred const]: items.generics.const.inferred
[literal]: ../tokens.md#字面量
[memory location]: ../expressions.md#位置表达式和值表达式
[panic]: ../panic.md
[path]: path-expr.md
[slice]: ../types/slice.md
