# ch12-12 Margin SDK 管理员接口和用户接口

[返回本章](README.md)

## 先定封装边界

SDK 小节先看封装边界：好的服务层不是隐藏 Move，而是减少对象、精度、权限和网络配置错误，同时保留 dry run 与错误定位能力。

## 源码入口

- `scripts/transactions/marginPrep.ts`、`supplyToMarginPool.ts`、`enableMarginVersion.ts`：管理员/供应方脚本。
- `packages/deepbook_margin/sources/margin_manager.move`：用户入口。
- `packages/deepbook_margin/sources/margin_pool.move`：资金池 supply/withdraw。

## SDK 读法

Margin 管理员路径来自 `scripts/transactions/marginPrep.ts`、`enableMarginVersion.ts`、`supplyToMarginPool.ts`：

- `marginAdmin.enableVersion`：启用 Margin 版本。
- `marginAdmin.newPythConfig`、`addConfig`、`removeConfig`：维护价格源配置。
- `marginMaintainer.newProtocolConfig`、`createMarginPool`：创建资金池和利率参数。
- `marginPool.supplyToMarginPool`：使用 supplier cap 向池子供给资产。

用户路径绑定 `packages/deepbook_margin/sources/margin_manager.move`，重点封装 `new`、`deposit`、`withdraw`、`borrow_base`、`borrow_quote`、`repay_base`、`repay_quote`、`add_conditional_order` 和风险查询。每个用户交易都必须先检查 oracle 新鲜度、价格容忍度、风险率和可借额度。

服务封装建议统一返回 `Transaction` 或交易 bytes、dry run 摘要和可读错误码。后端可以构造 PTB 和校验配置，但用户交易必须由钱包签名；管理员交易则按 `scripts/utils/utils.ts` 的 multisig/dry run 思路固定 gas、expiration 和 cap。

Predict 相关接口必须额外校验 `predictVersionStatus`、package ID、registry ID、predict object ID、oracle ID 和 quote coin type。由于迁移文档未把 Predict SDK、Indexer 或 Server 标为稳定完成，本章只把它们写成 raw Move 封装或未来服务边界。

## 封装判断

- 管理员方法强制检查 admin/maintainer/supplier cap 和版本。
- 用户 borrow/repay 前检查 oracle freshness、风险率、可借额度和 margin pool 状态。
- Margin 服务和 Predict 服务共享配置风格，但对象和风险模型分开。

## 动手检查

- Margin admin、maintainer、supplier 分别控制哪些操作？
- 用户 borrow 前最关键的风险检查是什么？
- 为什么不能把 Predict position 直接当作 Margin 风控输入？
