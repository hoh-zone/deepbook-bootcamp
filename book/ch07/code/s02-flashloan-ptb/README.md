# s02 闪电贷 PTB

目标：用 TypeScript 构造最小借出再归还 PTB。

PTB 步骤：

1. 调用 `pool::borrow_flashloan_base<Base, Quote>`，得到 `[coin, loan]`。
2. 可选插入 swap、套利或再平衡调用。
3. 调用 `pool::return_flashloan_base<Base, Quote>`，传入同一个 pool、精确本金 coin 和 loan。

运行方式：

```bash
pnpm install
pnpm tsx flashloan-ptb.ts --pool 0x... --amount 1000000 --asset base
```

环境变量：

```text
SUI_RPC_URL=https://fullnode.testnet.sui.io:443
DEEPBOOK_PACKAGE_ID=0x...
SUI_KEYPAIR=...
```

开发注意事项：

- 如果中间操作改变 coin 数量，先 split 出本金。
- 归还到另一个 pool 会触发 `EIncorrectLoanPool = 4`。
- 归还数量不等于借出数量会触发 `EIncorrectQuantityReturned = 6`。
