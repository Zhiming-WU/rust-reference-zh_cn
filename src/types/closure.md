<!-- https://github.com/rust-lang/reference/blob/master/src/types/closure.md -->
<!-- commit 68bdfd15fa9feebfbf94a06cf574de598e239198 -->

r[type.closure]
# 闭包类型

r[type.closure.intro]
 [闭包表达式][closure expression] 产生一个具有唯一的、匿名的且无法写出的类型的闭包值。
一个闭包类型大约等同于一个包含捕获值的结构体。
例如，以下闭包：

```rust
#[derive(Debug)]
struct Point { x: i32, y: i32 }
struct Rectangle { left_top: Point, right_bottom: Point }

fn f<F : FnOnce() -> String> (g: F) {
    println!("{}", g());
}

let mut rect = Rectangle {
    left_top: Point { x: 1, y: 1 },
    right_bottom: Point { x: 0, y: 0 }
};

let c = || {
    rect.left_top.x += 1;
    rect.right_bottom.x += 1;
    format!("{:?}", rect.left_top)
};
f(c); // 打印 "Point { x: 2, y: 1 }"。
```

生成一个大致如下的闭包类型：

<!-- ignore: simplified -->
```rust,ignore
// 注意：这并非准确的转换方式，仅用于说明。

struct Closure<'a> {
    left_top : &'a mut Point,
    right_bottom_x : &'a mut i32,
}

impl<'a> FnOnce<()> for Closure<'a> {
    type Output = String;
    extern "rust-call" fn call_once(self, args: ()) -> String {
        self.left_top.x += 1;
        *self.right_bottom_x += 1;
        format!("{:?}", self.left_top)
    }
}
```

使得对 `f` 的调用就像这样：

<!-- ignore: continuation of above -->
```rust,ignore
f(Closure{ left_top: &mut rect.left_top, right_bottom_x: &mut rect.right_bottom.x });
```

r[type.closure.capture]
## 捕获模式

r[type.closure.capture.intro]
一个 *捕获模式* 决定了环境中的 [位置表达式][place expression] 是如何被借用或移动到闭包中的。
捕获模式有：

1. 不可变借用 (`ImmBorrow`) --- 位置表达式作为 [共享引用][shared reference] 被捕获。
2. 唯一不可变借用 (`UniqueImmBorrow`) --- 类似于不可变借用，但必须是唯一的，如 [下文](#unique-immutable-borrows-in-captures) 所述。
3. 可变借用 (`MutBorrow`) --- 位置表达式作为 [可变引用][mutable reference] 被捕获。
4. 移动 (`ByValue`) --- 位置表达式通过 [移动值][moving the value] 到闭包中而被捕获。

r[type.closure.capture.precedence]
来自环境的位置表达式按与捕获值在闭包体内的使用方式相兼容的第一种模式进行捕获。
该模式不受闭包周围代码的影响，例如相关变量或字段的生命周期，或者闭包本身的生命周期。

[moving the value]: ../expressions.md#移动和复制类型
[mutable reference]: pointer.md#可变引用mut
[place expression]: ../expressions.md#位置表达式和值表达式
[shared reference]: pointer.md#引用和mut

r[type.closure.capture.copy]
### `Copy`值

实现了 [`Copy`] 的值如果被移动到闭包中，将以 `ImmBorrow` 模式捕获。

```rust
let x = [0; 1024];
let c = || {
    let y = x; // x 通过 ImmBorrow 捕获
};
```

r[type.closure.async.input]
### 异步输入捕获

异步闭包总是捕获所有输入参数，无论它们是否在闭包体中使用。

## 捕获精度

r[type.closure.capture.precision.capture-path]
一个 *捕获路径* 是一个序列，起始于环境中的变量，后跟该变量的零个或多个位置投影。

r[type.closure.capture.precision.place-projection]
一个 *位置投影* 是对变量应用的 [字段访问][field access]、[元组索引][tuple index]、[解引用][dereference]（以及自动解引用）、[数组或切片索引][array or slice index] 表达式或 [模式解构][pattern destructuring]。

> [!NOTE]
> 在 `rustc` 中，模式解构会被脱糖为一系列解引用以及字段或元素访问。

r[type.closure.capture.precision.intro]
闭包借用或移动捕获路径，该路径可能会根据下述规则被截断。

例如：

```rust
struct SomeStruct {
    f1: (i32, i32),
}
let s = SomeStruct { f1: (1, 2) };

let c = || {
    let x = s.f1.1; // s.f1.1 通过 ImmBorrow 捕获
};
c();
```

这里的捕获路径是局部变量 `s` ，后跟一个字段访问 `.f1` ，然后是一个元组索引 `.1` 。
这个闭包捕获了 `s.f1.1` 的不可变借用。

[field access]: ../expressions/field-expr.md
[pattern destructuring]: patterns.destructure
[tuple index]: ../expressions/tuple-expr.md#元组索引表达式
[dereference]: ../expressions/operator-expr.md#解引用运算符
[array or slice index]: ../expressions/array-expr.md#数组和切片索引表达式

r[type.closure.capture.precision.shared-prefix]
### 共享前缀

在捕获路径及其祖先路径都被闭包捕获的情况下，祖先路径以两者中最高的捕获模式捕获， `CaptureMode = max(AncestorCaptureMode, DescendantCaptureMode)` ，使用严格弱序：

`ImmBorrow < UniqueImmBorrow < MutBorrow < ByValue`

注意这可能需要递归应用。

```rust
// 在这个例子中，有三个具有共享祖先的不同捕获路径：
# fn move_value<T>(_: T){}
let s = String::from("S");
let t = (s, String::from("T"));
let mut u = (t, String::from("U"));

let c = || {
    println!("{:?}", u); // u 通过 ImmBorrow 捕获
    u.1.truncate(0); // u.0 通过 MutBorrow 捕获
    move_value(u.0.0); // u.0.0 通过 ByValue 捕获
};
c();
```

总的来说，这个闭包将通过 `ByValue` 捕获 `u` 。

r[type.closure.capture.precision.dereference-shared]
### 最右侧共享引用解引用截断

如果解引用应用于共享引用，捕获路径将在捕获路径中最右侧的解引用处截断。

允许这种截断是因为通过共享引用读取的字段将始终通过共享引用或副本读取。
当额外的精度在借用检查的角度下没有任何益处时，这有助于减小捕获的大小。

之所以是 *最右侧* 解引用，是为了帮助避免产生比必要生命周期更短的生命周期。
考虑以下示例：

```rust
struct Int(i32);
struct B<'a>(&'a i32);

struct MyStruct<'a> {
   a: &'static Int,
   b: B<'a>,
}

fn foo<'a, 'b>(m: &'a MyStruct<'b>) -> impl FnMut() + 'static {
    let c = || drop(&m.a.0);
    c
}
```

如果要捕获 `m` ，那么闭包将不再能比 `'static` 存活得更久，因为 `m` 被限制在 `'a` 。相反，它通过 `ImmBorrow` 捕获 `(*(*m).a)` 。

r[type.closure.capture.precision.wildcard]
### 通配符模式绑定

r[type.closure.capture.precision.wildcard.reads]
闭包仅捕获需要读取的数据。使用 [通配符模式][wildcard pattern] 绑定一个值不会读取该值，因此该位置不会被捕获。

```rust,no_run
struct S; // 一个非 `Copy` 类型。
let x = S;
let c = || {
    let _ = x;  // 不捕获 `x`。
};
let c = || match x {
    _ => (), // 不捕获 `x`。
};
x; // OK： `x` 可以在这里被移动。
c();
```

r[type.closure.capture.precision.wildcard.destructuring]
解构元组、结构体和单 变体 枚举本身不会导致读取或位置被捕获。

> [!NOTE]
> 来自其他 crate 的带有 [`#[non_exhaustive]`][attributes.type-system.non_exhaustive] 标记的枚举总是被视为具有多个 变体 。参见 *[type.closure.capture.precision.discriminants.non_exhaustive]*。

```rust,no_run
struct S; // 一个非 `Copy` 类型。

// 解构元组不会导致读取或捕获。
let x = (S,);
let c = || {
    let (..) = x; // 不捕获 `x`。
};
x; // OK： `x` 可以在这里被移动。
c();

// 解构类单元结构体不会导致读取或捕获。
let x = S;
let c = || {
    let S = x; // 不捕获 `x`。
};
x; // OK： `x` 可以在这里被移动。
c();

// 解构结构体不会导致读取或捕获。
struct W<T>(T);
let x = W(S);
let c = || {
    let W(..) = x; // 不捕获 `x`。
};
x; // OK： `x` 可以在这里被移动。
c();

// 解构单变体枚举不会导致读取或捕获。
enum E<T> { V(T) }
let x = E::V(S);
let c = || {
    let E::V(..) = x; // 不捕获 `x`。
};
x; // OK： `x` 可以在这里被移动。
c();
```

r[type.closure.capture.precision.wildcard.fields]
与 [RestPattern] (`..`) 或 [StructPatternEtCetera] (也是 `..`) 匹配的字段不会被读取，且这些字段不会被捕获。

```rust,no_run
struct S; // 一个非 `Copy` 类型。
let x = (S, S);
let c = || {
    let (x0, ..) = x;  // 通过 `ByValue` 捕获 `x.0`。
};
// 只有第一个元组字段被闭包捕获。
x.1; // OK： `x.1` 可以在这里被移动。
c();
```

r[type.closure.capture.precision.wildcard.array-slice]
不支持对数组和切片的部分捕获；即使使用通配符模式匹配、索引或子切片，整个切片或数组也总是会被捕获。

```rust,compile_fail,E0382
struct S; // 一个非 `Copy` 类型。
let mut x = [S, S];
let c = || {
    let [x0, _] = x; // 通过 `ByValue` 捕获全部 `x`。
};
let _ = &mut x[1]; // 错误：借用了已移动的值。
```

r[type.closure.capture.precision.wildcard.initialized]
与通配符匹配的值仍必须已初始化。

```rust,compile_fail,E0381
let x: u8;
let c = || {
    let _ = x; // 错误：绑定 `x` 未初始化。
};
```

[wildcard pattern]: ../patterns.md#通配符模式

r[type.closure.capture.precision.discriminants]
### 为读取判别式而捕获

r[type.closure.capture.precision.discriminants.reads]
如果模式匹配读取了判别式，则包含该判别式的位置将通过 `ImmBorrow` 被捕获。

r[type.closure.capture.precision.discriminants.multiple-variant]
与具有多于一个 变体 的枚举的 变体 进行匹配会读取判别式，从而通过 `ImmBorrow` 捕获该位置。

```rust,compile_fail,E0502
struct S; // 一个非 `Copy` 类型。
let mut x = (Some(S), S);
let c = || match x {
    (None, _) => (),
//   ^^^^
// 此模式需要读取判别式，这导致 `x.0` 被 `ImmBorrow` 捕获。
    _ => (),
};
let _ = &mut x.0; // 错误：不能将 `x.0` 借用为可变。
//           ^^^
// 闭包仍然存活，所以 `x.0` 在这里仍然被不可变借用。
c();
```

```rust,no_run
# struct S; // 一个非 `Copy` 类型。
# let x = (Some(S), S);
let c = || match x { // 通过 `ImmBorrow` 捕获 `x.0`。
    (None, _) => (),
    _ => (),
};
// 尽管 `x.0` 因判别式读取而被捕获，但 `x.1` 未被捕获。
x.1; // OK： `x.1` 可以在这里被移动。
c();
```

r[type.closure.capture.precision.discriminants.single-variant]
与单 变体 枚举的唯一 变体 进行匹配不会读取判别式，也不会捕获该位置。

```rust,no_run
enum E<T> { V(T) } // 一个单变体枚举。
let x = E::V(());
let c = || {
    let E::V(_) = x; // 不捕获 `x`。
};
x; // OK： `x` 可以在这里被移动。
c();
```

r[type.closure.capture.precision.discriminants.non_exhaustive]
如果 [`#[non_exhaustive]`][attributes.type-system.non_exhaustive] 应用于在外部 crate 中定义的枚举，则出于决定是否发生读取的目的，该枚举被视为具有多个 变体 ，即使它实际上只有一个 变体 。

r[type.closure.capture.precision.discriminants.uninhabited-variants]
即使除被匹配的 变体 外的所有 变体 都是未入驻的，使得模式 [不可驳回][patterns.refutable]，如果本来会读取判别式，则判别式仍会被读取。

```rust,compile_fail,E0502
enum Empty {}
let mut x = Ok::<_, Empty>(42);
let c = || {
    let Ok(_) = x; // 通过 `ImmBorrow` 捕获 `x`。
};
let _ = &mut x; // 错误：不能将 `x` 借用为可变。
c();
```


r[type.closure.capture.precision.range-patterns]
### 捕获与范围模式

r[type.closure.capture.precision.range-patterns.reads]
与 [范围模式][patterns.range] 进行匹配会读取被匹配的位置，即使范围包含了该类型的所有可能值，并会通过 `ImmBorrow` 捕获该位置。

```rust,compile_fail,E0502
let mut x = 0u8;
let c = || {
    let 0..=u8::MAX = x; // 通过 `ImmBorrow` 捕获 `x`。
};
let _ = &mut x; // 错误：不能将 `x` 借用为可变。
c();
```

r[type.closure.capture.precision.slice-patterns]
### 捕获与切片模式

r[type.closure.capture.precision.slice-patterns.slices]
将切片与除仅包含单个 [剩余模式][patterns.rest]（即 `[..]` ）以外的 [切片模式][patterns.slice] 进行匹配，将被视为对切片长度的读取，并通过 `ImmBorrow` 捕获切片。

```rust,compile_fail,E0502
let x: &mut [u8] = &mut [];
let c = || match x { // 通过 `ImmBorrow` 捕获 `*x`。
    &mut [] => (),
//       ^^
// 这匹配一个长度恰好为零的切片。为了知道被检查对象是否匹配，
// 必须读取长度，从而导致切片被捕获。
    _ => (),
};
let _ = &mut *x; // 错误：不能将 `*x` 借用为可变。
c();
```

```rust,no_run
let x: &mut [u8] = &mut [];
let c = || match x { // 不捕获 `*x`。
    [..] => (),
//   ^^ 剩余模式。
};
let _ = &mut *x; // OK： `*x` 可以在这里被借用。
c();
```

> [!NOTE]
> 也许令人惊讶的是，尽管长度包含在切片的（宽） *指针* 中，但被视为读取并被捕获的是 *被指物* （切片）的位置。
>
> ```rust,no_run
> fn f<'l: 's, 's>(x: &'s mut &'l [u8]) -> impl Fn() + 'l {
>     // 闭包存活时间超过 `'l`，因为它捕获了 `**x`。如果
>     // 它捕获的是 `*x`，它将存活得不够长，
>     // 无法满足 `impl Fn() + 'l` 界限。
>     || match *x { // 通过 `ImmBorrow` 捕获 `**x`。
>         &[] => (),
>         _ => (),
>     }
> }
> ```
>
> 这样，该行为与在被检查对象中解引用到切片是一致的。
>
> ```rust,no_run
> fn f<'l: 's, 's>(x: &'s mut &'l [u8]) -> impl Fn() + 'l {
>     || match **x { // 通过 `ImmBorrow` 捕获 `**x`。
>         [] => (),
>         _ => (),
>     }
> }
> ```
>
> 有关详细信息，请参见 [Rust PR #138961](https://github.com/rust-lang/rust/pull/138961)。

r[type.closure.capture.precision.slice-patterns.arrays]
由于数组的长度由其类型固定，因此将数组与切片模式匹配本身不会捕获该位置。

```rust,no_run
let x: [u8; 1] = [0];
let c = || match x { // 不捕获 `x`。
    [_] => (), // 长度是固定的。
};
x; // OK： `x` 可以在这里被移动。
c();
```

r[type.closure.capture.precision.move-dereference]
### 在移动语境中捕获引用

由于不允许从引用中移出字段， `move` 闭包将仅捕获捕获路径的前缀，该前缀一直延伸到但不包括引用的第一次解引用。
引用本身将被移动到闭包中。

```rust
struct T(String, String);

let mut t = T(String::from("foo"), String::from("bar"));
let t_mut_ref = &mut t;
let mut c = move || {
    t_mut_ref.0.push_str("123"); // 通过 ByValue 捕获 `t_mut_ref`
};
c();
```

r[type.closure.capture.precision.raw-pointer-dereference]
### 裸指针解引用

由于解引用裸指针是 `unsafe` 的，闭包将仅捕获捕获路径的前缀，该前缀一直延伸到但不包括裸指针的第一次解引用。

```rust
struct T(String, String);

let t = T(String::from("foo"), String::from("bar"));
let t_ptr = &t as *const T;

let c = || unsafe {
    println!("{}", (*t_ptr).0); // 通过 ImmBorrow 捕获 `t_ptr`
};
c();
```

r[type.closure.capture.precision.union]
### 联合体 字段

由于访问 联合体 字段是 `unsafe` 的，闭包将仅捕获捕获路径的前缀，该前缀一直延伸到 联合体 本身。

```rust
union U {
    a: (i32, i32),
    b: bool,
}
let u = U { a: (123, 456) };

let c = || {
    let x = unsafe { u.a.0 }; // 通过 ByValue 捕获 `u`
};
c();

// 这也包括写入字段。
let mut u = U { a: (123, 456) };

let mut c = || {
    u.b = true; // 通过 MutBorrow 捕获 `u`
};
c();
```

r[type.closure.capture.precision.unaligned]
### 对未对齐`struct`的引用

由于创建对结构体中未对齐字段的引用是 [未定义行为][undefined behavior] ，
闭包将仅捕获捕获路径的前缀，该前缀一直延伸到但不包括对使用 [ `packed` 表示][the `packed` representation] 的结构体的第一次字段访问。
这包括所有字段，甚至是那些已对齐的字段，以防止将来结构体中的任何字段发生更改时产生兼容性问题。

```rust
#[repr(packed)]
struct T(i32, i32);

let t = T(2, 5);
let c = || {
    let a = t.0; // 通过 ImmBorrow 捕获 `t`
};
// 从 `t` 中复制是可以的。
let (a, b) = (t.0, t.1);
c();
```

类似地，获取未对齐字段的地址也会捕获整个结构体：

```rust,compile_fail,E0505
#[repr(packed)]
struct T(String, String);

let mut t = T(String::new(), String::new());
let c = || {
    let a = std::ptr::addr_of!(t.1); // 通过 ImmBorrow 捕获 `t`
};
let a = t.0; // 错误：无法移出 `t.0` ，因为它已被借用
c();
```

但如果它不是 packed 的，上述代码就可以工作，因为它能精确地捕获字段：

```rust
struct T(String, String);

let mut t = T(String::new(), String::new());
let c = || {
    let a = std::ptr::addr_of!(t.1); // 通过 ImmBorrow 捕获 `t.1`
};
// 这里的移动是允许的。
let a = t.0;
c();
```

[undefined behavior]: ../behavior-considered-undefined.md
[the `packed` representation]: ../type-layout.md#对齐修饰符

r[type.closure.capture.precision.box-deref]
### `Box`与其他`Deref`实现

[`Box`] 的 [`Deref`] 特型 实现与其他 `Deref` 实现受到的待遇不同，因为它被视为一个特殊的实体。

例如，让我们看看涉及 `Rc` 和 `Box` 的例子。 `*rc` 被脱糖为对 `Rc` 上定义的特型方法 `deref` 的调用，但由于 `*box` 受到不同待遇，因此可以对 `Box` 的内容进行精确捕获。

[`Box`]: ../special-types-and-traits.md#boxt
[`Deref`]: ../special-types-and-traits.md#deref-and-derefmut

r[type.closure.capture.precision.box-non-move.not-moved]
#### 具有非`move`闭包的`Box`

在非 `move` 闭包中，如果 `Box` 的内容没有被移动到闭包体中，则 `Box` 的内容会被精确捕获。

```rust
struct S(String);

let b = Box::new(S(String::new()));
let c_box = || {
    let x = &(*b).0; // 通过 ImmBorrow 捕获 `(*b).0`
};
c_box();

// 将 `Box` 与另一个实现了 Deref 的类型进行对比：
let r = std::rc::Rc::new(S(String::new()));
let c_rc = || {
    let x = &(*r).0; // 通过 ImmBorrow 捕获 `r`
};
c_rc();
```

r[type.closure.capture.precision.box-non-move.moved]
然而，如果 `Box` 的内容被移动到闭包中，那么该 box 会被整体捕获。这样做是为了尽量减少需要移动到闭包中的数据量。

```rust
// 这与上面的例子相同，除了闭包
// 移动值而不是获取其引用。

struct S(String);

let b = Box::new(S(String::new()));
let c_box = || {
    let x = (*b).0; // 通过 ByValue 捕获 `b`
};
c_box();
```

r[type.closure.capture.precision.box-move.read]
#### 具有move闭包的`Box`

类似于在非 `move` 闭包中移动 `Box` 的内容，在 `move` 闭包中读取 `Box` 的内容将整体捕获 `Box` 。

```rust
struct S(i32);

let b = Box::new(S(10));
let c_box = move || {
    let x = (*b).0; // 通过 ByValue 捕获 `b`
};
```

r[type.closure.unique-immutable]
## 捕获中的唯一不可变借用

捕获可以通过一种称为 _唯一不可变借用_ 的特殊借用发生，这种借用在语言的其他地方无法使用，也无法显式写出。它发生在修改可变引用的引用物时，如下例所示：

```rust
let mut b = false;
let x = &mut b;
let mut c = || {
    // x 的一个 ImmBorrow 和一个 MutBorrow。
    let a = &x;
    *x = true; // `x` 通过 UniqueImmBorrow 捕获
};
// 下面这行是一个错误：
// let y = &x;
c();
// 然而，下面这样是可以的。
let z = &x;
```

在这种情况下，由于 `x` 不是 `mut` ，所以无法以可变方式借用 `x` 。但与此同时，不可变地借用 `x` 会使赋值操作非法，因为 `& &mut` 引用可能不是唯一的，因此无法安全地用于修改值。所以使用了唯一不可变借用：它不可变地借用 `x` ，但像可变借用一样，它必须是唯一的。

在上面的示例中，取消对 `y` 的声明的注释将产生错误，因为它会违反闭包对 `x` 借用的唯一性； z 的声明是有效的，因为闭包的生命周期在块结束时已过期，释放了借用。

r[type.closure.call]
## 调用特型与隐式类型转换

r[type.closure.call.intro]
闭包类型都实现了 [`FnOnce`] ，表示它们可以通过消耗闭包的所有权来调用一次。此外，一些闭包还实现了更具体的调用 特型 ：

r[type.closure.call.fn-mut]
* 不从任何捕获变量中移出的闭包实现了 [`FnMut`] ，表示它可以通过可变引用调用。

r[type.closure.call.fn]
* 不修改或从任何捕获变量中移出的闭包实现了 [`Fn`] ，表示它可以通过共享引用调用。

> [!NOTE]
> `move` 闭包仍可能实现 [`Fn`] 或 [`FnMut`] ，即使它们通过移动捕获变量。这是因为由闭包类型实现的 特型 是由闭包对捕获值的操作决定的，而不是由它如何捕获它们决定的。

r[type.closure.non-capturing]
*非捕获闭包* 是不从其环境中捕获任何内容的闭包。非异步、非捕获的闭包可以被 隐式类型转换 为具有匹配签名的函数指针（例如， `fn()` ）。

```rust
let add = |x, y| x + y;

let mut x = add(5,7);

type Binop = fn(i32, i32) -> i32;
let bo: Binop = add;
x = bo(5,7);
```

r[type.closure.async.traits]
### 异步闭包特型

r[type.closure.async.traits.fn-family]
异步闭包在是否实现 [`FnMut`] 或 [`Fn`] 方面有进一步的限制。

异步闭包返回的 [`Future`] 具有与闭包类似的捕获特性。它根据位置表达式的使用方式从异步闭包中捕获它们。如果异步闭包具有以下任一属性，则称其向其 [`Future`] *借出 (lending)* ：

-  `Future` 包含一个可变捕获。
- 异步闭包通过值捕获，除非该值是通过解引用投影访问的。

如果异步闭包向其 `Future` 借出，则 *不* 实现 [`FnMut`] 和 [`Fn`] 。 [`FnOnce`] 始终会被实现。

> **示例**：可变捕获的第一种情况可以通过以下方式说明：
>
> ```rust,compile_fail
> fn takes_callback<Fut: Future>(c: impl FnMut() -> Fut) {}
>
> fn f() {
>     let mut x = 1i32;
>     let c = async || {
>         x = 2;  // x 通过 MutBorrow 捕获
>     };
>     takes_callback(c);  // 错误：异步闭包未实现 `FnMut`
> }
> ```
>
> 普通值捕获的第二种情况可以通过以下方式说明：
>
> ```rust,compile_fail
> fn takes_callback<Fut: Future>(c: impl Fn() -> Fut) {}
>
> fn f() {
>     let x = &1i32;
>     let c = async move || {
>         let a = x + 2;  // x 通过 ByValue 捕获
>     };
>     takes_callback(c);  // 错误：异步闭包未实现 `Fn`
> }
> ```
>
> 第二种情况的例外可以通过使用解引用来说明，这确实允许实现 `Fn` 和 `FnMut` ：
>
> ```rust
> fn takes_callback<Fut: Future>(c: impl Fn() -> Fut) {}
>
> fn f() {
>     let x = &1i32;
>     let c = async move || {
>         let a = *x + 2;
>     };
>     takes_callback(c);  // OK：实现了 `Fn`
> }
> ```

r[type.closure.async.traits.async-family]
异步闭包实现 [`AsyncFn`] 、 [`AsyncFnMut`] 和 [`AsyncFnOnce`] 的方式，与普通闭包实现 [`Fn`] 、 [`FnMut`] 和 [`FnOnce`] 的方式类似；也就是说，取决于其体中对捕获变量的使用。

r[type.closure.traits]
### 其他特型

r[type.closure.traits.intro]
所有闭包类型都实现 [`Sized`] 。此外，如果闭包存储的捕获类型允许，闭包类型还会实现以下 特型 ：

* [`Clone`]
* [`Copy`]
* [`Sync`]
* [`Send`]

r[type.closure.traits.behavior]
 [`Send`] 和 [`Sync`] 的规则与普通结构体类型的规则一致，而 [`Clone`] 和 [`Copy`] 的行为就像是 [派生][derived] 的。对于 [`Clone`] ，捕获值的克隆顺序未指定。

由于捕获通常是通过引用进行的，因此会产生以下一般规则：

* 如果所有捕获的值都是 [`Sync`] ，则闭包是 [`Sync`] 。
* 如果所有通过非唯一不可变引用捕获的值都是 [`Sync`] ，且所有通过唯一不可变或可变引用、复制或移动捕获的值都是 [`Send`] ，则闭包是 [`Send`] 。
* 如果闭包未通过唯一不可变或可变引用捕获任何值，且它通过复制或移动捕获的所有值分别实现了 [`Clone`] 或 [`Copy`] ，则闭包是 [`Clone`] 或 [`Copy`] 。

[`Clone`]: ../special-types-and-traits.md#clone
[`Copy`]: ../special-types-and-traits.md#copy
[`Send`]: ../special-types-and-traits.md#send
[`Sized`]: ../special-types-and-traits.md#sized
[`Sync`]: ../special-types-and-traits.md#sync
[closure expression]: ../expressions/closure-expr.md
[derived]: ../attributes/derive.md

r[type.closure.drop-order]
## 丢弃顺序

如果闭包通过值捕获复合类型（如结构体、元组和枚举）的一个字段，则该字段的生命周期现在将与闭包绑定。因此，复合类型的互不相交字段可能会在不同时间被丢弃。

```rust
{
    let tuple =
      (String::from("foo"), String::from("bar")); // --+
    { //                                               |
        let c = || { // ----------------------------+  |
            // tuple.0 被捕获到闭包中                |  |
            drop(tuple.0); //                       |  |
        }; //                                       |  |
    } // 'c' 和 'tuple.0' 在这里被丢弃 -------------+  |
} // tuple.1 在这里被丢弃 -----------------------------+
```

r[type.closure.capture.precision.edition2018.entirety]
## 2018版次及更早版本

### 闭包类型差异

在2018版次及更早版本中，闭包总是整体捕获变量，而没有精确的捕获路径。这意味着对于 [闭包类型](#closure-types) 章节中使用的示例，生成的闭包类型将如下所示：

<!-- ignore: simplified -->
```rust,ignore
struct Closure<'a> {
    rect : &'a mut Rectangle,
}

impl<'a> FnOnce<()> for Closure<'a> {
    type Output = String;
    extern "rust-call" fn call_once(self, args: ()) -> String {
        self.rect.left_top.x += 1;
        self.rect.right_bottom.x += 1;
        format!("{:?}", self.rect.left_top)
    }
}
```

并且对 `f` 的调用将如下工作：

<!-- ignore: continuation of above -->
```rust,ignore
f(Closure { rect: rect });
```

r[type.closure.capture.precision.edition2018.composite]
### 捕获精度差异

复合类型（如结构体、元组和枚举）总是被整体捕获，而不是按单个字段捕获。因此，可能需要借用到局部变量中以便捕获单个字段：

```rust
# use std::collections::HashSet;
#
struct SetVec {
    set: HashSet<u32>,
    vec: Vec<u32>
}

impl SetVec {
    fn populate(&mut self) {
        let vec = &mut self.vec;
        self.set.iter().for_each(|&n| {
            vec.push(n);
        })
    }
}
```

相反，如果闭包直接使用 `self.vec` ，那么它将尝试通过可变引用捕获 `self` 。但由于 `self.set` 已经被借用用于迭代，代码将无法编译。

r[type.closure.capture.precision.edition2018.move]
如果使用了 `move` 关键字，那么所有的捕获都是通过移动进行的，或者对于 `Copy` 类型通过复制进行，无论借用是否可行。 `move` 关键字通常用于允许闭包存活时间超过捕获的值，例如闭包被返回或用于派生新线程的情况。

r[type.closure.capture.precision.edition2018.wildcard]
无论数据是否会被闭包读取（即在通配符模式的情况下），如果在闭包内提到了闭包外定义的变量，该变量都将被整体捕获。

r[type.closure.capture.precision.edition2018.drop-order]
### 丢弃顺序差异

由于复合类型是被整体捕获的，因此通过值捕获其中一种复合类型的闭包将在闭包被丢弃的同时丢弃整个捕获的变量。

```rust
{
    let tuple =
      (String::from("foo"), String::from("bar"));
    {
        let c = || { // --------------------------+
            // tuple 被捕获到闭包中                  |
            drop(tuple.0); //                     |
        }; //                                     |
    } // 'c' 和 'tuple' 在这里被丢弃 --------------+
}
```
