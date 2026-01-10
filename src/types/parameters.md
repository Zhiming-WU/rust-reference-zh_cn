r[type.generic]
# 类型参数

在具有类型参数声明的 项 的主体内，其类型参数的名称就是类型：

```rust
fn to_vec<A: Clone>(xs: &[A]) -> Vec<A> {
    if xs.is_empty() {
        return vec![];
    }
    let first: A = xs[0].clone();
    let mut rest: Vec<A> = to_vec(&xs[1..]);
    rest.insert(0, first);
    rest
}
```

这里， first 的类型为 A ，引用了 to_vec 的 A 类型参数；而 rest 的类型为 Vec<A> ，这是一个元素类型为 A 的向量。
