# ch11-05 mint binary position 的交易构造

[返回本章](README.md)

## 本节目标

- 构造 mint binary position 的 PTB，覆盖 UP/DOWN sentinel、oracle freshness 和 fee。
- 把 fair price、fee、all-in ask 和 vault exposure 展示给用户。
- 说明二元 mint 在当前版本下依赖GitHub 源码入口而非稳定 SDK。

## 源码关联

- `packages/predict/sources/predict.move`：`mint<Quote>` 与 `quote_mint_amounts`。
- `packages/predict/sources/market_key/range_key.move`：UP/DOWN sentinel。
- `packages/predict/simulations/src/sim.ts`：oracle refresh + mint 的 scenario 执行。

## 正文

仿真 `mintTx` 把二元方向转换成 `RangeKey`：UP 使用 `(strike, POS_INF]`，DOWN 使用 `(NEG_INF, strike]`。随后调用：

```ts
tx.moveCall({
  target: `${PACKAGE_ID}::predict::mint`,
  typeArguments: [DUSDC_TYPE],
  arguments: [
    tx.object(predictId),
    tx.object(managerId),
    tx.object(oracleId),
    key,
    tx.pure.u64(quantity),
    tx.object("0x6"),
  ],
});
```

交易前必须 dev inspect 或本地重算 quote，确认 all-in ask 没超过用户 slippage bound。当前 `predict::mint` 源码没有把用户自定义 max cost 作为参数，因此应用层要么用 PTB 前模拟，要么等待协议/SDK 增加保护参数后再面向生产。

应用实现应把这一节绑定到 localnet 仿真和可签名 PTB，而不是绑定到未完成的 Predict Server。所有对象 ID 都来自配置或 setup state，交易提交前先 dry run，并把 gas、wallMs、Move abort 和对象变化写入结果摘要。

版本状态上，本章示例参考 `packages/predict/simulations/*` 与本地 `packages/predict/sources/*`。`PREDICT_MIGRATION.md` 中未完成的 Indexer、Server、部署脚本和 Oracle services 只能作为后续集成点。

## 开发要点

- UP/DOWN 只改变 `RangeKey` sentinel，不改变 `predict::mint` 入口。
- mint 前用 dev inspect 或本地 quote 计算 all-in ask，并与用户 slippage bound 比较。
- dry run 摘要记录 fair price、fee、manager withdraw、vault payment 和 position increase。

## 检查问题

- UP 和 DOWN 的 lower/higher sentinel 分别如何设置？
- 当前 `mint` 没有 max cost 参数时，应用如何保护用户？
- 二元 mint 失败最可能来自 oracle stale、余额不足还是 ask bound？
