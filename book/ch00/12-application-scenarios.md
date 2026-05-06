# ch00-12 可以构建哪些应用

[返回本章](README.md)

理解 DeepBook 的最好方式之一，是从可以构建的应用倒推需要学习哪些模块。

最直接的是交易终端。它需要展示池列表、订单簿、成交、K 线、用户余额、open orders、历史订单、下单面板、撤单和订单状态。交易提交依赖 SDK 和 `BalanceManager`，行情展示依赖 Indexer 和 Server，交易前检查依赖链上查询和 dry run。

第二类是 swap 和聚合器。它不一定暴露完整订单簿，而是把 DeepBook 作为流动性来源之一。应用需要估算输出数量、检查 min out、构造 `swap_exact_base_for_quote` 或 `swap_exact_quote_for_base`，并处理 DEEP 输入和剩余 coin 输出。聚合器还要比较 AMM、CLOB 和其他路由。

第三类是做市和量化工具。它需要批量下单、批量撤单、实时订单簿、账户 open orders、成交事件、费用和 rebate 统计。做市系统通常还会管理多个 `BalanceManager` 或多个 trader 权限，并对异常成交、撤单失败和网络延迟做监控。

第四类是 Margin 应用。它要支持创建 `MarginManager`、存入抵押品、供应借贷池、借入资产、通过 DeepBook 池执行杠杆交易、还款、平仓、TPSL 和清算。风控 UI 必须显示风险率、债务、资产、oracle 时间戳和清算阈值。

第五类是数据服务。团队可以基于 Indexer 和 Server 构建行情 API、交易历史 API、研究面板、费用统计、rebate 报表、清算监控和组合资产视图。这类应用不一定提交交易，但对事件语义和表结构要求很高。

第六类是协议组合。其他 Move 包可以在 PTB 中调用 DeepBook swap 或闪电贷，完成套利、再平衡、清算、结构化产品或 treasury 管理。此时最重要的是对象传递、资产归还、失败回滚和交易原子性。

## 本节检查

- [ ] 能列出六类 DeepBook 应用。
- [ ] 能为每类应用选择需要学习的章节。
- [ ] 能判断一个需求是交易构造问题、链上状态问题，还是数据索引问题。
