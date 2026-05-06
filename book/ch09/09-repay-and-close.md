# ch09-09 部分还款、全部还款和平仓

[返回本章](README.md)

## 本节目标

- 实现部分还款、全额还款和组合平仓流程。
- 能正确处理 `Option<u64>` amount、repay shares、债务资产余额和 min repay 边界。
- 能设计取消订单、结算、反向交易、还款、提取剩余资产的平仓顺序。

## 源码关联

重点阅读：

- [packages/deepbook_margin/sources/margin_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/margin_manager.move)
- [packages/deepbook_margin/sources/margin_pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/margin_pool.move)
- `book/ch09/code/s04-repay-and-close/README.md`

阅读时先从这些文件定位结构体、入口函数和事件，再回到正文中的资金路径或应用流程。

## 正文

还款入口：

- `repay_base`：偿还 base 债务。
- `repay_quote`：偿还 quote 债务。

两者都接收 `Option<u64>` amount。传入 none 表示尽量全额还款，传入 some 表示部分还款。内部会根据当前 debt amount 和 shares 计算 `repay_shares`，从 manager 的 `BalanceManager` 取出债务资产，再调用 `MarginPool.repay`。

平仓通常是组合操作：

1. 取消所有 open orders。
2. 结算 DeepBook Pool。
3. 如果资产不是债务资产，先通过 DeepBook 反向交易换成债务资产。
4. 调 `repay_base` 或 `repay_quote`。
5. 债务清零后提取剩余资产。

错误处理重点：

- `ERepayAmountTooLow`：还款金额太小。
- `ERepaySharesTooLow`：金额换算出的 shares 为 0。
- `EIncorrectMarginPool`：传错了债务对应的 MarginPool。
- 余额不足：manager 里没有足够债务资产，用户需要先成交或存入。

补充说明：

还款金额和债务 shares 之间存在精度边界。全额还款通常用 none 语义让链上尽量还清；部分还款要确保金额能换算成非零 shares，并且 manager 中有足够债务资产。

平仓不是单个 repay 调用。用户可能需要先取消挂单、结算成交资产，再通过 DeepBook 反向交易把资产换成 debt asset，最后还款并提取剩余余额。

## 开发要点

- 还款 UI 默认显示 debt asset，禁止用非债务资产直接提交 repay。
- 全额还款后重新读取 borrow shares，确认是否真正清零。
- 平仓流程每一步都保留 dry run，尤其是反向交易和最终提款。

## 检查问题

- 传入 none 和 some amount 的链上行为有什么区别？
- manager 中是否有足够 debt asset，还是需要先反向交易？
- 平仓后还有 open orders、settled amounts 或 TPSL 残留吗？
