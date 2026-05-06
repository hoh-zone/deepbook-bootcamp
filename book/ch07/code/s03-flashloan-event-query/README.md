# s03 FlashLoanBorrowed 事件查询

目标：按事件类型查询 DeepBookV3 闪电贷借出事件。

事件类型：

```text
<DEEPBOOK_PACKAGE_ID>::vault::FlashLoanBorrowed
```

事件字段：

- `pool_id`
- `borrow_quantity`
- `type_name`

运行方式：

```bash
pnpm install
pnpm tsx query-flashloan-events.ts --package 0x... --cursor 0
```

注意事项：

- 只有借出事件，没有 return 事件。
- 只分析成功交易中的事件。
- `type_name` 用于区分借出的是 base 还是 quote，必须和 pool 泛型一起解释。
