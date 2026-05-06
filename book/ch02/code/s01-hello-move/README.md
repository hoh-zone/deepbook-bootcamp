# s01-hello-move 最小 Move package

本示例对应 DeepBook 源码 [packages/deepbook/sources/hello_move.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/hello_move.move)。该文件覆盖 module、struct、ability、泛型、对象、事件和测试辅助函数。

## 源码阅读目标

重点阅读这些定义：

- `module deepbook::hello_move`
- `Greeting has copy, drop, store`
- `NonCopyableResource has store`
- `GenericBox<phantom T>`
- `UserProfile has key`
- `create_user`
- `event::emit(UserCreated { ... })`

## 本地构建 DeepBook 包

```bash
cd deepbookv3/packages/deepbook
sui move build
sui move test
```

如果依赖下载失败，先确认网络和 Sui CLI 版本。`Move.toml` 依赖 [packages/token](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/token) 对应的 git 包配置。

## 最小练习包结构

可以在本目录下扩展为：

```text
s01-hello-move/
  Move.toml
  sources/
    hello.move
```

`Move.toml` 示例：

```toml
[package]
name = "hello_move_bootcamp"
edition = "2024.beta"
version = "0.0.1"

[addresses]
hello_move_bootcamp = "0x0"
```

## 检查点

- 普通值类型可以 `copy/drop/store`。
- 对象类型必须包含 `id: UID` 并具备 `key`。
- 创建对象需要 `object::new(ctx)`。
- 事件字段应使用 `ID`、address、数字、bool、vector 等可索引值。
