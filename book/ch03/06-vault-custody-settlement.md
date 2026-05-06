# ch03-06 Vault：最终资产保管和结算

[返回本章](README.md)

## 先抓住结构

先把“Vault：最终资产保管和结算”放进 DeepBook 的对象图里。这里不是罗列模块，而是建立阅读顺序：入口在哪里，状态放在哪里，资金最终在哪里结算。

## 源码入口

这一节只保留必要入口，目的不是让你马上读完源码，而是建立后续定位能力：

- [packages/deepbook/sources/vault/vault.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/vault/vault.move)
- [packages/deepbook/sources/state/balances.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/state/balances.move)
- [packages/deepbook/sources/balance_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/balance_manager.move)
- [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)

读源码时先确认对象、函数签名和事件名称；等正文讲到交易路径时，再回到这些入口核对。

## 读架构

`vault.move` 的 `Vault<BaseAsset, QuoteAsset>` 保存三类余额：`base_balance`、`quote_balance`、`deep_balance`。`settle_balance_manager` 接收 `balances_out` 和 `balances_in`：

- 当 `balances_out.base() > balances_in.base()`，Vault 把差额 split 出来并存入 BalanceManager。
- 当 `balances_in.base() > balances_out.base()`，BalanceManager 通过 `withdraw_with_proof` 提供差额，Vault join 到池余额。
- quote 和 DEEP 使用同样模式。

因此结算是“净额结算”，不是每个 fill 都立即转 coin。开发者排查资金问题时，应同时看 `OrderInfo`、`Fill`、`State` 返回的 `Balances`，以及最终 BalanceManager 的余额变化。

## 阅读补充

Vault 是资金守恒的最后关口。State 计算出本次交易的输入输出差额后，Vault 根据 base、quote、DEEP 三类余额方向，从池内余额 split 给 BalanceManager，或从 BalanceManager withdraw 后 join 回池内余额。

阅读 Vault 时要把每个方向都写成“谁多了、谁少了、差额是多少”。这样能快速识别 taker 支付 quote 获得 base、maker 收入、DEEP fee 这几条资金线是否被正确结算。

## 工程判断

- 结算表按 base、quote、DEEP 三列记录差额。
- Vault 调用 BalanceManager 时必须验证 proof，不要绕过授权解释资金变化。
- 闪电贷路径也在 Vault，但不要与普通订单净额结算混淆。

## 读完以后问自己

- Vault 根据什么判断从池转出还是向池转入？
- 为什么结算要使用净额而不是逐 fill 转账？
- Vault 与 BalanceManager 的授权关系在哪里体现？
