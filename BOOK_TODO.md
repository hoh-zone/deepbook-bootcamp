# DeepBook 开发权威指南 TODO

## 编写约定

- 书稿仓库：`deepbook-bootcamp`
- DeepBook 源码仓库：[MystenLabs/deepbookv3](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0)
- 章节编号：`ch00` 到 `ch15`
- 阅读顺序：以 `book/SUMMARY.md` 的六阶路线为准；文件编号只表示书稿组织，不强制代表学习顺序。
- 小节编号：每章内部使用 `01` 到 `99`，例如 `ch04-07`
- 每章代码目录：`book/chXX/code/`
- 每节代码建议：放在对应章节 `code` 目录下，命名为 `sNN-topic`
- 正文目标：从概念、源码、交易路径、SDK、Indexer、生产应用逐层推进
- 出版质量要求：每章包含学习目标、架构图、源码索引、代码实战、练习题、常见错误、延伸阅读
- 写作调性：高级 Move 案例书 + DeepBook 专题书，每章穿插 Move 技巧、源码旁白和工程提醒，避免写成 API 文档。
- 学习路线：按“先理解、先用起来、再读源码、再做高级协议、最后生产化”组织，不按源码文件复杂度硬排。

## 总体交付 TODO

- [ ] 建立 `book/chXX/README.md` 作为每章正文入口。
- [ ] 建立 `book/chXX/code/` 作为每章示例代码目录。
- [ ] 每章开头补充“本章你将学会什么”。
- [x] 增加 `LEARNING_PATH.md`，把全书改成六阶从易到难路线。
- [x] 每章补充“本章学习阶梯”，用 L1-L5 标注学习深度。
- [x] 每章补充“Move 高阶穿插点”，把 DeepBook 功能映射到 Move 资源、对象、权限和事件。
- [x] 首批重点小节加入“Move 技巧 / 源码旁白 / 工程提醒”提示框。
- [ ] 每章正文绑定 GitHub 源码文件和关键函数。
- [ ] 每章末尾补充“本章检查清单”和“进阶练习”。
- [ ] 所有 Move 代码示例给出可运行命令。
- [ ] 所有 TypeScript SDK 示例给出依赖、环境变量和运行方式。
- [ ] 所有 Indexer 示例给出 PostgreSQL schema、API 请求和响应样例。
- [ ] 最终统一术语表、函数索引、事件索引、表结构索引。

## ch00 DeepBook 是什么

代码目录：`book/ch00/code/`

源码范围：[README.md](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/README.md)、`packages/deepbook/sources/pool.move`、`balance_manager.move`、`vault/vault.move`、`packages/deepbook_margin/sources`、`packages/predict/sources`、`crates/indexer`、`crates/server`

- [x] 01 解释 DeepBook 解决什么问题，以及它为什么是链上 CLOB。
- [x] 02 说明为什么 Sui 对象模型适合链上订单簿。
- [x] 03 建立 DeepBookV3、Margin、Predict、SDK、Indexer、Server 的功能地图。
- [x] 04 梳理 DeepBookV3 spot 的现货交易能力。
- [x] 05 解释 `BalanceManager` 的账户模型、owner、trader、cap 和 proof。
- [x] 06 解释 `Pool`、`Book`、`State`、`Vault` 的分工。
- [x] 07 说明 DEEP token 在费用、staking、治理和 rebate 中的位置。
- [x] 08 明确闪电贷在 `pool.move` 和 `vault.move` 中的位置。
- [x] 09 介绍 DeepBook Margin 的产品边界和源码入口。
- [x] 10 介绍 DeepBook Predict 的产品边界和源码入口。
- [x] 11 解释 SDK、Indexer、Server 的职责差异。
- [x] 12 给出基于 DeepBook 可以构建的应用类型。
- [x] 13 给出不同读者目标的阅读路线。

代码 TODO：

- [x] `s01-feature-map/`：维护 DeepBook 功能域、源码路径和章节映射。

## ch01 DeepBook 与链上订单簿入门

代码目录：`book/ch01/code/`

源码范围：[README.md](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/README.md)、`packages/deepbook/README.md`

- [ ] 01 解释本书定位：面向 Sui Move、SDK、Indexer、前端和后端开发者。
- [ ] 02 说明 DeepBook 是什么：链上中央限价订单簿 CLOB，而不是 AMM。
- [ ] 03 对比 CLOB、AMM、RFQ 和聚合器的交易路径差异。
- [ ] 04 梳理 DeepBookV3、DeepBook Margin、DeepBook Predict 的产品边界。
- [ ] 05 解释为什么 Sui 的对象模型适合高并发交易场景。
- [ ] 06 介绍 DeepBook 的核心用户角色：taker、maker、LP、borrower、liquidator、predict trader。
- [ ] 07 建立资产、订单、池子、余额管理器、事件、索引器的全局心智模型。
- [ ] 08 说明主网、测试网、本地网在本书中的使用方式。
- [ ] 09 安装 Sui CLI、Node.js、pnpm、Rust、PostgreSQL。
- [ ] 10 检查 DeepBook 源码仓库结构。
- [ ] 11 给出本书项目目录规范和代码运行规范。
- [ ] 12 设计第一个读者任务：查询一个 DeepBook 池子的基础信息。
- [ ] 13 补充术语表：pool、base、quote、lot、tick、order id、checkpoint。
- [ ] 14 本章练习：用文字画出一次下单从钱包到链上事件的路径。

代码 TODO：

- [ ] `s01-env-check/`：环境检查脚本，输出 Sui、Node、Rust、PostgreSQL 版本。
- [ ] `s02-repo-map/`：读取 DeepBook 源码目录并生成模块清单。
- [ ] `s03-first-query/`：查询链上 DeepBook 池子对象的最小示例。

## ch02 Sui Move 开发基础

代码目录：`book/ch02/code/`

源码范围：`packages/deepbook/sources/hello_move.move`、`packages/deepbook/Move.toml`

- [ ] 01 解释 Move 的 module、struct、fun、ability。
- [ ] 02 讲清楚 `key`、`store`、`copy`、`drop` 对金融协议对象的影响。
- [ ] 03 介绍 Sui 对象：owned object、shared object、immutable object。
- [ ] 04 解释 `UID`、`ID`、`TxContext`、`Clock` 在合约中的作用。
- [ ] 05 讲解泛型资产类型：`BaseAsset`、`QuoteAsset`、`phantom`。
- [ ] 06 讲解 `Coin<T>` 与 `Balance<T>` 的差异。
- [ ] 07 讲解 `split`、`join`、`into_balance`、`into_coin` 的资金流转意义。
- [ ] 08 讲解动态字段在大规模订单簿和注册表里的用途。
- [ ] 09 介绍事件 `event::emit` 以及为什么 Indexer 依赖事件。
- [ ] 10 介绍 abort code 的设计方式和读源码时的定位方法。
- [ ] 11 讲解 Move 单元测试和 `test_scenario`。
- [ ] 12 说明如何阅读 DeepBook 的 `Move.toml` 依赖和地址映射。
- [ ] 13 本章练习：实现一个最小共享对象计数器。
- [ ] 14 本章练习：实现一个 `Coin<T>` 存取款示例。

代码 TODO：

- [ ] `s01-hello-move/`：最小 Move package。
- [ ] `s02-coin-balance/`：`Coin<T>` 与 `Balance<T>` 转换示例。
- [ ] `s03-shared-object/`：共享对象读写示例。
- [ ] `s04-event-indexing/`：发出事件并用 RPC 查询事件。

## ch03 DeepBookV3 总体架构

代码目录：`book/ch03/code/`

源码范围：`packages/deepbook/sources/pool.move`、`balance_manager.move`、`registry.move`、`state/*`、`book/*`、`vault/*`

- [ ] 01 用一张全景图说明 Pool、Book、State、Vault、BalanceManager、Registry。
- [ ] 02 解释 DeepBookV3 的 package、shared object、capability 布局。
- [ ] 03 讲解 `Pool<BaseAsset, QuoteAsset>` 的泛型资产设计。
- [ ] 04 说明 Pool 为什么是交易入口，Book 为什么是撮合核心。
- [ ] 05 解释 State 负责账户、历史、治理、费用、参数。
- [ ] 06 解释 Vault 负责最终资产保管和结算。
- [ ] 07 解释 BalanceManager 作为交易账户抽象的意义。
- [ ] 08 解释 Registry 如何管理池子注册、版本和权限。
- [ ] 09 讲解 DEEP token 在手续费、staking、rebate 中的位置。
- [ ] 10 梳理一次交易中的对象借用顺序和可并行执行边界。
- [ ] 11 对比 DeepBookV2 到 V3 的关键变化。
- [ ] 12 说明 DeepBookV3 与 Margin、Predict 的依赖关系。
- [ ] 13 建立源码阅读路线：先 Pool，再 Book，再 Vault，再 State。
- [ ] 14 本章练习：根据源码画出下单调用链。

代码 TODO：

- [ ] `s01-architecture-map/`：从源码提取模块依赖关系。
- [ ] `s02-pool-object-query/`：查询 Pool 对象字段。
- [ ] `s03-event-flow/`：按交易 digest 解析 DeepBook 事件。

## ch04 Pool、Book 与撮合源码精读

代码目录：`book/ch04/code/`

源码范围：`packages/deepbook/sources/pool.move`、`book/book.move`、`book/order.move`、`book/order_info.move`、`book/fill.move`

- [ ] 01 解释订单簿中的 bid、ask、price level、queue priority。
- [ ] 02 讲解 `Order` 结构的字段、方向、类型和生命周期。
- [ ] 03 讲解订单 ID 的生成、排序和可查询性。
- [ ] 04 讲解 `Book` 如何保存买卖两侧订单。
- [ ] 05 讲解限价单进入撮合引擎的完整路径。
- [ ] 06 讲解市价单和吃单行为的资金检查。
- [ ] 07 讲解 post-only、immediate-or-cancel、self match 等边界策略。
- [ ] 08 讲解 `Fill` 如何表示一次成交。
- [ ] 09 讲解 `OrderInfo` 如何生成订单和成交事件。
- [ ] 10 讲解部分成交、完全成交、剩余挂单的状态变化。
- [ ] 11 讲解取消单和批量取消的执行路径。
- [ ] 12 讲解 order query 模块如何服务前端查询。
- [ ] 13 结合测试文件分析撮合边界条件。
- [ ] 14 本章练习：手工推演一个三档订单簿的撮合结果。

代码 TODO：

- [ ] `s01-orderbook-simulator/`：用 TypeScript 实现最小订单簿模拟器。
- [ ] `s02-match-trace/`：输入订单列表，输出撮合过程日志。
- [ ] `s03-order-query/`：查询链上订单和本地模拟结果对比。

## ch05 BalanceManager、Vault、费用与治理

代码目录：`book/ch05/code/`

源码范围：`balance_manager.move`、`vault/vault.move`、`state/account.move`、`state/governance.move`、`state/trade_params.move`、`vault/deep_price.move`

- [ ] 01 解释为什么 DeepBook 使用 BalanceManager 而不是直接用钱包余额。
- [ ] 02 讲解 `BalanceManager` 的创建、owner、trade proof。
- [ ] 03 讲解存入、取出、锁定、结算的资金状态。
- [ ] 04 讲解 Vault 在交易后如何处理 `balances_in` 和 `balances_out`。
- [ ] 05 讲解 maker fee、taker fee、protocol fee、rebate 的差异。
- [ ] 06 讲解 DEEP staking 如何影响费用等级。
- [ ] 07 讲解 `trade_params` 中的交易参数。
- [ ] 08 讲解 `deep_price` 如何服务 DEEP 价格计算。
- [ ] 09 讲解治理提案、投票、参数更新的源码路径。
- [ ] 10 讲解 pause、version、maintainer 能力边界。
- [ ] 11 分析资金安全相关 abort code。
- [ ] 12 结合测试讲解余额不一致、权限错误、版本错误。
- [ ] 13 本章练习：设计一张交易费用结算表。
- [ ] 14 本章练习：解释一次成交后 maker 和 taker 的余额变化。

代码 TODO：

- [ ] `s01-balance-manager-create/`：创建 BalanceManager。
- [ ] `s02-deposit-withdraw/`：存入和取出资产。
- [ ] `s03-fee-calculator/`：按参数估算交易费用和 rebate。
- [ ] `s04-governance-query/`：查询治理相关事件。

## ch06 DeepBookV3 Spot 交易开发

代码目录：`book/ch06/code/`

源码范围：`pool.move`、`order_query.move`、`scripts/transactions/createPool.ts`、`scripts/transactions/deepbookMarketMaker.ts`

- [ ] 01 设计一个 Spot 交易应用的最小功能集。
- [ ] 02 讲解交易对发现、池子选择和 base/quote 显示。
- [ ] 03 讲解精度、tick size、lot size、最小下单量。
- [ ] 04 讲解钱包连接和交易签名流程。
- [ ] 05 讲解创建 BalanceManager 和首次充值。
- [ ] 06 讲解限价买单、限价卖单的 PTB 构造。
- [ ] 07 讲解市价买入、市价卖出的 PTB 构造。
- [ ] 08 讲解订单状态刷新和订单簿深度展示。
- [ ] 09 讲解成交历史、个人成交、未成交订单展示。
- [ ] 10 讲解撤单、批量撤单和失败重试。
- [ ] 11 讲解交易前模拟、Gas 预算和错误提示。
- [ ] 12 讲解做市机器人所需的数据和交易循环。
- [ ] 13 本章实战：最小命令行交易终端。
- [ ] 14 本章实战：最小 Web 交易面板。

代码 TODO：

- [ ] `s01-pool-list-cli/`：列出可交易池子。
- [ ] `s02-place-limit-order/`：限价单示例。
- [ ] `s03-place-market-order/`：市价单示例。
- [ ] `s04-cancel-order/`：撤单示例。
- [ ] `s05-mini-terminal/`：CLI 交易终端。

## ch07 DeepBookV3 闪电贷与组合交易

代码目录：`book/ch07/code/`

源码范围：`packages/deepbook/sources/pool.move`、`packages/deepbook/sources/vault/vault.move`、`crates/indexer/src/handlers/flash_loan_handler.rs`

- [ ] 01 明确 DeepBookV3 存在闪电贷，入口在 Pool，底层在 Vault。
- [ ] 02 解释 hot potato 模式为什么能强制同一交易归还资产。
- [ ] 03 讲解 `FlashLoan` 结构：`pool_id`、`borrow_quantity`、`type_name`。
- [ ] 04 讲解 `borrow_flashloan_base` 的完整源码路径。
- [ ] 05 讲解 `borrow_flashloan_quote` 的完整源码路径。
- [ ] 06 讲解 `return_flashloan_base` 的校验逻辑。
- [ ] 07 讲解 `return_flashloan_quote` 的校验逻辑。
- [ ] 08 讲解错误码：数量为零、余额不足、池子错误、类型错误、数量错误。
- [ ] 09 讲解闪电贷事件 `FlashLoanBorrowed` 及 Indexer 表 `flashloans`。
- [ ] 10 对比 DeepBook 闪电贷和 Margin 借贷的本质差异。
- [ ] 11 设计套利、再平衡、批量清算等组合交易场景。
- [ ] 12 讲解闪电贷没有跨交易债务状态的安全边界。
- [ ] 13 本章实战：构造一次借出再归还的 PTB。
- [ ] 14 本章练习：设计一个失败的闪电贷并解释 abort 原因。

代码 TODO：

- [ ] `s01-flashloan-move-wrapper/`：Move wrapper 调用借出和归还。
- [ ] `s02-flashloan-ptb/`：TypeScript PTB 构造示例。
- [ ] `s03-flashloan-event-query/`：查询 `FlashLoanBorrowed` 事件。
- [ ] `s04-flashloan-indexer-row/`：展示 `flashloans` 表记录。

## ch08 DeepBook Margin 协议源码精读

代码目录：`book/ch08/code/`

源码范围：`packages/deepbook_margin/sources/*`、`packages/margin_liquidation/sources/liquidation_vault.move`

- [ ] 01 解释 Margin 与 Spot 的关系：抵押、借贷、交易、清算。
- [ ] 02 讲解 `MarginRegistry` 的注册、版本和应用授权。
- [ ] 03 讲解 `MarginManager` 的账户抽象和仓位管理。
- [ ] 04 讲解 `MarginPool<Asset>` 的供应、借出、还款。
- [ ] 05 讲解 `pool_proxy.move` 如何连接 Margin 和 DeepBook Pool。
- [ ] 06 讲解抵押品和债务如何计入风险比率。
- [ ] 07 讲解 `position_manager.move` 的仓位状态。
- [ ] 08 讲解 `margin_state.move` 的供应、借款、利息累计。
- [ ] 09 讲解 `protocol_config.move` 的风险参数。
- [ ] 10 讲解 `protocol_fees.move` 的费用收取。
- [ ] 11 讲解 oracle 价格输入和 stale price 风险。
- [ ] 12 讲解 TPSL 条件订单。
- [ ] 13 讲解清算入口和 liquidation vault。
- [ ] 14 结合测试分析正常借款、还款、清算、失败边界。
- [ ] 15 本章练习：画出开多和开空的资金路径。
- [ ] 16 本章练习：解释风险比率下降到阈值以下后的处理流程。

代码 TODO：

- [ ] `s01-margin-source-map/`：生成 Margin 模块依赖图。
- [ ] `s02-risk-ratio-calculator/`：实现风险比率计算器。
- [ ] `s03-interest-accrual/`：模拟利息累计。
- [ ] `s04-liquidation-scenario/`：构造清算示例数据。

## ch09 DeepBook Margin 应用开发

代码目录：`book/ch09/code/`

源码范围：`scripts/transactions/marginPrep.ts`、`supplyToMarginPool.ts`、`updateInterestRates.ts`、`fundLiquidationVault.ts`

- [ ] 01 设计一个 Margin 交易应用的功能模块。
- [ ] 02 讲解如何初始化 Margin SDK 配置和 package ID。
- [ ] 03 讲解创建或加载 MarginManager。
- [ ] 04 讲解存入抵押品和提取抵押品。
- [ ] 05 讲解供应资产到 MarginPool。
- [ ] 06 讲解借入 base 或 quote 资产。
- [ ] 07 讲解通过 DeepBook Pool 执行杠杆买入。
- [ ] 08 讲解通过 DeepBook Pool 执行杠杆卖出。
- [ ] 09 讲解部分还款、全部还款和平仓。
- [ ] 10 讲解 TPSL 订单创建、触发和取消。
- [ ] 11 讲解清算机器人如何扫描和执行。
- [ ] 12 讲解 UI 中展示健康度、风险比率、利息、可借额度。
- [ ] 13 讲解 Margin 应用的错误提示和交易前模拟。
- [ ] 14 本章实战：命令行杠杆交易工具。
- [ ] 15 本章实战：Margin 风控面板。

代码 TODO：

- [ ] `s01-create-margin-manager/`：创建 MarginManager。
- [ ] `s02-deposit-collateral/`：存入抵押品。
- [ ] `s03-borrow-and-trade/`：借款并交易。
- [ ] `s04-repay-and-close/`：还款和平仓。
- [ ] `s05-liquidation-bot/`：清算扫描器骨架。

## ch10 DeepBook Predict 协议源码精读

代码目录：`book/ch10/code/`

源码范围：`packages/predict/sources/*`、`packages/predict/tests/*`、`PREDICT_MIGRATION.md`

- [ ] 01 说明 Predict 的版本状态、网络状态和与 DeepBook 的关系。
- [ ] 02 解释预测市场、二元期权、区间市场、LP vault。
- [ ] 03 讲解 `predict.move` 的对外入口。
- [ ] 04 讲解 `predict_manager.move` 的用户账户和头寸管理。
- [ ] 05 讲解 `oracle.move`、`oracle_config.move` 的价格输入。
- [ ] 06 讲解 `vault/vault.move` 的资金池、mint、settle。
- [ ] 07 讲解 `vault/plp.move` 的 LP 份额和收益。
- [ ] 08 讲解 `market_key/range_key.move` 的市场标识。
- [ ] 09 讲解 pricing config、risk config、treasury config。
- [ ] 10 讲解 strike matrix、rate limiter、math helper。
- [ ] 11 讲解 fee reserve 和协议收费路径。
- [ ] 12 讲解 Predict 事件和未来 Indexer 需求。
- [ ] 13 结合测试分析 mint、exercise、settle、withdraw。
- [ ] 14 结合 `PREDICT_MIGRATION.md` 标注已完成和待完成模块。
- [ ] 15 本章练习：设计一个 SUI 价格二元预测市场。
- [ ] 16 本章练习：解释 LP 面临的主要风险。

代码 TODO：

- [ ] `s01-predict-source-map/`：生成 Predict 模块依赖图。
- [ ] `s02-market-key-builder/`：构造市场 key。
- [ ] `s03-pricing-calculator/`：实现简化定价计算器。
- [ ] `s04-vault-accounting/`：模拟 Predict vault 账本。

## ch11 DeepBook Predict 应用开发与仿真

代码目录：`book/ch11/code/`

源码范围：`packages/predict/simulations/*`、`packages/predict/simulations/src/runtime.ts`

- [ ] 01 设计 Predict 应用的信息架构：市场、价格、赔率、头寸、结算。
- [ ] 02 讲解如何创建 Predict market。
- [ ] 03 讲解如何创建或加载 PredictManager。
- [ ] 04 讲解用户如何 deposit quote 资产。
- [ ] 05 讲解 mint binary position 的交易构造。
- [ ] 06 讲解 mint range position 的交易构造。
- [ ] 07 讲解 LP 如何 supply 和 withdraw。
- [ ] 08 讲解 oracle 更新和结算窗口。
- [ ] 09 讲解 settled market 的收益领取。
- [ ] 10 讲解 Predict 前端如何展示风险和收益。
- [ ] 11 讲解仿真脚本如何构造市场路径。
- [ ] 12 讲解仿真结果如何可视化。
- [ ] 13 讲解 Predict 与 Margin/Spot 组合产品的设计边界。
- [ ] 14 本章实战：预测市场 CLI。
- [ ] 15 本章实战：仿真一个价格路径和 LP 收益曲线。

代码 TODO：

- [ ] `s01-create-predict-market/`：市场创建示例。
- [ ] `s02-mint-position/`：mint position 示例。
- [ ] `s03-supply-lp/`：LP supply 示例。
- [ ] `s04-settle-market/`：结算和领取示例。
- [ ] `s05-simulation-report/`：仿真报告生成器。

## ch12 DeepBook SDK 集成专题

代码目录：`book/ch12/code/`

源码范围：`scripts/transactions/*`、`scripts/config/constants.ts`、`@mysten/deepbook-v3` SDK

- [ ] 01 说明 SDK 章的目标：把合约能力封装成应用可用接口。
- [ ] 02 安装和配置 `@mysten/deepbook-v3`。
- [ ] 03 配置 SuiClient、network、package IDs、object IDs。
- [ ] 04 讲解 `DeepBookClient` 的初始化方式。
- [ ] 05 讲解 constants 文件如何管理主网和测试网配置。
- [ ] 06 讲解如何查询池子、资产元数据和对象状态。
- [ ] 07 讲解 BalanceManager SDK 操作。
- [ ] 08 讲解下限价单 SDK 封装。
- [ ] 09 讲解下市价单 SDK 封装。
- [ ] 10 讲解撤单、批量撤单、订单查询 SDK 封装。
- [ ] 11 讲解 swap/route 类交易封装。
- [ ] 12 讲解 Margin SDK 管理员接口和用户接口。
- [ ] 13 讲解 Predict 交易在 SDK 层应如何封装。
- [ ] 14 讲解 PTB 组合、dry run、dev inspect、Gas 预算。
- [ ] 15 讲解钱包适配、签名、提交、确认和错误解析。
- [ ] 16 讲解后端服务如何安全地构造交易而不托管用户私钥。
- [ ] 17 讲解 SDK 单元测试、mock client、fixture。
- [ ] 18 本章实战：封装一个 `DeepBookService`。
- [ ] 19 本章实战：封装一个 `MarginService`。
- [ ] 20 本章实战：封装一个 `PredictService`。

代码 TODO：

- [ ] `s01-sdk-init/`：SDK 初始化模板。
- [ ] `s02-deepbook-service/`：Spot 服务封装。
- [ ] `s03-margin-service/`：Margin 服务封装。
- [ ] `s04-predict-service/`：Predict 服务封装。
- [ ] `s05-wallet-flow/`：前端钱包交易流程。
- [ ] `s06-dry-run-helper/`：dry run 和错误解析工具。

## ch13 DeepBook Indexer、Server 与数据系统

代码目录：`book/ch13/code/`

源码范围：`crates/indexer/*`、`crates/schema/*`、`crates/server/*`、`docker/deepbook-indexer/*`、`docker/deepbook-server/*`

- [ ] 01 解释为什么交易应用不能只依赖链上对象查询。
- [ ] 02 讲解 Sui checkpoint、event、transaction digest 的数据模型。
- [ ] 03 讲解 `sui-indexer-alt` 框架在本项目中的使用方式。
- [ ] 04 讲解 `crates/indexer/src/main.rs` 启动参数。
- [ ] 05 讲解 handler 的通用结构和 `map_event` 模式。
- [ ] 06 讲解 core DeepBook handlers：order fill、order update、pool created、balances。
- [ ] 07 讲解 flash loan handler 和 `flashloans` 表。
- [ ] 08 讲解 Margin handlers：loan borrowed、loan repaid、collateral、liquidation。
- [ ] 09 讲解 schema migrations 和 Diesel model。
- [ ] 10 讲解 PostgreSQL 表设计、索引和查询模式。
- [ ] 11 讲解 server REST API 路由和分页参数。
- [ ] 12 讲解 `/status` 健康检查和 checkpoint lag。
- [ ] 13 讲解 Prometheus metrics 和生产告警。
- [ ] 14 讲解订单簿缓存、K 线、成交聚合、用户历史。
- [ ] 15 讲解 Docker 部署 indexer 和 server。
- [ ] 16 讲解数据一致性：重放、幂等、重复事件、防止漏数据。
- [ ] 17 讲解前端如何消费 server API。
- [ ] 18 讲解自建 Indexer 与使用公共 API 的取舍。
- [ ] 19 本章实战：本地启动 PostgreSQL、Indexer、Server。
- [ ] 20 本章实战：实现一个行情 API 聚合服务。

代码 TODO：

- [ ] `s01-local-postgres/`：本地数据库启动说明和 schema 初始化。
- [ ] `s02-run-indexer/`：运行 indexer 的配置模板。
- [ ] `s03-query-events/`：查询订单、成交、闪电贷、Margin 事件。
- [ ] `s04-kline-builder/`：从成交表生成 K 线。
- [ ] `s05-api-client/`：调用 DeepBook Server API。
- [ ] `s06-monitoring/`：`/status` 和 Prometheus 检查脚本。

## ch14 构建自己的 DeepBook 应用

代码目录：`book/ch14/code/`

源码范围：综合使用 `packages/*`、`scripts/*`、`crates/server/*`

- [ ] 01 说明应用构建路线：协议理解、SDK 交易、Indexer 数据、前端交互。
- [ ] 02 设计 Spot 交易终端的信息架构。
- [ ] 03 设计订单簿和成交流组件。
- [ ] 04 设计下单表单和交易确认状态机。
- [ ] 05 设计资产余额、可用余额、锁定余额展示。
- [ ] 06 设计做市机器人：报价、撤单、库存、风控。
- [ ] 07 设计套利机器人：价格源、路由、闪电贷、利润检查。
- [ ] 08 设计 Margin 应用：仓位、风险率、借款、还款、清算提示。
- [ ] 09 设计 Predict 应用：市场列表、赔率、头寸、结算。
- [ ] 10 设计后端交易服务：配置、日志、dry run、错误码映射。
- [ ] 11 设计数据服务：K 线、深度、成交、用户历史、排行榜。
- [ ] 12 设计风控系统：价格偏离、订单失败率、Indexer 延迟、资金异常。
- [ ] 13 讲解如何把 Spot、Margin、Predict 组合成结构化产品。
- [ ] 14 讲解应用上线前的最小合规和风险披露文案。
- [ ] 15 本章实战：专业交易终端 MVP。
- [ ] 16 本章实战：做市机器人 MVP。
- [ ] 17 本章实战：Margin 仪表盘 MVP。
- [ ] 18 本章实战：Predict 市场 MVP。

代码 TODO：

- [ ] `s01-trading-terminal/`：交易终端 MVP。
- [ ] `s02-market-maker-bot/`：做市机器人骨架。
- [ ] `s03-arbitrage-bot/`：套利机器人骨架。
- [ ] `s04-margin-dashboard/`：Margin 仪表盘。
- [ ] `s05-predict-market-ui/`：Predict 市场 UI。
- [ ] `s06-risk-monitor/`：风险监控服务。

## ch15 测试、安全、部署与出版级交付

代码目录：`book/ch15/code/`

源码范围：`packages/*/tests/*`、`crates/indexer/tests/*`、`docker/*`、`.github/workflows/*`

- [ ] 01 建立 DeepBook 开发测试金字塔：Move 单测、SDK 单测、Indexer 快照、端到端测试。
- [ ] 02 讲解 Move 单元测试的场景组织方式。
- [ ] 03 讲解撮合、余额、费用、清算、结算的边界测试。
- [ ] 04 讲解 Indexer snapshot tests 的价值。
- [ ] 05 讲解 SDK mock、fixture、dry run 测试。
- [ ] 06 讲解前端端到端测试和交易状态机测试。
- [ ] 07 讲解金融协议安全清单：权限、资金、价格源、精度、溢出、重入式组合风险。
- [ ] 08 讲解对象权限和 capability 泄漏风险。
- [ ] 09 讲解版本升级、package upgrade、迁移脚本。
- [ ] 10 讲解部署拓扑：前端、API、Indexer、PostgreSQL、监控。
- [ ] 11 讲解日志、指标、告警、SLO。
- [ ] 12 讲解故障演练：RPC 延迟、Indexer 落后、交易失败、价格异常。
- [ ] 13 讲解审计前准备：威胁模型、资产清单、关键不变量。
- [ ] 14 讲解出版社级书稿标准：术语统一、图表统一、代码可运行、索引完整。
- [ ] 15 讲解每章交付验收标准。
- [ ] 16 编写全书最终项目 README。
- [ ] 17 编写全书代码运行手册。
- [ ] 18 编写附录：函数、事件、表结构、错误码、常见问题。

代码 TODO：

- [ ] `s01-move-test-template/`：Move 测试模板。
- [ ] `s02-sdk-test-template/`：SDK 测试模板。
- [ ] `s03-indexer-snapshot-template/`：Indexer 快照测试模板。
- [ ] `s04-e2e-test-template/`：端到端测试模板。
- [ ] `s05-deployment-compose/`：生产部署 compose 模板。
- [ ] `s06-audit-checklist/`：审计检查脚本和清单。

## 附录 TODO

- [ ] A DeepBookV3 函数速查表。
- [ ] B DeepBookV3 事件速查表。
- [ ] C DeepBook Margin 函数和事件速查表。
- [ ] D DeepBook Predict 函数和事件速查表。
- [ ] E Indexer PostgreSQL 表结构速查表。
- [ ] F SDK 常用代码片段。
- [ ] G Move 错误码和排障索引。
- [ ] H 术语中英文对照表。
- [ ] I 全书代码运行顺序。
- [ ] J 出版前事实核对清单。
