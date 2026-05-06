# ch11-09 settled market 的收益领取

[返回本章](README.md)

## 本节目标

- 构造 settled market 的 redeem/permissionless redeem 流程。
- 解释 settlement price、zero-fee redeem 和 manager 余额变化。
- 设计 keeper/自动领取时的迁移状态标注。

## 源码关联

- `packages/predict/sources/predict.move`：`redeem` 与 `redeem_permissionless`。
- `packages/predict/sources/market_key/range_key.move`：settled payout。
- `packages/predict/sources/oracle.move`：settlement price 冻结。

## 正文

到期后，oracle settlement price 冻结。用户可调用 `predict::redeem`，如果 oracle 已 settled，源码走 settled path；也可由 keeper 调 `redeem_permissionless`，把 payout 存入用户 manager。结算 payout 使用 `(lower, higher]`，刚好等于 higher strike 算命中，刚好等于 lower strike 不命中。

应用应为 settled position 提供 claim 操作，也应允许批量 claim。由于当前稳定 Indexer 尚未完成，批量列表需要读 manager position table 或维护自己的事件索引。

应用实现应把这一节绑定到 localnet 仿真和可签名 PTB，而不是绑定到未完成的 Predict Server。所有对象 ID 都来自配置或 setup state，交易提交前先 dry run，并把 gas、wallMs、Move abort 和对象变化写入结果摘要。

版本状态上，本章示例参考 `packages/predict/simulations/*` 与本地 `packages/predict/sources/*`。`PREDICT_MIGRATION.md` 中未完成的 Indexer、Server、部署脚本和 Oracle services 只能作为后续集成点。

## 开发要点

- claim PTB 调用 settled `redeem` 或 permissionless redeem，确认 oracle 已冻结 settlement price。
- 批量领取列表来自 manager position table、交易历史或自建临时索引。
- 领取结果检查 manager 余额增加、position 减少和 zero-fee 结算路径。

## 检查问题

- settled redeem 为什么不应再使用 live fair price？
- permissionless redeem 给 keeper 带来什么便利和边界？
- 没有稳定 Indexer 时，批量 claim 列表如何生成？
