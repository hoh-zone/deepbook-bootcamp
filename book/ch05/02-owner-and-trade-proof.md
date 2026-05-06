# ch05-02 创建、owner 与 TradeProof

[返回本章](README.md)

## 本节目标

- 读懂 创建、owner 与 TradeProof 对 BalanceManager、Vault、Account 或费用参数的影响。
- 能画出本节相关 base、quote、DEEP 在 deposit、trade、settle、withdraw 中的资金方向。
- 能把权限、余额、版本和治理参数错误拆成可定位的 Move 模块。

## 源码关联

- [packages/deepbook/sources/balance_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/balance_manager.move)：`BalanceManager` 余额、owner/cap、`TradeProof`、deposit/withdraw 和权限校验。
- [packages/deepbook/sources/state/account.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/state/account.move)：账户维度的 open orders、settled/owed balances、stake、rebate 和成交后状态。
- [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)：交易、stake、claim rebate、治理和 manager 结算的对外入口。

## 源码定义

`BalanceManager` 本体和交易证明：

```move
public struct BalanceManager has key, store {
    id: UID,
    owner: address,
    balances: Bag,
    allow_listed: VecSet<ID>,
}

public struct TradeProof has drop {
    balance_manager_id: ID,
    trader: address,
}
```

`TradeProof` 的生成入口：

```move
public fun generate_proof_as_owner(
    balance_manager: &BalanceManager,
    ctx: &TxContext,
): TradeProof

public fun generate_proof_as_trader(
    balance_manager: &BalanceManager,
    trade_cap: &TradeCap,
    ctx: &TxContext,
): TradeProof
```

`owner` 可以直接生成 proof。非 owner 必须持有仍在 `allow_listed` 里的 `TradeCap`。这就是 DeepBook 权限模型的核心：交易权限和提款权限可以分开，机器人可以交易但不应能提现。

## 正文

`balance_manager::new(ctx)` 创建 owner 为 `ctx.sender()` 的 manager，并发出 `BalanceManagerEvent { balance_manager_id, owner }`。`new_with_custom_owner(owner, ctx)` 允许为指定地址创建。应用若要由托管服务或机器人代操作，应使用 cap 模型：`mint_trade_cap`、`mint_deposit_cap`、`mint_withdraw_cap` 只能由 owner 调用，内部会把 cap ID 放入 allow list。

交易需要 `TradeProof`。owner 可调用 `generate_proof_as_owner(balance_manager, ctx)`；被授权交易者调用 `generate_proof_as_trader(balance_manager, trade_cap, ctx)`。`TradeProof` 只包含 `balance_manager_id` 和 `trader`，池子在结算时调用 `validate_proof`，校验 proof 指向当前 manager。错误码包括 `EInvalidOwner = 0`、`EInvalidTrader = 1`、`EInvalidProof = 2`、`ECapNotInList = 5`。

> **Move 技巧**：`TradeProof` 不是“登录态”，而是交易内可验证的权限证据。它把“谁签名”和“谁有资格代表这个 `BalanceManager` 交易”拆开，适合机器人、托管服务和多交易员系统。

生产应用应把 owner 操作和 trader 操作分离。owner 适合充值、提现和权限管理；做市机器人只应持有 `TradeCap`，不应持有 `WithdrawCap`。

补充阅读：创建、owner 与 TradeProof 的资金主线是“钱包 Coin -> [packages/deepbook/sources/balance_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/balance_manager.move) 托管余额 -> [packages/deepbook/sources/state/account.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/state/account.move) 记录成交后的 settled/owed -> [packages/deepbook/sources/vault/vault.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/vault/vault.move) 做实际资产进出”。撮合模块只产生成交结果，真正的资产划转要回到 BalanceManager 和 Vault 这两层看。

权限路径要区分 owner、`TradeCap`、`WithdrawCap` 和 `DepositCap`。应用给机器人做授权时通常只需要交易能力，不应同时授予提款能力；否则撮合之外的资金提取风险会扩大。

## 开发要点

- 交易前同时检查 `BalanceManager` 余额、授权 proof、pool version 和费用参数。
- 钱包余额只表示未托管资产；下单可用余额必须来自 `balance_manager.move` 的 manager 余额。
- 涉及费用或返佣时，分别记录 maker/taker、base/quote/DEEP 和当前 epoch 参数。

## 检查问题

- 创建、owner 与 TradeProof 中哪一步会读写 `BalanceManager`，哪一步会进入 `Vault`？
- 失败时应优先排查余额不足、cap/proof 权限、pool pause/version 还是治理参数未生效？
- 应用侧需要向用户展示 wallet balance、manager balance、locked balance 中的哪几项？
