# ch16-04 Dashboard 工作流

[返回本章](README.md)

Dashboard 的价值不是“有个页面好看”，而是把 DeepBook 开发中最容易分散的状态集中起来。它让读者不用一上来就写 SDK、SQL、curl 和钱包脚本，也能看到服务健康、部署地址、faucet、订单簿、BalanceManager 和交易动作。

## 五个页面怎么读

| 页面 | 主要回答的问题 |
| --- | --- |
| Health | localnet、indexer、oracle、market maker、faucet 等服务是否可用。 |
| Market Maker | 本地池子是否有双边报价，网格单是否在持续重平衡。 |
| Trading | 钱包能否连接 localnet，是否能创建 BalanceManager、充值、下单。 |
| Faucet | 测试地址能否拿到 SUI、DEEP、USDC。 |
| Deployment | 当前 localnet 的 package ID、pool ID、oracle object 和部署清单是什么。 |

建议第一次进入 dashboard 时，不要直接下单。先走一遍观察路径：

1. Health 页面确认服务可用。
2. Deployment 页面保存当前 package ID 和 pool ID。
3. Faucet 页面给测试钱包发资产。
4. Trading 页面创建 BalanceManager 并充值。
5. Market Maker 页面观察订单簿是否有深度。
6. 再回到 Trading 页面执行限价单或市价单。

## Trading 页背后的 Move 动作

第一次连接钱包时，页面会引导创建 BalanceManager。这个按钮背后不是一个普通前端状态，而是一组 PTB 动作：创建 `balance_manager::new`，注册 BalanceManager，并把对象共享出来。这样后续交易才能通过注册表发现和复用同一个交易账户。

这正好呼应本书前面反复强调的一个原则：DeepBook 不是直接拿钱包余额下单，而是通过 BalanceManager 管理可用余额、锁定余额和交易权限。Dashboard 把这个原则变成了可点击流程。

## Dashboard 不是黑盒

Dashboard 内联展示 SDK 片段，这一点对学习很重要。读者可以先用页面完成动作，再把对应 SDK 代码迁移到自己的脚本或应用里。推荐的学习方式是：

- 先用页面执行一次成功交易。
- 记录 digest、pool、BalanceManager、coin type 和数量。
- 再用 SDK 构造同一类交易。
- 最后用 Server 或链上查询确认结果。

## 常见误判

- 页面没有立刻刷新，就以为交易失败。先看 digest 和 checkpoint，再看 indexer 是否追上。
- Faucet 发币成功，但 Trading 页仍显示余额不足。检查钱包网络、coin type、BalanceManager 充值动作是否完成。
- Market Maker 没有订单，直接怀疑撮合引擎。先检查 oracle 和 market maker health。
- Deployment 页面地址变化，却还用旧配置调用 SDK。localnet 重置后必须更新配置。

## 本节验收

- 能按 Health、Deployment、Faucet、Trading、Market Maker 的顺序完成一次检查。
- 能解释 BalanceManager 创建按钮背后的 Move 对象变化。
- 能把 dashboard 的一次交易迁移成 SDK 调用的输入参数。
