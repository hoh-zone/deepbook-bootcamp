# ch16-10 用 Sandbox 承接本书练习

[返回本章](README.md)

前面章节的练习分散在 Move、SDK、Indexer、应用和部署里。Sandbox 可以把它们收束成一个本地学习闭环：读源码，跑本地链，执行交易，看事件，查读模型，改前端或脚本，再重置环境重来。

## 对应全书路线

| 本书章节 | Sandbox 中的练习方式 |
| --- | --- |
| ch02 Move 基础 | 用自定义合约模板练习 `Move.toml`、environment、发布和对象借用。 |
| ch04 撮合源码 | 观察 market maker 的 POST_ONLY 挂单，再手动吃单，复盘 fill。 |
| ch05 BalanceManager | 用 dashboard 创建、充值、交易，理解账户余额与钱包余额的差异。 |
| ch07 闪电贷 | 在 localnet 中验证组合交易 wrapper 和失败路径。 |
| ch09 Margin 应用 | 利用本地 oracle、USDC、SUI pool 和 margin package 做风险演练。 |
| ch12 SDK 集成 | 从 dashboard SDK 片段迁移到自己的 TypeScript 服务。 |
| ch13 Indexer/Server | 用交易 digest 对照 event、PostgreSQL 和 REST response。 |
| ch14 应用构建 | 把 dashboard 当参考产品，拆出自己的交易终端或机器人 MVP。 |
| ch15 测试部署 | 用 reset、logs、manifest、integration tests 形成交付证据。 |

## 推荐学习节奏

第一轮只跑通，不改代码。确认 dashboard、faucet、market maker 和交易页面都能工作。

第二轮开始记录证据。每做一个动作，记录输入对象、交易 digest、事件、读模型响应和 UI 变化。

第三轮改自定义合约。先写 wrapper，不急着做策略；先证明依赖、发布、对象传参和错误定位都清楚。

第四轮改应用。把 dashboard 里的流程拆成你自己的 SDK service、API client 和 UI 状态机。

第五轮做破坏性测试。停掉 oracle、让 indexer 落后、使用旧 manifest、让 faucet 地址错误，观察系统如何失败。

## 让练习不像文档

真正的练习不应该只是“运行命令得到输出”。可以把每个练习改成一个小问题：

- 为什么创建 BalanceManager 要注册并共享对象？
- 为什么 market maker 用 POST_ONLY，而不是普通限价单？
- 为什么交易成功后 UI 仍可能短暂看不到成交？
- 为什么自定义合约发布需要 `Pub.localnet.toml`？
- 为什么重置 localnet 后旧 SDK 配置会变成危险输入？

这些问题能把 DeepBook 学习从“会调用”推进到“懂系统”。

## 本节验收

- 能把本书至少五章的练习迁移到 sandbox。
- 能为每个练习写出链上证据、读模型证据和 UI 证据。
- 能设计一个专属于自己项目的 sandbox 学习路线。
