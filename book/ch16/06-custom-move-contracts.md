# ch16-06 自定义 Move 合约

[返回本章](README.md)

Sandbox 对高级 Move 学习最有价值的部分，是它允许你在本地写一个真正依赖 DeepBook 的 Move package。你不是在模拟接口，也不是在伪造对象，而是在 localnet 上引用已经发布的 DeepBook package，并把自己的合约发布到同一条链。

## 从模板开始

官方仓库提供 `example_contract` 模板。典型流程是：

```bash
cd deepbook-sandbox
cp -r sandbox/packages/example_contract sandbox/packages/my_contract
cd sandbox/packages/my_contract
```

模板的意义不是减少几行文件创建，而是给你一个正确的依赖形状。自定义合约通常会依赖这些本地 package：

| 依赖 | 典型用途 |
| --- | --- |
| `deepbook` | Pool、BalanceManager、订单和核心交易能力。 |
| `token` | DEEP token 类型。 |
| `deepbook_margin` | Margin 相关对象和入口。 |
| `margin_liquidation` | 清算逻辑。 |
| `pyth` | 价格对象与 oracle 相关依赖。 |
| `usdc` | 本地 USDC coin type。 |

## Move.toml 的关键点

本地合约的 `Move.toml` 需要同时处理 dependency 和 environment：

```toml
[package]
name = "my_contract"
edition = "2024"

[dependencies]
deepbook = { local = "../../.external-packages/deepbook" }
token = { local = "../../.external-packages/token" }

[environments]
localnet = "<chain-id>"
```

`<chain-id>` 从 `sandbox/Pub.localnet.toml` 读取。这个细节很重要：Move 编译器不是只看文件路径，它还要知道当前 build environment 对应哪条链。

## 发布到 localnet

```bash
sui client test-publish --build-env localnet --pubfile-path ../../Pub.localnet.toml
```

这里有两个参数需要认真理解：

- `--build-env localnet`：选择 `Move.toml` 中的 localnet 环境。
- `--pubfile-path ../../Pub.localnet.toml`：告诉 Sui CLI，DeepBook 等依赖已经在本地链上发布，地址记录在这个 pubfile 中。

没有 `--pubfile-path`，编译器可能能看到源码，却无法把依赖解析到当前 localnet 的发布地址。对依赖链上协议的 Move 合约来说，这是最常见也最隐蔽的错误之一。

## 如何设计自己的合约练习

第一版不要写复杂策略。建议先做三个小 wrapper：

1. 读取或传入 Pool、BalanceManager、Clock 等对象，验证对象借用顺序。
2. 调用一个只做校验或记录事件的函数，确认自定义 event 能被 indexer 读取。
3. 再尝试组合 DeepBook 交易入口，观察 PTB 中对象和 coin 的流转。

当 wrapper 失败时，先看 abort module 和 abort code，再看对象类型和 type argument。Move 的错误往往不是“业务错了”，而是资源、权限、类型或对象版本没有对齐。

## 本节验收

- 能复制 `example_contract` 并说明每个依赖的用途。
- 能写出带 `localnet` environment 的 `Move.toml`。
- 能解释 `--pubfile-path` 为什么是依赖 DeepBook 的本地合约发布关键。
