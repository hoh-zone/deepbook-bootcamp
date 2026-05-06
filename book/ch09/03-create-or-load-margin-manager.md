# ch09-03 创建或加载 MarginManager

[返回本章](README.md)

## 先看用户路径

这一节从 Margin 用户动作读起：围绕“创建或加载 MarginManager”，先把 manager、collateral、loan、trade 和 repay 的顺序排清楚，再看源码入口。

## 源码入口

重点阅读：

- [packages/deepbook_margin/sources/margin_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/margin_manager.move)
- [packages/deepbook_margin/sources/margin_registry.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/margin_registry.move)
- `book/ch09/code/s01-create-margin-manager/README.md`

> **源码旁白**：先定位结构体、入口函数和事件，再回到本节的资金路径或应用流程。不要从 helper 函数开始读。

## 把 PTB 串起来

创建 manager 对应链上 `margin_manager.move` 的 `new` 或 `new_with_initializer`。SDK 通常会封装成类似 `client.deepbook.marginManager.createMarginManager(poolKey)` 的交易构造器，最终需要传 DeepBook Pool、DeepBook Registry、MarginRegistry、Clock 和 owner。

加载 manager 的推荐顺序：

1. 先从本地缓存或后端数据库按 `(owner, poolKey)` 查 manager ID。
2. 如果没有，查询 `MarginRegistry.margin_managers` 对应的事件或对象索引。
3. 如果仍不存在，引导用户创建。
4. 创建成功后保存 `MarginManagerCreatedEvent` 中的 `margin_manager_id` 和 `balance_manager_id`。

创建后通常要注册到 registry，便于应用按用户地址枚举 manager。注销前必须检查：

- 没有 base debt。
- 没有 quote debt。
- `margin_pool_id` 为空。
- base、quote、DEEP 余额都为 0。

## 工程旁白

应用首次进入某个交易对时，应先从 registry 或 indexer 查 `(owner, pool)` 是否已有 manager。没有时再构造创建交易；有多个 manager 时要提示用户选择或按业务规则固定使用一个。

创建 manager 是账户初始化，不等于存入保证金或借款。创建成功后风险率通常仍为空或无债务状态，UI 应引导用户下一步存入抵押品，而不是展示可交易杠杆额度。

## Margin 应用判断

- manager ID 绑定 pool key 存储，切换交易对时重新加载。
- 创建后等待 shared object 可读再允许后续 deposit/borrow。
- 注销入口只在无债务、无余额、无挂单和无 TPSL 时显示。

## 动手检查

- 当前 owner 在这个 pool 是否已经有 manager？
- 创建交易是否传入正确的 base/quote type arguments 和 DeepBook Pool ID？
- 加载到的 manager 是否已经注册到 `MarginRegistry`？
