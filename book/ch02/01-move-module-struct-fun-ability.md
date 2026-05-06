# ch02-01 module、struct、fun、ability

[返回本章](README.md)

## 先建立手感

这一节用“module、struct、fun、ability”训练 Move 手感：先看对象和资源能不能被复制、丢弃、转移，再回到 DeepBook 里判断为什么这些限制有实际价值。

## Move 对照

先用下面几处源码建立 Move 概念的落点。这里不追完整协议流程，只确认类型、ability、对象、事件和测试入口如何在真实项目中出现。

- [packages/deepbook/sources/hello_move.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/hello_move.move)
- [packages/deepbook/Move.toml](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/Move.toml)
- [packages/deepbook/sources/balance_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/balance_manager.move)

读这些文件时，把语法点和真实对象放在一起看：ability、泛型、对象 ID、事件和测试入口分别承担什么约束。

## 拆开来看

Move 的 `module` 是代码和类型的命名空间。DeepBook 的基础示例模块是 [packages/deepbook/sources/hello_move.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/hello_move.move)，开头声明 `module deepbook::hello_move;`。这说明模块地址来自 `Move.toml` 的 `deepbook = "0x0"` 映射，发布时会被实际 package ID 替换。

`struct` 定义数据形状和 ability。`hello_move.move` 中的 `Greeting has copy, drop, store` 是普通值；`UserProfile has key` 是链上对象；`NonCopyableResource has store` 不能随意复制或丢弃，需要显式消费。

> **Move 技巧**：读金融协议里的 `struct`，第一眼不要看字段，先看 ability。没有 `copy` 的资源不能被复制，没有 `drop` 的资源不能随手丢弃，`key` 表示它会成为链上对象。这些约束比字段名更早决定安全边界。

`fun` 定义行为。`new_greeting(custom_message: vector<u8>): Greeting` 返回普通结构；`create_user(admin_cap, name, initial_balance, ctx): UserProfile` 创建对象并发出事件；`mint_resource(value, ctx)` 创建对象后 `transfer::transfer` 给交易发送者。

## 阅读补充

读 DeepBook 源码时，先把 `module` 当作命名空间和权限边界，把 `struct` 当作链上状态或事件形状，把 `fun` 当作状态迁移入口。ability 决定资源能否复制、丢弃、存储或成为对象，是金融协议最容易踩错的语法点。

建议先用 `hello_move.move` 建立最小语法样本，再打开 `balance_manager.move` 对照真实业务结构。这样能看到教学示例与生产代码之间的差异，而不是直接被完整协议的泛型和 cap 设计淹没。

## Move 判断

- 读源码先标注 module 名、公开函数和关键 struct ability。
- 看到 `has key` 时立刻判断它是不是链上对象。
- 看到无 `copy/drop` 的资源时，不要假设它能随意传值或销毁。

## 练习问题

- `module`、`struct`、`fun` 在 DeepBook 源码里分别对应什么阅读对象？
- 为什么 ability 是金融对象安全的一部分？
- `hello_move.move` 和 `balance_manager.move` 适合怎样配合阅读？
