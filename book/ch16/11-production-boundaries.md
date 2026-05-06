# ch16-11 生产边界和迁移判断

[返回本章](README.md)

Sandbox 适合开发内环，不适合伪装成生产环境。它的价值是快、完整、可重置；生产环境的价值是稳定、可审计、可恢复、权限清晰。两者目标不同，不能混用。

## 哪些结论可以从 Sandbox 带走

可以带走：

- PTB 构造是否合理。
- BalanceManager、Pool、coin type、shared object 的传参是否正确。
- 自定义 Move 合约能否依赖并调用 DeepBook。
- 事件是否能被 indexer 读取并进入读模型。
- 前端状态机是否能处理 pending、success、abort 和 indexer lag。
- 做市、faucet、oracle、server 之间的交互是否符合预期。

不能直接带走：

- 主网 package ID 和对象地址。
- 生产 oracle 安全假设。
- 生产 RPC 性能与稳定性结论。
- 真实流动性和滑点结论。
- 安全审计结论。
- 生产部署拓扑和权限管理结论。

## 从 Sandbox 到测试网

迁移到测试网前，至少重做这些检查：

1. 替换 network config、package ID、pool ID、coin type 和 server URL。
2. 移除本地 faucet 假设，改用测试网资产获取流程。
3. 检查钱包网络切换和签名提示。
4. 检查真实 RPC 延迟、交易确认时间和失败重试。
5. 检查 Indexer/Server 是否使用目标网络的数据源。
6. 重新跑 dry run、失败路径和边界值测试。

## 从测试网到生产

生产环境要额外关注：

- key 和权限：deployer、oracle、market maker、后端服务不能共用高权限 key。
- 数据：PostgreSQL 需要备份、迁移、重放、只读账号和告警。
- 服务：RPC、Indexer、Server、前端、交易服务要有健康检查和回滚策略。
- 风控：机器人暂停条件、价格偏离阈值、最大仓位、最大下单量必须可配置。
- 审计：自定义合约、后端交易构造、SDK 参数和权限边界都要有审查证据。

## 一个实用判断

如果某个问题在 sandbox 里都说不清，绝不要带到测试网；如果某个问题只能在测试网或生产暴露，就不要用 sandbox 的成功来证明它不存在。好的工程判断不是把一个环境用到底，而是知道每个环境能证明什么。

## 本节验收

- 能列出 sandbox 可以证明和不能证明的事项。
- 能写出从 sandbox 到测试网的迁移清单。
- 能说明生产环境还需要哪些安全、数据和运维证据。
