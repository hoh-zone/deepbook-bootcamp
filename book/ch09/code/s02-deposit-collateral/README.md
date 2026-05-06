# s02-deposit-collateral

目标：向 `MarginManager` 存入 base、quote 或 DEEP 抵押品，并展示交易前风险率变化。

输入参数：

- `--pool SUI_USDC`
- `--manager 0x...`
- `--coin USDC`
- `--amount 1000`
- `--dry-run`

链上入口：

- [packages/deepbook_margin/sources/margin_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/margin_manager.move)
- `deposit<BaseAsset, QuoteAsset, DepositAsset>`

实现要点：

1. 选择或合并用户 coin object。
2. 传入 base 和 quote Pyth price object。
3. 调用 SDK 的 margin manager deposit 封装，或直接构造 Move call。
4. dry run 后解析 `DepositCollateralEvent`。
5. 重新读取 `manager_state`，显示 base/quote asset 和 risk ratio。

错误提示：

- `EInvalidDeposit`：只能存入 base、quote 或 DEEP。
- `EInvalidMarginManagerOwner`：当前钱包不是 manager owner。
- oracle 错误：价格 object 过期、置信区间过大或 feed ID 不匹配。

运行方式建议：

```bash
pnpm tsx index.ts --pool SUI_USDC --coin USDC --amount 1000 --dry-run
```
