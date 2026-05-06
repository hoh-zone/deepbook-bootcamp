# ch02-06 Coin&lt;T&gt; 与 Balance&lt;T&gt;

[返回本章](README.md)

## 本节目标

- 区分钱包 coin 对象和协议内部余额。
- 能沿 `Coin<T>` 与 `Balance<T>` 定位相关 Move 源码、脚本或链下服务入口。
- 读完后能够用交易路径、对象职责或失败场景解释本节主题。

## 源码关联

本节重点对照以下源码或后续阅读入口：

- [packages/deepbook/sources/balance_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/balance_manager.move)
- [packages/deepbook/sources/vault/vault.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/vault/vault.move)
- [packages/deepbook/sources/state/balances.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/state/balances.move)

阅读时先从标题对应的入口文件开始，确认对象、函数签名和事件名称，再回到本节正文理解它在交易路径中的位置。

## 正文

`Coin<T>` 是可作为对象输入和输出的钱币对象。钱包里看到的 SUI 或 token 通常是 `Coin<T>` 对象。

`Balance<T>` 是合约内部保存的余额值，不能直接当作钱包对象传输。DeepBook 的 `BalanceManager` 会把用户存入的 `Coin<T>` 转成内部 `Balance<T>`，交易结算时再按需要转回 `Coin<T>`。

源码在 [packages/deepbook/sources/balance_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/balance_manager.move)：`deposit<T>` 接收 `coin: Coin<T>`，调用 `coin.into_balance()` 后进入内部余额；`withdraw<T>` 调用 `withdraw_with_proof(...).into_coin(ctx)` 返回 `Coin<T>`。

## 阅读补充

`Coin<T>` 是可在用户钱包中持有和转移的对象形态，`Balance<T>` 更适合协议内部合并、拆分和记账。DeepBook 的 `BalanceManager` 会把用户存入的 coin 转成内部余额，交易结算时再通过 Vault 与账户余额做净额变化。

调试“余额不见了”时，先判断资产处于钱包 coin、BalanceManager 余额、Pool Vault 余额还是 open order 锁定状态。它们都是资产路径的一部分，但查询方式和权限完全不同。

## 开发要点

- 存款路径关注 `into_balance`，取款路径关注 `into_coin(ctx)`。
- 不要用钱包 coin 余额直接判断能否下 DeepBook 订单。
- 结算问题要同时查看 BalanceManager 和 Vault 的差额处理。

## 检查问题

- 为什么存入 BalanceManager 后钱包 coin 数量会变化？
- `Balance<T>` 为什么更适合协议内部记账？
- 下单前应该检查钱包余额还是 BalanceManager 余额？
