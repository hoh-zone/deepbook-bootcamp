# ch00-05 BalanceManager 账户模型

[返回本章](README.md)

`BalanceManager` 是 DeepBookV3 最重要的开发者概念之一。它不是一个普通钱包地址，也不是某个池子的临时余额表，而是一个共享对象，用来保存单个交易账户在 DeepBook 中使用的多资产余额和权限。

源码入口是 [balance_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/balance_manager.move)。从结构上看，它有 owner、allow list、余额集合、cap 和 proof 相关逻辑。owner 可以存入、取出、交易、staking 和管理交易权限；trader 通常不能取款，但可以执行交易相关动作。这样的设计适合做市团队、托管策略、自动化机器人和多交易员系统。

为什么 DeepBook 不直接让每次交易都从钱包 `Coin` 中扣款？原因是订单簿交易有挂单、部分成交、撤单、settlement、费用和 rebate。一个挂单可能长期存在，资金需要被锁定或记账；一个 maker 可能在多个池中同时做市；一个后台系统可能需要让多个 bot 共用同一套资金。`BalanceManager` 把这些复杂性收进一个账户抽象里。

`BalanceManager` 还提供 cap 模型。`TradeCap`、`DepositCap` 和 `WithdrawCap` 可以把交易、存入、取出权限拆开。源码中很多 DeepBook 操作需要 `TradeProof`，它是“当前调用者有权代表这个 BalanceManager 交易”的证明。这样做比到处传 owner 地址更清晰，也更适合上层协议组合。

应用开发时，`BalanceManager` 是你必须先设计的对象。交易终端要决定用户是否自己创建 manager；做市系统要决定一个策略一个 manager，还是多个策略共享 manager；托管系统要决定是否给机器人发 trader 权限；Margin 则把自己的 `MarginManager` 包装在 `BalanceManager` 和 cap 之上。

## 读源码的重点

先读 `new`、`new_with_custom_owner_and_caps`、`mint_trade_cap`、`deposit`、`withdraw`、`generate_proof_as_owner`、`generate_proof_as_trader`、`deposit_with_proof` 和 `withdraw_with_proof`。这些函数解释了账户、权限和资产流转的骨架。

## 本节检查

- [ ] 能解释 `BalanceManager` 和钱包地址的区别。
- [ ] 能说明 owner、trader、cap、proof 的关系。
- [ ] 能判断一个应用是否需要为用户创建 `BalanceManager`。
