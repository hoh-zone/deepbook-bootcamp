# ch08-16 风险率跌破阈值后的处理流程

[返回本章](README.md)

## 先看风险边界

这里先把问题放到风险控制面里。“风险率跌破阈值后的处理流程”不是在 DeepBook 旁边加一层 UI，而是把 registry、manager、oracle、borrow/supply state 接进同一条链上风控路径。

## 源码入口

重点阅读：

- [packages/deepbook_margin/sources/margin_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/margin_manager.move)
- [packages/deepbook_margin/sources/pool_proxy.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/pool_proxy.move)
- [packages/margin_liquidation/sources/liquidation_vault.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/margin_liquidation/sources/liquidation_vault.move)

> **源码旁白**：先定位结构体、入口函数和事件，再回到本节的资金路径或应用流程。不要从 helper 函数开始读。

## 读风险控制面

当风险率低于 `liquidation_risk_ratio`：

1. 任何人都可以尝试清算，清算 vault 还要求调用者在授权 trader 集合中。
2. 清算入口先取消挂单并结算，避免订单继续改变仓位。
3. 协议计算把风险率拉回 `target_liquidation_risk_ratio` 所需的还款额。
4. 如果资产不足，可能发生 full liquidation，`pool_default` 记录借贷池坏账。
5. 清算事件记录剩余资产、剩余债务、风险率和预言机价格，应用端应把它作为仓位状态同步的权威信号。

## 工程旁白

风险率下降不一定立刻可清算。低于 `min_withdraw_risk_ratio` 时应禁止提款；低于 `min_borrow_risk_ratio` 时应禁止新增借款；只有低于 `liquidation_risk_ratio` 才进入清算条件。

用户侧优先提供降风险动作：补充抵押、还款、取消挂单并结算、reduce-only 反向交易。机器人侧只在满足清算阈值且 vault 有对应 debt asset 时提交清算。

## 风控判断

- UI 用同一条风险率刻度展示四个阈值，并把可用操作随状态切换。
- Pool 暂停时仍优先保留 reduce-only、还款和补抵押路径。
- 清算告警要包含价格来源时间，避免用 stale indexer 数据误触发。

## 动手检查

- 当前状态只是禁止提款/借款，还是已经达到 `can_liquidate`？
- 用户最小代价的降风险动作是什么：补抵押、还款还是反向交易？
- 清算机器人是否在提交前重新 dev inspect 风险率？
