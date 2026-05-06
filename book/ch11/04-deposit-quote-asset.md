# ch11-04 deposit quote 资产

[返回本章](README.md)

## 本节目标

- 实现 quote 资产 deposit 的 PTB 构造和余额检查。
- 说明 treasury quote whitelist、6 位 decimals 和内部 BalanceManager 的关系。
- 把 deposit 失败解析成 coin、manager、owner 或配置问题。

## 源码关联

- `packages/predict/sources/predict_manager.move`：`deposit<T>` 与内部 BalanceManager。
- `packages/predict/sources/config/treasury_config.move`：quote whitelist 和 decimals。
- `packages/predict/simulations/src/runtime.ts`：coin 选择和 PTB 执行。

## 正文

仿真里的 `depositToManagerTx(managerId, amount)` 先 mint DUSDC 测试币，再调用：

```ts
tx.moveCall({
  target: `${PACKAGE_ID}::predict_manager::deposit`,
  typeArguments: [DUSDC_TYPE],
  arguments: [tx.object(managerId), coin],
});
```

生产环境不会 mint 测试币，而是从钱包 coin 中 split 指定 amount。交易前检查包括 quote asset 是否被 `treasury_config` 接受、decimals 是否为 6、manager owner 是否等于当前地址、manager quote balance 是否足够支付 mint cost。

应用实现应把这一节绑定到 localnet 仿真和可签名 PTB，而不是绑定到未完成的 Predict Server。所有对象 ID 都来自配置或 setup state，交易提交前先 dry run，并把 gas、wallMs、Move abort 和对象变化写入结果摘要。

版本状态上，本章示例参考 `packages/predict/simulations/*` 与本地 `packages/predict/sources/*`。`PREDICT_MIGRATION.md` 中未完成的 Indexer、Server、部署脚本和 Oracle services 只能作为后续集成点。

## 开发要点

- deposit 前检查 quote coin type、coin balance、treasury whitelist 和 6 位 decimals。
- PTB 参数明确 manager object、coin object/merged coin 和 sender。
- 错误提示区分 coin 不足、quote asset 不支持、manager owner 错误。

## 检查问题

- 为什么 deposit 到 PredictManager 后才适合调用 mint？
- quote decimals 与 treasury config 不匹配会影响什么金额计算？
- 如何在 UI 上区分钱包余额和 manager 内部余额？
