# ch00-03 功能地图

[返回本章](README.md)

DeepBook 的功能可以按三层理解：交易核心、扩展产品、数据和开发工具。

交易核心是 DeepBookV3 spot。它包含池创建、限价单、市价单、直接 swap、订单修改、订单取消、批量撤单、settled amount 提取、账户订单查询、level2 order book 查询、费用、staking、治理、rebate、referral、闪电贷和管理操作。核心源码集中在 [packages/deepbook/sources](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources)。

扩展产品主要是 DeepBook Margin 和 DeepBook Predict。Margin 引入 `MarginManager`、借贷池、抵押品、oracle、风险率、清算和 TPSL 条件订单。它不是现货池里的一个参数，而是独立 Move 包，通过 DeepBook 池完成杠杆交易。Predict 则是预测市场方向，包含区间头寸、LP vault、oracle 配置、风险配置、费用储备和结算逻辑。

数据和开发工具包括 TypeScript SDK、脚本、Indexer 和 Server。SDK 负责把复杂对象、类型参数和 PTB 组织成可调用接口；Indexer 负责把链上事件处理成数据库记录；Server 负责在数据库和 RPC 之上提供行情、订单簿、成交、资产、费用、Margin 状态、组合数据等查询接口。

可以把功能地图压缩成下面这张表：

| 层级 | 典型功能 | 主要源码 |
| --- | --- | --- |
| Spot 交易 | 限价单、市价单、swap、撤单、查询、闪电贷 | [pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move) |
| 账户和资金 | 余额、owner、trader、cap、proof | [balance_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/balance_manager.move) |
| 撮合和订单 | bid、ask、order id、fill、order info | [book](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/book) |
| 费用和治理 | DEEP fee、staking、proposal、vote、rebate | [state](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/state) |
| Margin | 借贷、抵押、风险、清算、TPSL | [deepbook_margin](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources) |
| Predict | 预测头寸、LP、oracle、settlement | [predict](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/predict/sources) |
| 数据服务 | 事件处理、REST API、行情和历史数据 | [crates/indexer](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/indexer)、[crates/server](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/server) |

## 怎么使用这张地图

读源码时先问三件事：这个功能是否改变链上资产，是否依赖 `BalanceManager`，是否需要 Indexer 才能高效查询。如果改变链上资产，优先读 Move；如果组织交易，读 SDK 和脚本；如果服务页面和后台，读事件、schema、server route。

## 本节检查

- [ ] 能把 DeepBook 功能分成交易核心、扩展产品、数据工具三层。
- [ ] 能根据一个需求快速定位源码目录。
- [ ] 能解释为什么 Indexer 不是交易核心的一部分。
