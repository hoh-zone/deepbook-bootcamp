# ch09-02 初始化 Margin SDK 配置

[返回本章](README.md)

## 先看用户路径

这里先从用户动作往回看。“初始化 Margin SDK 配置”在应用里通常是一组 PTB 和状态刷新，不是一条孤立 move call；每一步都要同时考虑余额、债务和风险率。

## 源码入口

重点阅读：

- [scripts/transactions/marginPrep.ts](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/scripts/transactions/marginPrep.ts)
- [scripts/transactions/updateInterestRates.ts](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/scripts/transactions/updateInterestRates.ts)
- [packages/deepbook_margin/sources/margin_registry.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/margin_registry.move)
- [packages/deepbook_margin/sources/helper/oracle.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/helper/oracle.move)

> **源码旁白**：先定位结构体、入口函数和事件，再回到本节的资金路径或应用流程。不要从 helper 函数开始读。

## 把 PTB 串起来

脚本使用 `@mysten/sui` 和 `@mysten/deepbook-v3`：

```ts
import { SuiGrpcClient } from "@mysten/sui/grpc";
import { Transaction } from "@mysten/sui/transactions";
import { deepbook } from "@mysten/deepbook-v3";

const env = "mainnet";
const client = new SuiGrpcClient({
  baseUrl: "https://sui-mainnet.mystenlabs.com",
  network: "mainnet",
}).$extend(
  deepbook({
    address: adminCapOwner[env],
    adminCap: adminCapID[env],
    marginAdminCap: marginAdminCapID[env],
    marginMaintainerCap: marginMaintainerCapID[env],
  }),
);

const tx = new Transaction();
```

普通用户操作通常只需要 `address`。管理员和维护者脚本才传 `adminCap`、`marginAdminCap`、`marginMaintainerCap`。`marginPrep.ts` 是管理员视角，不要把其中 cap 配置复制到用户前端。

生产配置建议用一份 network config：

- `network` 和 RPC endpoint。
- DeepBook package 和 Margin package 版本。
- `MarginRegistry` shared object。
- 支持的 pool key，如 `SUI_USDC`、`DEEP_USDC`。
- base/quote Pyth price object。
- base/quote MarginPool ID。
- liquidation vault ID。

## 工程旁白

初始化脚本展示的是管理员流程，普通应用只需要读取结果并构造用户 PTB。不要把 `AdminCap`、maintainer cap 或 pause cap 放进前端配置；前端配置应只包含 package ID、registry ID、DeepBook Pool ID、MarginPool ID、Pyth price object 和 type arguments。

不同网络的对象 ID 可能完全不同。SDK 配置要支持按 network 加载，并在启动时用链上读取校验 registry 版本、pool enabled、loan enabled 和 price feed 是否匹配，避免用户签名后才发现配置错。

## Margin 应用判断

- 把 admin 初始化配置和用户交易配置分文件管理。
- 启动时校验 package version、registry allowed version 和 PoolConfig。
- price object 配置要和 base/quote 类型绑定，不能只按 symbol 字符串匹配。

## 动手检查

- 当前配置是否包含任何不该给前端的 cap？
- DeepBook Pool、base MarginPool、quote MarginPool 和 Pyth price object 是否来自同一网络？
- SDK 在构造 PTB 前是否验证 registry 中的 pool enabled 状态？
