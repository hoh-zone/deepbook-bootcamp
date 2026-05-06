# ch12 DeepBook SDK 集成专题

## 本章目标

本章把前面章节中的 Move 合约能力落到应用工程：用 `@mysten/deepbook-v3`、Sui SDK 和 PTB 把 DeepBook Spot、Margin、Predict 的核心交易封装成可复用服务。目标不是堆 API 清单，而是建立一套生产应用可以采用的 SDK 边界：配置集中管理、交易只构造不托管私钥、提交前 dry run、失败后能解析错误并定位到 Move 入口。

本章代码放在 `book/ch12/code/`，每个目录都是可以扩展成 TypeScript 工程的骨架。

## 本章学习阶梯

- L1 先安装 SDK、连接 RPC、读取配置。
- L2 查询池子、资产、BalanceManager 和对象状态。
- L3 封装限价单、市价单、swap、撤单、Margin 和 Predict 交易。
- L4 加入 dry run、错误解析、钱包签名和后端安全构造交易。

## 源码地图

- `scripts/config/constants.ts`：主网、测试网的 admin cap、margin cap、supplier cap、market maker 地址等配置来源。
- `scripts/transactions/deepbookMarketMaker.ts`：继承 `DeepBookClient`，封装 keypair、`SuiClient`、签名提交、限价单和闪电贷 PTB。
- `scripts/transactions/createPool.ts`：用 `SuiGrpcClient().$extend(deepbook(...))` 初始化 SDK，并调用 `deepBookAdmin.createPoolAdmin`。
- `scripts/transactions/prepBalanceManager.ts`：配置 `balanceManagers`，调用 `balanceManager.createAndShareBalanceManager`。
- `scripts/transactions/marginPrep.ts`、`supplyToMarginPool.ts`、`enableMarginVersion.ts`：Margin admin、maintainer、supplier 侧 SDK 调用。
- `scripts/utils/utils.ts`：读取 signer、设置 gas payment、`dryRunTransactionBlock`、生成 multisig 交易 bytes。
- `packages/deepbook/sources/pool.move`：Spot 交易入口，例如 `place_limit_order`、`place_market_order`、`swap_exact_base_for_quote`、`cancel_order`、`borrow_flashloan_base`。
- `packages/deepbook/sources/balance_manager.move`：`BalanceManager`、deposit、withdraw、cap 和 trade proof。
- `packages/deepbook_margin/sources/margin_manager.move`：Margin 用户入口，例如 deposit、withdraw、borrow、repay、条件单和风险查询。
- `packages/deepbook_margin/sources/margin_pool.move`：Margin 资金池 supply、withdraw、费用和利率状态。
- `packages/predict/sources/predict.move`、`predict_manager.move`、`registry.move`：Predict 当前 Move 层入口。Predict 需要按实际发布网络和对象 ID 标注，不应把迁移中的能力写成稳定主网接口。

## 小节目录

- [01 SDK 章的边界](01-sdk-scope.md)
- [02 安装和配置](02-install-and-configure.md)
- [03 SuiClient、network 和对象配置](03-sui-client-network-objects.md)
- [04 DeepBookClient 初始化](04-deepbook-client-init.md)
- [05 constants 管理主网和测试网](05-constants-mainnet-testnet.md)
- [06 查询池子、资产元数据和对象状态](06-query-pools-assets-objects.md)
- [07 BalanceManager 操作](07-balance-manager-operations.md)
- [08 限价单封装](08-limit-order-wrapper.md)
- [09 市价单封装](09-market-order-wrapper.md)
- [10 撤单、批量撤单和订单查询](10-cancel-orders-and-query.md)
- [11 swap/route 封装](11-swap-route-wrapper.md)
- [12 Margin SDK 管理员接口和用户接口](12-margin-sdk-interfaces.md)
- [13 Predict SDK 封装方式](13-predict-sdk-wrapper.md)
- [14 PTB、dry run、dev inspect 和 Gas](14-ptb-dryrun-devinspect-gas.md)
- [15 钱包签名、提交、确认和错误解析](15-wallet-sign-submit-confirm-errors.md)
- [16 后端安全构造交易](16-secure-backend-transaction-building.md)
- [17 SDK 单元测试、mock client 和 fixture](17-sdk-tests-mocks-fixtures.md)
- [18 本章实战：DeepBookService](18-deepbook-service.md)
- [19 本章实战：MarginService](19-margin-service.md)
- [20 本章实战：PredictService](20-predict-service.md)

## 本章代码

- `book/ch12/code/s01-sdk-init/`：SDK 初始化模板。
- `book/ch12/code/s02-deepbook-service/`：Spot 服务封装。
- `book/ch12/code/s03-margin-service/`：Margin 服务封装。
- `book/ch12/code/s04-predict-service/`：Predict 服务封装。
- `book/ch12/code/s05-wallet-flow/`：前端钱包交易流程。
- `book/ch12/code/s06-dry-run-helper/`：dry run 和错误解析工具。

## Move 高阶穿插点

- SDK 集成的本质是把 Move 函数签名翻译成类型安全的交易构造器。
- 每个 wrapper 都应保留 type arguments、object ids、coin 输入输出和 dry run 结果，方便定位失败。
- 不要让 SDK 隐藏关键资源语义；用户至少要知道哪些对象被 mutably borrowed，哪些 coin 被消费。

## 常见错误

- 把主网对象 ID 用在测试网交易中。
- 前端要求用户导出私钥，而不是调用钱包签名。
- 忽略 `BalanceManager` 所有权和 cap，导致交易 dry run 通过不了。
- 只检查 SDK 抛错字符串，不解析 `effects.status.error`。
- Predict 未标注网络和版本状态，把迁移中的接口写成稳定接口。
- 管理员交易不设置 expiration 和固定 gas payment，导致 multisig 流程不可复现。

## 本章检查清单

- [ ] 所有服务初始化都能明确 `network`、`rpcUrl`、`address`。
- [ ] 所有 cap、pool、manager、registry 对象 ID 都来自配置层。
- [ ] 用户交易返回 PTB 或 bytes，不要求后端托管私钥。
- [ ] 每个写交易都有 dry run 和错误解析路径。
- [ ] Margin 交易检查 oracle、风险率和版本。
- [ ] Predict 交易标注 package/object IDs、网络和版本状态。

## 进阶练习

1. 为 `DeepBookService` 增加 route swap，并在 dry run 后校验最小输出。
2. 为 `MarginService` 增加 `borrowAndPlaceLimitOrder` 组合 PTB。
3. 为 `PredictService` 增加 quote 缓存和 oracle stale 检查。
4. 用 fixture 写一个 dry run 错误解析测试表，覆盖 Move abort、对象不存在、余额不足和 gas 不足。

