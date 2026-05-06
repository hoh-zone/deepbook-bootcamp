# ch05 BalanceManager、Vault、费用与治理

## 本章目标

读完本章后，你应该能把一笔 DeepBookV3 交易拆成三层资金路径：用户资产先进入 `BalanceManager`，撮合产生的应收应付记录在 `Account`，最后由 `Vault` 和 `BalanceManager` 做实际资产划转。你还应该能解释 maker fee、taker fee、rebate、DEEP staking 和治理参数之间的关系，并能在应用侧提前检查余额、版本、权限和常见 abort code。

## 本章学习阶梯

- L2 先区分钱包余额、manager 余额、locked balance 和 settled amounts。
- L3 读 `BalanceManager`、`Vault`、费用、staking 和治理源码。
- L4 能设计 owner/trader/cap/proof 的权限模型。
- L5 能为交易终端或做市系统设计资金安全检查。

## 源码地图

- `packages/deepbook/sources/balance_manager.move`：用户资金账户、owner、cap、`TradeProof`、存取款事件。
- `packages/deepbook/sources/vault/vault.move`：池子资产金库、交易结算、闪电贷底层借还。
- `packages/deepbook/sources/state/account.move`：每个 `BalanceManager` 在池子内的订单、成交、stake、rebate、settled/owed 状态。
- `packages/deepbook/sources/state/governance.move`：费用参数提案、投票、epoch 更新。
- `packages/deepbook/sources/state/trade_params.move`：`taker_fee`、`maker_fee`、`stake_required` 和用户折扣规则。
- `packages/deepbook/sources/vault/deep_price.move`：DEEP 与 base/quote 的换算价格，用于 DEEP 支付手续费。
- `packages/deepbook/sources/pool.move`：交易、stake、治理、claim rebate、burn DEEP 的对外入口。

## 小节目录

- [01 为什么使用 BalanceManager](01-why-balance-manager.md)
- [02 创建、owner 与 TradeProof](02-owner-and-trade-proof.md)
- [03 存入、取出、锁定与结算](03-deposit-withdraw-lock-settle.md)
- [04 Vault 如何结算 balances_in 与 balances_out](04-vault-settlement-balances.md)
- [05 maker fee、taker fee、protocol fee 与 rebate](05-fees-and-rebates.md)
- [06 DEEP staking 如何影响费用等级](06-deep-staking-fee-tiers.md)
- [07 trade_params 中的交易参数](07-trade-params.md)
- [08 deep_price 如何服务 DEEP 价格计算](08-deep-price.md)
- [09 治理提案、投票与参数更新](09-governance-proposals-voting.md)
- [10 pause、version 与 maintainer 能力边界](10-pause-version-maintainer.md)
- [11 资金安全相关 abort code](11-fund-safety-abort-codes.md)
- [12 余额不一致、权限错误、版本错误](12-balance-permission-version-errors.md)

## 本章代码

- `book/ch05/code/s01-balance-manager-create/`：创建 manager、注册到 registry、生成 proof 和 cap。
- `book/ch05/code/s02-deposit-withdraw/`：deposit、withdraw、withdraw_all 的资金路径。
- `book/ch05/code/s03-fee-calculator/`：按 trade params、DEEP price、订单方向估算手续费。
- `book/ch05/code/s04-governance-query/`：查询治理参数、proposal、`TradeParamsUpdateEvent`。

## Move 高阶穿插点

- `BalanceManager` 是学习 Move 权限设计的核心案例：owner、trader、cap、proof 分别解决不同信任边界。
- `Vault` 展示了协议资产保管的正确姿势：撮合模块不直接转 coin，结算模块统一处理余额变化。
- 费用和治理章节要把经济规则落回 Move 类型和事件，避免只停留在产品说明。

## 常见错误

- 把钱包余额当作可下单余额。
- 忘记为 `pay_with_deep = true` 准备 DEEP 余额和价格点。
- 在 permissionless settlement 中尝试结算仍有 owed 的账户。
- 认为 stake 后立即获得折扣；实际要看 active stake、成交量和 epoch。
- 给机器人同时发放 `TradeCap` 和 `WithdrawCap`。

## 本章检查清单

- 能说明 `settled` 和 `owed` 分别由谁支付。
- 能按成交方向解释 base、quote、DEEP 的流向。
- 能列出 BalanceManager、Vault、Governance 的关键 abort code。
- 能区分当前参数和下一 epoch 参数。
- 能为交易前检查设计余额、版本、proof、价格点四类校验。

## 进阶练习

1. 设计一张交易费用结算表，列出 maker/taker、买/卖、DEEP/input fee 四种组合。
2. 手工推演一次卖单成交后 maker 和 taker 的 `settled_balances` 与 `owed_balances`。
3. 写一个监控脚本，订阅 `BalanceEvent`、`OrderFilled`、`TradeParamsUpdateEvent`，重建某个 manager 的资金流水。

