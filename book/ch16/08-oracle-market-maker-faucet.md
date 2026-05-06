# ch16-08 Oracle、Market Maker 和 Faucet

[返回本章](README.md)

Sandbox 中最容易被低估的三个服务是 oracle、market maker 和 faucet。它们看起来像“辅助工具”，但没有它们，本地 DeepBook 很快会变成一个空壳：没有价格输入，没有订单簿深度，没有测试资产，读者只能看合约，无法真正交易。

## Oracle Service

Oracle service 会维护本地 price object，让 SUI、DEEP、USDC 等资产在 localnet 上有可读价格。它的健康检查入口是：

```bash
curl http://localhost:9010/
```

对 Margin 和 market maker 来说，价格不是页面装饰，而是风控和报价的输入。Oracle 如果停止更新，market maker 可能停止合理报价，Margin 场景也无法正确演练风险。

## Market Maker

Market maker 维护本地池子的双边报价。官方 README 描述的默认策略是网格型：围绕 oracle mid price 在买卖两边放置多个 POST_ONLY 限价单，并按固定间隔撤单、重算、重新挂单。

健康检查：

```bash
curl http://localhost:3001/health
```

查看日志：

```bash
docker compose logs -f market-maker
```

这对学习撮合引擎很有用。你可以先看 market maker 如何挂 maker-only 订单，再用自己的 taker 订单吃掉一部分深度，然后观察事件、余额和 Server 读模型。

## Faucet

DeepBook faucet 与 Sui localnet 原生 faucet 不完全相同。原生 faucet 负责 SUI gas，DeepBook faucet 还能分发 DEEP 和 USDC。

请求 DEEP：

```bash
curl -X POST http://localhost:9009/faucet \
  -H "Content-Type: application/json" \
  -d '{"address":"0x<your-address>","token":"DEEP","amount":1000}'
```

请求 USDC：

```bash
curl -X POST http://localhost:9009/faucet \
  -H "Content-Type: application/json" \
  -d '{"address":"0x<your-address>","token":"USDC","amount":1000}'
```

请求 SUI：

```bash
curl -X POST http://localhost:9009/faucet \
  -H "Content-Type: application/json" \
  -d '{"address":"0x<your-address>","token":"SUI"}'
```

## 工程判断

这三个服务共同支撑“可交易性”：

- Faucet 解决测试资金。
- Oracle 解决价格输入。
- Market maker 解决订单簿流动性。

如果本地交易失败，不要只看合约。先问自己：钱包有 gas 吗？BalanceManager 里有资产吗？池子有对手单吗？oracle 是否更新？market maker 是否健康？

## 本节验收

- 能用 faucet 给测试地址发 SUI、DEEP 和 USDC。
- 能检查 oracle 与 market maker 健康状态。
- 能解释为什么本地订单簿需要自动做市服务。
