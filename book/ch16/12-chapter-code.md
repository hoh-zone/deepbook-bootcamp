# ch16-12 本章代码

[返回本章](README.md)

本章代码目录不是替代 DeepBook Sandbox 仓库，而是给读者一组随书运行手册。真实源码、脚本和服务以 [MystenLabs/deepbook-sandbox](https://github.com/MystenLabs/deepbook-sandbox) 为准；本书代码目录负责说明怎么用、怎么验收、怎么把 sandbox 连接到前面章节。

## 目录

- [s01-sandbox-runbook](code/s01-sandbox-runbook/README.md)：从 clone 到部署、检查、清理的完整 runbook。
- [s02-dashboard-checks](code/s02-dashboard-checks/README.md)：Dashboard 五个页面的手工验收清单。
- [s03-custom-contract-template](code/s03-custom-contract-template/README.md)：自定义 Move 合约模板使用说明。
- [s04-service-health-checks](code/s04-service-health-checks/README.md)：端口、健康检查和排障命令。

## 使用方式

建议读者先在独立目录克隆 sandbox，不要把官方仓库直接复制进本书 `code/` 目录：

```bash
git clone --recurse-submodules https://github.com/MystenLabs/deepbook-sandbox.git
cd deepbook-sandbox/sandbox
pnpm install
cp .env.example .env
pnpm deploy-all
```

本章 `code/` 下的 README 用来记录“怎么跑”和“怎么判断跑对了”。如果你在自己的项目里改了 SDK、合约或前端，可以把每次实验的 manifest、digest 和日志摘要写进这些目录的笔记中。

## 本章验收顺序

1. 跑通 `s01-sandbox-runbook`。
2. 用 `s02-dashboard-checks` 完成一次页面验收。
3. 用 `s04-service-health-checks` 检查 oracle、market maker、faucet 和 manifest。
4. 最后进入 `s03-custom-contract-template`，发布自己的 Move package。

不要反过来。自定义合约依赖 `.external-packages/` 和 `Pub.localnet.toml`，它们只有在 `pnpm deploy-all` 成功之后才可靠。
