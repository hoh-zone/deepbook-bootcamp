# ch01 DeepBook 与链上订单簿入门

## 本章目标

读完本章后，你应该能够：

- 说清 DeepBookV3 是 Sui 上的中央限价订单簿，交易路径不同于 AMM。
- 在本地找到 DeepBookV3、DeepBook Margin、DeepBook Predict、Indexer、Server 的源码边界。
- 解释一次下单涉及的钱包、交易、`Pool`、`BalanceManager`、事件和 Indexer。
- 完成开发环境检查，并查询一个 DeepBook 池子对象的基础字段。

## 本章学习阶梯

- L1 建立 CLOB、AMM、订单、池子和角色直觉。
- L2 完成环境检查和第一个池子对象查询。
- L3 把查询结果映射到 Sui object、type、shared owner 和网络配置。
- L4 能用自己的话画出一次下单从钱包到事件的路径。

## 源码地图

本章只读取项目入口和产品边界，不进入撮合细节。

| 主题 | GitHub 源码 | 阅读重点 |
| --- | --- | --- |
| 仓库入口 | [README.md](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/README.md) | DeepBookV3 的 CLOB 定位、BalanceManager、Pool、DEEP 费用和治理 |
| DeepBook Move 包 | [packages/deepbook/README.md](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/README.md) | 和仓库入口一致，是合约包级别说明 |
| DeepBookV3 合约 | [packages/deepbook](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook) | Spot CLOB、BalanceManager、Pool、Book、State、Vault |
| DeepBook Margin | [packages/deepbook_margin](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin) | 保证金和普通借贷，不等同于 DeepBookV3 闪电贷 |
| DeepBook Predict | [packages/predict](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/predict) | 预测市场相关包，版本和网络状态以后按源码核对 |
| Indexer | [crates/indexer](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/indexer) | 从链上事件落库 |
| Server | [crates/server](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/server) | 对外查询服务 |
| 运维脚本 | [scripts/transactions](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/scripts/transactions) | 创建池子、升级、参数更新和做市脚本 |

## 小节目录

- [01 本书定位](01-book-positioning.md)
- [02 DeepBook 是 CLOB，不是 AMM](02-deepbook-is-clob-not-amm.md)
- [03 CLOB、AMM、RFQ 和聚合器](03-clob-amm-rfq-aggregator.md)
- [04 产品边界：V3、Margin、Predict](04-product-boundaries-v3-margin-predict.md)
- [05 为什么 Sui 对象模型适合订单簿](05-sui-object-model-for-orderbooks.md)
- [06 核心用户角色](06-core-user-roles.md)
- [07 全局心智模型](07-mental-model.md)
- [08 网络使用方式](08-network-usage.md)
- [09 环境安装](09-environment-setup.md)
- [10 检查 DeepBook 源码仓库结构](10-inspect-source-repo.md)
- [11 本书项目目录规范](11-book-project-structure.md)
- [12 第一个读者任务：查询池子对象](12-first-pool-query.md)
- [13 术语表](13-glossary.md)
- [14 本章代码](14-chapter-code.md)

## Move 高阶穿插点

- 环境搭建不只是安装工具，还要建立“源码、链上对象、SDK、Indexer”四个视角之间的跳转习惯。
- 查询池子对象时，优先记录对象 ID、type、version、owner 和 shared object 状态，这些字段决定后续 PTB 能否正确构造。
- 遇到 DeepBook 术语时，立刻追问它是链上对象、事件字段、SDK 配置，还是纯前端展示概念。

## 常见错误

- 把 DeepBookV3 当成 AMM，只查储备量，不查订单簿、订单和事件。
- 把 [packages/deepbook_margin](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin) 的借贷逻辑误认为 DeepBookV3 spot 交易路径。
- 在 mainnet 对象 ID 上使用 testnet RPC，导致对象不存在。
- 构造交易前没有确认 `BalanceManager` 所属 owner、cap 和交易者关系。
- 只看前端返回值，不订阅或索引 `event::emit` 产生的链上事件。

## 本章检查清单

- [ ] 能指出 DeepBookV3 源码包路径。
- [ ] 能解释 CLOB 和 AMM 的交易路径差异。
- [ ] 能画出钱包到 `Pool`、`BalanceManager`、事件、Indexer 的路径。
- [ ] 能运行环境检查命令。
- [ ] 能查询一个池子对象并确认网络一致。

## 进阶练习

1. 用文字画出一次限价买单从钱包签名到 `OrderPlaced` 事件的路径。
2. 阅读 [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)，列出 `place_limit_order` 的每个参数含义。
3. 阅读 [packages/deepbook/sources/balance_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/balance_manager.move)，解释为什么 owner 和 trader 的权限不同。

