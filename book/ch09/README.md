# ch09 DeepBook Margin 应用开发

## 本章目标

- 把 ch08 的协议结构落到 TypeScript SDK、PTB 构造、交易前检查和 UI 风控展示。
- 能创建或加载 `MarginManager`，完成抵押、供应、借款、交易、TPSL、还款和平仓流程。
- 能设计清算扫描器和 Margin 风控面板，正确展示健康度、风险率、利息和可借额度。

## 本章学习阶梯

- L1 先从用户动作理解 Margin 应用：抵押、借款、交易、还款、清算。
- L2 用 SDK/PTB 封装 manager 创建、抵押、借款和杠杆交易。
- L3 把 UI 风险提示映射回 Move assert、oracle 和风险参数。
- L4 做出清算机器人或 Margin 风控面板。

## 源码地图

- [scripts/transactions/marginPrep.ts](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/scripts/transactions/marginPrep.ts)：管理员初始化脚本，展示 Margin SDK 配置、Pyth config、MarginPool 创建、DeepBook Pool 注册启用和借贷池授权。
- [scripts/transactions/supplyToMarginPool.ts](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/scripts/transactions/supplyToMarginPool.ts)：供应资产到 MarginPool 的脚本，调用 `client.deepbook.marginPool.supplyToMarginPool`。
- [scripts/transactions/updateInterestRates.ts](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/scripts/transactions/updateInterestRates.ts)：维护者更新利率参数和池配置的脚本。
- [scripts/transactions/fundLiquidationVault.ts](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/scripts/transactions/fundLiquidationVault.ts)：给 liquidation vault 注资的脚本。
- [packages/deepbook_margin/sources/margin_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/margin_manager.move)：应用侧用户操作最终落到的账户入口。
- [packages/deepbook_margin/sources/pool_proxy.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/pool_proxy.move)：Margin 下单和撤单的链上入口。
- [packages/margin_liquidation/sources/liquidation_vault.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/margin_liquidation/sources/liquidation_vault.move)：清算机器人和后台资金库的链上入口。

## 小节目录

- [01 Margin 交易应用模块设计](01-margin-app-modules.md)
- [02 初始化 Margin SDK 配置](02-margin-sdk-config.md)
- [03 创建或加载 MarginManager](03-create-or-load-margin-manager.md)
- [04 存入和提取抵押品](04-deposit-withdraw-collateral.md)
- [05 供应资产到 MarginPool](05-supply-to-margin-pool.md)
- [06 借入 base 或 quote](06-borrow-base-or-quote.md)
- [07 通过 DeepBook Pool 执行杠杆买入](07-leveraged-buy-through-deepbook.md)
- [08 通过 DeepBook Pool 执行杠杆卖出](08-leveraged-sell-through-deepbook.md)
- [09 部分还款、全部还款和平仓](09-repay-and-close.md)
- [10 TPSL 订单](10-tpsl-orders.md)
- [11 清算机器人](11-liquidation-bot.md)
- [12 UI 风控展示](12-risk-ui.md)
- [13 错误提示和交易前模拟](13-errors-and-dry-run.md)
- [14 实战：命令行杠杆交易工具](14-cli-leverage-tool.md)
- [15 实战：Margin 风控面板](15-margin-risk-dashboard.md)

## 本章代码

- `code/s01-create-margin-manager/`：创建或加载 `MarginManager` 的 CLI 骨架。
- `code/s02-deposit-collateral/`：存入抵押品并做 dry run。
- `code/s03-borrow-and-trade/`：借款并通过 `pool_proxy` 下单。
- `code/s04-repay-and-close/`：还款、取消订单、结算和平仓流程。
- `code/s05-liquidation-bot/`：清算扫描器骨架，连接 risk ratio、vault 余额和清算入口。

## Move 高阶穿插点

- Margin 应用开发要把用户动作拆成链上对象变化，而不是只按页面按钮设计流程。
- 每个风险提示都应该能追溯到 Move 里的某个 assert、配置字段或 oracle 条件。
- 清算机器人要同时懂 Move 入口、Indexer 延迟、oracle 新鲜度和 vault 资产可用性。

## 常见错误

- 在用户前端暴露 admin cap 或 maintainer cap。普通用户不需要这些 cap。
- 把 `supplyToMarginPool` 当成保证金充值。它是借贷池供应，不是 manager collateral。
- 只查 manager 余额，不查 open order 和 settled amounts。风险率可能与 UI 显示余额不一致。
- 不做 dry run，直接提交复杂 PTB。Margin 交易依赖多个 shared object、oracle 和风险参数，失败面很多。
- 忽略 `Option<u64>` 语义。还款和平仓中 none 与 some 的行为不同。
- 清算机器人只看风险率，不看 vault 是否有对应 debt asset。

## 本章检查清单

- [ ] 能区分用户配置、管理员配置和维护者配置。
- [ ] 能创建或加载某个 `(owner, poolKey)` 的 `MarginManager`。
- [ ] 能解释抵押、供应、借款三种资金动作的区别。
- [ ] 能用 SDK 构造借 quote 后买 base 的 PTB。
- [ ] 能用 SDK 构造借 base 后卖 base 的 PTB。
- [ ] 能处理 TPSL 的创建、触发、取消和失败事件。
- [ ] 能设计一个清算扫描器，并说明何时调用 `liquidate_base` 或 `liquidate_quote`。

## 进阶练习

- 给 `s03-borrow-and-trade` 增加 `--dry-run-only` 参数，输出风险率变化但不提交交易。
- 给 `s04-repay-and-close` 增加 `--reduce-only` 模式，在 Pool 被禁用时仍允许用户降风险。
- 给 `s05-liquidation-bot` 增加 vault 资产再平衡提示：当 USDC 不足但 SUI 充足时，生成 `swap_base_to_quote` 交易建议。
- 为 UI 错误提示写一张 abort code 映射表，并把每个错误绑定到用户可执行的下一步。

