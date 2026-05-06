# ch03-12 DeepBookV3 与 Margin、Predict 的依赖关系

[返回本章](README.md)

## 先抓住结构

先把“DeepBookV3 与 Margin、Predict 的依赖关系”放进 DeepBook 的对象图里。这里不是罗列模块，而是建立阅读顺序：入口在哪里，状态放在哪里，资金最终在哪里结算。

## 源码入口

这一节只保留必要入口，目的不是让你马上读完源码，而是建立后续定位能力：

- [packages/deepbook](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook)
- [packages/deepbook_margin](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin)
- [packages/predict](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/predict)
- [packages/deepbook/sources/vault/vault.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/vault/vault.move)

读源码时先确认对象、函数签名和事件名称；等正文讲到交易路径时，再回到这些入口核对。

## 读架构

DeepBookV3 是现货 CLOB 和池级资产保管层。DeepBook Margin 与 Predict 是独立 package 范围内的上层协议或相邻协议，源码目录分别在 [packages/deepbook_margin](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin) 和 [packages/predict](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/predict)。

本章讨论的 `vault::FlashLoan` 是 DeepBookV3 闪电贷 hot potato，不是 Margin 普通借贷债务。读 Indexer 或事件表时也要区分 DeepBookV3 的 `FlashLoanBorrowed` 与 Margin 的借贷事件。

## 阅读补充

DeepBookV3、Margin、Predict 可以共享生态和资产，但源码包和状态机需要分开阅读。spot 下单路径不应引入 Margin 借贷债务，Predict 的市场结算也不应被解释成订单簿撮合。

判断依赖关系时优先看 `Move.toml`、模块 import、对象类型和事件模块。只有源码中出现明确调用或共享对象关系，才把它写成依赖；否则写成相邻协议或后续章节入口。

## 工程判断

- 文档中提到借贷时明确来自 Margin 还是 V3 flash loan。
- 事件索引表要按 package/module 区分 spot、margin、predict。
- 跨产品组合交易要分别列出每个包的对象和权限。

## 读完以后问自己

- V3 flash loan 和 Margin loan 的状态生命周期有什么不同？
- Predict 包为什么不能用 Pool/Book 流程解释？
- 如何用源码判断两个产品是否存在真实依赖？
