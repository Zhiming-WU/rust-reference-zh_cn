r[attributes.codegen]
# 代码生成属性

以下 [属性][attributes] 用于控制代码生成。

<!-- template:attributes -->
r[attributes.codegen.inline]
### `inline`属性

r[attributes.codegen.inline.intro]
*`inline` [属性][attribute]* 建议是将所修饰函数代码的副本放置在调用者中，而不是生成对函数的调用。

> [!EXAMPLE]
> ```rust
> #[inline]
> pub fn example1() {}
>
> #[inline(always)]
> pub fn example2() {}
>
> #[inline(never)]
> pub fn example3() {}
> ```

> [!NOTE]
> `rustc` 在看起来值得时会自动内联函数。请谨慎使用此属性，因为内联决策不当可能会降低程序速度。

r[attributes.codegen.inline.syntax]
`inline` 属性的 语法格式 如下：

```grammar,属性
@root InlineAttribute ->
      `inline` `(` `always` `)`
    | `inline` `(` `never` `)`
    | `inline`
```

r[attributes.codegen.inline.allowed-positions]
`inline` 属性只能应用于带有 [函数体][bodies] 的函数——[闭包][closures]、[异步块][async blocks]、[自由函数][free functions]、[固有实现][inherent impl] 或 [特型实现][trait impl] 中的 [关联函数][associated functions]，以及 [特型定义][trait definition] 中带有 [默认定义][default definition] 的关联函数。

> [!NOTE]
> `rustc` 忽略在其他位置的使用，但会对其进行 lint 检查。这在将来可能会成为错误。

> [!NOTE]
> 尽管该属性可以应用于 [闭包][closures] 和 [异步块][async blocks]，但其用处有限，因为我们尚不支持表达式上的属性。
>
> ```rust
> // 我们允许在语句上使用属性。
> #[inline] || (); // OK
> #[inline] async {}; // OK
> ```
>
> ```rust,compile_fail,E0658
> // 我们尚不支持在表达式上使用属性。
> let f = #[inline] || (); // ERROR
> ```

r[attributes.codegen.inline.duplicates]
`inline` 在函数上的首次使用才生效。

> [!NOTE]
> `rustc` 会对首次使用之后的任何使用进行 lint 检查。这在将来可能会成为错误。

r[attributes.codegen.inline.modes]
`inline` 属性支持以下模式：

- `#[inline]` *建议* 执行内联展开。
- `#[inline(always)]` *建议* 始终执行内联展开。
- `#[inline(never)]` *建议* 永不执行内联展开。

> [!NOTE]
> 无论何种形式，该属性都只是一个提示。编译器可能会忽略它。

r[attributes.codegen.inline.trait]
当 `inline` 应用于 [特型][trait] 中的函数时，它仅应用于 [默认定义][default definition] 的代码。

r[attributes.codegen.inline.async]
当 `inline` 应用于 [异步函数][async function] 或 [异步闭包][async closure] 时，它仅应用于生成的 `poll` 函数的代码。

> [!NOTE]
> 欲了解更多详情，请参阅 [Rust issue #129347](https://github.com/rust-lang/rust/issues/129347)。

r[attributes.codegen.inline.externally-exported]
如果函数通过 [`no_mangle`] 或 [`export_name`] 外部导出，则 `inline` 属性会被忽略。

<!-- template:attributes -->
r[attributes.codegen.cold]
### `cold`属性

r[attributes.codegen.cold.intro]
*`cold` [属性][attribute]* 建议所修饰的函数不太可能被调用，这可能有助于编译器生成更好的代码。

> [!EXAMPLE]
> ```rust
> #[cold]
> pub fn example() {}
> ```

r[attributes.codegen.cold.syntax]
`cold` 属性使用 [MetaWord] 语法格式。

r[attributes.codegen.cold.allowed-positions]
`cold` 属性只能应用于带有 [函数体][bodies] 的函数——[闭包][closures]、[异步块][async blocks]、[自由函数][free functions]、[固有实现][inherent impl] 或 [特型实现][trait impl] 中的 [关联函数][associated functions]，以及 [特型定义][trait definition] 中带有 [默认定义][default definition] 的关联函数。

> [!NOTE]
> `rustc` 忽略在其他位置的使用，但会对其进行 lint 检查。这在将来可能会成为错误。

> [!NOTE]
> 尽管该属性可以应用于 [闭包][closures] 和 [异步块][async blocks]，但其用处有限，因为我们尚不支持表达式上的属性。

<!-- TODO: rustc currently seems to allow cold on a trait function without a body, but it appears to be ignored. I think that may be a bug, and it should at least warn if not reject (like inline does). -->

r[attributes.codegen.cold.duplicates]
`cold` 在函数上的首次使用才生效。

> [!NOTE]
> `rustc` 会对首次使用之后的任何使用进行 lint 检查。这在将来可能会成为错误。

r[attributes.codegen.cold.trait]
当 `cold` 应用于 [特型][trait] 中的函数时，它仅应用于 [默认定义][default definition] 的代码。

r[attributes.codegen.naked]
## `naked`属性

r[attributes.codegen.naked.intro]
*`naked` [属性][attribute]* 阻止编译器为所修饰的函数发出函数序言和函数尾声。

r[attributes.codegen.naked.body]
[函数体][function body] 必须由且仅由一个 [`naked_asm!`] 宏调用组成。

r[attributes.codegen.naked.prologue-epilogue]
不会为所修饰的函数生成函数序言或函数尾声。`naked_asm!` 块中的汇编代码构成了裸函数的完整函数体。

r[attributes.codegen.naked.unsafe-attribute]
`naked` 属性是一个 [不安全 属性][unsafe attribute]。使用 `#[unsafe(naked)]` 注解函数附带的安全义务是，函数体必须遵守函数的调用约定，维护其签名，并且要么返回，要么发散（即，不能在汇编代码末尾之后继续执行）。

r[attributes.codegen.naked.call-stack]
汇编代码可以假定调用栈和寄存器状态在入口处是有效的，符合函数的签名和调用约定。

r[attributes.codegen.naked.no-duplication]
除了多态函数单态化时，编译器不得复制汇编代码。

> [!NOTE]
> 保证汇编代码何时可以或不可以被复制对于定义符号的裸函数很重要。

r[attributes.codegen.naked.unused-variables]
[`unused_variables`] lint 在裸函数内部被抑制。

r[attributes.codegen.naked.inline]
[`inline`](#inline属性) 属性不能应用于裸函数。

r[attributes.codegen.naked.track_caller]
[`track_caller`](#track_caller属性) 属性不能应用于裸函数。

r[attributes.codegen.naked.testing]
[测试属性](testing.md) 不能应用于裸函数。

<!-- template:attributes -->
r[attributes.codegen.no_builtins]
## `no_builtins`属性

r[attributes.codegen.no_builtins.intro]
*`no_builtins` [属性][attribute]* 禁用与对假定存在的库函数调用相关的某些代码模式的优化。

<!-- TODO: This needs expanding, see <https://github.com/rust-lang/reference/issues/542>. -->

> [!EXAMPLE]
> ```rust
> #![no_builtins]
> ```

r[attributes.codegen.no_builtins.syntax]
`no_builtins` 属性使用 [MetaWord] 语法格式。

r[attributes.codegen.no_builtins.allowed-positions]
`no_builtins` 属性只能应用于 crate 根。

r[attributes.codegen.no_builtins.duplicates]
`no_builtins` 属性的首次使用才生效。

> [!NOTE]
> `rustc` 会对首次使用之后的任何使用进行 lint 检查。

r[attributes.codegen.target_feature]
## `target_feature`属性

r[attributes.codegen.target_feature.intro]
*`target_feature` [属性][attribute]* 可以应用于函数，以启用该函数针对特定平台架构 特性 的代码生成。它使用 [MetaListNameValueStr] 语法格式，其中包含一个 `enable` 的键，其值是逗号分隔的 特性 名称字符串，以启用这些 特性。

```rust
# #[cfg(target_feature = "avx2")]
#[target_feature(enable = "avx2")]
fn foo_avx2() {}
```

r[attributes.codegen.target_feature.arch]
每个 [目标架构][target architecture] 都有一组可以启用的 特性。为 crate 未编译的目标架构指定 特性 是一个错误。

r[attributes.codegen.target_feature.closures]
在带有 `target_feature` 注解的函数中定义的闭包会从封闭函数继承该属性。

r[attributes.codegen.target_feature.target-ub]
调用使用当前代码运行平台不支持的 特性 编译的函数是 [未定义行为][undefined behavior]，*除非* 平台明确文档说明这是安全的。

r[attributes.codegen.target_feature.safety-restrictions]
除非以下平台规则另有规定，否则适用以下限制：

- 安全的 `#[target_feature]` 函数（以及继承该属性的闭包）只能在启用了被调用者启用所有 `target_feature` 的调用者中安全地调用。此限制不适用于 `unsafe` 上下文。
- 安全的 `#[target_feature]` 函数（以及继承该属性的闭包）只能在启用了被强制转换者启用所有 `target_feature` 的上下文中被强制转换为 *安全* 的函数指针。此限制不适用于 `unsafe` 函数指针。

隐式启用的 特性 也包含在此规则中。例如，一个 `sse2` 函数可以调用标记有 `sse` 的函数。

```rust
# #[cfg(target_feature = "sse2")] {
#[target_feature(enable = "sse")]
fn foo_sse() {}

fn bar() {
    // 在这里调用 `foo_sse` 是不安全的，因为我们必须首先确保 SSE 可用，
    // 即使 `sse` 在目标平台上默认启用或通过编译器标志手动启用。
    unsafe {
        foo_sse();
    }
}

#[target_feature(enable = "sse")]
fn bar_sse() {
    // 在这里调用 `foo_sse` 是安全的。
    foo_sse();
    || foo_sse();
}

#[target_feature(enable = "sse2")]
fn bar_sse2() {
    // 在这里调用 `foo_sse` 是安全的，因为 `sse2` 意味着 `sse`。
    foo_sse();
}
# }
```

r[attributes.codegen.target_feature.fn-traits]
带有 `#[target_feature]` 属性 的函数 *从不* 实现 `Fn` 特型 家族，尽管从封闭函数继承 特性 的闭包会实现。

r[attributes.codegen.target_feature.allowed-positions]
`#[target_feature]` 属性不允许在以下位置使用：

- [主函数][crate.main]
- [恐慌处理器函数][panic.panic_handler]
- 安全的 特型 方法
- 特型 中安全的默认函数

r[attributes.codegen.target_feature.inline]
标记有 `target_feature` 的函数不会内联到不支持给定 特性 的上下文中。`#[inline(always)]` 属性不能与 `target_feature` 属性一起使用。

r[attributes.codegen.target_feature.availability]
### 可用的特性

以下是可用 特性 名称的列表。

r[attributes.codegen.target_feature.x86]
#### `x86`或`x86_64`

在此平台上执行带有不支持 特性 的代码是未定义行为。因此，在此平台上使用 `#[target_feature]` 函数遵循 [上述限制][attributes.codegen.target_feature.safety-restrictions]。

特性     | 隐式启用      | 描述
------------|--------------------|-------------------
`adx`       |          | [ADX] --- 多精度加进位指令扩展
`aes`       | `sse2`   | [AES] --- 高级加密标准
`avx`       | `sse4.2` | [AVX] --- 高级向量扩展
`avx2`      | `avx`    | [AVX2] --- 高级向量扩展 2
`avx512bf16`        | `avx512bw`           | [AVX512-BF16] --- 高级向量扩展 512 位 - Bfloat16 扩展
`avx512bitalg`      | `avx512bw`           | [AVX512-BITALG] --- 高级向量扩展 512 位 - 位算法
`avx512bw`          | `avx512f`            | [AVX512-BW] --- 高级向量扩展 512 位 - 字节和字指令
`avx512cd`          | `avx512f`            | [AVX512-CD] --- 高级向量扩展 512 位 - 冲突检测指令
`avx512dq`          | `avx512f`            | [AVX512-DQ] --- 高级向量扩展 512 位 - 双字和四字指令
`avx512f`           | `avx2`, `fma`, `f16c`| [AVX512-F] --- 高级向量扩展 512 位 - 基础
`avx512fp16`        | `avx512bw`           | [AVX512-FP16] --- 高级向量扩展 512 位 - 浮点16 扩展
`avx512ifma`        | `avx512f`            | [AVX512-IFMA] --- 高级向量扩展 512 位 - 整数融合乘加
`avx512vbmi`        | `avx512bw`           | [AVX512-VBMI] --- 高级向量扩展 512 位 - 向量字节操作指令
`avx512vbmi2`       | `avx512bw`           | [AVX512-VBMI2] --- 高级向量扩展 512 位 - 向量字节操作指令 2
`avx512vl`          | `avx512f`            | [AVX512-VL] --- 高级向量扩展 512 位 - 向量长度扩展
`avx512vnni`        | `avx512f`            | [AVX512-VNNI] --- 高级向量扩展 512 位 - 向量神经网络指令
`avx512vp2intersect`| `avx512f`            | [AVX512-VP2INTERSECT] --- 高级向量扩展 512 位 - 向量对交集到一对掩码寄存器
`avx512vpopcntdq`   | `avx512f`            | [AVX512-VPOPCNTDQ] --- 高级向量扩展 512 位 - 向量人口计数指令
`avxifma`           | `avx2`               | [AVX-IFMA] --- 高级向量扩展 - 整数融合乘加
`avxneconvert`      | `avx2`               | [AVX-NE-CONVERT] --- 高级向量扩展 - 无异常浮点转换指令
`avxvnni`           | `avx2`               | [AVX-VNNI] --- 高级向量扩展 - 向量神经网络指令
`avxvnniint16`      | `avx2`               | [AVX-VNNI-INT16] --- 高级向量扩展 - 带有 16 位整数的向量神经网络指令
`avxvnniint8`       | `avx2`               | [AVX-VNNI-INT8] --- 高级向量扩展 - 带有 8 位整数的向量神经网络指令
`bmi1`      |          | [BMI1] --- 位操作指令集
`bmi2`      |          | [BMI2] --- 位操作指令集 2
`cmpxchg16b`|          | [`cmpxchg16b`] --- 原子地比较和交换 16 字节（128 位）数据
`f16c`      | `avx`    | [F16C] --- 16 位浮点转换指令
`fma`       | `avx`    | [FMA3] --- 三操作数融合乘加
`fxsr`      |          | [`fxsave`] 和 [`fxrstor`] --- 保存和恢复 x87 FPU、MMX 技术和 SSE 状态
`gfni`      | `sse2`   | [GFNI] --- 伽罗瓦域新指令
`kl`        | `sse2`   | [KEYLOCKER] --- 英特尔密钥锁定指令
`lzcnt`     |          | [`lzcnt`] --- 前导零计数
`movbe`     |          | [`movbe`] --- 字节交换后移动数据
`pclmulqdq` | `sse2`   | [`pclmulqdq`] --- 打包无进位乘法四字
`popcnt`    |          | [`popcnt`] --- 设置为 1 的位数计数
`rdrand`    |          | [`rdrand`] --- 读取随机数
`rdseed`    |          | [`rdseed`] --- 读取随机种子
`sha`       | `sse2`   | [SHA] --- 安全散列算法
`sha512`    | `avx2`   | [SHA512] --- 安全散列算法与 512 位摘要
`sm3`       | `avx`    | [SM3] --- 商密 3 散列算法
`sm4`       | `avx2`   | [SM4] --- 商密 4 密码算法
`sse`       |          | [SSE] --- 流式 <abbr title="Single Instruction Multiple Data">SIMD</abbr> 扩展
`sse2`      | `sse`    | [SSE2] --- 流式 SIMD 扩展 2
`sse3`      | `sse2`   | [SSE3] --- 流式 SIMD 扩展 3
`sse4.1`    | `ssse3`  | [SSE4.1] --- 流式 SIMD 扩展 4.1
`sse4.2`    | `sse4.1` | [SSE4.2] --- 流式 SIMD 扩展 4.2
`sse4a`     | `sse3`   | [SSE4a] --- 流式 SIMD 扩展 4a
`ssse3`     | `sse3`   | [SSSE3] --- 补充流式 SIMD 扩展 3
`tbm`       |          | [TBM] --- 尾随位操作
`vaes`      | `avx2`, `aes`     | [VAES] --- 向量 AES 指令
`vpclmulqdq`| `avx`, `pclmulqdq`| [VPCLMULQDQ] --- 四字向量无进位乘法
`widekl`    | `kl`     | [KEYLOCKER_WIDE] --- 英特尔宽密钥锁定指令
`xsave`     |          | [`xsave`] --- 保存处理器扩展状态
`xsavec`    |          | [`xsavec`] --- 保存处理器带压缩的扩展状态
`xsaveopt`  |          | [`xsaveopt`] --- 保存处理器优化扩展状态
`xsaves`    |          | [`xsaves`] --- 保存处理器主管扩展状态

<!-- Keep links near each table to make it easier to move and update. -->

[ADX]: https://en.wikipedia.org/wiki/Intel_ADX
[AES]: https://en.wikipedia.org/wiki/AES_instruction_set
[AVX]: https://en.wikipedia.org/wiki/Advanced_Vector_Extensions
[AVX2]: https://en.wikipedia.org/wiki/Advanced_Vector_Extensions#AVX2
[AVX512-BF16]: https://en.wikipedia.org/wiki/AVX-512#BF16
[AVX512-BITALG]: https://en.wikipedia.org/wiki/AVX-512#VPOPCNTDQ_and_BITALG
[AVX512-BW]: https://en.wikipedia.org/wiki/AVX-512#BW,_DQ_and_VBMI
[AVX512-CD]: https://en.wikipedia.org/wiki/AVX-512#Conflict_detection
[AVX512-DQ]: https://en.wikipedia.org/wiki/AVX-512#BW,_DQ_and_VBMI
[AVX512-F]: https://en.wikipedia.org/wiki/AVX-512
[AVX512-FP16]: https://en.wikipedia.org/wiki/AVX-512#FP16
[AVX512-IFMA]: https://en.wikipedia.org/wiki/AVX-512#IFMA
[AVX512-VBMI]: https://en.wikipedia.org/wiki/AVX-512#BW,_DQ_and_VBMI
[AVX512-VBMI2]: https://en.wikipedia.org/wiki/AVX-512#VBMI2
[AVX512-VL]: https://en.wikipedia.org/wiki/AVX-512
[AVX512-VNNI]: https://en.wikipedia.org/wiki/AVX-512#VNNI
[AVX512-VP2INTERSECT]: https://en.wikipedia.org/wiki/AVX-512#VP2INTERSECT
[AVX512-VPOPCNTDQ]:https://en.wikipedia.org/wiki/AVX-512#VPOPCNTDQ_and_BITALG
[AVX-IFMA]: https://en.wikipedia.org/wiki/Advanced_Vector_Extensions#AVX-VNNI,_AVX-IFMA
[AVX-NE-CONVERT]: https://en.wikipedia.org/wiki/Advanced_Vector_Extensions#AVX-VNNI,_AVX-IFMA
[AVX-VNNI]: https://en.wikipedia.org/wiki/Advanced_Vector_Extensions#AVX-VNNI,_AVX-IFMA
[AVX-VNNI-INT16]: https://en.wikipedia.org/wiki/Advanced_Vector_Extensions#AVX-VNNI,_AVX-IFMA
[AVX-VNNI-INT8]: https://en.wikipedia.org/wiki/Advanced_Vector_Extensions#AVX-VNNI,_AVX-IFMA
[BMI1]: https://en.wikipedia.org/wiki/Bit_Manipulation_Instruction_Sets
[BMI2]: https://en.wikipedia.org/wiki/Bit_Manipulation_Instruction_Sets#BMI2
[`cmpxchg16b`]: https://www.felixcloutier.com/x86/cmpxchg8b:cmpxchg16b
[F16C]: https://en.wikipedia.org/wiki/F16C
[FMA3]: https://en.wikipedia.org/wiki/FMA_instruction_set
[`fxsave`]: https://www.felixcloutier.com/x86/fxsave
[`fxrstor`]: https://www.felixcloutier.com/x86/fxrstor
[GFNI]: https://en.wikipedia.org/wiki/AVX-512#GFNI
[KEYLOCKER]: https://en.wikipedia.org/wiki/List_of_x86_cryptographic_instructions#Intel_Key_Locker_instructions
[KEYLOCKER_WIDE]: https://en.wikipedia.org/wiki/List_of_x86_cryptographic_instructions#Intel_Key_Locker_instructions
[`lzcnt`]: https://www.felixcloutier.com/x86/lzcnt
[`movbe`]: https://www.felixcloutier.com/x86/movbe
[`pclmulqdq`]: https://www.felixcloutier.com/x86/pclmulqdq
[`popcnt`]: https://www.felixcloutier.com/x86/popcnt
[`rdrand`]: https://en.wikipedia.org/wiki/RdRand
[`rdseed`]: https://en.wikipedia.org/wiki/RdRand
[SHA]: https://en.wikipedia.org/wiki/Intel_SHA_extensions
[SHA512]: https://en.wikipedia.org/wiki/Intel_SHA_extensions
[SM3]: https://en.wikipedia.org/wiki/List_of_x86_cryptographic_instructions#Intel_SHA_and_SM3_instructions
[SM4]: https://en.wikipedia.org/wiki/List_of_x86_cryptographic_instructions#Intel_SHA_and_SM3_instructions
[SSE]: https://en.wikipedia.org/wiki/Streaming_SIMD_Extensions
[SSE2]: https://en.wikipedia.org/wiki/SSE2
[SSE3]: https://en.wikipedia.org/wiki/SSE3
[SSE4.1]: https://en.wikipedia.org/wiki/SSE4#SSE4.1
[SSE4.2]: https://en.wikipedia.org/wiki/SSE4#SSE4.2
[SSE4a]: https://en.wikipedia.org/wiki/SSE4#SSE4a
[SSSE3]: https://en.wikipedia.org/wiki/SSSE3
[TBM]: https://en.wikipedia.org/wiki/X86_Bit_manipulation_instruction_set#TBM_(Trailing_Bit_Manipulation)
[VAES]: https://en.wikipedia.org/wiki/AVX-512#VAES
[VPCLMULQDQ]: https://en.wikipedia.org/wiki/AVX-512#VPCLMULQDQ
[`xsave`]: https://www.felixcloutier.com/x86/xsave
[`xsavec`]: https://www.felixcloutier.com/x86/xsavec
[`xsaveopt`]: https://www.felixcloutier.com/x86/xsaveopt
[`xsaves`]: https://www.felixcloutier.com/x86/xsaves

r[attributes.codegen.target_feature.aarch64]
#### `aarch64`

在此平台上，`#[target_feature]` 函数的使用遵循 [上述限制][attributes.codegen.target_feature.safety-restrictions]。

有关这些 特性 的更多文档可以在 [ARM Architecture Reference Manual] 或 [developer.arm.com] 上的其他位置找到。

[ARM Architecture Reference Manual]: https://developer.arm.com/documentation/ddi0487/latest
[developer.arm.com]: https://developer.arm.com

> [!NOTE]
> 如果使用以下 特性 对，应同时将其标记为启用或禁用：
> - `paca` 和 `pacg`，LLVM 目前将其实现为一个 特性。

特性        | 隐式启用           | 特性 名称
-------        | ------------------ | ------------
`aes`          | `neon`             | FEAT_AES & FEAT_PMULL --- 高级 <abbr title="Single Instruction Multiple Data">SIMD</abbr> AES 和 PMULL 指令
`bf16`         |                    | FEAT_BF16 --- BFloat16 指令
`bti`          |                    | FEAT_BTI --- 分支目标识别
`crc`          |                    | FEAT_CRC --- CRC32 校验和指令
`dit`          |                    | FEAT_DIT  --- 数据无关时序指令
`dotprod`      | `neon`             | FEAT_DotProd --- 高级 SIMD Int8 点积指令
`dpb`          |                    | FEAT_DPB --- 数据缓存清理到持久点
`dpb2`         | `dpb`              | FEAT_DPB2 --- 数据缓存清理到深度持久点
`f32mm`        | `sve`              | FEAT_F32MM --- SVE 单精度浮点矩阵乘法指令
`f64mm`        | `sve`              | FEAT_F64MM --- SVE 双精度浮点矩阵乘法指令
`fcma`         | `neon`             | FEAT_FCMA --- 浮点复数支持
`fhm`          | `fp16`             | FEAT_FHM --- 半精度浮点 FMLAL 指令
`flagm`        |                    | FEAT_FLAGM --- 条件标志操作
`fp16`         | `neon`             | FEAT_FP16 --- 半精度浮点数据处理
`frintts`      |                    | FEAT_FRINTTS --- 浮点到整数辅助指令
`i8mm`         |                    | FEAT_I8MM --- Int8 矩阵乘法
`jsconv`       | `neon`             | FEAT_JSCVT --- JavaScript 转换指令
`lor`          |                    | FEAT_LOR --- 有限排序区域扩展
`lse`          |                    | FEAT_LSE --- 大型系统扩展
`mte`          |                    | FEAT_MTE & FEAT_MTE2 --- 内存标记扩展
`neon`         |                    | FEAT_AdvSimd & FEAT_FP --- 浮点和高级 SIMD 扩展
`paca`         |                    | FEAT_PAUTH --- 指针认证（地址认证）
`pacg`         |                    | FEAT_PAUTH --- 指针认证（通用认证）
`pan`          |                    | FEAT_PAN --- 特权访问永不扩展
`pmuv3`        |                    | FEAT_PMUv3 --- 性能监视器扩展 (v3)
`rand`         |                    | FEAT_RNG --- 随机数生成器
`ras`          |                    | FEAT_RAS & FEAT_RASv1p1 --- 可靠性、可用性和可服务性扩展
`rcpc`         |                    | FEAT_LRCPC --- 释放一致处理器一致
`rcpc2`        | `rcpc`             | FEAT_LRCPC2 --- 带立即偏移的 RcPc
`rdm`          | `neon`             | FEAT_RDM --- 舍入双精度乘累加
`sb`           |                    | FEAT_SB --- 推测屏障
`sha2`         | `neon`             | FEAT_SHA1 & FEAT_SHA256 --- 高级 SIMD SHA 指令
`sha3`         | `sha2`             | FEAT_SHA512 & FEAT_SHA3 --- 高级 SIMD SHA 指令
`sm4`          | `neon`             | FEAT_SM3 & FEAT_SM4 --- 高级 SIMD SM3/4 指令
`spe`          |                    | FEAT_SPE --- 统计分析扩展
`ssbs`         |                    | FEAT_SSBS & FEAT_SSBS2 --- 推测存储旁路安全
`sve`          | `neon`             | FEAT_SVE --- 可伸缩向量扩展
`sve2`         | `sve`              | FEAT_SVE2 --- 可伸缩向量扩展 2
`sve2-aes`     | `sve2`, `aes`      | FEAT_SVE_AES & FEAT_SVE_PMULL128 --- SVE AES 指令
`sve2-bitperm` | `sve2`             | FEAT_SVE2_BitPerm --- SVE 位置换
`sve2-sha3`    | `sve2`, `sha3`     | FEAT_SVE2_SHA3 --- SVE SHA3 指令
`sve2-sm4`     | `sve2`, `sm4`      | FEAT_SVE2_SM4 --- SVE SM4 指令
`tme`          |                    | FEAT_TME --- 事务性内存扩展
`vh`           |                    | FEAT_VHE --- 虚拟化主机扩展

r[attributes.codegen.target_feature.loongarch]
#### `loongarch`

在此平台上，`#[target_feature]` 函数的使用遵循 [上述限制][attributes.codegen.target_feature.safety-restrictions]。

特性     | 隐式启用            | 描述
------------|---------------------|-------------------
`f`         |                     | [F][la-f] --- 单精度浮点指令
`d`         | `f`                 | [D][la-d] --- 双精度浮点指令
`frecipe`   |                     | [FRECIPE][la-frecipe] --- 倒数近似指令
`lasx`      | `lsx`               | [LASX][la-lasx] --- 256 位向量指令
`lbt`       |                     | [LBT][la-lbt] --- 二进制翻译指令
`lsx`       | `d`                 | [LSX][la-lsx] --- 128 位向量指令
`lvz`       |                     | [LVZ][la-lvz] --- 虚拟化指令

<!-- Keep links near each table to make it easier to move and update. -->

[la-f]: https://loongson.github.io/LoongArch-Documentation/LoongArch-Vol1-EN.html#cpucfg-fp_sp
[la-d]: https://loongson.github.io/LoongArch-Documentation/LoongArch-Vol1-EN.html#cpucfg-fp_dp
[la-frecipe]: https://loongson.github.io/LoongArch-Documentation/LoongArch-Vol1-EN.html#cpucfg-frecipe
[la-lasx]: https://loongson.github.io/LoongArch-Documentation/LoongArch-Vol1-EN.html#cpucfg-lasx
[la-lbt]: https://loongson.github.io/LoongArch-Documentation/LoongArch-Vol1-EN.html#cpucfg-lbt_x86
[la-lsx]: https://loongson.github.io/LoongArch-Documentation/LoongArch-Vol1-EN.html#cpucfg-lsx
[la-lvz]: https://loongson.github.io/LoongArch-Documentation/LoongArch-Vol1-EN.html#cpucfg-lvz

r[attributes.codegen.target_feature.riscv]
#### `riscv32`或`riscv64`

在此平台上，`#[target_feature]` 函数的使用遵循 [上述限制][attributes.codegen.target_feature.safety-restrictions]。

有关这些 特性 的更多文档可以在它们各自的规范中找到。许多规范在 [RISC-V ISA Manual]，[version 20250508]，或 [RISC-V GitHub Account] 上的另一本手册中描述。

[RISC-V ISA Manual]: https://github.com/riscv/riscv-isa-manual
[version 20250508]: https://github.com/riscv/riscv-isa-manual/tree/20250508
[RISC-V GitHub Account]: https://github.com/riscv

特性     | 隐式启用                        | 描述
------------|---------------------------------|-------------------
`a`         |                                 | [A][rv-a] --- 原子指令
`c`         |                                 | [C][rv-c] --- 压缩指令
`m`         |                                 | [M][rv-m] --- 整数乘法和除法指令
`zba`       |                                 | [Zba][rv-zba] --- 地址生成指令
`zbb`       |                                 | [Zbb][rv-zbb] --- 基本位操作
`zbc`       | `zbkc`                          | [Zbc][rv-zbc] --- 无进位乘法
`zbkb`      |                                 | [Zbkb][rv-zbkb] --- 密码学位操作指令
`zbkc`      |                                 | [Zbkc][rv-zbkc] --- 密码学无进位乘法
`zbkx`      |                                 | [Zbkx][rv-zbkx] --- 交叉置换
`zbs`       |                                 | [Zbs][rv-zbs] --- 单比特指令
`zk`        | `zkn`, `zkr`, `zks`, `zkt`, `zbkb`, `zbkc`, `zkbx` | [Zk][rv-zk] --- 标量密码学
`zkn`       | `zknd`, `zkne`, `zknh`, `zbkb`, `zbkc`, `zkbx`     | [Zkn][rv-zkn] --- NIST 算法套件扩展
`zknd`      |                                                    | [Zknd][rv-zknd] --- NIST 套件: AES 解密
`zkne`      |                                                    | [Zkne][rv-zkne] --- NIST 套件: AES 加密
`zknh`      |                                                    | [Zknh][rv-zknh] --- NIST 套件: 哈希函数指令
`zkr`       |                                                    | [Zkr][rv-zkr] --- 熵源扩展
`zks`       | `zksed`, `zksh`, `zbkb`, `zbkc`, `zkbx`            | [Zks][rv-zks] --- 商密算法套件
`zksed`     |                                                    | [Zksed][rv-zksed] --- 商密套件: SM4 分组密码指令
`zksh`      |                                                    | [Zksh][rv-zksh] --- 商密套件: SM3 哈希函数指令
`zkt`       |                                                    | [Zkt][rv-zkt] --- 数据无关执行延迟子集

<!-- Keep links near each table to make it easier to move and update. -->

[rv-a]: https://github.com/riscv/riscv-isa-manual/blob/20250508/src/a-st-ext.adoc
[rv-c]: https://github.com/riscv/riscv-isa-manual/blob/20250508/src/c-st-ext.adoc
[rv-m]: https://github.com/riscv/riscv-isa-manual/blob/20250508/src/m-st-ext.adoc
[rv-zba]: https://github.com/riscv/riscv-isa-manual/blob/20250508/src/b-st-ext.adoc
[rv-zbb]: https://github.com/riscv/riscv-isa-manual/blob/20250508/src/b-st-ext.adoc
[rv-zbc]: https://github.com/riscv/riscv-isa-manual/blob/20250508/src/b-st-ext.adoc
[rv-zbkb]: https://github.com/riscv/riscv-isa-manual/blob/20250508/src/b-st-ext.adoc
[rv-zbkc]: https://github.com/riscv/riscv-isa-manual/blob/20250508/src/b-st-ext.adoc
[rv-zbkx]: https://github.com/riscv/riscv-isa-manual/blob/20250508/src/b-st-ext.adoc
[rv-zbs]: https://github.com/riscv/riscv-isa-manual/blob/20250508/src/b-st-ext.adoc
[rv-zk]: https://github.com/riscv/riscv-isa-manual/blob/20250508/src/scalar-crypto.adoc
[rv-zkn]: https://github.com/riscv/riscv-isa-manual/blob/20250508/src/scalar-crypto.adoc
[rv-zkne]: https://github.com/riscv/riscv-isa-manual/blob/20250508/src/scalar-crypto.adoc
[rv-zknd]: https://github.com/riscv/riscv-isa-manual/blob/20250508/src/scalar-crypto.adoc
[rv-zknh]: https://github.com/riscv/riscv-isa-manual/blob/20250508/src/scalar-crypto.adoc
[rv-zkr]: https://github.com/riscv/riscv-isa-manual/blob/20250508/src/scalar-crypto.adoc
[rv-zks]: https://github.com/riscv/riscv-isa-manual/blob/20250508/src/scalar-crypto.adoc
[rv-zksed]: https://github.com/riscv/riscv-isa-manual/blob/20250508/src/scalar-crypto.adoc
[rv-zksh]: https://github.com/riscv/riscv-isa-manual/blob/20250508/src/scalar-crypto.adoc
[rv-zkt]: https://github.com/riscv/riscv-isa-manual/blob/20250508/src/scalar-crypto.adoc

r[attributes.codegen.target_feature.wasm]
#### `wasm32`或`wasm64`

安全的 `#[target_feature]` 函数始终可以在 Wasm 平台上在安全上下文中使用。通过 `#[target_feature]` 属性 造成未定义行为是不可能的，因为尝试使用 Wasm 引擎不支持的指令会在加载时失败，而不会有被以编译器预期之外的方式解释的风险。

特性               | 隐式启用            | 描述
----------------------|---------------------|-------------------
`bulk-memory`         |                     | [WebAssembly bulk memory operations proposal][bulk-memory]
`extended-const`      |                     | [WebAssembly extended const expressions proposal][extended-const]
`mutable-globals`     |                     | [WebAssembly mutable global proposal][mutable-globals]
`nontrapping-fptoint` |                     | [WebAssembly non-trapping float-to-int conversion proposal][nontrapping-fptoint]
`relaxed-simd`        | `simd128`           | [WebAssembly relaxed simd proposal][relaxed-simd]
`sign-ext`            |                     | [WebAssembly sign extension operators Proposal][sign-ext]
`simd128`             |                     | [WebAssembly simd proposal][simd128]
`multivalue`          |                     | [WebAssembly multivalue proposal][multivalue]
`reference-types`     |                     | [WebAssembly reference-types proposal][reference-types]
`tail-call`           |                     | [WebAssembly tail-call proposal][tail-call]

[bulk-memory]: https://github.com/WebAssembly/bulk-memory-operations
[extended-const]: https://github.com/WebAssembly/extended-const
[mutable-globals]: https://github.com/WebAssembly/mutable-global
[nontrapping-fptoint]: https://github.com/WebAssembly/nontrapping-float-to-int-conversions
[relaxed-simd]: https://github.com/WebAssembly/relaxed-simd
[sign-ext]: https://github.com/WebAssembly/sign-extension-ops
[simd128]: https://github.com/webassembly/simd
[reference-types]: https://github.com/webassembly/reference-types
[tail-call]: https://github.com/webassembly/tail-call
[multivalue]: https://github.com/webassembly/multi-value

r[attributes.codegen.target_feature.s390x]
#### `s390x`

在 `s390x` 目标上，使用带有 `#[target_feature]` 属性 的函数遵循 [上述限制][attributes.codegen.target_feature.safety-restrictions]。

有关这些 特性 的更多文档可以在 *[z/Architecture Principles of Operation]* 的第 1 章“对 z/Architecture 的补充”部分中找到。

特性                                | 隐式启用                          | 描述
---------------------------------------|---------------------------------------|---------------------
`vector`                               |                                       | 128 位向量指令
`vector-enhancements-1`                | `vector`                              | 向量增强 1
`vector-enhancements-2`                | `vector-enhancements-1`               | 向量增强 2
`vector-enhancements-3`                | `vector-enhancements-2`               | 向量增强 3
`vector-packed-decimal`                | `vector`                              | 向量压缩十进制
`vector-packed-decimal-enhancement`    | `vector-packed-decimal`               | 向量压缩十进制增强
`vector-packed-decimal-enhancement-2`  | `vector-packed-decimal-enhancement-2` | 向量压缩十进制增强 2
`vector-packed-decimal-enhancement-3`  | `vector-packed-decimal-enhancement-3` | 向量压缩十进制增强 3
`nnp-assist`                           | `vector`                              | NNP 辅助
`miscellaneous-extensions-2`           |                                       | 杂项扩展 2
`miscellaneous-extensions-3`           |                                       | 杂项扩展 3
`miscellaneous-extensions-4`           |                                       | 杂项扩展 4

[z/Architecture Principles of Operation]: https://publibfp.dhe.ibm.com/epubs/pdf/a227832d.pdf

r[attributes.codegen.target_feature.info]
### 附加信息

r[attributes.codegen.target_feature.remark-cfg]
有关根据编译时设置有选择地启用或禁用代码编译，请参阅 [`target_feature` conditional compilation option]。请注意，此选项不受 `target_feature` 属性 的影响，仅由整个 crate 启用的 特性 驱动。

r[attributes.codegen.target_feature.remark-rt]
可以在运行时使用标准库中平台特定的宏检查 特性 是否启用，例如 [`is_x86_feature_detected`] 或 [`is_aarch64_feature_detected`]。

> [!NOTE]
> `rustc` 为每个目标和 CPU 启用了一组默认 特性。可以使用 [`-C target-cpu`] 标志选择 CPU。可以使用 [`-C target-feature`] 标志为整个 crate 启用或禁用单个 特性。

r[attributes.codegen.track_caller]
## `track_caller`属性

r[attributes.codegen.track_caller.allowed-positions]
`track_caller` 属性可以应用于任何具有 [`"Rust"` ABI][rust-abi] 的函数，但入口点 `fn main` 除外。

r[attributes.codegen.track_caller.traits]
当应用于 特型 声明中的函数和方法时，该属性适用于所有实现。如果 特型 提供带有该属性 的默认实现，则该属性也适用于覆盖实现。

r[attributes.codegen.track_caller.extern]
当应用于 `extern` 块中的函数时，该属性也必须应用于任何链接的实现，否则会导致未定义行为。当应用于可用于 `extern` 块的函数时，`extern` 块中的声明也必须具有该属性，否则会导致未定义行为。

r[attributes.codegen.track_caller.behavior]
### 行为

将该属性应用于函数 `f` 允许 `f` 内的代码获取导致 `f` 被调用的“最顶层”跟踪调用的 [`Location`] 提示。在观察点，实现的行为就像它从 `f` 的栈帧向上查找最近的 *未修饰* 函数 `outer` 的栈帧，并返回 `outer` 中跟踪调用的 [`Location`]。

```rust
#[track_caller]
fn f() {
    println!("{}", std::panic::Location::caller());
}
```

> [!NOTE]
> `core` 提供 [`core::panic::Location::caller`] 用于观察调用者位置。它包装了 `rustc` 实现的 [`core::intrinsics::caller_location`] 内部函数。

> [!NOTE]
> 因为生成的 `Location` 是一个提示，实现可能会提前停止其栈向上查找。有关重要的注意事项，请参阅 [限制](#limitations)。

#### 示例

当 `f` 被 `calls_f` 直接调用时，`f` 中的代码观察到其在 `calls_f` 中的调用点：

```rust
# #[track_caller]
# fn f() {
#     println!("{}", std::panic::Location::caller());
# }
fn calls_f() {
    f(); // <-- f() prints this location
}
```

当 `f` 被另一个带有属性 的函数 `g` 调用，而 `g` 又被 `calls_g` 调用时，`f` 和 `g` 中的代码都观察到 `g` 在 `calls_g` 中的调用点：

```rust
# #[track_caller]
# fn f() {
#     println!("{}", std::panic::Location::caller());
# }
#[track_caller]
fn g() {
    println!("{}", std::panic::Location::caller());
    f();
}

fn calls_g() {
    g(); // <-- g() prints this location twice, once itself and once from f()
}
```

当 `g` 被另一个带有属性 的函数 `h` 调用，而 `h` 又被 `calls_h` 调用时，`f`、`g` 和 `h` 中的所有代码都观察到 `h` 在 `calls_h` 中的调用点：

```rust
# #[track_caller]
# fn f() {
#     println!("{}", std::panic::Location::caller());
# }
# #[track_caller]
# fn g() {
#     println!("{}", std::panic::Location::caller());
#     f();
# }
#[track_caller]
fn h() {
    println!("{}", std::panic::Location::caller());
    g();
}

fn calls_h() {
    h(); // <-- prints this location three times, once itself, once from g(), once from f()
}
```

依此类推。

r[attributes.codegen.track_caller.limits]
### 限制

r[attributes.codegen.track_caller.hint]
此信息是一个提示，不要求实现保留它。

r[attributes.codegen.track_caller.decay]
特别是，将带有 `#[track_caller]` 的函数强制转换为函数指针会创建一个 shim，该 shim 在观察者看来是在所修饰函数的定义点被调用，从而在虚拟调用中丢失实际的调用者信息。这种强制转换的一个常见示例是创建其方法带有属性 的 特型 对象。

> [!NOTE]
> 上述函数指针的 shim 是必要的，因为 `rustc` 在代码生成上下文中通过向函数 ABI 附加一个隐式参数来实现 `track_caller`，但对于间接调用来说，这会是不健全的，因为该参数不是函数类型的一部分，并且给定的函数指针类型可能引用也可能不引用带有该属性 的函数。创建 shim 隐藏了函数指针调用者中的隐式参数，从而保持了健全性。

<!-- template:attributes -->
r[attributes.codegen.instruction_set]
## `instruction_set`属性

r[attributes.codegen.instruction_set.intro]
*`instruction_set` [属性][attribute]* 指定函数在代码生成过程中将使用的指令集。这允许在单个程序中混合使用多个指令集。

> [!EXAMPLE]
> <!-- ignore: arm-only -->
> ```rust,ignore
> #[instruction_set(arm::a32)]
> fn arm_code() {}
>
> #[instruction_set(arm::t32)]
> fn thumb_code() {}
> ```

r[attributes.codegen.instruction_set.syntax]
`instruction_set` 属性使用 [MetaListPaths] 语法格式来指定一个由架构家族名称和指令集名称组成的单一路径。

r[attributes.codegen.instruction_set.allowed-positions]
`instruction_set` 属性只能应用于带有 [函数体][bodies] 的函数——[闭包][closures]、[异步块][async blocks]、[自由函数][free functions]、[固有实现][inherent impl] 或 [特型实现][trait impl] 中的 [关联函数][associated functions]，以及 [特型定义][trait definition] 中带有 [默认定义][default definition] 的关联函数。

> [!NOTE]
> `rustc` 忽略在其他位置的使用，但会对其进行 lint 检查。这在将来可能会成为错误。

> [!NOTE]
> 尽管该属性可以应用于 [闭包][closures] 和 [异步块][async blocks]，但其用处有限，因为我们尚不支持表达式上的属性。

r[attributes.codegen.instruction_set.duplicates]
`instruction_set` 属性在函数上只能使用一次。

r[attributes.codegen.instruction_set.target-limits]
`instruction_set` 属性只能与支持给定值的目标一起使用。

r[attributes.codegen.instruction_set.inline-asm]
当使用 `instruction_set` 属性时，函数中的任何内联汇编都必须使用指定的指令集而不是目标默认指令集。

r[attributes.codegen.instruction_set.arm]
### ARM上的`instruction_set`

当目标为 `ARMv4T` 和 `ARMv5te` 架构时，`instruction_set` 支持的值有：

- `arm::a32` --- 将函数生成为 A32 "ARM" 代码。
- `arm::t32` --- 将函数生成为 T32 "Thumb" 代码。

如果将函数的地址作为函数指针，则地址的低位将取决于所选的指令集：

- 对于 `arm::a32` ("ARM")，它将为 0。
- 对于 `arm::t32` ("Thumb")，它将为 1。

[`-C target-cpu`]: https://doc.rust-lang.org/rustc/codegen-options/index.html#target-cpu
[`-C target-feature`]: https://doc.rust-lang.org/rustc/codegen-options/index.html#target-feature
[`export_name`]: abi.export_name
[`is_aarch64_feature_detected`]: https://doc.rust-lang.org/std/arch/macro.is_aarch64_feature_detected.html
[`is_x86_feature_detected`]: https://doc.rust-lang.org/std/arch/macro.is_x86_feature_detected.html
[`Location`]: core::panic::Location
[`naked_asm!`]: ../inline-assembly.md
[`no_mangle`]: abi.no_mangle
[`target_feature` conditional compilation option]: ../conditional-compilation.md#target_feature
[`unused_variables`]: https://doc.rust-lang.org/rustc/lints/listing/warn-by-default.html#unused-variables
[associated functions]: items.associated.fn
[async blocks]: expr.block.async
[async closure]: expr.closure.async
[async function]: items.fn.async
[attribute]: ../attributes.md
[attributes]: ../attributes.md
[bodies]: items.fn.body
[closures]: expr.closure
[default definition]: items.traits.associated-item-decls
[free functions]: items.fn
[function body]: ../items/functions.md#function-body
[functions]: ../items/functions.md
[inherent impl]: items.impl.inherent
[rust-abi]: ../items/external-blocks.md#abi
[target architecture]: ../conditional-compilation.md#target_arch
[trait]: items.traits
[trait definition]: items.traits
[trait impl]: items.impl.trait
[undefined behavior]: ../behavior-considered-undefined.md
[unsafe attribute]: ../attributes.md#r-attributes.safety
