<!-- https://github.com/rust-lang/reference/blob/master/src/influences.md -->
<!-- commit 68bdfd15fa9feebfbf94a06cf574de598e239198 -->

# 影响

Rust 并非一种特别原创的语言，其设计元素来自广泛的来源。其中一些列举如下（包括后来已被移除的元素）：

* SML, OCaml: 代数数据类型，模式匹配，类型推断，分号语句分隔
* C++: 引用，RAII，智能指针，移动语义，单态化，内存模型
* ML Kit, Cyclone: 基于区域的内存管理
* Haskell (GHC): 类型类，类型家族
* Newsqueak, Alef, Limbo: 通道，并发
* Erlang: 消息传递，线程失败，~~关联线程失败~~，~~轻量级并发~~
* Swift: 可选绑定
* Scheme: 卫生宏
* C#: 属性
* Ruby: 闭包 语法格式，~~块 语法格式~~
* NIL, Hermes: ~~类型状态~~
* [Unicode 附录 #31](http://www.unicode.org/reports/tr31/): 标识符 和 模式 语法格式
