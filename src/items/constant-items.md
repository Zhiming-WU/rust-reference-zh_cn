<!-- https://github.com/rust-lang/reference/blob/master/src/items/constant-items.md -->
<!-- commit 68bdfd15fa9feebfbf94a06cf574de598e239198 -->

r[items.const]
# 常量项

r[items.const.syntax]
```grammar,项
ConstantItem ->
    `const` ( IDENTIFIER | `_` ) `:` Type ( `=` Expression )? `;`
```

r[items.const.intro]
 *常量项*  是一个可选命名的  [常量值][constant value] ，它不与程序中特定的内存位置相关联。

r[items.const.behavior]
常量在本质上是在其被使用的地方内联的，这意味着在使用时它们被直接复制到相关的上下文中。这包括使用来自外部 crate 的常量，以及非 [`Copy`] 类型。指向同一常量的引用不一定保证指向相同的内存地址。

r[items.const.namespace]
常量声明在其所在的模块或代码块的  [值命名空间][value namespace]  中定义常量值。

r[items.const.static]
常量必须显式指定类型。该类型必须具有 `'static` 生命周期：初始化程序中的任何引用都必须具有 `'static` 生命周期。常量类型中的引用默认为 `'static` 生命周期；请参见  [静态生命周期省略][static lifetime elision] 。

r[items.const.static-temporary]
如果常量值符合  [提升][promotion]  条件，则指向该常量的引用将具有 `'static` 生命周期；否则，将创建一个临时变量。

```rust
const BIT1: u32 = 1 << 0;
const BIT2: u32 = 1 << 1;

const BITS: [u32; 2] = [BIT1, BIT2];
const STRING: &'static str = "bitstring";

struct BitsNStrings<'a> {
    mybits: [u32; 2],
    mystring: &'a str,
}

const BITS_N_STRINGS: BitsNStrings<'static> = BitsNStrings {
    mybits: BITS,
    mystring: STRING,
};
```

r[items.const.expr-omission]
常量表达式仅在  [特型定义][trait definition]  中可以省略。

r[items.const.destructor]
## 带有析构函数的常量

常量可以包含析构函数。当值超出作用域时，析构函数将运行。

```rust
struct TypeWithDestructor(i32);

impl Drop for TypeWithDestructor {
    fn drop(&mut self) {
        println!("Dropped. Held {}.", self.0);
    }
}

const ZERO_WITH_DESTRUCTOR: TypeWithDestructor = TypeWithDestructor(0);

fn create_and_drop_zero_with_destructor() {
    let x = ZERO_WITH_DESTRUCTOR;
    // x 在函数结束时被 drop，调用 drop。
    // 打印 "Dropped. Held 0."。
}
```

r[items.const.unnamed]
## 匿名常量

r[items.const.unnamed.intro]
与  [关联常量][associated constant]  不同， [自由][free]  常量可以通过使用下划线代替名称来保持匿名。例如：

```rust
const _: () =  { struct _SameNameTwice; };

// 尽管名称与上面相同，但是 OK 的：
const _: () =  { struct _SameNameTwice; };
```

r[items.const.unnamed.repetition]
与  [下划线导入][underscore imports]  一样，宏可以在同一作用域内安全地多次生成同一个匿名常量。例如，以下代码不应产生错误：

```rust
macro_rules! m {
    ($item: item) => { $item $item }
}

m!(const _: () = (););
// 这将展开为：
// const _: () = ();
// const _: () = ();
```

r[items.const.eval]
## 求值

 [自由][free]  常量始终在编译时  [求值][const_eval]  以暴露 恐慌。即使在未使用的函数中也会发生这种情况：

```rust,compile_fail
// 编译时恐慌
const PANIC: () = std::unimplemented!();

fn unused_generic_function<T>() {
    // 失败的编译时断言
    const _: () = assert!(usize::BITS == 0);
}
```

[const_eval]: ../const_eval.md
[associated constant]: ../items/associated-items.md#关联常量
[constant value]: ../const_eval.md#常量表达式
[free]: ../glossary.md#自由项
[static lifetime elision]: ../lifetime-elision.md#const和static省略
[trait definition]: traits.md
[underscore imports]: use-declarations.md#下划线导入
[`Copy`]: ../special-types-and-traits.md#copy
[value namespace]: ../names/namespaces.md
[promotion]: destructors.scope.const-promotion
