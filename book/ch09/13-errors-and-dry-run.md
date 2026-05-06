# ch09-13 错误提示和交易前模拟

[返回本章](README.md)

## 先看用户路径

这一节从 Margin 用户动作读起：围绕“错误提示和交易前模拟”，先把 manager、collateral、loan、trade 和 repay 的顺序排清楚，再看源码入口。

## 源码入口

重点阅读：

- [packages/deepbook_margin/sources/margin_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/margin_manager.move)
- [packages/deepbook_margin/sources/margin_pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/margin_pool.move)
- [packages/deepbook_margin/sources/pool_proxy.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/pool_proxy.move)
- [packages/deepbook_margin/sources/helper/oracle.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/helper/oracle.move)

> **源码旁白**：先定位结构体、入口函数和事件，再回到本节的资金路径或应用流程。不要从 helper 函数开始读。

## 把 PTB 串起来

应用需要维护 Move abort 到业务提示的映射。常见映射：

- `EBorrowRiskRatioExceeded`：借款后风险率不足，请降低借款或增加抵押品。
- `EWithdrawRiskRatioExceeded`：提款后风险率不足，请减少提款或先还款。
- `ECannotHaveLoanInMoreThanOneMarginPool`：同一 manager 当前只能从一个 MarginPool 借款。
- `EDeepbookPoolNotAllowedForLoan`：该交易对暂未被授权使用目标借贷池。
- `EPoolNotEnabledForMarginTrading`：该 Pool 的 Margin 交易未启用或已暂停。
- `EInvalidDeposit`：只能存入 base、quote 或 DEEP。
- `ECannotLiquidate`：仓位尚未达到清算条件。
- oracle 相关错误：价格过期、置信区间过大或价格源不匹配。

交易前检查顺序：

1. 本地静态检查：参数范围、余额、pool key、manager owner。
2. 链上读取：PoolConfig、MarginPool 状态、manager_state、oracle 更新时间。
3. 本地估算：风险率、滑点、利息、费用。
4. dry run 或 dev inspect：确认 Move 调用真实可执行。
5. 提交交易并订阅事件更新 UI。

## 工程旁白

交易前模拟应该输出状态差异，而不仅是“success/fail”。对借款加交易的 PTB，至少展示借款前后 debt amount、成交后资产、风险率变化、预估利息和可能的价格保护失败点。

错误映射要按用户可行动作设计：风险率不足提示减少金额或补抵押；oracle stale 提示等待更新；pool 未启用提示只能 reduce-only 或稍后再试；min borrow/repay 提示提高金额或选择全额。

## Margin 应用判断

- 本地静态检查不替代 dry run，只用于减少明显错误。
- 记录 abort module、function、code、PTB 输入和链上对象版本，方便复现。
- 对于 stale price 和 orderbook liquidity，错误提示要附带刷新或调整参数的操作。

## 动手检查

- 这个错误用户能通过改金额解决，还是必须等待维护者/价格更新？
- dry run 是否使用了和提交交易相同的 shared object 与 price object？
- 错误映射是否覆盖了 Margin 借贷和 DeepBook Pool 撮合两类 abort？
