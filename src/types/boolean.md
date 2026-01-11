<!-- https://github.com/rust-lang/reference/blob/master/src/types/boolean.md -->
<!-- commit 68bdfd15fa9feebfbf94a06cf574de598e239198 -->

r[type.bool]
# 布尔类型

```rust
let b: bool = true;
```

r[type.bool.intro]
 * 布尔类型 * 或 * bool * 是一种原始数据类型，可以取两个值之一，称为 * true * 和 * false * 。

r[type.bool.literal]
该类型的值可以使用 [字面量表达式][literal expression] 创建，使用对应于同名值的关键字 `true` 和 `false` 。

r[type.bool.namespace]
该类型是 [语言预导入][language prelude] 的一部分， [名称][name] 为 `bool` 。

r[type.bool.layout]
布尔类型的对象其 [大小和对齐][size and alignment] 均为 1 。

r[type.bool.repr]
值 false 的位模式为 `0x00` ，值 true 的位模式为 `0x01` 。布尔类型的对象具有任何其他位模式都是 [未定义行为][undefined behavior] 。

r[type.bool.usage]
布尔类型是各种 [表达式][expressions] 中许多操作数的类型：

r[type.bool.usage-condition]
* [if 表达式][if expressions] 和 [while 表达式][while expressions] 中的条件操作数

r[type.bool.usage-lazy-operator]
* [惰性布尔运算符表达式][lazy] 中的操作数

> [!NOTE]
> 布尔类型的作用类似于但不是 [枚举类型][enumerated type] 。在实践中，这主要意味着构造函数不与类型相关联 (例如 `bool::true`) 。

r[type.bool.traits]
与所有原始类型一样，布尔类型 [实现][p-impl] 了 [特型][p-traits] [`Clone`][p-clone]、 [`Copy`][p-copy]、 [`Sized`][p-sized]、 [`Send`][p-send] 和 [`Sync`][p-sync] 。

> [!NOTE]
> 有关库操作，请参阅 [标准库文档](bool) 。

r[type.bool.expr]
## 布尔值的运算

当对布尔类型的操作数使用某些运算符表达式时，它们按照 [布尔逻辑][boolean logic] 的规则进行求值。

r[type.bool.expr.not]
### 逻辑非

| `b` | [`!b`][op-not] |
|- | - |
| `true` | `false` |
| `false` | `true` |

r[type.bool.expr.or]
### 逻辑或

| `a` | `b` | [`a \| b`][op-or] |
|- | - | - |
| `true` | `true` | `true` |
| `true` | `false` | `true` |
| `false` | `true` | `true` |
| `false` | `false` | `false` |

r[type.bool.expr.and]
### 逻辑与

| `a` | `b` | [`a & b`][op-and] |
|- | - | - |
| `true` | `true` | `true` |
| `true` | `false" | `false` |
| `false` | `true` | `false` |
| `false` | `false` | `false` |

r[type.bool.expr.xor]
### 逻辑异或

| `a` | `b` | [`a ^ b`][op-xor] |
|- | - | - |
| `true` | `true` | `false` |
| `true` | `false` | `true` |
| `false` | `true` | `true` |
| `false` | `false` | `false` |

r[type.bool.expr.cmp]
### 比较

r[type.bool.expr.cmp.eq]
| `a` | `b` | [`a == b`][op-compare] |
|- | - | - |
| `true` | `true` | `true` |
| `true` | `false` | `false` |
| `false` | `true` | `false` |
| `false` | `false` | `true` |

r[type.bool.expr.cmp.greater]
| `a` | `b` | [`a > b`][op-compare] |
|- | - | - |
| `true` | `true` | `false` |
| `true` | `false` | `true` |
| `false` | `true` | `false` |
| `false` | `false` | `false` |

r[type.bool.expr.cmp.not-eq]
* `a != b` 与 `!(a == b)` 相同

r[type.bool.expr.cmp.greater-eq]
* `a >= b` 与 `a == b | a > b` 相同

r[type.bool.expr.cmp.less]
* `a < b` 与 `!(a >= b)` 相同

r[type.bool.expr.cmp.less-eq]
* `a <= b` 与 `a == b | a < b` 相同

r[type.bool.validity]
## 位有效性

`bool` 的单个字节保证被初始化 (换句话说， `transmute::<bool, u8>(...)` 总是健全的 -- 但由于某些位模式是无效的 `bool` ，其反向操作并不总是健全的) 。

[boolean logic]: https://en.wikipedia.org/wiki/Boolean_algebra
[enumerated type]: enum.md
[expressions]: ../expressions.md
[if expressions]: ../expressions/if-expr.md#if表达式
[language prelude]: ../names/preludes.md#语言预导入
[lazy]: ../expressions/operator-expr.md#惰性布尔运算符
[literal expression]: ../expressions/literal-expr.md
[name]: ../names.md
[op-and]: ../expressions/operator-expr.md#算术和逻辑二元运算符
[op-compare]: ../expressions/operator-expr.md#比较运算符
[op-not]: ../expressions/operator-expr.md#求负运算符
[op-or]: ../expressions/operator-expr.md#算术和逻辑二元运算符
[op-xor]: ../expressions/operator-expr.md#算术和逻辑二元运算符
[p-clone]: ../special-types-and-traits.md#clone
[p-copy]: ../special-types-and-traits.md#copy
[p-impl]: ../items/implementations.md
[p-send]: ../special-types-and-traits.md#send
[p-sized]: ../special-types-and-traits.md#sized
[p-sync]: ../special-types-and-traits.md#sync
[p-traits]: ../items/traits.md
[size and alignment]: ../type-layout.md#大小和对齐
[undefined behavior]: ../behavior-considered-undefined.md
[while expressions]: ../expressions/loop-expr.md#谓词循环
