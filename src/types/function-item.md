r[type.fn-item]
# 函数项类型

r[type.fn-item.intro]
当引用时，函数 项 ，或者类元组 结构体 或枚举 变体 的构造函数，会产生其  *函数项类型*  的一个零大小值。

r[type.fn-item.unique]
该类型明确标识了函数 —— 它的名称、它的类型参数以及它的早绑定生命周期参数（但不包括它的晚绑定生命周期参数，后者仅在调用函数时分配） —— 因此该值不需要包含实际的函数指针，并且在调用函数时不需要间接寻址。

r[type.fn-item.name]
没有直接引用函数项类型的 语法格式 ，但编译器在错误消息中会将该类型显示为类似于 `fn(u32) -> i32 {fn_name}` 的内容。

由于函数项类型明确标识了函数，不同函数的 项 类型 —— 不同的 项 ，或具有不同 泛型 的相同 项  —— 是不同的，将它们混合会产生类型错误：

```rust,compile_fail,E0308
fn foo<T>() { }
let x = &mut foo::<i32>;
*x = foo::<u32>; //~ 错误：类型不匹配
```

r[type.fn-item.coercion]
然而，存在从函数项到具有相同签名的 [函数指针][function pointers] 的 [隐式类型转换][coercion]，这不仅在直接预期函数指针的情况下使用函数项时触发，而且在具有相同签名的不同函数项类型出现在同一个 `if` 或 `match` 的不同分支中时也会触发：

```rust
# let want_i32 = false;
# fn foo<T>() { }

// 这里 `foo_ptr_1` 具有函数指针类型 `fn()`
let foo_ptr_1: fn() = foo::<i32>;

// ... `foo_ptr_2` 也是如此 —— 这通过了类型检查。
let foo_ptr_2 = if want_i32 {
    foo::<i32>
} else {
    foo::<u32>
};
```

r[type.fn-item.traits]
所有函数项都实现 [`Copy`]、[`Clone`]、[`Send`] 和 [`Sync`]。

[`Fn`]、[`FnMut`] 和 [`FnOnce`] 都会被实现，除非函数具有以下任何一种情况：

-  [`unsafe`][unsafe.fn] 限定符
-  [`target_feature` 属性][attributes.codegen.target_feature]
- 除 `"Rust"` 以外的 [ABI][items.fn.extern]

[`Clone`]: ../special-types-and-traits.md#clone
[`Copy`]: ../special-types-and-traits.md#copy
[`Send`]: ../special-types-and-traits.md#send
[`Sync`]: ../special-types-and-traits.md#sync
[coercion]: ../type-coercions.md
[function pointers]: function-pointer.md
