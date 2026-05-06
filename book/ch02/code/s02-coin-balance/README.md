# s02-coin-balance Coin 与 Balance

本示例解释 `Coin<T>` 和 `Balance<T>` 的转换路径，绑定源码 [packages/deepbook/sources/balance_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/balance_manager.move)。

## 关键源码

`deposit<T>` 的资金方向：

```move
public fun deposit<T>(balance_manager: &mut BalanceManager, coin: Coin<T>, ctx: &mut TxContext) {
    balance_manager.emit_balance_event(
        type_name::with_defining_ids<T>(),
        coin.value(),
        true,
    );

    let proof = balance_manager.generate_proof_as_owner(ctx);
    balance_manager.deposit_with_proof(&proof, coin.into_balance());
}
```

`withdraw<T>` 的资金方向：

```move
public fun withdraw<T>(
    balance_manager: &mut BalanceManager,
    withdraw_amount: u64,
    ctx: &mut TxContext,
): Coin<T> {
    let proof = generate_proof_as_owner(balance_manager, ctx);
    let coin = balance_manager.withdraw_with_proof(&proof, withdraw_amount, false).into_coin(ctx);
    balance_manager.emit_balance_event(
        type_name::with_defining_ids<T>(),
        coin.value(),
        false,
    );

    coin
}
```

## 构建命令

```bash
cd deepbookv3/packages/deepbook
sui move build
sui move test
```

## 开发要点

- `Coin<T>` 是钱包和 PTB 输入输出层面的对象。
- `Balance<T>` 是合约内部会计值。
- `into_balance()` 会消费 coin。
- `into_coin(ctx)` 需要 `TxContext` 创建新的 coin 对象。
- 存取款必须伴随事件，DeepBook 使用 `BalanceEvent` 记录 `asset`、`amount`、`deposit`。

## 扩展任务

实现一个独立 Move 模块：

1. 定义 `Vault<phantom T> has key`，字段包含 `id: UID` 和 `balance: Balance<T>`。
2. `deposit<T>(&mut Vault<T>, Coin<T>)` 使用 `into_balance` 后 `join`。
3. `withdraw<T>(&mut Vault<T>, amount, &mut TxContext): Coin<T>` 使用 `split` 和 `into_coin`。
4. 为存取款发出事件。
