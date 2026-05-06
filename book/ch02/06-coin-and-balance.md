# ch02-06 Coin&lt;T&gt; 与 Balance&lt;T&gt;

[返回本章](README.md)

## 先建立手感

先不要把“Coin&lt;T&gt; 与 Balance&lt;T&gt;”当成孤立语法点。DeepBook 里每个资产、订单和权限对象都会受 Move 类型系统约束，读这一节时要看语法如何变成资金安全边界。

## Move 对照

先用下面几处源码建立 Move 概念的落点。这里不追完整协议流程，只确认类型、ability、对象、事件和测试入口如何在真实项目中出现。

- [packages/deepbook/sources/balance_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/balance_manager.move)
- [packages/deepbook/sources/vault/vault.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/vault/vault.move)
- [packages/deepbook/sources/state/balances.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/state/balances.move)

读这些文件时，把语法点和真实对象放在一起看：ability、泛型、对象 ID、事件和测试入口分别承担什么约束。

## 拆开来看

`Coin<T>` 是可作为对象输入和输出的钱币对象。钱包里看到的 SUI 或 token 通常是 `Coin<T>` 对象。

`Balance<T>` 是合约内部保存的余额值，不能直接当作钱包对象传输。DeepBook 的 `BalanceManager` 会把用户存入的 `Coin<T>` 转成内部 `Balance<T>`，交易结算时再按需要转回 `Coin<T>`。

源码在 [packages/deepbook/sources/balance_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/balance_manager.move)：`deposit<T>` 接收 `coin: Coin<T>`，调用 `coin.into_balance()` 后进入内部余额；`withdraw<T>` 调用 `withdraw_with_proof(...).into_coin(ctx)` 返回 `Coin<T>`。

## 阅读补充

`Coin<T>` 是可在用户钱包中持有和转移的对象形态，`Balance<T>` 更适合协议内部合并、拆分和记账。DeepBook 的 `BalanceManager` 会把用户存入的 coin 转成内部余额，交易结算时再通过 Vault 与账户余额做净额变化。

调试“余额不见了”时，先判断资产处于钱包 coin、BalanceManager 余额、Pool Vault 余额还是 open order 锁定状态。它们都是资产路径的一部分，但查询方式和权限完全不同。

## Move 判断

- 存款路径关注 `into_balance`，取款路径关注 `into_coin(ctx)`。
- 不要用钱包 coin 余额直接判断能否下 DeepBook 订单。
- 结算问题要同时查看 BalanceManager 和 Vault 的差额处理。

## 练习问题

- 为什么存入 BalanceManager 后钱包 coin 数量会变化？
- `Balance<T>` 为什么更适合协议内部记账？
- 下单前应该检查钱包余额还是 BalanceManager 余额？
