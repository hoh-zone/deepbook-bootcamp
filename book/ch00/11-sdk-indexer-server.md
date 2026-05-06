# ch00-11 SDK、Indexer 与 Server

[返回本章](README.md)

DeepBook 开发不是只写 Move。真正的应用通常需要三类链下组件：SDK、Indexer 和 Server。

SDK 负责构造交易。DeepBook 的交易经常需要 shared object、类型参数、`BalanceManager`、cap、clock、registry、pool id、coin、order 参数和 dry run。手写 PTB 很容易传错对象、类型或顺序。SDK 的价值是把这些细节包装成业务动作，让应用代码表达“下限价单”“swap exact quote for base”“创建 MarginManager”“借款并交易”，而不是在每个页面重复拼交易。

官方 DeepBookV3 SDK 文档以 `@mysten/deepbook-v3` 为主入口，强调 constants、coin/pool/manager key 配置和 `DeepBookClient`。本书会在此基础上补齐生产应用需要的部分：钱包签名、对象刷新、dry run、错误解析、网络隔离和服务端交易构造边界。

Indexer 负责处理历史事实。链上事件是 DeepBook 数据的关键来源，包括订单更新、成交、池创建、参数更新、stake、proposal、vote、rebate、闪电贷、Margin 借款、还款、清算、抵押、供应和 Predict 事件。源码中 [crates/indexer/src/handlers](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/indexer/src/handlers) 展示了大量事件处理器。没有 Indexer，前端仍能直接读链上对象，但历史成交、K 线、账户活动和筛选查询会很困难。

官方 Indexer 文档提供 public mainnet/testnet endpoint、资产最小单位转换、pool 信息、historical volume、OHLCV 等 API。书中不会把 public endpoint 当作无限可用的生产依赖；专业交易终端和机器人需要明确延迟、健康检查、自建服务和降级策略。

Server 负责把数据库和 RPC 组合成查询 API。[crates/server/src/server.rs](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/server/src/server.rs) 中可以看到 pools、historical volume、ticker、trades、orders、assets、OHLCV、level2、fees、status、Margin events、portfolio 等 route。它服务的不是链上执行，而是产品体验和后台系统。

三者的关系可以这样记：SDK 写入交易，Indexer 读取事件，Server 提供查询。一个最小交易工具可能只需要 SDK 和 RPC；一个专业交易终端通常需要 SDK、Indexer 和 Server；一个风控系统还需要自定义数据库视图、告警和任务调度。

## 本节检查

- [ ] 能解释 SDK、Indexer、Server 各自解决的问题。
- [ ] 能判断一个页面数据应该来自链上对象、Indexer 表，还是 Server API。
- [ ] 能说明为什么历史数据不能只靠实时 RPC 查询。
- [ ] 能说明 public indexer 和自建 indexer 的取舍。
