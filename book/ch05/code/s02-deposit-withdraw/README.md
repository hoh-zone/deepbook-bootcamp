# s02 存入与取出

目标：演示钱包 `Coin<T>` 如何进入 `BalanceManager`，以及如何从 manager 提回钱包。

核心函数：

- `balance_manager::deposit<T>(&mut BalanceManager, Coin<T>, &mut TxContext)`
- `balance_manager::deposit_with_cap<T>(&mut BalanceManager, &DepositCap, Coin<T>, &TxContext)`
- `balance_manager::withdraw<T>(&mut BalanceManager, amount, &mut TxContext): Coin<T>`
- `balance_manager::withdraw_all<T>(&mut BalanceManager, &mut TxContext): Coin<T>`

资金路径：

1. 钱包选择或合并 `Coin<T>`。
2. deposit 把 `Coin<T>` 转成 `Balance<T>`。
3. manager 的 `balances[BalanceKey<T>]` 增加。
4. 链上发出 `BalanceEvent { asset, amount, deposit: true }`。
5. withdraw 时从 manager split 指定数量，返回 `Coin<T>`，并发出 `deposit: false` 事件。

运行方式：

```bash
pnpm install
pnpm tsx deposit.ts
pnpm tsx withdraw.ts
```

生产检查：

- 下单前检查的是 manager 余额，不是钱包余额。
- 提现前先调用 `pool.withdraw_settled_amounts`，否则成交后的 settled 余额可能还在 pool account 内。
- 余额不足会触发 `EBalanceManagerBalanceTooLow = 3`。
