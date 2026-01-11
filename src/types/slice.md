r[type.slice]
# 切片类型

r[type.slice.syntax]
```grammar,类型
SliceType -> `[` Type `]`
```

r[type.slice.intro]
切片是一种 [动态大小类型][dynamically sized type] ，表示一个 T 类型元素序列的 '视图' 。切片类型写作 `[T]` 。

r[type.slice.unsized]
切片类型通常通过指针类型使用。例如：

* `&[T]`: 一个 '共享切片' ，通常只被称为 '切片' 。它不拥有它指向的数据；它借用它。
* `&mut [T]`: 一个 '可变切片' 。它可变地借用它指向的数据。
* `Box<[T]>`: 一个 '装箱切片'

示例：

```rust
// 一个堆分配的数组， 隐式类型转换 为切片
let boxed_array: Box<[i32]> = Box::new([1, 2, 3]);

// 一个指向数组的 (共享) 切片
let slice: &[i32] = &boxed_array[..];
```

r[type.slice.safe]
切片的所有元素总是被初始化的，并且在安全方法和运算符中访问切片总是会进行边界检查。

[dynamically sized type]: ../dynamically-sized-types.md
