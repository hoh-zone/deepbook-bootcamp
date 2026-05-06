# ch14-09 Predict 应用

[返回本章](README.md)

## 本节目标

- 明确“Predict 应用”在 DeepBook 应用闭环中的用户场景、交易构造和数据依赖。
- 能把 UI、SDK/PTB、Server API、Indexer 数据和监控信号连接成可运行流程。
- 能识别本节应用上线前必须处理的余额、风险、延迟、失败和披露问题。

## 源码关联

- [packages/predict/sources/predict.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/predict/sources/predict.move)：Predict 市场、oracle、定价、结算或测试逻辑。
- [packages/predict/sources/predict_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/predict/sources/predict_manager.move)：Predict 市场、oracle、定价、结算或测试逻辑。
- [packages/predict/sources/oracle.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/predict/sources/oracle.move)：Predict 市场、oracle、定价、结算或测试逻辑。
- [packages/predict/tests](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/predict/tests)：测试 fixture、断言或可复现验证材料。

## 正文

Predict 应用要展示市场到期时间、oracle 价格源、赔率、最大损失、LP 风险和结算规则。对于还在迁移或测试阶段的能力，产品文案必须明确标注网络和版本状态。


Predict 应用的核心不是订单簿，而是市场定义、oracle 更新、下注/赎回和结算披露。UI 必须展示市场 key、价格来源、结算时间和费用，避免用户把预测份额当作普通现货仓位。

## 开发要点

- 从用户动作出发写清 PTB 输入、type arguments、权限对象、签名者和预期 digest。
- 前端状态要区分钱包签名、链上确认、Indexer 可见和 Server API 缓存刷新。
- 机器人和后端服务必须有库存、价格、checkpoint lag、错误率和人工暂停阈值。
- 上线前为费用、滑点、oracle、清算、数据延迟和失败重试准备明确披露。

## 检查问题

- 用户完成本节场景时，最小 PTB、API 查询和 UI 状态分别是什么？
- 失败时应展示权限、余额、价格、oracle、清算、RPC 还是 Indexer 延迟中的哪一类原因？
- 这个应用 MVP 上线前还缺哪项测试、监控、暂停开关或风险披露？
