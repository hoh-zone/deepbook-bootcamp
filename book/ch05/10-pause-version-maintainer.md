# ch05-10 pause、version 与 maintainer 能力边界

[返回本章](README.md)

## 本节目标

- 读懂 pause、version 与 maintainer 能力边界 对 BalanceManager、Vault、Account 或费用参数的影响。
- 能画出本节相关 base、quote、DEEP 在 deposit、trade、settle、withdraw 中的资金方向。
- 能把权限、余额、版本和治理参数错误拆成可定位的 Move 模块。

## 源码关联

- [packages/deepbook/sources/balance_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/balance_manager.move)：`BalanceManager` 余额、owner/cap、`TradeProof`、deposit/withdraw 和权限校验。
- [packages/deepbook/sources/state/governance.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/state/governance.move)：治理提案、投票、参数更新、暂停和维护者能力边界。
- [packages/deepbook/sources/state/account.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/state/account.move)：账户维度的 open orders、settled/owed balances、stake、rebate 和成交后状态。
- [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)：交易、stake、claim rebate、治理和 manager 结算的对外入口。

## 正文

DeepBookV3 的版本边界主要在 `registry.move` 和 `pool.move`。`Registry` 维护全局 enabled versions，`enable_version`、`disable_version` 只能由 `DeepbookAdminCap` 调用；禁用当前版本会触发 `ECannotDisableCurrentVersion = 6`。Pool 内部也有 `allowed_versions`，`update_allowed_versions` 是 admin 入口。

创建池和注册池有两条路径：`create_permissionless_pool` 要支付固定 DEEP creation fee，并在 registry 注册；`create_pool_admin` 由 admin 创建，可指定 whitelisted 或 stable 状态。`PoolCreated` 事件记录 `pool_id`、`taker_fee`、`maker_fee`、`tick_size`、`lot_size`、`min_size`、`whitelisted_pool` 和 `treasury_address`。

应用侧不要假设一个池永远可交易。交易前应检查 registry 是否启用版本、pool 是否 registered、以及池对象类型是否与 base/quote 匹配。

补充阅读：pause、version 与 maintainer 能力边界 的资金主线是“钱包 Coin -> [packages/deepbook/sources/balance_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/balance_manager.move) 托管余额 -> [packages/deepbook/sources/state/account.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/state/account.move) 记录成交后的 settled/owed -> [packages/deepbook/sources/vault/vault.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/vault/vault.move) 做实际资产进出”。撮合模块只产生成交结果，真正的资产划转要回到 BalanceManager 和 Vault 这两层看。

治理和维护能力影响的是参数生效、暂停和版本门禁，不应被前端当作普通余额错误处理。PTB dry run 返回这些 abort 时，提示语应引导用户等待版本升级或市场恢复，而不是要求用户继续充值。

## 开发要点

- 交易前同时检查 `BalanceManager` 余额、授权 proof、pool version 和费用参数。
- 钱包余额只表示未托管资产；下单可用余额必须来自 `balance_manager.move` 的 manager 余额。
- 涉及费用或返佣时，分别记录 maker/taker、base/quote/DEEP 和当前 epoch 参数。

## 检查问题

- pause、version 与 maintainer 能力边界 中哪一步会读写 `BalanceManager`，哪一步会进入 `Vault`？
- 失败时应优先排查余额不足、cap/proof 权限、pool pause/version 还是治理参数未生效？
- 应用侧需要向用户展示 wallet balance、manager balance、locked balance 中的哪几项？
