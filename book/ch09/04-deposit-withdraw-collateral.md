# ch09-04 存入和提取抵押品

[返回本章](README.md)

## 先看用户路径

这里先从用户动作往回看。“存入和提取抵押品”在应用里通常是一组 PTB 和状态刷新，不是一条孤立 move call；每一步都要同时考虑余额、债务和风险率。

## 源码入口

重点阅读：

- [packages/deepbook_margin/sources/margin_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/margin_manager.move)
- [packages/deepbook_margin/sources/margin_registry.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/margin_registry.move)
- `book/ch09/code/s02-deposit-collateral/README.md`

> **源码旁白**：先定位结构体、入口函数和事件，再回到本节的资金路径或应用流程。不要从 helper 函数开始读。

## 把 PTB 串起来

抵押品存入调用 `MarginManager.deposit<BaseAsset, QuoteAsset, DepositAsset>`。`DepositAsset` 必须是 base、quote 或 DEEP，否则 `deposit_int` 会抛 `EInvalidDeposit`。

应用流程：

1. 查询用户 coin objects，并合并或选择足够余额。
2. 构造 PTB，把 coin 传给 manager deposit。
3. 传入 base 和 quote Pyth price object，链上会在 base/quote 抵押存入时记录价格事件。
4. 执行前 dry run，捕获无效 coin、owner 不匹配、price object 错误等问题。

提取调用 `withdraw`，会先从 `BalanceManager` 取出 coin，再重新计算风险率。若 manager 有债务且提款后低于 `min_withdraw_risk_ratio`，交易会 abort，状态回滚。

UI 应提供“最大可提”而不是只显示余额。最大可提要考虑当前债务、价格、open order、`min_withdraw_risk_ratio` 和 stale price 风险。

## 工程旁白

存入抵押品会增加 manager 内 BalanceManager 的资产余额，可能提高风险率；供应到 MarginPool 则把资产放入借贷池 vault，换取 supply shares，不会直接改善某个 manager 的健康度。

提款是高风险操作，因为它会降低资产侧价值。应用在允许提款前要用最新 price object 模拟 `withdraw` 后的风险率，并检查 `min_withdraw_risk_ratio`，否则用户会在签名后遇到 abort。

## Margin 应用判断

- 充值页面明确标注“存入保证金”和“供应赚息”两个不同入口。
- 提款金额默认给出安全上限，而不是全部 BalanceManager 余额。
- DEEP 可作为协议相关资产展示，但仍要遵守 `EInvalidDeposit` 的资产类型限制。

## 动手检查

- 这次资金进入 manager 还是 MarginPool vault？
- 提款后风险率是否仍高于 `min_withdraw_risk_ratio`？
- 用户要提取的是可用余额，还是仍锁在 open orders 或未结算金额中？
