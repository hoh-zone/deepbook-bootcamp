# ch02-12 阅读 Move.toml

[返回本章](README.md)

## 先建立手感

先不要把“阅读 Move.toml”当成孤立语法点。DeepBook 里每个资产、订单和权限对象都会受 Move 类型系统约束，读这一节时要看语法如何变成资金安全边界。

## 源码入口

这一节只保留必要入口，目的不是让你马上读完源码，而是建立后续定位能力：

- [packages/deepbook/Move.toml](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/Move.toml)
- [packages/deepbook_margin/Move.toml](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/Move.toml)
- [packages/predict/Move.toml](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/predict/Move.toml)

读源码时先确认对象、函数签名和事件名称；等正文讲到交易路径时，再回到这些入口核对。

## 拆开来看

DeepBookV3 的包配置在 [packages/deepbook/Move.toml](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/Move.toml)：

```toml
[package]
name = "deepbook"
edition = "2024.beta"
version = "0.0.1"

[dependencies]
token = { git = "https://github.com/MystenLabs/deepbookv3.git", subdir = "packages/token", rev = "main"}

[addresses]
deepbook = "0x0"
```

`name` 决定包名，`edition` 决定 Move 语法版本，`dependencies` 说明依赖 `token` 包，`addresses` 给源码中的 `deepbook::...` 提供编译期地址占位。

开发实践：本地练习包可以先用 `0x0`，发布后 CLI 会给出真实 package ID。调用链上函数时必须使用真实 package ID，而不是 `0x0`。

## 阅读补充

`Move.toml` 是进入一个 Move 包的第一张地图。它告诉你包名、edition、依赖来源和地址别名；这些信息会影响源码里的 module 路径、发布地址和测试构建行为。

比较 DeepBook、Margin、Predict 的 `Move.toml`，可以快速看出它们是独立包还是共享依赖。不要只凭目录名判断协议关系，包配置里的依赖才是更可靠的入口。

## Move 判断

- 读源码前先确认当前包名和地址别名。
- 升级或切换分支后重新检查 dependency rev/path。
- 引用模块时用 Move.toml 中的地址映射解释来源。

## 动手检查

- `Move.toml` 中哪些字段会影响源码阅读？
- 为什么包依赖比产品名称更能说明边界？
- 地址别名写错会造成哪类调用问题？
