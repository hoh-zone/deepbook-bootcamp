# ch16-02 一条命令拉起本地栈

[返回本章](README.md)

DeepBook Sandbox 的开发入口非常克制：克隆仓库、安装依赖、复制 `.env` 示例，然后执行 `pnpm deploy-all`。第一轮会慢，因为 Docker 镜像、Rust indexer/server 构建、本地 Move package 发布都要完成；第二轮开始，缓存会让反馈快很多。

## 前置条件

官方仓库当前要求的本地工具包括：

| 工具 | 要求 |
| --- | --- |
| Docker Desktop | 需要包含 `docker compose`，建议给 Docker 至少 8 GB 内存。 |
| Node.js | 18 或更高版本。 |
| pnpm | 用于安装 sandbox 工作区依赖和运行脚本。 |
| Sui CLI | README 建议 `1.63.2` 到 `1.64.1` 区间，并用 `sui --version` 核对。 |

这里不要偷懒。Sandbox 是多服务环境，Docker 内存不足、Sui CLI 版本不匹配、没有递归拉取子模块，都会表现成“部署失败”，但根因并不在 DeepBook 合约。

## 最小启动路径

```bash
git clone --recurse-submodules https://github.com/MystenLabs/deepbook-sandbox.git
cd deepbook-sandbox/sandbox
pnpm install
cp .env.example .env
pnpm deploy-all
```

如果只是想快速体验，并接受使用预构建镜像，可以使用：

```bash
pnpm deploy-all --quick
```

看到 `DeepBook Sandbox Ready!` 之后，本地环境已经具备几类能力：Sui RPC、Sui faucet、DeepBook faucet、oracle status、market maker health、DeepBook REST API 和 dashboard。

## 第一次启动要观察什么

不要只盯着 dashboard 页面是否打开。更可靠的启动检查是按依赖顺序看：

1. `docker compose ps`：容器是否都处于健康或运行状态。
2. `curl http://localhost:9010/`：oracle 是否能返回状态和价格。
3. `curl http://localhost:3001/health`：market maker 是否已经开始维护订单。
4. `curl http://localhost:9009/manifest`：是否能看到部署后的 package ID、pool ID、oracle object。
5. `http://localhost:5173`：dashboard 是否能代理和展示服务状态。

这里的顺序很重要。Dashboard 是观察面，不是根因面。页面空白可能是前端问题，也可能是服务未就绪、proxy 失败、faucet 未启动、manifest 没生成，甚至是 localnet 没有起来。

## deploy-all 在做什么

可以把 `pnpm deploy-all` 看成一个开发编排器：

- 启动 Sui localnet 和 PostgreSQL。
- 等待 RPC 与 faucet 可用。
- 导入或生成本地 key，配置 Sui CLI 的 `localnet` 环境。
- 发布 DEEP token、DeepBook、Pyth、USDC、Margin 和 liquidation 相关 package。
- 写入 `.env`、`deployments/localnet.json` 和 `Pub.localnet.toml`。
- 启动 indexer、server、sandbox API、oracle、market maker 和 dashboard。
- 创建本地池子，并给 market maker 准备 BalanceManager 和初始资金。

这条流水线解释了为什么不要绕过它直接执行 `docker compose up`。容器起来不等于 DeepBook 可用，package ID、object ID、oracle object、pool ID 和 pubfile 都必须被正确写入。

## 停止和清理

```bash
pnpm down
```

这个命令适合结束当天实验或重新开始。执行前先保存需要保留的地址、digest、日志和 manifest。默认情况下，本地链状态会被清掉，重新部署后 package ID 和 object ID 可能变化。

## 本节验收

- 能独立完成首次启动并解释第一轮为什么较慢。
- 能说出 `pnpm deploy-all` 和普通 `docker compose up` 的差异。
- 能在 dashboard 之前用命令行判断核心服务是否可用。
