# ch01-04 产品边界：V3、Margin、Predict

[返回本章](README.md)

## 本节目标

- 把 spot CLOB、保证金借贷和预测市场拆成独立源码边界。
- 能沿“产品边界：V3、Margin、Predict”定位相关 Move 源码、脚本或链下服务入口。
- 读完后能够用交易路径、对象职责或失败场景解释本节主题。

## 源码关联

本节重点对照以下源码或后续阅读入口：

- [packages/deepbook](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook)
- [packages/deepbook_margin](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin)
- [packages/predict](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/predict)
- [packages/deepbook/sources/vault/vault.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/vault/vault.move)

阅读时先从标题对应的入口文件开始，确认对象、函数签名和事件名称，再回到本节正文理解它在交易路径中的位置。

## 正文

DeepBookV3 是 spot CLOB。源码包在 [packages/deepbook](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook)，核心对象包括 `Pool`、`BalanceManager`、`Book`、`State`、`Vault`。

DeepBook Margin 在 [packages/deepbook_margin](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin)。它处理保证金相关状态，不要把 Margin 的普通借贷事件写成 DeepBookV3 闪电贷。DeepBookV3 闪电贷入口后续会在 `pool.move` 和 `vault/vault.move` 中单独分析。

DeepBook Predict 在 [packages/predict](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/predict)。本书只在源码明确支持的范围内描述 Predict，涉及迁移状态时以仓库中的 `PREDICT_MIGRATION.md` 和包代码为准。

## 阅读补充

DeepBookV3 的本体是 spot CLOB，`deepbook_margin` 和 `predict` 是相邻产品，不应在第一遍阅读时把状态机合并。尤其是 V3 的闪电贷 hot potato 与 Margin 的借贷仓位不是同一种债务模型，混用术语会直接导致交易路径和风险解释错误。

做源码地图时，先按包目录划边界，再看包之间是否通过对象、cap 或事件产生依赖。没有明确调用关系时，文档应写“相邻产品”或“后续章节入口”，不要推断为同一个协议状态。

## 开发要点

- 涉及普通 spot 交易时优先引用 `packages/deepbook`，不要引用 Margin 仓位逻辑解释成交。
- 提到闪电贷时明确是 `vault::FlashLoan` hot potato，还是 Margin 借贷。
- Predict 相关内容先标为预测市场包入口，等后续章节按源码确认具体流程。

## 检查问题

- DeepBookV3、Margin、Predict 在本地分别对应哪个包？
- V3 闪电贷和 Margin 借贷最容易混淆的地方是什么？
- 如果事件表里同时有交易和借贷事件，应按什么字段或模块区分？
