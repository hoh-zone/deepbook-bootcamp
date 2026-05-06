# ch03-12 DeepBookV3 与 Margin、Predict 的依赖关系

[返回本章](README.md)

## 本节目标

- 区分 DeepBookV3 spot、Margin 借贷和 Predict 市场的源码依赖边界。
- 能沿“DeepBookV3 与 Margin、Predict 的依赖关系”定位相关 Move 源码、脚本或链下服务入口。
- 读完后能够用交易路径、对象职责或失败场景解释本节主题。

## 源码关联

本节重点对照以下源码或后续阅读入口：

- [packages/deepbook](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook)
- [packages/deepbook_margin](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin)
- [packages/predict](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/predict)
- [packages/deepbook/sources/vault/vault.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/vault/vault.move)

阅读时先从标题对应的入口文件开始，确认对象、函数签名和事件名称，再回到本节正文理解它在交易路径中的位置。

## 正文

DeepBookV3 是现货 CLOB 和池级资产保管层。DeepBook Margin 与 Predict 是独立 package 范围内的上层协议或相邻协议，源码目录分别在 [packages/deepbook_margin](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin) 和 [packages/predict](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/predict)。

本章讨论的 `vault::FlashLoan` 是 DeepBookV3 闪电贷 hot potato，不是 Margin 普通借贷债务。读 Indexer 或事件表时也要区分 DeepBookV3 的 `FlashLoanBorrowed` 与 Margin 的借贷事件。

## 阅读补充

DeepBookV3、Margin、Predict 可以共享生态和资产，但源码包和状态机需要分开阅读。spot 下单路径不应引入 Margin 借贷债务，Predict 的市场结算也不应被解释成订单簿撮合。

判断依赖关系时优先看 `Move.toml`、模块 import、对象类型和事件模块。只有源码中出现明确调用或共享对象关系，才把它写成依赖；否则写成相邻协议或后续章节入口。

## 开发要点

- 文档中提到借贷时明确来自 Margin 还是 V3 flash loan。
- 事件索引表要按 package/module 区分 spot、margin、predict。
- 跨产品组合交易要分别列出每个包的对象和权限。

## 检查问题

- V3 flash loan 和 Margin loan 的状态生命周期有什么不同？
- Predict 包为什么不能用 Pool/Book 流程解释？
- 如何用源码判断两个产品是否存在真实依赖？
