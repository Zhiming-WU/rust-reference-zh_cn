<!-- https://github.com/rust-lang/reference/blob/master/src/items/unions.md -->
<!-- commit 68bdfd15fa9feebfbf94a06cf574de598e239198 -->

r[items.union]
# 联合体

r[items.union.syntax]
```grammar,项
Union ->
    `union` IDENTIFIER GenericParams? WhereClause? `{` StructFields? `}`
```

r[items.union.intro]
 联合体 声明使用与 结构体 声明相同的 语法格式，除了用 `union` 代替 `struct`。

r[items.union.namespace]
 联合体 声明在其所在的模块或代码块的  [类型命名空间][type namespace]  中定义给定的名称。

```rust
#[repr(C)]
union MyUnion {
    f1: u32,
    f2: f32,
}
```

r[items.union.common-storage]
 联合体 的关键属性是 联合体 的所有字段共享公共存储。因此，对 联合体 的一个字段进行写入可能会覆盖其其他字段，并且 联合体 的大小由其最大字段的大小决定。

r[items.union.field-restrictions]
 联合体 字段类型仅限于以下类型子集：

r[items.union.field-copy]
- `Copy` 类型

r[items.union.field-references]
- 引用（针对任意 `T` 的 `&T` 和 `&mut T`）

r[items.union.field-manually-drop]
- `ManuallyDrop<T>`（针对任意 `T`）

r[items.union.field-tuple]
- 仅包含允许的 联合体 字段类型的元组和数组

r[items.union.drop]
这种限制特别确保了 联合体 字段永远不需要被 drop。与 结构体 和枚举一样，可以为 联合体 `impl Drop` 以手动定义其在被 drop 时发生的情况。

r[items.union.fieldless]
编译器不接受没有任何字段的 联合体，但宏可以接受。

r[items.union.init]
## 初始化联合体

r[items.union.init.intro]
 联合体 类型的值可以使用与 结构体 类型相同的 语法格式 创建，不同之处在于它必须恰好指定一个字段：

```rust
# union MyUnion { f1: u32, f2: f32 }
#
let u = MyUnion { f1: 1 };
```

r[items.union.init.result]
上面的表达式创建了一个 `MyUnion` 类型的值，并使用字段 `f1` 初始化存储。可以使用与 结构体 字段相同的 语法格式 访问 联合体：

```rust
# union MyUnion { f1: u32, f2: f32 }
#
# let u = MyUnion { f1: 1 };
let f = unsafe { u.f1 };
```

r[items.union.fields]
## 读写联合体字段

r[items.union.fields.intro]
 联合体 没有“活动字段”的概念。相反，每次对 联合体 的访问只是将存储解释为用于访问的字段的类型。

r[items.union.fields.read]
读取 联合体 字段会按该字段的类型读取 联合体 的位。

r[items.union.fields.offset]
字段可能具有非零偏移量（除非使用了  [C 表示][the C representation] ）；在这种情况下，将读取从字段偏移量开始的位。

r[items.union.fields.validity]
程序员有责任确保数据在该字段的类型下是有效的。否则会导致  [未定义行为][undefined behavior] 。例如，从  [布尔类型][boolean type]  字段中读取值 `3` 是 未定义行为。实际上，在使用  [C 表示][the C representation]  的 联合体 中，先写入然后再读取，类似于从用于写入的类型到用于读取的类型的  [`transmute`] 。

r[items.union.fields.read-safety]
因此，所有对 联合体 字段的读取都必须放在 `unsafe` 块中：

```rust
# union MyUnion { f1: u32, f2: f32 }
# let u = MyUnion { f1: 1 };
#
unsafe {
    let f = u.f1;
}
```

通常，使用 联合体 的代码会为不安全的 联合体 字段访问提供安全包装。

r[items.union.fields.write-safety]
相比之下，对 联合体 字段的写入是安全的，因为它们只是覆盖任意数据，不会导致 未定义行为。（注意，联合体 字段类型永远不会有 drop glue，因此 联合体 字段写入永远不会隐式地 drop 任何内容。）

r[items.union.pattern]
## 在联合体上进行模式匹配

r[items.union.pattern.intro]
另一种访问 联合体 字段的方法是使用模式匹配。

r[items.union.pattern.one-field]
在 联合体 字段上进行模式匹配使用与 结构体 模式相同的 语法格式，但模式必须恰好指定一个字段。

r[items.union.pattern.safety]
由于模式匹配就像使用特定字段读取 联合体，因此它也必须放在 `unsafe` 块中。

```rust
# union MyUnion { f1: u32, f2: f32 }
#
fn f(u: MyUnion) {
    unsafe {
        match u {
            MyUnion { f1: 10 } => { println!("ten"); }
            MyUnion { f2 } => { println!("{}", f2); }
        }
    }
}
```

r[items.union.pattern.subpattern]
模式匹配可以将 联合体 作为更大结构的一个字段进行匹配。特别是，当使用 Rust 联合体 通过 FFI 实现 C 标签联合体时，这允许同时对标签和相应的字段进行匹配：

```rust
#[repr(u32)]
enum Tag { I, F }

#[repr(C)]
union U {
    i: i32,
    f: f32,
}

#[repr(C)]
struct Value {
    tag: Tag,
    u: U,
}

fn is_zero(v: Value) -> bool {
    unsafe {
        match v {
            Value { tag: Tag::I, u: U { i: 0 } } => true,
            Value { tag: Tag::F, u: U { f: num } } if num == 0.0 => true,
            _ => false,
        }
    }
}
```

r[items.union.ref]
## 指向联合体字段的引用

r[items.union.ref.intro]
由于 联合体 字段共享公共存储，获得对 联合体 一个字段的写访问权可能会获得对其所有其余字段的写访问权。

r[items.union.ref.borrow]
借用检查规则必须进行调整以考虑到这一事实。因此，如果 联合体 的一个字段被借用，其所有其余字段也会在相同的生命周期内被借用。

```rust,compile_fail
# union MyUnion { f1: u32, f2: f32 }
// 错误：一次不能多次将 `u`（通过 `u.f2`）借用为可变
fn test() {
    let mut u = MyUnion { f1: 1 };
    unsafe {
        let b1 = &mut u.f1;
//                    ---- 第一次可变借用发生在这里（通过 `u.f1`）
        let b2 = &mut u.f2;
//                    ^^^^ 第二次可变借用发生在这里（通过 `u.f2`）
        *b1 = 5;
    }
//  - 第一次借用在这里结束
    assert_eq!(unsafe { u.f1 }, 5);
}
```

r[items.union.ref.usage]
如你所见，在许多方面（除了布局、安全性和所有权），联合体 的行为与 结构体 完全相同，这在很大程度上是由于继承了 结构体 的 语法格式 形状。对于 Rust 语言中许多未提及的方面也是如此（如私有性、名称解析、类型推导、 泛型 、 特型 实现、固有实现、一致性、模式检查等等）。

[`transmute`]: std::mem::transmute
[boolean type]: ../types/boolean.md
[the C representation]: ../type-layout.md#reprc联合体
[type namespace]: ../names/namespaces.md
[undefined behavior]: ../behavior-considered-undefined.md
