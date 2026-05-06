# ch15-17 代码运行手册

[返回本章](README.md)

## 先定交付标准

这一节按上线视角读“代码运行手册”。如果某个判断不能被测试、监控或运行手册证明，它就还不是出版级和生产级内容。

## 运行入口

- [book/ch13/code/s02-run-indexer/README.md](../ch13/code/s02-run-indexer/README.md)：启动 Indexer 的配置模板。
- [book/ch14/code/s03-arbitrage-bot/README.md](../ch14/code/s03-arbitrage-bot/README.md)：组合交易和闪电贷机器人骨架。
- [book/ch15/code/s04-e2e-test-template/README.md](code/s04-e2e-test-template/README.md)：端到端测试模板。
- [docker/deepbook-server/Dockerfile](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/docker/deepbook-server/Dockerfile)：Server 容器边界。

## 运行手册的固定格式

每个代码目录都应该按同一套证据组织，而不是只列“模块”。推荐格式：

```text
目标：这个示例验证哪条交易、数据或运维路径。
前置条件：Sui CLI / Node / Rust / Docker / PostgreSQL / sandbox。
环境变量：每个变量的来源、示例值、是否敏感。
启动命令：最小命令，不混入解释。
验证命令：RPC、digest、API、SQL、日志或页面检查。
停止/清理：如何回滚状态，哪些数据会丢失。
失败排查：按链上、SDK、Indexer、Server、UI 分层定位。
```

这套格式的意义是让读者在失败时有证据可追，而不是只能重跑全部步骤。DeepBook 应用涉及资金和共享对象，运行手册必须记录 transaction digest、checkpoint、object ID 和关键日志。

## 分层运行顺序

1. 先跑 `mdbook build`，确认书稿导航和链接可生成。
2. 再跑 ch16 sandbox，获得 localnet、package ID、pool ID 和 faucet。
3. 跑 ch12 SDK 初始化，确认 RPC 与 manifest 可读取。
4. 跑 ch13 Indexer/Server 示例，确认读模型能跟上 checkpoint。
5. 跑 ch14 应用或机器人示例，验证交易构造、状态刷新和风险熔断。
6. 最后跑 ch15 测试模板，把成功和失败证据固化。

不要一开始就跑完整产品。越靠近用户界面的示例，越依赖前面的链上和数据层证据。

## 交付检查

- 每个示例是否有“启动”和“验证”两类命令？
- 失败时是否能从 digest、checkpoint、日志或 SQL 中定位责任层？
- 清理命令会不会删除链状态、数据库或生成配置？README 是否提前说明？
