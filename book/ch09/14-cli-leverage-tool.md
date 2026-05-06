# ch09-14 实战：命令行杠杆交易工具

[返回本章](README.md)

## 先看用户路径

这里先从用户动作往回看。“实战：命令行杠杆交易工具”在应用里通常是一组 PTB 和状态刷新，不是一条孤立 move call；每一步都要同时考虑余额、债务和风险率。

## 源码入口

重点阅读：

- [scripts/transactions/marginPrep.ts](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/scripts/transactions/marginPrep.ts)
- `book/ch09/code/s01-create-margin-manager/README.md`
- `book/ch09/code/s03-borrow-and-trade/README.md`
- `book/ch09/code/s04-repay-and-close/README.md`

> **源码旁白**：先定位结构体、入口函数和事件，再回到本节的资金路径或应用流程。不要从 helper 函数开始读。

## 把 PTB 串起来

CLI 可以按命令拆分：

```bash
pnpm tsx margin.ts manager create --pool SUI_USDC
pnpm tsx margin.ts collateral deposit --pool SUI_USDC --coin USDC --amount 1000
pnpm tsx margin.ts borrow --pool SUI_USDC --asset USDC --amount 500
pnpm tsx margin.ts trade market --pool SUI_USDC --side buy --quantity 100
pnpm tsx margin.ts repay --pool SUI_USDC --asset USDC --all
pnpm tsx margin.ts state --pool SUI_USDC
```

每个命令共享同一套配置加载、对象查询、dry run 和错误映射。输出应固定包含 transaction digest、changed objects、事件摘要和最新风险率。

## 工程旁白

CLI 的价值在于把复杂 PTB 分解成可重复的命令。每条命令都应支持 `--dry-run-only`，并输出对象 ID、type arguments、gas 估算、事件摘要和风险率变化，便于定位问题。

命令层要明确区分 `collateral deposit`、`pool supply` 和 `borrow`。这三个命令都涉及资金，但分别改变 manager collateral、MarginPool supply shares 和 manager debt shares。

## Margin 应用判断

- 配置读取和对象查询做成共享模块，避免每条命令重复硬编码 ID。
- 输出格式保留 JSON 模式，方便风控面板或测试脚本消费。
- 危险命令如 borrow、trade、withdraw 默认先预览，用户确认后再提交。

## 动手检查

- 这个命令改变 collateral、supply shares 还是 borrow shares？
- CLI 输出是否足够复现失败交易，包括对象 ID 和 abort 映射？
- `state` 命令是否展示利息后的 debt amount，而不只是 borrow shares？
