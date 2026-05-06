# ch16-05 部署产物和地址清单

[返回本章](README.md)

在 Sui Move 里，发布不是最后一步，发布之后的地址管理才是应用开发真正开始的地方。DeepBook Sandbox 会把本地部署过程中产生的 package ID、object ID、pool ID 和 oracle object 写入文件与服务，让后续合约、SDK、dashboard 和脚本能共享同一份事实。

## 关键产物

| 产物 | 用途 |
| --- | --- |
| `sandbox/.env` | 给容器和脚本读取 package ID、object ID、key、RPC、market maker 参数。 |
| `sandbox/deployments/localnet.json` | 记录本次 localnet 部署的地址、池子、oracle 等 manifest。 |
| `sandbox/Pub.localnet.toml` | 给 Sui CLI 和自定义 Move 合约解析本地已发布 package。 |
| `.external-packages/` | `pnpm deploy-all` 生成的本地依赖目录，自定义合约会引用它。 |

这些文件不是普通配置。它们连接了三类世界：Move 编译器看到的依赖、localnet 上真实发布的地址、应用运行时需要传入的对象 ID。

## 部署顺序的含义

Sandbox 的部署不是随便发布几个 package。它需要按依赖关系推进：

1. 发布 token，得到 DEEP coin 类型和 treasury。
2. 发布 core DeepBook，得到订单簿和 BalanceManager 相关能力。
3. 发布 Pyth 和 USDC，为价格与稳定币测试准备对象。
4. 发布 Margin 和 liquidation 相关 package。
5. 创建 pool、oracle object、margin pool 和注册关系。
6. 写入 manifest，启动 indexer/server/oracle/market maker。

这个顺序是 Move 开发中很好的教材。一个后发布的 package 如果依赖前一个 package，就不能只靠源码路径；在本地链上，它还要知道前一个 package 的发布地址。

## manifest 怎么用

最直接的读取方式：

```bash
curl http://localhost:9009/manifest
```

拿到 manifest 后，不要急着全部复制到应用配置里。先区分：

- package ID：交易构造和 Move call 需要。
- shared object ID：Pool、Registry、BalanceManager、oracle object 等运行时对象需要。
- coin type：泛型 type argument 需要。
- checkpoint 起点：Indexer 或数据校验需要。

对 SDK 应用而言，最重要的是把 package ID、pool ID、coin type、RPC URL 放进同一份 network config。对自定义合约而言，最重要的是让 `Move.toml` 和 `Pub.localnet.toml` 同步。

## 重置后的地址风险

执行 `pnpm down` 或重新部署后，localnet 状态会重建，很多地址会变化。读者应该养成一个习惯：每次开始实验，都先把当前 manifest 视为新的事实源。旧 digest 可以用来写学习笔记，但旧 object ID 不应继续进入新交易。

## 本节验收

- 能说明 `.env`、`localnet.json`、`Pub.localnet.toml` 的区别。
- 能解释为什么本地发布后的地址会影响 Move 编译和 SDK 调用。
- 能从 manifest 中挑出 SDK、合约和 indexer 分别需要的字段。
