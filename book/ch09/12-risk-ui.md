# ch09-12 UI 风控展示

[返回本章](README.md)

## 本节目标

- 设计 Margin 仓位页所需的风险率、债务、利息、阈值、订单和 oracle 展示。
- 能把链上风险率和前端输入模拟结合，实时提示用户操作后的健康度。
- 能清楚区分抵押余额、供应份额、借款债务和 DeepBook open orders。

## 源码关联

重点阅读：

- [packages/deepbook_margin/sources/margin_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/margin_manager.move)
- [packages/deepbook_margin/sources/margin_pool/margin_state.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/margin_pool/margin_state.move)
- [packages/deepbook_margin/sources/helper/oracle.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/helper/oracle.move)
- [packages/deepbook_margin/sources/tpsl.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/tpsl.move)

阅读时先从这些文件定位结构体、入口函数和事件，再回到正文中的资金路径或应用流程。

## 正文

仓位页至少展示：

- manager ID、pool key、owner。
- base balance、quote balance、DEEP balance。
- base debt、quote debt、debt asset。
- risk ratio 和四个阈值。
- borrow shares 和折算 debt amount。
- MarginPool utilization、borrow rate、supply rate 估算。
- Pyth 价格更新时间、confidence 状态。
- open orders、locked balance、TPSL 数量。
- 可借额度、可提额度、清算距离。

健康度可以做成区间：

- 安全：risk ratio 高于 `min_withdraw_risk_ratio`。
- 注意：低于提款阈值但高于借款阈值。
- 危险：接近 `liquidation_risk_ratio`。
- 可清算：低于 `liquidation_risk_ratio`。

每次用户输入借款、提款、下单、TPSL 参数时，都要重新模拟风险率，而不是等点击提交才报错。

补充说明：

风控面板要围绕 debt asset 组织信息。用户最需要知道的是欠什么、欠多少、利息如何增长、哪个价格方向会触发清算，而不是只看账户净值。

输入框每次变化都应触发本地估算和必要的 dry run。比如增加借款、提款或创建 TPSL，都会改变未来风险率；等到提交交易才发现失败，会让用户无法判断该补抵押还是降低规模。

## 开发要点

- 风险率条展示 min withdraw、min borrow、liquidation、target liquidation 四个刻度。
- 余额表分为 manager collateral、MarginPool supply position、open orders 和 settled amounts。
- 价格组件显示 Pyth 更新时间和 stale 状态，避免用户误信过期健康度。

## 检查问题

- 用户当前风险由债务资产价格、利息增长还是订单锁仓主导？
- UI 是否把 lending supply shares 与 margin collateral 分开显示？
- 用户输入一笔新交易后，风险率预览是否即时更新？
