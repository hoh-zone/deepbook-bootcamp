# ch08-14 正常路径与失败边界

[返回本章](README.md)

## 本节目标

- 把 Margin 正常操作路径和失败边界整理成可测试的检查表。
- 能按借款、提款、下单、还款、TPSL 和清算分别定位常见 abort 原因。
- 能设计交易前模拟，让用户在签名前知道风险率、利息、价格和权限问题。

## 源码关联

重点阅读：

- [packages/deepbook_margin/sources/margin_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/margin_manager.move)
- [packages/deepbook_margin/sources/margin_pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/margin_pool.move)
- [packages/deepbook_margin/sources/pool_proxy.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/pool_proxy.move)
- [packages/deepbook_margin/sources/helper/oracle.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/helper/oracle.move)

阅读时先从这些文件定位结构体、入口函数和事件，再回到正文中的资金路径或应用流程。

## 正文

正常借款路径：`deposit` 抵押品，`borrow_base` 或 `borrow_quote` 从对应 MarginPool 借款，借款 coin 存入 manager，然后立即检查 `min_borrow_risk_ratio`。

正常还款路径：用户先确保 manager 中有债务资产，调用 `repay_base` 或 `repay_quote`。内部按 amount 和当前 debt 计算 repay shares，从 `BalanceManager` 取 coin，调用 `MarginPool.repay`，债务清零后 `margin_pool_id` 变为 none。

常见失败边界：

- Pool 未启用 Margin：`EPoolNotEnabledForMarginTrading`。
- 借贷池未允许该 DeepBook Pool 借款：`EDeepbookPoolNotAllowedForLoan`。
- 同一个 manager 已有另一借贷池债务：`ECannotHaveLoanInMoreThanOneMarginPool`。
- 借款后风险率不足：`EBorrowRiskRatioExceeded`。
- 提款后风险率不足：`EWithdrawRiskRatioExceeded`。
- 清算风险率尚未跌破阈值：`ECannotLiquidate`。
- oracle stale、confidence 过高或 EWMA 偏离过大：oracle 模块抛错。

补充说明：

正常路径的关键是对象一致性：registry 中的 PoolConfig、manager 绑定的 DeepBook Pool、MarginPool 授权列表、oracle price object 和 PTB type arguments 必须互相匹配。任何一个错位都可能表现为 Move abort。

失败边界应转成产品语言。风险率不足提示用户补抵押或降低借款；oracle stale 提示等待价格更新；reduce-only 失败提示订单方向会扩大仓位；min borrow/min repay 提示用户调整金额。

## 开发要点

- 为每个写操作维护 preflight checklist，而不是只在 catch 里展示原始 abort code。
- 把权限、余额、风险率、价格和配置错误分组，便于客服和日志分析。
- 复杂 PTB 先 dev inspect，再 dry run，再提交，尤其是借款加交易的组合操作。

## 检查问题

- 这次失败属于对象配置错误、用户余额错误、风险率错误还是 oracle 错误？
- 如果交易部分成功不可能发生，PTB 原子性是否已向用户解释清楚？
- 错误提示是否给出用户下一步，而不只是 Move abort 名称？
