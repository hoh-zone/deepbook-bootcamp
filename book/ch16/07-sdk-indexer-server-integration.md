# ch16-07 SDK、Indexer、Server 集成

[返回本章](README.md)

DeepBook 应用不能只靠一个 SDK，也不能只靠一个 REST API。SDK 负责构造交易，Indexer 负责把链上事件转成可查询读模型，Server 负责把读模型整理成应用可消费的接口。Sandbox 的好处是这三件事在本地同时存在，读者可以用一笔交易贯穿它们。

## 推荐调试闭环

一次交易的本地调试流程可以写成这样：

1. 从 dashboard 或 manifest 获取 package ID、pool ID 和 coin type。
2. 用 SDK 或 dashboard 构造交易，提交到 `http://localhost:9000`。
3. 保存 transaction digest。
4. 用 Sui RPC 查询交易 effects 和 events。
5. 等 indexer 追到对应 checkpoint。
6. 用 DeepBook Server 查询订单、成交、池子或历史数据。
7. 再回到 dashboard 看 UI 是否与读模型一致。

这套流程比“页面能用就行”严格得多，但它能帮你在真实应用里分清责任层。

## SDK 在本地怎么配

本地 SDK 配置至少包含：

```ts
export const localnet = {
  rpcUrl: "http://localhost:9000",
  deepbookServerUrl: "http://localhost:9008",
  faucetUrl: "http://localhost:9009",
  manifestUrl: "http://localhost:9009/manifest",
};
```

实际项目里不要手写所有 ID。更好的方式是启动时读取 manifest，生成本地 network config，再把 config 注入交易服务。这样每次 `pnpm down` 后重新部署，都不会因为旧地址残留而构造错误交易。

## Indexer 和 Server 怎么判断健康

命令行先看容器：

```bash
docker compose ps
docker compose logs -f deepbook-indexer
docker compose logs -f deepbook-server
```

再看 API 和 dashboard。需要注意的是，RPC 成功不等于 Server 已经能查到数据。Indexer 需要读取 checkpoint、解析事件、写 PostgreSQL，Server 才能返回最新读模型。交易刚提交后，页面短暂落后是正常现象；长期落后则要检查 indexer 日志和 checkpoint lag。

## 本地 API 的使用边界

Dashboard 会把 `/api/deepbook` 代理到 `localhost:9008`，但后端服务或脚本可以直接调用 `http://localhost:9008`。对于读者自己的应用，建议把链上查询和 Server 查询分开封装：

- 链上查询用于最终状态、对象内容、交易 effects 和事件核验。
- Server 查询用于订单列表、成交历史、池子数据、聚合展示和页面刷新。

这不是重复劳动，而是交易应用必须具备的双重证据。链上状态解释“发生了什么”，读模型解释“应用现在能展示什么”。

## 本节验收

- 能用一笔交易串起 SDK、RPC、Indexer、Server 和 dashboard。
- 能解释 RPC 成功但 UI 未刷新的原因。
- 能设计一个读取 manifest 后生成本地 network config 的应用初始化流程。
