r[items.impl]
# 实现

r[items.impl.syntax]
```grammar,items
Implementation -> InherentImpl | TraitImpl

InherentImpl ->
    `impl` GenericParams? Type WhereClause? `{`
        InnerAttribute*
        AssociatedItem*
    `}`

TraitImpl ->
    `unsafe`? `impl` GenericParams? `!`? TypePath `for` Type
    WhereClause?
    `{`
        InnerAttribute*
        AssociatedItem*
    `}`
```

r[items.impl.intro]
一个  *实现 (implementation)*  是一个将 项 与一个  *实现类型 (implementing type)*  相关联的 项。实现 使用关键字 `impl` 定义，并包含属于正在实现的类型的实例或静态属于该类型的函数。

r[items.impl.kinds]
实现 有两种类型：

- 固有实现 (inherent implementations)
- [特型][trait] 实现

r[items.impl.inherent]
## 固有实现

r[items.impl.inherent.intro]
固有实现被定义为 `impl` 关键字、 泛型 类型声明、指向标称类型的路径、where 子句以及一组用大括号括起来的可关联 项 的序列。

r[items.impl.inherent.implementing-type]
该标称类型被称为  *实现类型 (implementing type)* ，而可关联 项 是该实现类型的  *关联项 (associated items)* 。

r[items.impl.inherent.associated-items]
固有实现将包含的 项 与实现类型相关联。

r[items.impl.inherent.associated-items.allowed-items]
固有实现可以包含 [关联函数][associated functions]（包括 [方法][methods]）和 [关联常量][associated constants]。

r[items.impl.inherent.type-alias]
它们不能包含关联类型别名。

r[items.impl.inherent.associated-item-path]
指向关联项的 [路径][path] 是指向实现类型的任何路径，后跟关联项的标识符作为路径的最后一个组件。

r[items.impl.inherent.coherence]
一个类型也可以有多个固有实现。实现类型必须定义在与原始类型定义相同的 crate 中。

``` rust
pub mod color {
    pub struct Color(pub u8, pub u8, pub u8);

    impl Color {
        pub const WHITE: Color = Color(255, 255, 255);
    }
}

mod values {
    use super::color::Color;
    impl Color {
        pub fn red() -> Color {
            Color(255, 0, 0)
        }
    }
}

pub use self::color::Color;
fn main() {
    // 实现类型和 impl 在同一个模块中的实际路径。
    color::Color::WHITE;

    // 不同模块中的 impl 块仍然通过指向该类型的路径进行访问。
    color::Color::red();

    // 实现类型的重导出路径也有效。
    Color::red();

    // 无效，因为 values 中的 use 不是 pub。
    // values::Color::red();
}
```

r[items.impl.trait]
## 特型实现

r[items.impl.trait.intro]
 *特型实现*  的定义类似于固有实现，不同之处在于可选的 泛型 类型声明后面跟着一个 [特型][trait]，然后是关键字 `for`，最后是到一个标称类型的路径。

r[items.impl.trait.implemented-trait]
该 特型 被称为  *实现的特型 (implemented trait)* 。实现类型实现该 实现的特型。

r[items.impl.trait.def-requirement]
特型实现必须定义由 实现的特型 声明的所有非默认 关联项，可以重新定义由 实现的特型 定义的默认 关联项，并且不能定义任何其他 项。

r[items.impl.trait.associated-item-path]
指向关联项的路径是 `<` 后跟指向实现类型的路径，后跟 `as`，后跟指向 特型 的路径，后跟 `>` 作为路径组件，再后跟关联项的路径组件。

r[items.impl.trait.safety]
[不安全特型][Unsafe traits] 要求特型实现以 `unsafe` 关键字开头。

```rust
# #[derive(Copy, Clone)]
# struct Point {x: f64, y: f64};
# type Surface = i32;
# struct BoundingBox {x: f64, y: f64, width: f64, height: f64};
# trait Shape { fn draw(&self, s: Surface); fn bounding_box(&self) -> BoundingBox; }
# fn do_draw_circle(s: Surface, c: Circle) { }
struct Circle {
    radius: f64,
    center: Point,
}

impl Copy for Circle {}

impl Clone for Circle {
    fn clone(&self) -> Circle { *self }
}

impl Shape for Circle {
    fn draw(&self, s: Surface) { do_draw_circle(s, *self); }
    fn bounding_box(&self) -> BoundingBox {
        let r = self.radius;
        BoundingBox {
            x: self.center.x - r,
            y: self.center.y - r,
            width: 2.0 * r,
            height: 2.0 * r,
        }
    }
}
```

r[items.impl.trait.coherence]
### 特型实现一致性

r[items.impl.trait.coherence.intro]
如果孤儿规则检查失败或存在重叠的实现实例，则 特型实现 被认为是不一致的。

r[items.impl.trait.coherence.overlapping]
当两个 特型实现 所针对的 特型 存在非空交集时，即这两个实现可以用相同的类型实例化，则它们发生重叠。

r[items.impl.trait.orphan-rule]
#### 孤儿规则

r[items.impl.trait.orphan-rule.intro]
 *孤儿规则 (orphan rule)*  规定，只有当 特型 或实现中的至少一个类型定义在当前 crate 中时，才允许进行 特型实现。它防止了不同 crate 之间冲突的 特型实现，并且是确保一致性的关键。

一个孤儿实现是指为外部类型实现外部 特型。如果允许自由定义这些实现，则两个 crate 可能会以不兼容的方式为相同的类型实现相同的 特型，从而导致添加或更新依赖项可能会因实现冲突而破坏编译。

孤儿规则 允许库作者为其 特型 添加新的实现，而无需担心会破坏下游代码。如果没有这些限制，库将无法添加像 `impl<T: Display> MyTrait for T` 这样的实现，因为这可能会与下游实现产生冲突。

r[items.impl.trait.orphan-rule.general]
对于 `impl<P1..=Pn> Trait<T1..=Tn> for T0`，只有当以下至少一项为真时，`impl` 才是有效的：

- `Trait` 是一个 [本地特型][local trait]
- 以下全部成立：
  - 类型 `T0..=Tn` 中必须至少有一个是 [本地类型][local type]。设 `Ti` 为第一个此类类型。
  - 在 `T0..Ti`（不包括 `Ti`）中不得出现 [未覆盖的类型][uncovered type] 参数 `P1..=Pn`

r[items.impl.trait.uncovered-param]
仅对  *未覆盖的 (uncovered)*  类型参数的出现进行限制。

r[items.impl.trait.fundamental]
请注意，为了保持一致性，[基础类型][fundamental types] 是特殊的。`Box<T>` 中的 `T` 不被视为已覆盖，且 `Box<LocalType>` 被视为本地的。

r[items.impl.generics]
## 泛型实现

r[items.impl.generics.intro]
实现可以接受 [泛型参数][generic parameters]，这些参数可以用于实现的其余部分。实现参数直接写在 `impl` 关键字后面。

```rust
# trait Seq<T> { fn dummy(&self, _: T) { } }
impl<T> Seq<T> for Vec<T> {
    /* ... */
}
impl Seq<bool> for u32 {
    /* 将整数视为位序列 */
}
```

r[items.impl.generics.usage]
如果泛型参数在以下位置至少出现一次，则该参数会  *约束 (constrain)*  实现：

* 实现的 特型（如果存在）
* 实现类型
* 作为包含另一个约束实现的参数的类型的 [绑定][bounds] 中的 [关联类型][associated type]

r[items.impl.generics.constrain]
类型参数和常量参数必须始终约束实现。如果生命周期在关联类型中使用，则生命周期必须约束实现。

产生约束的情况示例：

```rust
# trait Trait{}
# trait GenericTrait<T> {}
# trait HasAssocType { type Ty; }
# struct Struct;
# struct GenericStruct<T>(T);
# struct ConstGenericStruct<const N: usize>([(); N]);
// T 通过作为 GenericTrait 的参数来产生约束。
impl<T> GenericTrait<T> for i32 { /* ... */ }

// T 通过作为 GenericStruct 的参数来产生约束
impl<T> Trait for GenericStruct<T> { /* ... */ }

// 同样地，N 通过作为 ConstGenericStruct 的参数来产生约束
impl<const N: usize> Trait for ConstGenericStruct<N> { /* ... */ }

// T 通过作为类型 U 的绑定中的关联类型来产生约束，而 U 本身是约束该特型的泛型参数。
impl<T, U> GenericTrait<U> for u32 where U: HasAssocType<Ty = T> { /* ... */ }

// 与前一个类似，除了类型是 (U, isize)。U 出现在包含 T 的类型内部，而不是该类型本身。
impl<T, U> GenericStruct<U> where (U, isize): HasAssocType<Ty = T> { /* ... */ }
```

不产生约束的情况示例：

```rust,compile_fail
// 以下其余部分都是错误，因为它们具有不产生约束的类型或常量参数。

// T 不产生约束，因为它根本没有出现。
impl<T> Struct { /* ... */ }

// N 出因同样的原因不产生约束。
impl<const N: usize> Struct { /* ... */ }

// 在实现内部使用 T 并不约束 impl。
impl<T> Struct {
    fn uses_t(t: &T) { /* ... */ }
}

// T 在 U 的绑定中被用作关联类型，但 U 并不产生约束。
impl<T, U> Struct where U: HasAssocType<Ty = T> { /* ... */ }

// T 在绑定中使用，但不是作为关联类型使用，因此它不产生约束。
impl<T, U> GenericTrait<U> for u32 where U: GenericTrait<T> {}
```

允许的不产生约束的生命周期参数示例：

```rust
# struct Struct;
impl<'a> Struct {}
```

不允许的不产生约束的生命周期参数示例：

```rust,compile_fail
# struct Struct;
# trait HasAssocType { type Ty; }
impl<'a> HasAssocType for Struct {
    type Ty = &'a Struct;
}
```

r[items.impl.attributes]
## 实现上的属性

实现可以在 `impl` 关键字之前包含外部 [属性][attributes]，并在包含关联项的大括号内包含内部 [属性][attributes]。内部属性必须位于任何关联项之前。此处有意义的属性是 [`cfg`]、[`deprecated`]、[`doc`] 和 [lint 检查属性][the lint check attributes]。

[trait]: traits.md
[associated constants]: associated-items.md#associated-constants
[associated functions]: associated-items.md#associated-functions-and-methods
[associated type]: associated-items.md#associated-types
[attributes]: ../attributes.md
[bounds]: ../trait-bounds.md
[`cfg`]: ../conditional-compilation.md
[`deprecated`]: ../attributes/diagnostics.md#the-deprecated-attribute
[`doc`]: https://doc.rust-lang.org/rustdoc/the-doc-attribute.html
[generic parameters]: generics.md
[methods]: associated-items.md#方法
[path]: ../paths.md
[the lint check attributes]: ../attributes/diagnostics.md#lint-check-attributes
[Unsafe traits]: traits.md#unsafe-traits
[local trait]: ../glossary.md#local-trait
[local type]: ../glossary.md#local-type
[fundamental types]: ../glossary.md#fundamental-type-constructors
[uncovered type]: ../glossary.md#uncovered-type
