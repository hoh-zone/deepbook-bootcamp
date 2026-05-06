# ch14 构建自己的 DeepBook 应用

## 本章目标

- 把前面章节的协议、SDK 和 Indexer 能力组合成真实应用。
- 设计 Spot、Margin、Predict、做市、套利和风控系统。
- 建立从交易构造、数据查询、状态确认到生产监控的工程闭环。

## 本章学习阶梯

- L1 先选应用类型：交易终端、做市机器人、Margin 面板、Predict 市场或数据服务。
- L2 搭出最小信息架构和核心交易流程。
- L3 接入 SDK、Indexer、Server、风控和错误状态机。
- L5 做到可上线 MVP，能解释资金、安全和数据边界。

## 源码地图

- [packages/deepbook](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook)：Spot 交易和闪电贷底层能力。
- [packages/deepbook_margin](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin)：Margin 抵押、借贷、清算。
- [packages/predict](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/predict)：Predict 市场和仿真。
- [scripts/transactions](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/scripts/transactions)：TypeScript 交易脚本样例。
- [crates/server](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/crates/server)：行情和历史数据 API。

## 小节目录

- [01 应用构建路线](01-app-build-route.md)
- [02 Spot 交易终端信息架构](02-spot-terminal-architecture.md)
- [03 订单簿和成交流组件](03-orderbook-trades-components.md)
- [04 下单表单和交易确认状态机](04-order-form-state-machine.md)
- [05 余额展示](05-balance-display.md)
- [06 做市机器人](06-market-maker-bot.md)
- [07 套利机器人与闪电贷](07-arbitrage-bot-flashloans.md)
- [08 Margin 应用](08-margin-app.md)
- [09 Predict 应用](09-predict-app.md)
- [10 后端交易服务](10-backend-transaction-service.md)
- [11 数据服务](11-data-service.md)
- [12 风控系统](12-risk-system.md)
- [13 组合结构化产品](13-structured-products.md)
- [14 上线前披露](14-launch-risk-disclosures.md)
- [15 专业交易终端 MVP](15-professional-terminal-mvp.md)
- [16 做市机器人 MVP](16-market-maker-mvp.md)
- [17 Margin 仪表盘 MVP](17-margin-dashboard-mvp.md)
- [18 Predict 市场 MVP](18-predict-market-mvp.md)

## 本章代码

- `code/s01-trading-terminal/`：交易终端 MVP。
- `code/s02-market-maker-bot/`：做市机器人骨架。
- `code/s03-arbitrage-bot/`：套利机器人骨架。
- `code/s04-margin-dashboard/`：Margin 仪表盘。
- `code/s05-predict-market-ui/`：Predict 市场 UI。
- `code/s06-risk-monitor/`：风险监控服务。

## Move 高阶穿插点

- 构建应用时，先定义你依赖的 Move 不变量，再设计页面、任务队列和后端服务。
- 协议组合要画资源流图：哪些 coin 进入、哪些对象被借用、哪些资源必须归还或销毁。
- 生产应用的高级能力不是功能堆叠，而是把失败路径、权限边界和资金安全讲清楚。

## 常见错误

- 前端直接信任单一数据源，不做交易 digest 校验。
- 把钱包余额当成 DeepBook 可用余额。
- 做市机器人没有库存上限。
- Margin UI 不展示清算阈值。
- Predict UI 不标注 oracle 和结算规则。

## 本章检查清单

- [ ] 能画出交易从 UI 到 PTB、链上、Indexer、UI 刷新的路径。
- [ ] 能区分交易服务和数据服务。
- [ ] 能说明机器人暂停条件。
- [ ] 能说明每类应用的最小可用版本。

## 进阶练习

- 为交易终端设计一个错误码到用户提示的映射表。
- 为做市机器人设计一个库存偏移后的报价调整公式。
- 为 Margin 仪表盘设计一个风险率颜色和告警规则。


