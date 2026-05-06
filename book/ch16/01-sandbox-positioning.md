# ch16-01 Sandbox 的定位

[返回本章](README.md)

DeepBook 的学习有一个很现实的断层：只读 Move 源码，很难感受到交易系统的完整形态；直接接主网或测试网，又会被地址、gas、RPC、Indexer 延迟、测试资金和环境污染拖住。DeepBook Sandbox 填补的是这个断层。它把协议、服务、数据、页面和测试资产放进一个本地、可重置、可观察的环境里。

这也是它和本书其他代码目录的区别。前面的章节更像“局部实验”：一次下单、一个 BalanceManager、一个 handler、一段 SDK 封装。Sandbox 是“全链路实验”：同一个动作会同时经过 localnet、Move package、shared object、Indexer、PostgreSQL、Server、Dashboard 和日志系统。

## 它适合解决什么问题

| 场景 | Sandbox 的价值 |
| --- | --- |
| 第一次集成 DeepBook | 不需要先整理主网 package ID 和测试资产，先在本地跑通交易流。 |
| SDK 调试 | 可以反复构造 PTB、dry run、提交交易，并立即清理状态重来。 |
| Indexer/Server 调试 | 本地 checkpoint 和事件稳定可控，适合验证读模型。 |
| 自定义 Move 合约 | 本地已经发布 DeepBook 依赖，可以测试 wrapper、组合交易和权限边界。 |
| 演示和培训 | Dashboard 能直接展示服务健康、池子、faucet、部署地址和交易动作。 |

## 它不替代什么

Sandbox 不替代测试网验收，也不替代生产部署。它默认是本地私有链，地址会变，资产是测试资产，oracle 和 market maker 是为开发便利服务的模拟组件。用它验证逻辑和工程路径没有问题，用它证明生产安全就不够。

真正的边界应该这样划：

- 用 Sandbox 验证交易构造、合约依赖、事件读模型和应用交互。
- 用测试网验证外部 RPC、钱包、网络延迟、真实对象地址和升级流程。
- 用生产环境验证监控、告警、SLO、权限管理、备份和事故响应。

## Move 开发提醒

Move 合约开发最容易忽略“地址来自哪里”。在本地 sandbox 中，DeepBook 不是一个抽象依赖，而是已经发布到 localnet 的 package。自定义合约依赖 `deepbook` 时，编译器需要本地源码路径，发布时还需要 localnet 上已经存在的 package 地址。`Pub.localnet.toml` 正是把这两件事连起来的文件。

读这一章时，不要把 Docker 当主角。真正的主角是：一个 Move package 如何从源码、依赖、发布地址、shared object、事件，再进入应用读模型。

## 本节验收

- 能解释为什么本书需要一个全栈 sandbox 章。
- 能区分 sandbox、测试网、生产环境的职责。
- 能说出 sandbox 对 Move 合约依赖解析和 DeepBook 应用开发的意义。
