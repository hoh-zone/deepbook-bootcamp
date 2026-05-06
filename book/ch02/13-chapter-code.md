# ch02-13 本章代码

[返回本章](README.md)

## 先建立手感

这一节用“本章代码”训练 Move 手感：先看对象和资源能不能被复制、丢弃、转移，再回到 DeepBook 里判断为什么这些限制有实际价值。

## Move 对照

先用下面几处源码建立 Move 概念的落点。这里不追完整协议流程，只确认类型、ability、对象、事件和测试入口如何在真实项目中出现。

- `book/ch02/code/s01-hello-move/README.md`
- `book/ch02/code/s02-coin-balance/README.md`
- `book/ch02/code/s04-event-indexing/README.md`
- [packages/deepbook/sources/hello_move.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/hello_move.move)

读这些文件时，把语法点和真实对象放在一起看：ability、泛型、对象 ID、事件和测试入口分别承担什么约束。

## 拆开来看

本章示例目录：

- `book/ch02/code/s01-hello-move`
- `book/ch02/code/s02-coin-balance`
- `book/ch02/code/s03-shared-object`
- `book/ch02/code/s04-event-indexing`

## 阅读补充

本章代码的目标是把 Move 基础语法压缩成可执行的小实验。`hello-move` 负责语法和 ability，`coin-balance` 负责资产转换，`shared-object` 负责对象访问模式，`event-indexing` 负责链上事件到链下读模型。

这些练习不需要复刻 DeepBook 全量逻辑，但每个练习都应该能指向 DeepBook 中的真实源码位置。这样读者写完小例子后，能自然迁移到 `balance_manager.move`、`pool.move` 和 `order_info.move`。

## Move 判断

- 每个练习 README 都写明对应的 DeepBook 源码文件。
- 练习代码优先验证一个 Move 概念，不混入完整交易所流程。
- 事件练习要同时给出 emit 位置和查询方式。

## 练习问题

- 四个代码练习分别对应哪些 Move 基础概念？
- 为什么小练习要绑定真实 DeepBook 文件？
- 完成本章代码后应能读懂哪些生产模块？
