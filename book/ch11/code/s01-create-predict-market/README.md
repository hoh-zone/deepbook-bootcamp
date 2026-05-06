# s01 Create Predict market

目标：记录创建 Predict market 的 PTB/交易顺序。参考 [packages/predict/simulations/src/runtime.ts](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/predict/simulations/src/runtime.ts) 和 `sim.ts`。

前置状态：

- 已发布 `deepbook_predict` package。
- 已有 `Registry` 和 `AdminCap`。
- 已注册 `PLP` treasury cap。
- quote asset 是 6 decimals，并有 `Currency<Quote>`。

交易顺序：

1. `registry::create_predict<Quote>(registry, admin_cap, currency, plp_treasury_cap, clock)`。
2. `registry::enable_quote_asset<Quote>(predict, admin_cap, currency)`。
3. `registry::create_oracle_cap(admin_cap)` 并转给 operator。
4. `registry::set_asset_feed_id(predict, admin_cap, "BTC", feed_id)`。
5. 可选：`registry::set_asset_basis_bounds(...)`。
6. `registry::create_oracle(registry, predict, oracle_cap, "BTC", expiry, min_strike, tick_size, clock)`。
7. `oracle::activate(oracle, oracle_cap, clock)`。

本地仿真命令：

```bash
cd deepbookv3
bash packages/predict/simulations/run.sh --setup
```

注意：迁移计划中的部署脚本和 oracle services 尚未完成，本示例是 localnet/runtime 参考。
