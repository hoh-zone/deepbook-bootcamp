# ch12-17 SDK 单元测试、mock client 和 fixture

[返回本章](README.md)

## 本节目标

- 为 SDK 服务编写 mock client、fixture 和错误表测试。
- 覆盖配置缺失、PTB target、dry run 成功/失败和对象状态。
- 用 fixture 固定 Predict 迁移状态，避免依赖实时主网。

## 源码关联

- `book/ch12/code/s06-dry-run-helper/`：dry run fixture 和错误解析。
- `scripts/config/constants.ts`：配置测试样本。
- `packages/predict/sources/*`：Predict fixture 应覆盖的对象类型。

## 正文

单元测试应 mock 三层：

- 配置解析：网络、对象 ID、空值检查。
- 交易构造：断言 PTB 包含预期 move target 或 SDK builder 调用。
- dry run 结果：成功、Move abort、余额不足、对象不存在。

fixture 应包含典型 pool、BalanceManager、MarginManager、Predict object 的 object response。不要让单元测试依赖实时主网状态。

服务封装建议统一返回 `Transaction` 或交易 bytes、dry run 摘要和可读错误码。后端可以构造 PTB 和校验配置，但用户交易必须由钱包签名；管理员交易则按 `scripts/utils/utils.ts` 的 multisig/dry run 思路固定 gas、expiration 和 cap。

Predict 相关接口必须额外校验 `predictVersionStatus`、package ID、registry ID、predict object ID、oracle ID 和 quote coin type。由于迁移文档未把 Predict SDK、Indexer 或 Server 标为稳定完成，本章只把它们写成 raw Move 封装或未来服务边界。

## 开发要点

- mock `SuiClient` 覆盖 object query、dry run、waitForTransaction。
- fixture 包含 pool、BalanceManager、MarginManager、Predict、Oracle、Vault。
- 错误表覆盖 Move abort、对象不存在、版本冲突、余额不足、gas 不足。

## 检查问题

- PTB 构造测试应断言 target 还是最终 digest？
- 为什么单元测试不应依赖实时主网对象？
- Predict fixture 中哪些版本状态必须覆盖？
