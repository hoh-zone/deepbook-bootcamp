# ch11-03 创建或加载 PredictManager

[返回本章](README.md)

## 先跑通场景

这一节从 Predict 的用户动作切入：先确认用户或 operator 要提交哪些对象和参数，再回到源码看市场状态如何被约束。

## 源码入口

- `packages/predict/sources/registry.move`：manager 创建入口和派生 key。
- `packages/predict/sources/predict_manager.move`：owner、BalanceManager、positions。
- `packages/predict/simulations/src/runtime.ts`：对象 ID 查询、执行重试和状态保存。

## 从仿真到交易

用户交易前需要 `PredictManager`。仿真代码 [packages/predict/simulations/src/runtime.ts](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/predict/simulations/src/runtime.ts) 中 `deriveManagerId(owner, 0n)` 使用 `PredictManagerKey(address, u64)` BCS 派生对象 ID，`createManagerTx()` 调用 `registry::create_and_share_manager`。

前端流程应先按 owner 派生 manager ID 并查询对象；不存在时提示用户创建。创建后，quote coin 不直接传给 `predict::mint`，而是先 deposit 到 manager，交易时由合约从 manager 内部 `BalanceManager` 扣款。

应用实现应把这一节绑定到 localnet 仿真和可签名 PTB，而不是绑定到未完成的 Predict Server。所有对象 ID 都来自配置或 setup state，交易提交前先 dry run，并把 gas、wallMs、Move abort 和对象变化写入结果摘要。

版本状态上，本章示例参考 `packages/predict/simulations/*` 与本地 `packages/predict/sources/*`。`PREDICT_MIGRATION.md` 中未完成的 Indexer、Server、部署脚本和 Oracle services 只能作为后续集成点。

## Predict 应用判断

- 先按 owner 派生 manager ID 并查询对象，不存在再创建。
- 创建 manager 与 deposit 可以组合进同一 PTB，但后续 mint 必须引用正确 manager。
- 缓存 manager 时同时保存 owner、network、package version 和 object version。

## 动手检查

- `PredictManagerKey(address, 0)` 解决了什么查找问题？
- owner 不匹配时交易应在前端、服务层还是 Move 层失败？
- 用户已有 manager 但本地缓存丢失时如何恢复？
