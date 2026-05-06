# ch12-13 Predict SDK 封装方式

[返回本章](README.md)

## 先定封装边界

读这一节时把自己当成 SDK 维护者：封装应该暴露业务意图，隐藏重复参数，同时保留 dry run 和错误定位能力。

## 源码入口

- `packages/predict/sources/predict.move`：mint/redeem/supply/withdraw。
- `packages/predict/sources/predict_manager.move`：manager deposit/withdraw/positions。
- `packages/predict/sources/registry.move`、`market_key/range_key.move`：创建对象和 market key。

## SDK 读法

Predict 当前应按 Move 入口封装，不应假设已有与 Spot 完全同形的稳定 SDK。源码在 `packages/predict/sources/predict.move`、`predict_manager.move`、`registry.move`。应用层可以先定义服务接口：

- `createPredictManager`：绑定 `registry::create_predict_manager` 或当前发布版本的 manager 创建入口。
- `deposit`、`withdraw`：绑定 `predict_manager::deposit`、`predict_manager::withdraw`。
- `mint`、`redeem`、`supply`：绑定 `predict.move` 的 `mint`、`redeem`、`supply`。
- `quote`：优先使用只读查询或 dev inspect，不直接提交交易。

Predict 交易必须在配置中显式标注 package ID、registry ID、predict object ID、oracle ID、quote coin type、网络和版本状态。

服务封装建议统一返回 `Transaction` 或交易 bytes、dry run 摘要和可读错误码。后端可以构造 PTB 和校验配置，但用户交易必须由钱包签名；管理员交易则按 `scripts/utils/utils.ts` 的 multisig/dry run 思路固定 gas、expiration 和 cap。

Predict 相关接口必须额外校验 `predictVersionStatus`、package ID、registry ID、predict object ID、oracle ID 和 quote coin type。由于迁移文档未把 Predict SDK、Indexer 或 Server 标为稳定完成，本章只把它们写成 raw Move 封装或未来服务边界。

## 封装判断

- PredictService target 白名单覆盖 registry、predict_manager、predict、oracle。
- 所有方法要求 package ID、registry ID、predict ID、oracle ID、quote coin type、version status。
- quote 优先 dev inspect 或只读模拟，不提交真实交易。

## 动手检查

- `createPredictManager`、`deposit`、`mint` 分别绑定哪些 Move 模块？
- Predict SDK 封装为什么必须暴露版本状态？
- 没有 max cost 参数时 mint 服务如何做用户保护？
