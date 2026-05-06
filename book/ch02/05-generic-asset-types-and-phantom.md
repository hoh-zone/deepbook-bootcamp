# ch02-05 泛型资产类型与 phantom

[返回本章](README.md)

## 先建立手感

这一节用“泛型资产类型与 phantom”训练 Move 手感：先看对象和资源能不能被复制、丢弃、转移，再回到 DeepBook 里判断为什么这些限制有实际价值。

## Move 对照

先用下面几处源码建立 Move 概念的落点。这里不追完整协议流程，只确认类型、ability、对象、事件和测试入口如何在真实项目中出现。

- [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)
- [packages/deepbook/sources/balance_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/balance_manager.move)
- [packages/deepbook/sources/registry.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/registry.move)
- [packages/deepbook/sources/state/balances.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/state/balances.move)

读这些文件时，把语法点和真实对象放在一起看：ability、泛型、对象 ID、事件和测试入口分别承担什么约束。

## 拆开来看

DeepBook 使用泛型资产类型表达交易对。`pool.move` 中 `Pool<phantom BaseAsset, phantom QuoteAsset>` 和 `PoolInner<phantom BaseAsset, phantom QuoteAsset>` 通过类型参数区分交易池，不需要在结构体字段中实际保存 `BaseAsset` 或 `QuoteAsset` 值。

`phantom` 表示类型参数只参与类型标识，不参与字段存储约束。这样 `Pool<SUI, USDC>` 和 `Pool<DEEP, SUI>` 是不同类型，编译器能帮助防止把错误资产传进错误池子。

实践中，TypeScript 构造 move call 时也要写完整泛型类型参数。泛型错了，即使对象 ID 看起来正确，也会在类型检查或执行阶段失败。

## 阅读补充

DeepBook 用泛型类型表示资产对，而不是把资产地址当普通字符串字段。`Pool<BaseAsset, QuoteAsset>` 的类型本身就是市场身份的一部分，调用 move function 时漏写或写错 type arguments，会让交易指向完全不同的类型空间。

`phantom` 适合表达“类型参与静态区分，但不以值的形式存储”。阅读资产相关 struct 时，注意它是否真正持有 `Coin<T>`/`Balance<T>`，还是只用类型参数约束市场或权限。

## Move 判断

- TypeScript move call 必须显式核对 base/quote type arguments 顺序。
- 创建池或查询池时不要只看对象 ID，也要看对象类型里的泛型。
- 看到 `phantom` 时检查它是否用于类型隔离而不是运行时字段。

## 练习问题

- 为什么 SUI/USDC 和 USDC/SUI 不是同一个泛型池？
- 漏写 type arguments 会导致哪类问题？
- `phantom` 资产参数和真实 `Balance<T>` 持有有什么区别？
