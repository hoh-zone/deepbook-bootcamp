# s06-dry-run-helper

本示例封装 dry run 和错误解析工具。它适用于 Spot、Margin 和 Predict 的所有写交易。

## 源码对应

[scripts/utils/utils.ts](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/scripts/utils/utils.ts) 中的 `prepareMultisigTx` 会：

- 设置 sender。
- 设置 gas price 和 expiration。
- 设置 gas payment。
- 构建交易 bytes。
- 调用 `dryRunTransactionBlock`。
- 失败时停止输出 multisig bytes。

应用服务也应遵循这个顺序。

## dry run helper

```ts
import type { SuiClient } from "@mysten/sui/client";
import type { Transaction } from "@mysten/sui/transactions";

export type DryRunOk = {
  ok: true;
  digest?: string;
  gasUsed: string;
  balanceChanges: unknown[];
  objectChanges: unknown[];
  raw: unknown;
};

export type DryRunFail = {
  ok: false;
  code: string;
  message: string;
  moveAbort?: {
    package?: string;
    module?: string;
    function?: string;
    abortCode?: string;
  };
  raw: unknown;
};

export async function dryRunOrExplain(
  client: SuiClient,
  tx: Transaction,
): Promise<DryRunOk | DryRunFail> {
  const bytes = await tx.build({ client });
  const result = await client.dryRunTransactionBlock({ transactionBlock: bytes });
  const status = result.effects.status;

  if (status.status === "success") {
    return {
      ok: true,
      gasUsed: JSON.stringify(result.effects.gasUsed),
      balanceChanges: result.balanceChanges ?? [],
      objectChanges: result.objectChanges ?? [],
      raw: result,
    };
  }

  return {
    ok: false,
    ...parseSuiExecutionError(status.error ?? "transaction failed"),
    raw: result,
  };
}
```

## 错误解析

```ts
export function parseSuiExecutionError(error: string) {
  if (error.includes("InsufficientCoinBalance")) {
    return { code: "INSUFFICIENT_BALANCE", message: error };
  }

  if (error.includes("ObjectNotFound")) {
    return { code: "OBJECT_NOT_FOUND", message: error };
  }

  if (error.includes("InputObjectDeleted") || error.includes("ObjectVersion")) {
    return { code: "OBJECT_VERSION_EXPIRED", message: error };
  }

  const moveAbort = parseMoveAbort(error);
  if (moveAbort) {
    return {
      code: mapMoveAbort(moveAbort),
      message: error,
      moveAbort,
    };
  }

  if (error.toLowerCase().includes("gas")) {
    return { code: "GAS_ERROR", message: error };
  }

  return { code: "SUI_EXECUTION_ERROR", message: error };
}
```

Move abort 字符串格式会随 Sui SDK 版本变化，所以解析器要保留原始错误：

```ts
function parseMoveAbort(error: string) {
  const abortCode = error.match(/abort(?:ed)?(?: with code)?[: ]+([0-9]+)/i)?.[1];
  const moduleMatch = error.match(/([0-9a-fx]+)::([A-Za-z0-9_]+)::([A-Za-z0-9_]+)/);

  if (!abortCode && !moduleMatch) return undefined;

  return {
    package: moduleMatch?.[1],
    module: moduleMatch?.[2],
    function: moduleMatch?.[3],
    abortCode,
  };
}
```

## 业务错误映射

```ts
function mapMoveAbort(abort: { module?: string; abortCode?: string }) {
  if (abort.module === "pool") return "DEEPBOOK_POOL_ABORT";
  if (abort.module === "balance_manager") return "BALANCE_MANAGER_ABORT";
  if (abort.module === "margin_manager") return "MARGIN_MANAGER_ABORT";
  if (abort.module === "margin_pool") return "MARGIN_POOL_ABORT";
  if (abort.module === "predict") return "PREDICT_ABORT";
  if (abort.module === "predict_manager") return "PREDICT_MANAGER_ABORT";
  return "MOVE_ABORT";
}
```

生产映射表应进一步读取 Move 常量：

- `packages/deepbook/sources/helper/constants.move`
- `packages/deepbook_margin/sources/helper/margin_constants.move`
- `packages/predict/sources/helper/constants.move`

## 交易前检查

dry run 不是唯一防线。构造交易前还要检查：

- 配置对象 ID 非空。
- `client.getObject` 能读取 Pool、BalanceManager、MarginManager、Predict object。
- coin object 余额足够。
- pool tick、lot、min size 满足输入。
- Margin oracle 未过期，风险率满足要求。
- Predict oracle、trading paused、ask bounds 和 vault capacity 满足要求。

dry run 成功后仍要把 `balanceChanges` 和 `objectChanges` 返回给调用方，用于用户确认和测试断言。
