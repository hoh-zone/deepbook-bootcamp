# ch05-03 存入、取出、锁定与结算

[返回本章](README.md)

## 先沿资金问问题

这一节把“存入、取出、锁定与结算”放到余额流里看。只看钱包余额会误判交易可用性，真正要追的是权限 proof、托管余额、费用参数和结算路径。

## 源码入口

- [packages/deepbook/sources/balance_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/balance_manager.move)：`BalanceManager` 余额、owner/cap、`TradeProof`、deposit/withdraw 和权限校验。
- [packages/deepbook/sources/vault/vault.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/vault/vault.move)：池子 `base_balance`/`quote_balance`、`balances_in`/`balances_out` 结算和资金安全断言。
- [packages/deepbook/sources/state/account.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/state/account.move)：账户维度的 open orders、settled/owed balances、stake、rebate 和成交后状态。
- [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)：交易、stake、claim rebate、治理和 manager 结算的对外入口。

## 关键定义

存入和取出入口很直接：

```move
public fun deposit<T>(
    balance_manager: &mut BalanceManager,
    coin: Coin<T>,
    ctx: &mut TxContext,
)

public fun withdraw<T>(
    balance_manager: &mut BalanceManager,
    withdraw_amount: u64,
    ctx: &mut TxContext,
): Coin<T>

public fun withdraw_all<T>(
    balance_manager: &mut BalanceManager,
    ctx: &mut TxContext,
): Coin<T>
```

每次余额变化都会发出事件：

```move
public struct BalanceEvent has copy, drop {
    balance_manager_id: ID,
    asset: TypeName,
    amount: u64,
    deposit: bool,
}
```

这说明前端余额页不能只查当前对象字段。要做流水、审计或用户历史，就必须读 `BalanceEvent`。`deposit: true` 是入账，`deposit: false` 是出账；`asset` 是完整类型名，不是 UI ticker。

## 沿资金流看

存入路径是 `balance_manager::deposit<T>(manager, coin, ctx)` 或 `deposit_with_cap<T>`。函数先发出 `BalanceEvent { balance_manager_id, asset, amount, deposit: true }`，再把 `Coin<T>` 转为 `Balance<T>`，通过 `deposit_with_proof` 合并到 `balances`。

取出路径是 `withdraw<T>`、`withdraw_all<T>` 或 `withdraw_with_cap<T>`。底层 `withdraw_with_proof<T>` 会按 `BalanceKey<T>` 取余额；如果余额小于提现量，抛出 `EBalanceManagerBalanceTooLow = 3`。提现成功后发出 `BalanceEvent { deposit: false }`。

下单时的锁定不是直接在 `BalanceManager` 内创建锁字段，而是在 `pool.place_order_int` 中经过 `state.process_create` 计算 `settled` 和 `owed`。`owed` 表示用户要付给 vault 的资产，`settled` 表示 vault 要付回用户的资产。对于挂在订单簿中的 maker 订单，订单对象记录了锁定数量；后续成交或取消时再把差额写入 account 的 settled balances。

> **资金旁白**：这条线不要从撮合引擎开始。钱包里的 `Coin<T>` 先进入 [balance_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/balance_manager.move) 的托管余额，成交后的 settled/owed 记录在 [account.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/state/account.move)，真正资产进出由 [vault.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/vault/vault.move) 收尾。读费用和余额时，始终沿这条资金线核对。

余额类错误最常见于把钱包余额、manager 可用余额、open order 锁定量和 owed balance 混为一谈。交易前检查应分别展示这些数值，并说明哪一项会被本次 PTB 消耗。

## 资金安全判断

- 交易前同时检查 `BalanceManager` 余额、授权 proof、pool version 和费用参数。
- 钱包余额只表示未托管资产；下单可用余额必须来自 `balance_manager.move` 的 manager 余额。
- 涉及费用或返佣时，分别记录 maker/taker、base/quote/DEEP 和当前 epoch 参数。

## 动手检查

- 存入、取出、锁定与结算 中哪一步会读写 `BalanceManager`，哪一步会进入 `Vault`？
- 失败时应优先排查余额不足、cap/proof 权限、pool pause/version 还是治理参数未生效？
- 应用侧需要向用户展示 wallet balance、manager balance、locked balance 中的哪几项？
