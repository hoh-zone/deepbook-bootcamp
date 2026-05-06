# s03 Supply LP

目标：说明 Predict LP 如何向 vault 提供 quote 资产并获得 PLP 份额。

## 交易顺序

1. 准备 quote coin。
2. 调用 `predict::supply` 或对应封装入口。
3. 传入 Predict object、quote coin 和 `Clock`。
4. 检查交易返回对象和事件。
5. 从 Indexer 或 RPC 更新 LP 份额展示。

## 风险展示

LP UI 不能只展示存入金额，还要展示 vault MTM、max payout、withdrawal limiter 和结算状态。Predict LP 不是无风险存款。

本地仿真命令：

```bash
cd deepbookv3
bash packages/predict/simulations/run.sh --setup
```

验收时记录 supply 交易 digest、PLP 份额、vault quote balance 和 max payout。
