# ch16-09 测试、重置和数据管理

[返回本章](README.md)

Sandbox 的核心优点是可重置，核心风险也是可重置。它允许你每天从干净 localnet 开始，快速排除旧状态干扰；但如果你没有保存证据，重置会让 package ID、object ID、交易状态和数据库内容一起消失。

## 日常命令

```bash
cd deepbook-sandbox/sandbox

pnpm deploy-all
pnpm deploy-all --quick
docker compose ps
docker compose logs -f
docker compose logs -f market-maker
docker compose logs -f oracle-service
pnpm down
```

集成测试可用：

```bash
pnpm test:integration
pnpm test:integration <pattern>
```

## 重置前保存什么

每次准备 `pnpm down` 前，建议保存四类证据：

| 证据 | 为什么要保存 |
| --- | --- |
| `deployments/localnet.json` | 记录本轮 package、pool、oracle object 和部署关系。 |
| 交易 digest | 后续复盘交易 effects、events 和 abort 需要。 |
| 自定义合约源码 | localnet 会重置，源码不会；但发布地址会变。 |
| 关键日志 | indexer、server、oracle、market maker 的失败原因可能只在日志里。 |

`pnpm down` 会停止并清理容器、卷、链状态和自动生成的部分配置。它不会删除你的源码，但会让本地链重新开始。重新部署后，旧地址不能继续作为应用配置。

## 测试策略

把 sandbox 作为测试环境时，不要只写 happy path。至少覆盖：

- Faucet 成功和余额不足。
- BalanceManager 创建、重复发现、充值、提现。
- 限价单、市价单、post-only 失败、撤单。
- Oracle 不可用时 market maker 的行为。
- Indexer 落后时前端如何显示 pending。
- 重置后旧配置是否会被拒绝或自动刷新。

这些用例不一定都要写成自动化测试。对书稿和早期项目，runbook 加手工验收也有价值。关键是每一步能复现，有命令，有预期，有失败解释。

## 数据管理提醒

PostgreSQL 是读模型，不是资金事实源。重置数据库不会改变链上事实；重置 localnet 会让链上事实本身消失。开发者要明确自己当前是在清理读模型、清理容器，还是重建整条链。

这个区分在生产环境更重要。生产不能用“重置”解决数据问题，只能用重放、迁移、补偿任务、备份恢复和人工审计。

## 本节验收

- 能说明 `pnpm down` 会清理什么、保留什么。
- 能为一次本地交易保存 digest、manifest 和日志。
- 能设计 sandbox 的最小回归测试清单。
