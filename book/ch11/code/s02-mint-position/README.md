# s02 Mint position

目标：构造用户 mint binary position 的应用流程。

步骤：

1. 派生 manager ID：参考 runtime.ts 的 `deriveManagerId(owner, 0n)`。
2. 查询对象，不存在则调用 `registry::create_and_share_manager(registry)`。
3. 将 quote coin 存入 manager：`predict_manager::deposit<Quote>(manager, coin)`。
4. 构造 `RangeKey`：
   - UP: `(strike, POS_INF]`
   - DOWN: `(NEG_INF, strike]`
   - RANGE: `(lower, higher]`
5. 调用 `predict::mint<Quote>(predict, manager, oracle, key, quantity, clock)`。

交易前检查：

- manager owner 等于当前钱包地址。
- quote balance 足够支付 fair value + fee。
- oracle active、未 stale、未 settled。
- strike 按 oracle tick size 对齐。
- all-in ask price 在用户可接受范围内。

仿真源码入口：

- [packages/predict/simulations/src/runtime.ts](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/predict/simulations/src/runtime.ts) 的 `mintTx`。
- [packages/predict/simulations/src/runtime.ts](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/predict/simulations/src/runtime.ts) 的 `refreshOracleAndMintTx`。

本地仿真命令：

```bash
cd deepbookv3
bash packages/predict/simulations/run.sh --setup
```
