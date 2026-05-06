# ch12-05 constants 管理主网和测试网

[返回本章](README.md)

## 先定封装边界

读这一节时把自己当成 SDK 维护者：封装应该暴露业务意图，隐藏重复参数，同时保留 dry run 和错误定位能力。

## 源码入口

- `scripts/config/constants.ts`：主网/测试网对象 ID 表。
- `scripts/transactions/marginPrep.ts`、`enableMarginVersion.ts`：cap 配置消费方式。
- `packages/predict/sources/predict.move`、`registry.move`：Predict 配置应包含的对象。

## SDK 读法

配置文件建议按“网络 -> 对象角色 -> object ID”组织，而不是按业务函数组织。对象角色包括：

- DeepBook admin：创建池、版本管理、授权 Margin app。
- Margin admin：版本开关、Pyth config、pause cap。
- Margin maintainer：创建 margin pool、更新利率和风控参数。
- Supplier cap：向 margin pool 供给流动性。
- BalanceManager：交易资金账户。

读取配置时要检查三类错误：网络不存在、对象 ID 为空、对象 ID 不属于当前网络。第三类需要用 `client.getObject` 查询 owner/type/version，并在 dry run 前失败。

服务封装建议统一返回 `Transaction` 或交易 bytes、dry run 摘要和可读错误码。后端可以构造 PTB 和校验配置，但用户交易必须由钱包签名；管理员交易则按 `scripts/utils/utils.ts` 的 multisig/dry run 思路固定 gas、expiration 和 cap。

Predict 相关接口必须额外校验 `predictVersionStatus`、package ID、registry ID、predict object ID、oracle ID 和 quote coin type。由于迁移文档未把 Predict SDK、Indexer 或 Server 标为稳定完成，本章只把它们写成 raw Move 封装或未来服务边界。

## 封装判断

- `constants.ts` 按 network 分组保存 cap、pool、manager、registry、predict 对象。
- 测试网字段为空时 fail fast，不发送空对象 ID。
- Predict 增加 `predictVersionStatus`、package ID、registry ID、predict ID、oracle IDs。

## 动手检查

- 哪些对象 ID 绝不能散落在业务函数里？
- 管理员 cap 和用户 manager 配置的生命周期有什么不同？
- 如何防止主网 cap 被测试网交易误用？
