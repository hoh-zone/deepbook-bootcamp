# s03-shared-object 共享对象计数器

目标：实现一个最小共享对象，练习 `key`、`UID`、`TxContext` 和 shared object 的读写模型。

## 目录结构

```text
s03-shared-object/
  Move.toml
  sources/counter.move
  tests/counter_tests.move
```

## 合约要点

`Counter` 应包含 `id: UID` 和 `value: u64`。初始化函数创建对象后使用 `transfer::share_object(counter)` 共享出去。递增函数接收 `&mut Counter`，读取函数接收 `&Counter`。

DeepBook 的 `Pool`、`Registry`、Margin pool、Predict vault 都是围绕 shared object 设计业务状态。本示例的价值不是计数，而是理解为什么共享对象可以成为多用户共同访问的协议入口。

## 测试重点

- 初始化后 `value == 0`。
- 调用 `increment` 后数值增加。
- 只读函数不需要可变借用。

运行方式：

```bash
sui move test
```

