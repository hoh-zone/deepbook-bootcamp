# s01 BalanceManager 创建

目标：用 PTB 创建 `BalanceManager`，可选注册到 `Registry`，并说明 owner、cap 和 `TradeProof` 的边界。

核心源码：

- `packages/deepbook/sources/balance_manager.move`
- `packages/deepbook/sources/registry.move`

最小调用路径：

1. `balance_manager::new(ctx)` 创建 owner 为发送者的 manager，发出 `BalanceManagerEvent`。
2. `balance_manager::register_balance_manager(&manager, &mut registry, ctx)` 把 manager ID 加入 registry 的 owner 映射。
3. owner 可调用 `mint_trade_cap`、`mint_deposit_cap`、`mint_withdraw_cap`。
4. 交易前调用 `generate_proof_as_owner` 或 `generate_proof_as_trader`，再把 `TradeProof` 传给 `pool.place_limit_order`。

TypeScript 示例建议：

```bash
pnpm install
pnpm tsx create-balance-manager.ts
```

环境变量：

```text
SUI_RPC_URL=https://fullnode.testnet.sui.io:443
DEEPBOOK_PACKAGE_ID=0x...
DEEPBOOK_REGISTRY_ID=0x...
SUI_KEYPAIR=...
```

注意事项：

- `new_with_owner` 和 `new_with_custom_owner_and_caps` 是 deprecated，源码中直接 abort 1337。
- 机器人只需要 `TradeCap`；提现服务才需要 `WithdrawCap`。
- cap 达到上限会触发 `EMaxCapsReached = 4`；撤销不存在 cap 会触发 `ECapNotInList = 5`。
