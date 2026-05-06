# ch16 DeepBook Sandbox 本地开发环境

DeepBook Sandbox 不是另一个“示例项目”。它更像本书前面十五章的本地实验室：一条命令拉起 Sui localnet、发布 DeepBook V3、部署 Margin、接上 oracle、跑起 market maker、启动 indexer/server、打开 dashboard，然后让读者在可重置的环境里验证交易、余额、事件、REST API 和自定义 Move 合约。

这章放在全书后段，是因为它需要读者已经理解 DeepBook 的几条主线：链上对象负责资金和规则，SDK 负责交易构造，Indexer/Server 负责读模型，Dashboard 负责把状态暴露给人。到了 sandbox 这里，这些能力第一次同时出现在一个本地拓扑里。

## 本章目标

- 理解 [MystenLabs/deepbook-sandbox](https://github.com/MystenLabs/deepbook-sandbox) 的定位：降低 DeepBook V3 本地开发、演示、集成和自定义合约实验的摩擦。
- 用 `pnpm deploy-all` 建立可复现的本地 DeepBook 全栈环境。
- 读懂 localnet、PostgreSQL、Indexer、DeepBook Server、Faucet、Oracle、Market Maker、Dashboard 的职责边界。
- 学会把自定义 Move package 依赖到本地已发布的 DeepBook package，并用 `Pub.localnet.toml` 解析地址。
- 建立一套从 dashboard 到命令行、从合约到数据服务的日常调试流程。

## 本章学习阶梯

- L2 把 sandbox 当成一套可重置的本地环境：先跑起来，再看服务是否健康。
- L3 从 dashboard 反推交易流：BalanceManager 创建、充值、下单、成交、事件、读模型。
- L4 进入自定义 Move 合约：依赖本地发布的 DeepBook、编译、发布、调用、失败排查。
- L5 把 sandbox 变成产品开发内环：SDK 调试、Indexer 校验、机器人测试、风险演练和上线前预演。

## 官方仓库基线

| 能力 | 本章如何使用 |
| --- | --- |
| Sui Localnet | 提供私有链、RPC 和原生 faucet，避免污染测试网状态。 |
| DeepBook V3 contracts | 在本地按依赖顺序发布 token、deepbook、pyth、usdc、margin 和 liquidation 相关 package。 |
| PostgreSQL + Indexer | 把 checkpoint 和事件写入数据库，支撑 DeepBook Server 查询。 |
| DeepBook Server | 给 dashboard、脚本和应用提供 REST 读接口。 |
| Oracle Service | 更新本地 Pyth price object，让 market maker 和 Margin 场景有价格输入。 |
| Market Maker | 持续挂 POST_ONLY 网格单，让本地池子有可交易深度。 |
| Faucet | 给测试地址发 SUI、DEEP、USDC。 |
| Dashboard | 用页面检查服务健康、池子、部署地址、faucet、交易动作和 SDK 片段。 |
| Example Contract | 提供依赖 DeepBook 的自定义 Move 合约模板。 |

## 小节目录

- [01 Sandbox 的定位](01-sandbox-positioning.md)
- [02 一条命令拉起本地栈](02-one-command-local-stack.md)
- [03 服务拓扑和端口](03-service-topology-ports.md)
- [04 Dashboard 工作流](04-dashboard-workflow.md)
- [05 部署产物和地址清单](05-contracts-and-deployments.md)
- [06 自定义 Move 合约](06-custom-move-contracts.md)
- [07 SDK、Indexer、Server 集成](07-sdk-indexer-server-integration.md)
- [08 Oracle、Market Maker 和 Faucet](08-oracle-market-maker-faucet.md)
- [09 测试、重置和数据管理](09-testing-reset-data-management.md)
- [10 用 Sandbox 承接本书练习](10-use-sandbox-for-book-exercises.md)
- [11 生产边界和迁移判断](11-production-boundaries.md)
- [12 本章代码](12-chapter-code.md)

## 本章代码

- `code/s01-sandbox-runbook/`：从 clone 到 teardown 的本地运行手册。
- `code/s02-dashboard-checks/`：Dashboard 页面验收清单。
- `code/s03-custom-contract-template/`：自定义 Move 合约模板使用说明。
- `code/s04-service-health-checks/`：服务健康检查、端口和排障命令。

## Move 高阶穿插点

Sandbox 最值得学的不是 Docker，而是“链上依赖如何被本地化”。`Pub.localnet.toml` 记录本地链上已经发布的 package 地址，自定义合约通过 `--pubfile-path` 指向它，就能把 `deepbook`、`token`、`deepbook_margin` 这些依赖解析到真实 localnet 对象上。这个动作让 Move 的编译期依赖、发布地址、运行期 shared object 三件事连到一起，是读懂 Sui 应用开发内环的关键。

## 常见错误

- 没有用 `--recurse-submodules` 克隆，导致 DeepBook 源码子模块缺失。
- 直接跑 `docker compose`，绕过 `pnpm deploy-all`，结果 `.env`、package ID 或 key 没有被正确生成。
- 忘记首次启动会编译 Rust indexer/server，把长时间构建误判为失败。
- 把 `pnpm down` 当作普通停止命令，忽视它会清理链状态和部分自动生成配置。
- 自定义合约没有先执行 `pnpm deploy-all`，`.external-packages/` 尚未生成，依赖无法解析。
- 用 dashboard 的读模型判断交易最终性，却没有核对 transaction digest、checkpoint 和链上对象。

## 本章检查清单

- [ ] 能完整说出 `pnpm deploy-all` 发布了哪些 package、启动了哪些服务。
- [ ] 能解释 dashboard 里的 Trading、Faucet、Deployment 页面分别验证什么。
- [ ] 能用 faucet 给测试地址发 SUI、DEEP、USDC。
- [ ] 能复制 `example_contract`，添加 localnet 环境并用 `test-publish` 发布。
- [ ] 能在重置前保存 package ID、pool ID、oracle object 和测试 digest。
- [ ] 能说明 sandbox 与生产部署的差异。

## 进阶练习

1. 先用 dashboard 创建 BalanceManager、充值 DEEP/SUI，再用命令行或 SDK 查询同一个对象。
2. 修改 market maker 参数，让订单簿深度变化，然后观察 dashboard 与 DeepBook Server 是否一致。
3. 写一个最小 Move wrapper，读取或组合 DeepBook 的交易对象，再发布到 localnet。
4. 用 `pnpm down` 重置后重新部署，比较两次 `deployments/localnet.json` 的地址差异。
