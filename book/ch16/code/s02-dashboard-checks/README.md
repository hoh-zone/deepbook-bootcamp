# s02 Dashboard Checks

Dashboard 是本地环境的操作台。手工验收建议按下面顺序完成。

## 页面清单

| 页面 | 检查点 |
| --- | --- |
| Health | localnet、indexer、server、oracle、market maker、faucet 是否可用。 |
| Deployment | package ID、pool ID、oracle object 是否已写入。 |
| Faucet | 能否给测试地址发 SUI、DEEP、USDC。 |
| Trading | 能否连接钱包、创建 BalanceManager、充值和下单。 |
| Market Maker | 是否有双边报价，订单是否随时间重平衡。 |

## 记录模板

```text
date:
manifest source: http://localhost:9009/manifest
wallet:
balance manager:
pool:
tx digest:
expected UI result:
server/indexer observation:
notes:
```

## 判断

页面成功只是第一层证据。重要交易要同时记录 digest、事件、Server response 和 dashboard 状态。
