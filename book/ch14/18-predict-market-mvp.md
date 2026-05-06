# ch14-18 Predict 市场 MVP

[返回本章](README.md)

## 先定义产品场景

产品小节先看上线闭环：交易、数据、风控、钱包和运维必须一起设计，不能只列功能按钮。

## 源码入口

- [packages/predict/sources/predict.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/predict/sources/predict.move)：Predict 市场主流程、用户头寸和结算入口。
- [packages/predict/sources/oracle_config.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/predict/sources/oracle_config.move)：价格源配置、oracle 权限和结算依赖。
- [packages/predict/sources/config/risk_config.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/predict/sources/config/risk_config.move)：市场风险参数和限额配置。
- [packages/predict/tests](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/predict/tests)：Predict 市场行为、定价和 oracle 测试。

## 把系统拼起来

第一版的边界要收紧：市场列表、市场详情、价格源、mint position、用户头寸、结算状态、LP supply。测试网阶段重点验证状态机和用户理解，不追求复杂收益策略。


Predict 市场 MVP 应覆盖市场列表、详情、下注、赎回/结算状态和 oracle 披露。测试和 UI 都要验证市场关闭、价格源不可用、结算延迟和费用展示。

## 产品落地判断

- 从用户动作出发写清 PTB 输入、type arguments、权限对象、签名者和预期 digest。
- 前端状态要区分钱包签名、链上确认、Indexer 可见和 Server API 缓存刷新。
- 机器人和后端服务必须有库存、价格、checkpoint lag、错误率和人工暂停阈值。
- 上线前为费用、滑点、oracle、清算、数据延迟和失败重试准备明确披露。

## 动手检查

- 用户完成本节场景时，最小 PTB、API 查询和 UI 状态分别是什么？
- 失败时应展示权限、余额、价格、oracle、清算、RPC 还是 Indexer 延迟中的哪一类原因？
- 这个应用 MVP 上线前还缺哪项测试、监控、暂停开关或风险披露？
