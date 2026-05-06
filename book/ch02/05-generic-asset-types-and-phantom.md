# ch02-05 泛型资产类型与 phantom

[返回本章](README.md)

## 本节目标

- 理解 `Pool<BaseAsset, QuoteAsset>` 和资产类型参数如何隔离市场。
- 能沿“泛型资产类型与 phantom”定位相关 Move 源码、脚本或链下服务入口。
- 读完后能够用交易路径、对象职责或失败场景解释本节主题。

## 源码关联

本节重点对照以下源码或后续阅读入口：

- [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)
- [packages/deepbook/sources/balance_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/balance_manager.move)
- [packages/deepbook/sources/registry.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/registry.move)
- [packages/deepbook/sources/state/balances.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/state/balances.move)

阅读时先从标题对应的入口文件开始，确认对象、函数签名和事件名称，再回到本节正文理解它在交易路径中的位置。

## 正文

DeepBook 使用泛型资产类型表达交易对。`pool.move` 中 `Pool<phantom BaseAsset, phantom QuoteAsset>` 和 `PoolInner<phantom BaseAsset, phantom QuoteAsset>` 通过类型参数区分交易池，不需要在结构体字段中实际保存 `BaseAsset` 或 `QuoteAsset` 值。

`phantom` 表示类型参数只参与类型标识，不参与字段存储约束。这样 `Pool<SUI, USDC>` 和 `Pool<DEEP, SUI>` 是不同类型，编译器能帮助防止把错误资产传进错误池子。

实践中，TypeScript 构造 move call 时也要写完整泛型类型参数。泛型错了，即使对象 ID 看起来正确，也会在类型检查或执行阶段失败。

## 阅读补充

DeepBook 用泛型类型表示资产对，而不是把资产地址当普通字符串字段。`Pool<BaseAsset, QuoteAsset>` 的类型本身就是市场身份的一部分，调用 move function 时漏写或写错 type arguments，会让交易指向完全不同的类型空间。

`phantom` 适合表达“类型参与静态区分，但不以值的形式存储”。阅读资产相关 struct 时，注意它是否真正持有 `Coin<T>`/`Balance<T>`，还是只用类型参数约束市场或权限。

## 开发要点

- TypeScript move call 必须显式核对 base/quote type arguments 顺序。
- 创建池或查询池时不要只看对象 ID，也要看对象类型里的泛型。
- 看到 `phantom` 时检查它是否用于类型隔离而不是运行时字段。

## 检查问题

- 为什么 SUI/USDC 和 USDC/SUI 不是同一个泛型池？
- 漏写 type arguments 会导致哪类问题？
- `phantom` 资产参数和真实 `Balance<T>` 持有有什么区别？
