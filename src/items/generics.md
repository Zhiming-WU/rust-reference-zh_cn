r[items.generics]
# 泛型参数

r[items.generics.syntax]
```grammar,items
GenericParams -> `<` ( GenericParam (`,` GenericParam)* `,`? )? `>`

GenericParam -> OuterAttribute* ( LifetimeParam | TypeParam | ConstParam )

LifetimeParam -> Lifetime ( `:` LifetimeBounds )?

TypeParam -> IDENTIFIER ( `:` TypeParamBounds? )? ( `=` Type )?

ConstParam ->
    `const` IDENTIFIER `:` Type
    ( `=` ( BlockExpression | IDENTIFIER | `-`?LiteralExpression ) )?
```

r[items.generics.syntax.intro]
[函数][Functions]、[类型别名][type aliases]、[结构体][structs]、[枚举][enumerations]、[联合体][unions]、[特型][traits] 和 [实现][implementations] 可以通过类型、常量和生命周期进行  *参数化 (parameterized)* 。这些参数列在尖括号 <span class="parenthetical">（`<...>`）</span>中，通常紧跟在 项 的名称之后、其定义之前。对于没有名称的 实现，它们直接跟在 `impl` 后面。

r[items.generics.syntax.decl-order]
泛型参数 的顺序受限，必须先是 生命周期参数，然后是混合在一起的 类型参数 和 常量参数。

r[items.generics.syntax.duplicate-params]
在 [GenericParams] 列表中，同一个参数名不得声明多次。

带类型、常量和生命周期参数的 项 示例：

```rust
fn foo<'a, T>() {}
trait A<U> {}
struct Ref<'a, T> where T: 'a { r: &'a T }
struct InnerArray<T, const N: usize>([T; N]);
struct EitherOrderWorks<const N: bool, U>(U);
```

r[items.generics.syntax.scope]
泛型参数 在其声明所在的 项 定义的作用域内。它们不在函数体内部声明的 项 的作用域内，如 [项声明][item declarations] 中所述。有关更多详细信息，请参阅 [泛型参数作用域][generic parameter scopes]。

r[items.generics.builtin-generic-types]
[引用][References]、[原始指针][raw pointers]、[数组][arrays]、[切片][slices]、[元组][tuples] 和 [函数指针][function pointers] 也有生命周期或类型参数，但不是通过路径语法引用的。

r[items.generics.invalid-lifetimes]
`'_` 和 `'static` 不是有效的 生命周期参数 名称。

r[items.generics.const]
### 常量泛型

r[items.generics.const.intro]
 *常量泛型参数 (Const generic parameters)*  允许 项 对常量值进行泛型化。

r[items.generics.const.namespace]
常量标识符在 [值命名空间][value namespace] 中为 常量参数 引入一个名称，并且该 项 的所有实例必须使用给定类型的值进行实例化。

r[items.generics.const.allowed-types]
常量参数 唯一允许的类型是 `u8`、`u16`、`u32`、`u64`、`u128`、`usize`、`i8`、`i16`、`i32`、`i64`、`i128`、`isize`、`char` 和 `bool`。

r[items.generics.const.usage]
常量参数 可以用在任何可以使用 [常量项][const item] 的地方，但在 [类型][type] 或 [数组重复表达式][array repeat expression] 中使用时，它必须是  *独立的 (standalone)* （如下所述）。也就是说，它们允许出现在以下位置：

1. 作为应用于构成相关 项 签名一部分的任何类型的常量。
2. 作为用于定义 [关联常量][associated const] 的常量表达式的一部分，或作为 [关联类型][associated type] 的参数。
3. 作为该 项 中任何函数体内的任何运行时表达式的值。
4. 作为该 项 中任何函数体内使用的任何类型的参数。
5. 作为该 项 中任何字段类型的一部分。

```rust
// 常量泛型参数可以使用的示例。

// 在项本身的签名中使用。
fn foo<const N: usize>(arr: [i32; N]) {
    // 在函数体中作为类型使用。
    let x: [i32; N];
    // 作为表达式使用。
    println!("{}", N * 2);
}

// 作为结构体的字段使用。
struct Foo<const N: usize>([i32; N]);

impl<const N: usize> Foo<N> {
    // 作为关联常量使用。
    const CONST: usize = N * 4;
}

trait Trait {
    type Output;
}

impl<const N: usize> Trait for Foo<N> {
    // 作为关联类型使用。
    type Output = [i32; N];
}
```

```rust,compile_fail
// 常量泛型参数不能使用的示例。
fn foo<const N: usize>() {
    // 不能在函数体内的项定义中使用。
    const BAD_CONST: [usize; N] = [1; N];
    static BAD_STATIC: [usize; N] = [1; N];
    fn inner(bad_arg: [usize; N]) {
        let bad_value = N * 2;
    }
    type BadAlias = [usize; N];
    struct BadStruct([usize; N]);
}
```

r[items.generics.const.standalone]
作为进一步的限制， 常量参数 只能作为 [类型][type] 或 [数组重复表达式][array repeat expression] 内部的独立参数出现。在这些上下文中，它们只能用作单段 [路径表达式][path expression]，可能位于 [块][block] 内部（例如 `N` 或 `{N}`）。也就是说，它们不能与其他表达式结合使用。

```rust,compile_fail
// 不允许使用常量参数的示例。

// 不允许在类型的其他表达式中进行组合，例如
// 这里返回类型中的算术表达式。
fn bad_function<const N: usize>() -> [u8; {N + 1}] {
    // 数组重复表达式同样不允许。
    [1; {N + 1}]
}
```

r[items.generics.const.argument]
[路径][path] 中的常量参数指定了用于该 项 的常量值。

r[items.generics.const.argument.const-expr]
该参数必须是 [推断常量][inferred const] 或者是归属于 常量参数 类型的 [常量表达式][const expression]。常量表达式必须是 [块表达式][block]（用大括号包围），除非它是单个路径段（一个 [IDENTIFIER]）或 [字面量][literal]（可能带有前导的 `-` 词法单元）。

> [!NOTE]
> 这种语法限制是必要的，以避免在解析类型内部的表达式时需要无限前瞻。

```rust
struct S<const N: i64>;
const C: i64 = 1;
fn f<const N: i64>() -> S<N> { S }

let _ = f::<1>(); // 字面量。
let _ = f::<-1>(); // 负数字面量。
let _ = f::<{ 1 + 2 }>(); // 常量表达式。
let _ = f::<C>(); // 单段路径。
let _ = f::<{ C + 1 }>(); // 常量表达式。
let _: S<1> = f::<_>(); // 推断常量。
let _: S<1> = f::<(((_)))>(); // 推断常量。
```

> [!NOTE]
> 在泛型参数列表中，[推断常量][inferred const] 被解析为 [推断类型][InferredType]，但在语义上被视为一种单独的 [常量泛型参数][const generic argument]。

r[items.generics.const.inferred]
在需要常量参数的地方，可以使用 `_`（可选地由任意数量的匹配括号包围），称为  *推断常量 (inferred const)* （[路径规则][paths.expr.complex-const-params]，[数组表达式规则][expr.array.length-restriction]）。这要求编译器在可能的情况下根据周围信息推断常量参数。

```rust
fn make_buf<const N: usize>() -> [u8; N] {
    [0; _]
    //  ^ 推断出 `N`。
}
let _: [u8; 1024] = make_buf::<_>();
//                             ^ 推断出 `1024`。
```

> [!NOTE]
> [推断常量][inferred const] 在语义上不是一个 [表达式][Expression]，因此不被大括号接受。
>
> ```rust,compile_fail
> fn f<const N: usize>() -> [u8; N] { [0; _] }
> let _: [_; 1] = f::<{ _ }>();
> //                    ^ 错误：这里不允许使用 `_`
> ```

r[items.generics.const.inferred.constraint]
推断常量 不能在 项 签名中使用。

```rust,compile_fail
fn f<const N: usize>(x: [u8; N]) -> [u8; _] { x }
//                                       ^ 错误：不允许
```

r[items.generics.const.type-ambiguity]
当泛型参数是否可以被解析为类型参数或常量参数存在歧义时，它总是被解析为类型。将参数放置在块表达式中可以强制将其解释为常量参数。

<!-- TODO: Rewrite the paragraph above to be in terms of namespaces, once
    namespaces are introduced, and it is clear which namespace each parameter
    lives in. -->

```rust,compile_fail
type N = u32;
struct Foo<const N: usize>;
// 以下是一个错误，因为 `N` 被解释为类型别名 `N`。
fn foo<const N: usize>() -> Foo<N> { todo!() } // 错误
// 可以通过包裹在大括号中来修复，以强制将其解释为 `N` 
// 常量参数：
fn bar<const N: usize>() -> Foo<{ N }> { todo!() } // ok
```

r[items.generics.const.variance]
与类型和生命周期参数不同， 常量参数 可以在不在 参数化项 内部使用的情况下声明，但 [泛型实现][generic implementations] 中描述的 实现 除外：

```rust,compile_fail
// ok
struct Foo<const N: usize>;
enum Bar<const M: usize> { A, B }

// 错误：未使用的参数
struct Baz<T>;
struct Biz<'a>;
struct Unconstrained;
impl<const N: usize> Unconstrained {}
```

r[items.generics.const.exhaustiveness]
在解析 特型绑定 义务时，确定绑定是否满足时不考虑 常量参数 所有实现的  *完备性 (exhaustiveness)* 。例如，在下文中，即使实现了 `bool` 类型所有可能的常量值，但 特型绑定 未满足仍然是一个错误：

```rust,compile_fail
struct Foo<const B: bool>;
trait Bar {}
impl Bar for Foo<true> {}
impl Bar for Foo<false> {}

fn needs_bar(_: impl Bar) {}
fn generic<const B: bool>() {
    let v = Foo::<B>;
    needs_bar(v); // 错误：特型绑定 `Foo<B>: Bar` 未满足
}
```

r[items.generics.where]
## Where子句

r[items.generics.where.syntax]
```grammar,items
WhereClause -> `where` ( WhereClauseItem `,` )* WhereClauseItem?

WhereClauseItem ->
      LifetimeWhereClauseItem
    | TypeBoundWhereClauseItem

LifetimeWhereClauseItem -> Lifetime `:` LifetimeBounds

TypeBoundWhereClauseItem -> ForLifetimes? Type `:` TypeParamBounds?
```

r[items.generics.where.intro]
 *Where 子句*  提供了另一种指定类型和生命周期参数绑定的方法，以及一种为非类型参数的类型指定绑定的方法。

r[items.generics.where.higher-ranked-lifetimes]
`for` 关键字可用于引入 [高阶生命周期][higher-ranked lifetimes]。它只允许 [LifetimeParam] 参数。

```rust
struct A<T>
where
    T: Iterator,            // 也可以使用 A<T: Iterator>
    T::Item: Copy,          // 关联类型上的绑定
    String: PartialEq<T>,   // 在 `String` 上的绑定，使用类型参数
    i32: Default,           // 允许，但没什么用
{
    f: T,
}
```

r[items.generics.attributes]
## 属性

泛型生命周期和类型参数允许在其上使用 [属性][attributes]。虽然自定义派生属性可能会赋予其含义，但目前没有内置属性在此位置执行任何操作。

此示例显示使用自定义派生属性来修改 泛型参数 的含义。

<!-- ignore: requires proc macro derive -->
```rust,ignore
// 假设 MyFlexibleClone 的派生宏将 `my_flexible_clone` 声明为
// 其理解的属性。
#[derive(MyFlexibleClone)]
struct Foo<#[my_flexible_clone(unbounded)] H> {
    a: *const H
}
```

[array repeat expression]: ../expressions/array-expr.md
[arrays]: ../types/array.md
[slices]: ../types/slice.md
[associated const]: associated-items.md#associated-constants
[associated type]: associated-items.md#associated-types
[attributes]: ../attributes.md
[block]: ../expressions/block-expr.md
[const contexts]: ../const_eval.md#const-context
[const expression]: ../const_eval.md#constant-expressions
[const generic argument]: items.generics.const.argument
[const item]: constant-items.md
[enumerations]: enumerations.md
[functions]: functions.md
[function pointers]: ../types/function-pointer.md
[generic implementations]: implementations.md#generic-implementations
[generic parameter scopes]: ../names/scopes.md#generic-parameter-scopes
[higher-ranked lifetimes]: ../trait-bounds.md#higher-ranked-trait-bounds
[implementations]: implementations.md
[inferred const]: items.generics.const.inferred
[item declarations]: ../statements.md#item-declarations
[item]: ../items.md
[literal]: ../expressions/literal-expr.md
[path]: ../paths.md
[path expression]: ../expressions/path-expr.md
[raw pointers]: ../types/pointer.md#raw-pointers-const-and-mut
[references]: ../types/pointer.md#shared-references-
[structs]: structs.md
[tuples]: ../types/tuple.md
[trait object]: ../types/trait-object.md
[traits]: traits.md
[type aliases]: type-aliases.md
[type]: ../types.md
[unions]: unions.md
[value namespace]: ../names/namespaces.md
