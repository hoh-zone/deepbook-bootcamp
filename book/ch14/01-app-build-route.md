# ch14-01 应用构建路线

[返回本章](README.md)

## 先定义产品场景

构建 DeepBook 应用时，最容易犯的错误是从“我要调用哪个 move function”开始。真正的产品路线应该从用户动作和失败路径开始：用户想交易、做市、借贷、清算还是购买预测头寸；失败时是余额不足、权限不对、oracle stale、Indexer 延迟，还是对象版本过期。

## 源码入口

- [packages/deepbook](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook)：DeepBook Spot、BalanceManager、订单簿或闪电贷逻辑。
- [packages/deepbook_margin](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin)：Margin 合约对象、借贷、抵押、清算或风控逻辑。
- [packages/predict](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/predict)：Predict 市场、oracle、定价、结算或测试逻辑。
- [crates/server/src/server.rs](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/server/src/server.rs)：REST API、数据库查询、健康检查或指标实现。

## 把系统拼起来

DeepBook 应用有四层：协议理解、SDK 交易、Indexer 数据、产品交互。协议层定义资金安全和状态机；SDK 层负责构造 PTB；Indexer 层负责历史数据和聚合；前端、机器人或后端 worker 负责用户操作、自动策略和风控降级。

应用路线要从“用户动作”倒推 PTB、数据查询和风控闭环。Spot 终端依赖 DeepBook 包与 Server 行情；Margin 还要引入风险率、借贷池、oracle 和清算；Predict 还要引入 expiry、range key、vault exposure 和 settlement。它们可以共享钱包、配置、dry run 和数据服务，但不能被塞进同一个通用交易表单。

出版社级应用章必须写出取舍：MVP 做什么，不做什么；哪些数据来自 public service，哪些必须自建；哪些交易允许用户直接签名，哪些应该由后端先生成 bytes；什么时候因为 checkpoint lag 或 oracle stale 暂停按钮。

## 产品落地判断

- 从用户动作出发写清 PTB 输入、type arguments、权限对象、签名者和预期 digest。
- 前端状态要区分钱包签名、链上确认、Indexer 可见和 Server API 缓存刷新。
- 机器人和后端服务必须有库存、价格、checkpoint lag、错误率和人工暂停阈值。
- 上线前为费用、滑点、oracle、清算、数据延迟和失败重试准备明确披露。
- 每个 MVP 都要写出降级模式：RPC 慢、Indexer 落后、oracle stale、钱包拒签时页面如何响应。

## 动手检查

- 用户完成本节场景时，最小 PTB、API 查询和 UI 状态分别是什么？
- 失败时应展示权限、余额、价格、oracle、清算、RPC 还是 Indexer 延迟中的哪一类原因？
- 这个应用 MVP 上线前还缺哪项测试、监控、暂停开关或风险披露？
