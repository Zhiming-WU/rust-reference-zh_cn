r[attributes.testing]
# 测试属性

以下 [属性][attributes] 用于指定执行测试的函数。在“测试”模式下编译 crate 会启用测试函数的构建以及执行测试的测试工具。启用测试模式还会启用 [`test` 条件编译选项][`test` conditional compilation option]。

<!-- template:attributes -->
r[attributes.testing.test]
## `test` 属性

r[attributes.testing.test.intro]
*`test` [属性][attributes]* 将一个函数标记为要作为测试执行。

> [!EXAMPLE]
> ```rust,no_run
> # pub fn add(left: u64, right: u64) -> u64 { left + right }
> #[test]
> fn it_works() {
>     let result = add(2, 2);
>     assert_eq!(result, 4);
> }
> ```

r[attributes.testing.test.syntax]
`test` 属性使用 [元字][MetaWord] 语法格式。

r[attributes.testing.test.allowed-positions]
`test` 属性只能应用于单态的、不带参数且返回类型实现 [`Termination`] 特型 的 [自由函数][free functions]。

> [!NOTE]
> 实现了 [`Termination`] 特型 的一些类型包括：
> * `()`
> * `Result<T, E> where T: Termination, E: Debug`

r[attributes.testing.test.duplicates]
`test` 在函数上的第一次使用才有效。

> [!NOTE]
> `rustc` 对第一次使用之后的任何使用都会发出 lint 警告。这在将来可能会变为错误。

<!-- TODO: This is a minor lie. Currently rustc warns that duplicates are ignored, but it then generates multiple test entries with the same name. I would vote for rejecting this in the future. -->

r[attributes.testing.test.stdlib]
`test` 属性从标准库 预导入 [`std::prelude::v1::test`] 中导出。

r[attributes.testing.test.enabled]
这些函数只在测试模式下编译。

> [!NOTE]
> 测试模式通过向 `rustc` 传递 `--test` 参数或使用 `cargo test` 启用。

r[attributes.testing.test.success]
测试工具调用返回值的 [`report`] 方法，并根据生成的 [`ExitCode`] 是否表示成功终止来将测试分类为通过或失败。
特别是：
* 返回 `()` 的测试只要终止且不 恐慌 就通过。
* 返回 `Result<(), E>` 的测试只要返回 `Ok(())` 就通过。
* 返回 `ExitCode::SUCCESS` 的测试通过，返回 `ExitCode::FAILURE` 的测试失败。
* 不终止的测试既不通过也不失败。

> [!EXAMPLE]
> ```rust,no_run
> # use std::io;
> # fn setup_the_thing() -> io::Result<i32> { Ok(1) }
> # fn do_the_thing(s: &i32) -> io::Result<()> { Ok(()) }
> #[test]
> fn test_the_thing() -> io::Result<()> {
>     let state = setup_the_thing()?; // 预期成功
>     do_the_thing(&state)?;          // 预期成功
>     Ok(())
> }
> ```

<!-- template:attributes -->
r[attributes.testing.ignore]
## `ignore` 属性

r[attributes.testing.ignore.intro]
*`ignore` [属性][attributes]* 可以与 [`test` 属性][attributes.testing.test] 一起使用，以告知测试工具不要将该函数作为测试执行。

> [!EXAMPLE]
> ```rust,no_run
> #[test]
> #[ignore]
> fn check_thing() {
>     // ...
> }
> ```

> [!NOTE]
> `rustc` 测试工具支持 `--include-ignored` 标志来强制运行被忽略的测试。

r[attributes.testing.ignore.syntax]
`ignore` 属性使用 [元字][MetaWord] 和 [元名称值字符串][MetaNameValueStr] 语法格式。

r[attributes.testing.ignore.reason]
`ignore` 属性的 [元名称值字符串][MetaNameValueStr] 形式提供了一种方法来指定忽略测试的原因。

> [!EXAMPLE]
> ```rust,no_run
> #[test]
> #[ignore = "not yet implemented"] // 尚未实现
> fn mytest() {
>     // ...
> }
> ```

r[attributes.testing.ignore.allowed-positions]
`ignore` 属性只能应用于带有 `test` 属性注解的函数。

> [!NOTE]
> `rustc` 在其他位置的使用会被忽略，但会发出 lint 警告。这在将来可能会变为错误。

r[attributes.testing.ignore.duplicates]
`ignore` 在函数上的第一次使用才有效。

> [!NOTE]
> `rustc` 对第一次使用之后的任何使用都会发出 lint 警告。这在将来可能会变为错误。

r[attributes.testing.ignore.behavior]
被忽略的测试在测试模式下仍然会被编译，但不会执行。

<!-- template:attributes -->
r[attributes.testing.should_panic]
## `should_panic` 属性

r[attributes.testing.should_panic.intro]
*`should_panic` [属性][attributes]* 会使测试仅在应用该属性的 [测试函数][attributes.testing.test] 发生 恐慌 时才通过。

> [!EXAMPLE]
> ```rust,no_run
> #[test]
> #[should_panic(expected = "values don't match")] // 预期为“值不匹配”
> fn mytest() {
>     assert_eq!(1, 2, "values don't match"); // 值不匹配
> }
> ```

r[attributes.testing.should_panic.syntax]
`should_panic` 属性有以下形式：

- [元字][MetaWord]
  > [!EXAMPLE]
  > ```rust,no_run
  > #[test]
  > #[should_panic]
  > fn mytest() { panic!("error: some message, and more"); } // 错误：一些消息，还有更多
  > ```

- [元名称值字符串][MetaNameValueStr] --- 给定字符串必须出现在 恐慌 消息中，测试才能通过。
  > [!EXAMPLE]
  > ```rust,no_run
  > #[test]
  > #[should_panic = "some message"] // 一些消息
  > fn mytest() { panic!("error: some message, and more"); } // 错误：一些消息，还有更多
  > ```

- [元列表名称值字符串][MetaListNameValueStr] --- 与 [元名称值字符串][MetaNameValueStr] 语法格式 一样，给定字符串必须出现在 恐慌 消息中。
  > [!EXAMPLE]
  > ```rust,no_run
  > #[test]
  > #[should_panic(expected = "some message")] // 预期为“一些消息”
  > fn mytest() { panic!("error: some message, and more"); } // 错误：一些消息，还有更多
  > ```

r[attributes.testing.should_panic.allowed-positions]
`should_panic` 属性只能应用于带有 `test` 属性注解的函数。

> [!NOTE]
> `rustc` 在其他位置的使用会被忽略，但会发出 lint 警告。这在将来可能会变为错误。

r[attributes.testing.should_panic.duplicates]
`should_panic` 在函数上的第一次使用才有效。

> [!NOTE]
> `rustc` 对第一次使用之后的任何使用都会发出未来兼容性警告。这在将来可能会变为错误。

r[attributes.testing.should_panic.expected]
当使用 [元名称值字符串][MetaNameValueStr] 形式或带有 `expected` 键的 [元列表名称值字符串][MetaListNameValueStr] 形式时，给定字符串必须出现在 恐慌 消息中的某个位置，测试才能通过。

r[attributes.testing.should_panic.return]
测试函数的返回类型必须是 `()`。

[`Termination`]: std::process::Termination
[`report`]: std::process::Termination::report
[`test` conditional compilation option]: ../conditional-compilation.md#test
[attributes]: ../attributes.md
[`ExitCode`]: std::process::ExitCode
[free functions]: ../glossary.md#自由项
