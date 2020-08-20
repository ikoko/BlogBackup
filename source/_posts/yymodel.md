title: YYModel
date: 2017-05-12 15:12:34
tags: iOS
---

### YYModel

- Github：[YYModel](https://github.com/ibireme/YYModel)

- 作者： [ibireme](http://blog.ibireme.com/)

- 功能：高性能的 iOS JSON 模型框架，将JSON格式数据自动映射成Model

​<!-- more -->

### YYModel的特性

- **高性能**: 模型转换性能接近手写解析代码。
- **自动类型转换**: 对象类型可以自动转换。
- **类型安全**: 转换过程中，所有的数据类型都会被检测一遍，以保证类型安全，避免崩溃问题。
- **无侵入性**: 使用Category，模型无需继承自其他基类。
- **轻量**: 该框架只有 5 个文件 (包括.h文件)。
- **文档和单元测试**: 文档覆盖率100%, 代码覆盖率99.6%。







### 主流Model库性能对比

|                     | 性能                             | 容错性                      | 功能                       | 侵入性                                  |
| ------------------- | ------------------------------ | ------------------------ | ------------------------ | ------------------------------------ |
| **Mantle**          | E                              | B 会进行对象类型检查              | A 可定制性最高                 | B 需要 Model 继承自某个基类，灵活性稍差，但功能丰富       |
| **JSONModel**       | C 和 MJExtension 差不多，Mantle 性能高 | D 没有对错误类型的检测，没有对 App 的保护 | B 使用比较简单，但功能相对 Mantle 稍弱 | B 需要 Model 继承自某个基类，灵活性稍差，但功能丰富       |
| **MJExtension**     | C 和JSONModel 差不多，Mantle 性能高    | C 对部分对象进行自动转换            | B 使用比较简单，但功能相对 Mantle 稍弱 | A- Category 方式来实现,添加了一些没有前缀的方法，易引起冲突 |
| **FastEasyMapping** | B 较快                           | E 没有自动转换的机制              | C 功能最少，使用也不算方便           | C 采用工具类来实现，使用灵活，但不方便                 |
| **YYModel**         | A 快，接近手写代码的效率                  | A 会进行对象类型检查              | B 使用比较简单，但功能相对 Mantle 稍弱 | A Category 方式来实现                     |







### YYModel性能是如何提升的？

1. 缓存

   Model JSON 转换过程中需要很多类的元数据，如果数据足够小，则全部缓存到内存中。

2. 查表

   当遇到多项选择的条件时，要尽量使用查表法实现，比如 switch/case，C Array，如果查表条件是对象，则可以用 NSDictionary 来实现。

3. 避免 KVC

   Key-Value Coding 使用起来非常方便，但性能上要差于直接调用 Getter/Setter，所以如果能避免 KVC 而用 Getter/Setter 代替，性能会有较大提升。

4. 避免 Getter/Setter 调用

   如果能直接访问 ivar，则尽量使用 ivar 而不要使用 Getter/Setter 这样也能节省一部分开销。

5. 避免多余的内存管理方法

   在 ARC 条件下，默认声明的对象是 __strong 类型的，赋值时有可能会产生 retain/release 调用，如果一个变量在其生命周期内不会被释放，则使用 __unsafe_unretained 会节省很大的开销。

   访问具有 __weak 属性的变量时，实际上会调用 objc_loadWeak() 和 objc_storeWeak() 来完成，这也会带来很大的开销，所以要避免使用 __weak 属性。

   创建和使用对象时，要尽量避免对象进入 autoreleasepool，以避免额外的资源开销。

6. 遍历容器类时，选择更高效的方法

   相对于 Foundation 的方法来说，CoreFoundation 的方法有更高的性能，用 CFArrayApplyFunction() 和 CFDictionaryApplyFunction() 方法来遍历容器类能带来不少性能提升，但代码写起来会非常麻烦。

7. 尽量用纯 C 函数、内联函数

   使用纯 C 函数可以避免 ObjC 的消息发送带来的开销。如果 C 函数比较小，使用 inline 可以避免一部分压栈弹栈等函数调用的开销。

8. 减少遍历的循环次数

   在 JSON 和 Model 转换前，Model 的属性个数和 JSON 的属性个数都是已知的，这时选择数量较少的那一方进行遍历，会节省很多时间。