# ch01-04 产品边界：V3、Margin、Predict

[返回本章](README.md)

## 先看问题

先把“产品边界：V3、Margin、Predict”放到读者路径里看：你不是在背一个协议名，而是在建立一套判断 DeepBook 能做什么、不能做什么的坐标。读这一节时，重点看产品边界如何落到对象、交易和数据系统上。

## 源码入口

本节先用官方文档定边界，再对照源码包。官方文档给出产品定位和当前集成状态，源码包解释这些定位如何落到对象和函数。

- [Sui Docs: DeepBookV3](https://docs.sui.io/onchain-finance/deepbookv3/deepbook)
- [Sui Docs: DeepBook Margin](https://docs.sui.io/onchain-finance/deepbook-margin/)
- [Sui Docs: DeepBook Predict](https://docs.sui.io/onchain-finance/deepbook-predict/)
- [packages/deepbook](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook)
- [packages/deepbook_margin](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin)
- [packages/predict](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/predict)
- [packages/deepbook/sources/vault/vault.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/vault/vault.move)

读源码时先确认对象、函数签名和事件名称；等正文讲到交易路径时，再回到这些入口核对。

## 建立直觉

DeepBookV3 是 spot CLOB。官方文档还强调它本身不是终端用户交易界面，而是供 DEX、钱包、做市系统和其他应用集成的链上交易基础设施。源码包在 [packages/deepbook](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook)，核心对象包括 `Pool`、`BalanceManager`、`Book`、`State`、`Vault`。

DeepBook Margin 在 [packages/deepbook_margin](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin)。官方定位是杠杆交易扩展，核心是借款、抵押、风险率、利息和清算。不要把 Margin 的普通借贷事件写成 DeepBookV3 闪电贷。DeepBookV3 闪电贷入口后续会在 `pool.move` 和 `vault/vault.move` 中单独分析。

DeepBook Predict 在 [packages/predict](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/predict)。官方当前把它标为 Testnet integration surface，因此本书描述 Predict 时必须持续标注网络状态、public server、package/object ID 和 Mainnet 前可能变化的边界。

## 阅读补充

DeepBookV3 的本体是 spot CLOB，`deepbook_margin` 和 `predict` 是相邻产品，不应在第一遍阅读时把状态机合并。尤其是 V3 的闪电贷 hot potato 与 Margin 的借贷仓位不是同一种债务模型，混用术语会直接导致交易路径和风险解释错误。

做源码地图时，先按包目录划边界，再看包之间是否通过对象、cap 或事件产生依赖。没有明确调用关系时，文档应写“相邻产品”或“后续章节入口”，不要推断为同一个协议状态。

## 落地判断

- 涉及普通 spot 交易时优先引用 `packages/deepbook`，不要引用 Margin 仓位逻辑解释成交。
- 提到闪电贷时明确是 `vault::FlashLoan` hot potato，还是 Margin 借贷。
- Predict 相关内容先标为预测市场包入口，等后续章节按源码确认具体流程。

## 读完以后问自己

- DeepBookV3、Margin、Predict 在本地分别对应哪个包？
- V3 闪电贷和 Margin 借贷最容易混淆的地方是什么？
- 如果事件表里同时有交易和借贷事件，应按什么字段或模块区分？
