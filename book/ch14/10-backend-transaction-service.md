# ch14-10 后端交易服务

[返回本章](README.md)

## 本节目标

- 明确“交易服务”在 DeepBook 应用闭环中的用户场景、交易构造和数据依赖。
- 能把 UI、SDK/PTB、Server API、Indexer 数据和监控信号连接成可运行流程。
- 能识别本节应用上线前必须处理的余额、风险、延迟、失败和披露问题。

## 源码关联

- [scripts/transactions](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/scripts/transactions)：TypeScript 交易构造或运行脚本样例。
- [packages/deepbook](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook)：DeepBook Spot、BalanceManager、订单簿或闪电贷逻辑。
- [packages/deepbook_margin](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin)：Margin 合约对象、借贷、抵押、清算或风控逻辑。
- [crates/server/src/admin/routes.rs](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/server/src/admin/routes.rs)：REST API、数据库查询、健康检查或指标实现。

## 正文

后端交易服务可以帮助前端构造交易，但不应该托管用户私钥。服务端返回待签名 PTB 或交易参数，用户钱包签名提交。服务端负责配置校验、对象 ID 管理、dry run、错误码映射和日志。


后端交易服务适合封装报价、模拟、风控和交易模板，但不应代替用户签名权限。服务返回的是待签 PTB、仿真结果和风险提示；真正资产转移仍要由钱包或受控 signer 执行并记录 digest。

## 开发要点

- 从用户动作出发写清 PTB 输入、type arguments、权限对象、签名者和预期 digest。
- 前端状态要区分钱包签名、链上确认、Indexer 可见和 Server API 缓存刷新。
- 机器人和后端服务必须有库存、价格、checkpoint lag、错误率和人工暂停阈值。
- 上线前为费用、滑点、oracle、清算、数据延迟和失败重试准备明确披露。

## 检查问题

- 用户完成本节场景时，最小 PTB、API 查询和 UI 状态分别是什么？
- 失败时应展示权限、余额、价格、oracle、清算、RPC 还是 Indexer 延迟中的哪一类原因？
- 这个应用 MVP 上线前还缺哪项测试、监控、暂停开关或风险披露？
