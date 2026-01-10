r[subtype]
# 子类型和变型

r[subtype.intro]
子类型是隐式的，可以发生在类型检查或推断的任何阶段。

r[subtype.kinds]
子类型仅限于两种情况：关于生命周期的变型，以及具有高阶生命周期的类型之间。如果我们从类型中擦除生命周期，那么唯一的子类型关系将仅源于类型相等。

考虑以下示例：字符串字面量始终具有 `'static` 生命周期。尽管如此，我们仍可以将 `s` 赋值给 `t`：

```rust
fn bar<'a>() {
    let s: &'static str = "hi";
    let t: &'a str = s;
}
```

由于 `'static` 的存活时间长于生命周期参数 `'a`，因此 `&'static str` 是 `&'a str` 的子类型。

r[subtype.higher-ranked]
[高阶][Higher-ranked]&#32;[函数指针][function pointers] 和 [特型对象][trait objects] 具有另一种子类型关系。它们是那些通过替换高阶生命周期而得到的类型的子类型。一些示例：

```rust
// 这里 'a 被替换为 'static
let subtype: &(for<'a> fn(&'a i32) -> &'a i32) = &((|x| x) as fn(&_) -> &_);
let supertype: &(fn(&'static i32) -> &'static i32) = subtype;

// 这对特型对象的作用类似
let subtype: &(dyn for<'a> Fn(&'a i32) -> &'a i32) = &|x| x;
let supertype: &(dyn Fn(&'static i32) -> &'static i32) = subtype;

// 我们也可以将一个高阶生命周期替换为另一个
let subtype: &(for<'a, 'b> fn(&'a i32, &'b i32)) = &((|x, y| {}) as fn(&_, &_));
let supertype: &for<'c> fn(&'c i32, &'c i32) = subtype;
```

r[subtyping.variance]
## 变型

r[subtyping.variance.intro]
变型是泛型类型相对于其参数所具有的属性。泛型类型在参数上的 *变型* 是指参数的子类型关系如何影响类型的子类型关系。

r[subtyping.variance.covariant]
* 如果 `T` 是 `U` 的子类型意味着 `F<T>` 是 `F<U>` 的子类型，则 `F<T>` 在 `T` 上是 *协变* 的（子类型关系“透传”）

r[subtyping.variance.contravariant]
* 如果 `T` 是 `U` 的子类型意味着 `F<U>` 是 `F<T>` 的子类型，则 `F<T>` 在 `T` 上是 *逆变* 的

r[subtyping.variance.invariant]
* 否则， `F<T>` 在 `T` 上是 *不变* 的（无法推导出子类型关系）

r[subtyping.variance.builtin-types]
类型的变型自动确定如下

| 类型                          | `'a` 的变型  | `T` 的变型   |
|-------------------------------|-------------------|-------------------|
| `&'a T`                       | 协变         | 协变         |
| `&'a mut T`                   | 协变         | 不变         |
| `*const T`                    |                   | 协变         |
| `*mut T`                      |                   | 不变         |
| `[T]` 和 `[T; n]`            |                   | 协变         |
| `fn() -> T`                   |                   | 协变         |
| `fn(T) -> ()`                 |                   | 逆变     |
| `std::cell::UnsafeCell<T>`    |                   | 不变         |
| `std::marker::PhantomData<T>` |                   | 协变         |
| `dyn 特型<T> + 'a`           | 协变         | 不变         |

r[subtyping.variance.user-composite-types]
其他 `struct` 、 `enum` 和 `union` 类型的变型是通过查看其字段类型的变型来确定的。如果参数被用于具有不同变型的位置，那么该参数就是不变的。例如，以下结构体在 `'a` 和 `T` 上是协变的，在 `'b` 、 `'c` 和 `U` 上是不变的。

```rust
use std::cell::UnsafeCell;
struct Variance<'a, 'b, 'c, T, U: 'a> {
    x: &'a U,               // 这使得 Variance 在 'a 上是协变的，并且本会
                            // 使其在 U 上也是协变的，但 U 在后面被用到了
    y: *const T,            // 在 T 上协变
    z: UnsafeCell<&'b f64>, // 在 'b 上不变
    w: *mut U,              // 在 U 上不变，使得整个结构体变为不变

    f: fn(&'c ()) -> &'c () // 既是协变也是逆变，使得 'c 在结构体中是不变的。
}
```

r[subtyping.variance.builtin-composite-types]
当在 `struct` 、 `enum` 或 `union` 之外使用时，参数的变型是在每个位置分别检查的。

```rust
# use std::cell::UnsafeCell;
fn generic_tuple<'short, 'long: 'short>(
    // 'long 在元组内部同时用于协变和不变的位置。
    x: (&'long u32, UnsafeCell<&'long u32>),
) {
    // 由于这些位置的变型是分别计算的，
    // 我们可以自由地缩小协变位置上的 'long。
    let _: (&'short u32, UnsafeCell<&'long u32>) = x;
}

fn takes_fn_ptr<'short, 'middle: 'short>(
    // 'middle 同时用于协变和逆变的位置。
    f: fn(&'middle ()) -> &'middle (),
) {
    // 由于这些位置的变型是分别计算的，
    // 我们可以自由地缩小协变位置上的 'middle
    // 并扩大逆变位置上的 'middle。
    let _: fn(&'static ()) -> &'short () = f;
}
```

[function pointers]: types/function-pointer.md
[Higher-ranked]: ../nomicon/hrtb.html
[trait objects]: types/trait-object.md
