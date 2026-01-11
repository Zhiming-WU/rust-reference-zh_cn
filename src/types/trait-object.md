<!-- https://github.com/rust-lang/reference/blob/master/src/types/trait-object.md -->
<!-- commit 68bdfd15fa9feebfbf94a06cf574de598e239198 -->

r[type.trait-object]
# 特型对象

r[type.trait-object.syntax]
```grammar,类型
TraitObjectType -> `dyn`? TypeParamBounds

TraitObjectTypeOneBound -> `dyn`? TraitBound
```

r[type.trait-object.intro]
一个 *特型对象* 是实现了的一组特型的另一种类型的不透明值。这组特型由一个 [dyn 兼容][dyn compatible] 的 *基础特型* 加上任意数量的 [自动特型][auto traits] 组成。

r[type.trait-object.impls]
特型对象实现了基础特型、其自动特型以及基础特型的任何 [超特型][supertraits] 。

r[type.trait-object.name]
特型对象写作关键字 `dyn` 后跟一组特型界限，但对特型界限有以下限制。

r[type.trait-object.constraint]
非自动特型不得超过一个，生命周期不得超过一个，且不允许退出界限（例如 `?Sized` ）。此外，特型路径可以用括号括起来。

例如，给定一个特型 `Trait` ，以下都是特型对象：

* `dyn Trait`
* `dyn Trait + Send`
* `dyn Trait + Send + Sync`
* `dyn Trait + 'static`
* `dyn Trait + Send + 'static`
* `dyn Trait +`
* `dyn 'static + Trait`.
* `dyn (Trait)`

r[type.trait-object.syntax-edition2021]
> [!EDITION-2021]
> 在 2021 版次 之前， `dyn` 关键字可以省略。

r[type.trait-object.syntax-edition2018]
> [!EDITION-2018]
> 在 2015 版次 中，如果特型对象的第一个界限是以 `::` 开头的路径，那么 `dyn` 将被视为路径的一部分。可以将第一个路径放在括号中来解决这个问题。因此，如果你想要一个具有 `::your_module::Trait` 特型的特型对象，你应该写成 `dyn (::your_module::Trait)` 。
>
> 从 2018 版次 开始， `dyn` 是一个真正的关键字，不允许出现在路径中，因此括号不是必需的。

r[type.trait-object.alias]
如果基础特型互为别名，且自动特型集合相同，生命周期界限也相同，则两个特型对象类型互为别名。例如， `dyn Trait + Send + UnwindSafe` 与 `dyn Trait + UnwindSafe + Send` 相同。

r[type.trait-object.unsized]
由于值属于哪种具体类型是不透明的，特型对象是 [动态大小类型][dynamically sized types] 。像所有 <abbr title="dynamically sized types">DST</abbr> 一样，特型对象在某种类型的指针后面使用；例如 `&dyn SomeTrait` 或 `Box<dyn SomeTrait>` 。特型对象指针的每个实例包括：

 - 一个指向实现了 `SomeTrait` 的类型 `T` 实例的指针
 - 一个 *虚方法表* ，通常简称为 *vtable* ，其中包含 `T` 针对 `SomeTrait` 及其 [超特型][supertraits] 的每个方法的实现指针（即函数指针）。

特型对象的目的是允许方法的 “后期绑定” 。在特型对象上调用方法会导致运行时的虚拟分派：即从特型对象 vtable 中加载函数指针并间接调用。每个 vtable 条目的实际实现可以因对象而异。

一个特型对象的例子：

```rust
trait Printable {
    fn stringify(&self) -> String;
}

impl Printable for i32 {
    fn stringify(&self) -> String { self.to_string() }
}

fn print(a: Box<dyn Printable>) {
    println!("{}", a.stringify());
}

fn main() {
    print(Box::new(10) as Box<dyn Printable>);
}
```

在这个例子中，特型 `Printable` 在 `print` 的类型签名和 `main` 中的类型转换表达式中都作为特型对象出现。

r[type.trait-object.lifetime-bounds]
## 特型对象生命周期界限

由于特型对象可以包含引用，这些引用的生命周期需要作为特型对象的一部分来表达。该生命周期写为 `Trait + 'a` 。有一些 [默认值][defaults] 允许通常以合理的选择推断此生命周期。

[auto traits]: ../special-types-and-traits.md#auto-traits
[defaults]: ../lifetime-elision.md#默认特型对象生命周期
[dyn compatible]: ../items/traits.md#dyn兼容性
[dynamically sized types]: ../dynamically-sized-types.md
[supertraits]: ../items/traits.md#父特型
