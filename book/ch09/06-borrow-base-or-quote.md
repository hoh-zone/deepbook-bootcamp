# ch09-06 借入 base 或 quote

[返回本章](README.md)

## 先看用户路径

这里先从用户动作往回看。“借入 base 或 quote”在应用里通常是一组 PTB 和状态刷新，不是一条孤立 move call；每一步都要同时考虑余额、债务和风险率。

## 源码入口

重点阅读：

- [packages/deepbook_margin/sources/margin_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/margin_manager.move)
- [packages/deepbook_margin/sources/margin_pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/margin_pool.move)
- [packages/deepbook_margin/sources/margin_pool/margin_state.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/margin_pool/margin_state.move)
- `book/ch09/code/s03-borrow-and-trade/README.md`

> **源码旁白**：先定位结构体、入口函数和事件，再回到本节的资金路径或应用流程。不要从 helper 函数开始读。

## 把 PTB 串起来

借款入口是 `borrow_base` 和 `borrow_quote`。应用在构造交易前要确认：

- DeepBook Pool 在 `MarginRegistry` 中 enabled。
- 目标 MarginPool 已通过 `enable_deepbook_pool_for_loan` 授权该 DeepBook Pool。
- 当前 manager 没有来自其它 MarginPool 的债务。
- 借款金额大于 `min_borrow`。
- 借款后风险率仍高于 `min_borrow_risk_ratio`。
- Pyth price object 新鲜且 confidence 合格。

借入 base 常用于开空：借 SUI 后卖成 USDC。借入 quote 常用于开多：借 USDC 后买入 SUI。

应用展示“可借额度”时不要只用池可用余额。还要取 min：

- MarginPool vault 可用余额。
- 最大利用率下剩余可借额度。
- 用户风险率约束下的最大借款。
- 交易规模和订单簿深度。

## 工程旁白

借入 quote 通常用于买入 base，借入 base 通常用于卖出 base。借款会增加 borrow shares，真实 debt amount 会随 `margin_state` 利息增长；应用应在借款确认页展示当前 APR 和预估日利息。

可借额度不是一个单一链上字段。后端可以先本地估算，再用 dev inspect 验证最终 PTB；如果用户同时借款并下单，还要把成交后资产结构和滑点纳入风险率估算。

## Margin 应用判断

- 借款按钮旁显示 debt asset、borrow APR 和清算阈值距离。
- 小额借款要检查 `min_borrow`，避免 shares 换算为 0 或低于池配置。
- 借款成功后立即刷新 manager debt shares 和 pool utilization。

## 动手检查

- 用户借的是 base 还是 quote，对价格风险方向有什么影响？
- 可借额度是否同时受池端 max utilization 和用户端 risk ratio 限制？
- 这笔借款会在同一交易归还吗？如果不会，UI 是否展示持续利息？
