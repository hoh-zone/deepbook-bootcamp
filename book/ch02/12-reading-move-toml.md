# ch02-12 阅读 Move.toml

[返回本章](README.md)

## 本节目标

- 从包配置理解 package、依赖、地址映射和 edition。
- 能沿“阅读 Move.toml”定位相关 Move 源码、脚本或链下服务入口。
- 读完后能够用交易路径、对象职责或失败场景解释本节主题。

## 源码关联

本节重点对照以下源码或后续阅读入口：

- [packages/deepbook/Move.toml](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/Move.toml)
- [packages/deepbook_margin/Move.toml](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/Move.toml)
- [packages/predict/Move.toml](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/predict/Move.toml)

阅读时先从标题对应的入口文件开始，确认对象、函数签名和事件名称，再回到本节正文理解它在交易路径中的位置。

## 正文

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

## 开发要点

- 读源码前先确认当前包名和地址别名。
- 升级或切换分支后重新检查 dependency rev/path。
- 引用模块时用 Move.toml 中的地址映射解释来源。

## 检查问题

- `Move.toml` 中哪些字段会影响源码阅读？
- 为什么包依赖比产品名称更能说明边界？
- 地址别名写错会造成哪类调用问题？
