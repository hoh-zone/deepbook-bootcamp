# ch11-04 deposit quote 资产

[返回本章](README.md)

## 先跑通场景

deposit 不是“给页面充值”这么简单。Predict 的用户资金先进入 `PredictManager`，后续 mint、redeem、settle 都围绕 manager 内部余额和头寸表发生。钱包余额、manager 余额和 vault 余额是三层不同状态，任何一个混在一起都会导致金额展示和错误提示失真。

## 源码入口

- `packages/predict/sources/predict_manager.move`：`deposit<T>` 与内部 BalanceManager。
- `packages/predict/sources/config/treasury_config.move`：quote whitelist 和 decimals。
- `packages/predict/simulations/src/runtime.ts`：coin 选择和 PTB 执行。

## 从仿真到交易

仿真里的 `depositToManagerTx(managerId, amount)` 先 mint DUSDC 测试币，再调用：

```ts
tx.moveCall({
  target: `${PACKAGE_ID}::predict_manager::deposit`,
  typeArguments: [DUSDC_TYPE],
  arguments: [tx.object(managerId), coin],
});
```

生产环境不会 mint 测试币，而是从钱包 coin 中 split 指定 amount。交易前检查包括 quote asset 是否被 `treasury_config` 接受、decimals 是否为 6、manager owner 是否等于当前地址、manager quote balance 是否足够支付 mint cost。

应用层要把 deposit 做成一个明确的账户动作：用户选择 coin、输入金额、钱包签名、链上确认、manager balance 刷新。只有 manager 余额刷新后，mint 面板才应该把这笔资金计入可用 quote。

本节仍然参考 localnet 仿真和当前源码，不依赖未完成的 Predict Server。对象 ID 来自配置或 setup state，提交前先 dry run，并把 gas、wallMs、Move abort 和对象变化写入结果摘要。

## Predict 应用判断

- deposit 前检查 quote coin type、coin balance、treasury whitelist 和 6 位 decimals。
- PTB 参数明确 manager object、coin object/merged coin 和 sender。
- 错误提示区分 coin 不足、quote asset 不支持、manager owner 错误。
- UI 上分开展示 wallet quote balance、manager quote balance 和预计 mint cost。

## 动手检查

- 为什么 deposit 到 PredictManager 后才适合调用 mint？
- quote decimals 与 treasury config 不匹配会影响什么金额计算？
- 如何在 UI 上区分钱包余额和 manager 内部余额？
