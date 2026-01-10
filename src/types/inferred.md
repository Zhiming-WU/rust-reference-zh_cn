r[type.inferred]
# 推断类型

r[type.inferred.syntax]
```grammar,types
InferredType -> `_`
```

r[type.inferred.intro]
推断类型 要求 编译器 根据可用的周边信息尽可能地推断类型。

> [!EXAMPLE]
> 推断类型 常用于 泛型 参数：
>
> ```rust
> let x: Vec<_> = (0..10).collect();
> ```

r[type.inferred.constraint]
推断类型 不能用于 项 签名。

<!--
  这里还需要说什么？
  我所知的唯一文档是 https://rustc-dev-guide.rust-lang.org/type-inference.html
  应该在某个地方更广泛地讨论 类型推断。
-->
