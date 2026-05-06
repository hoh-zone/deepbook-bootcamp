# ch04-12 order query 如何服务前端查询

[返回本章](README.md)

## 先看订单现场

先把“order query 如何服务前端查询”放进一笔订单里看。用户看到的是买卖按钮，链上真正发生的是 Pool 接住入口、Book 决定撮合、OrderInfo 带着执行结果回到资金结算。

## 源码入口

- [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)：下单、撤单和修改订单的 public 入口，负责进入对应 PoolInner。
- [packages/deepbook/sources/book/book.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/book/book.move)：`match_against_book`、`inject_limit_order`、`cancel_order` 和买卖两侧 `BigVector<Order>`。
- [packages/deepbook/sources/helper/utils.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/helper/utils.move)：`encode_order_id`、`decode_order_id` 的 side、price、sequence 编码规则。
- [packages/deepbook/sources/order_query.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/order_query.move)：分页读取 bids/asks 的只读接口，服务前端订单簿和 indexer 对账。

## 关键定义

`order_query.move` 给前端和 indexer 一个轻量分页视图。它返回的是 `Order` 副本集合，而不是把整个 `Book` 暴露给调用方。

```move
public struct OrderPage has drop {
    orders: vector<Order>,
    has_next_page: bool,
}

public fun iter_orders<BaseAsset, QuoteAsset>(
    self: &Pool<BaseAsset, QuoteAsset>,
    start_order_id: Option<u128>,
    end_order_id: Option<u128>,
    min_expire_timestamp: Option<u64>,
    limit: u64,
    bids: bool,
): OrderPage
```

这里的 `start_order_id` 和 `end_order_id` 不是数据库 offset，而是订单簿排序键。bids 按从高到低遍历，asks 按从低到高遍历；如果前端把它们转成 JavaScript number，分页游标和排序都会在大整数区间失真。

## 把订单走一遍

`order_query.move` 的 `iter_orders` 返回 `OrderPage { orders, has_next_page }`。调用方可传：

- `start_order_id`
- `end_order_id`
- `min_expire_timestamp`
- `limit`
- `bids`

bids 默认范围是 `[0, 1 << 127)`，asks 默认范围是 `[1 << 127, max_u128]`。函数会跳过 `expire_timestamp < min_expire` 的订单。

前端展示深度时，优先使用聚合 level2 查询；展示个人订单时，需要结合账户 open orders、BalanceManager ID 和 `Order` 详情。不要仅通过事件重建实时订单簿，除非后端 indexer 已处理重组、去重和补偿扫描。

> **源码旁白**：撮合相关小节都从 [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move) 的 public 入口进入，再沿 [packages/deepbook/sources/book/book.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/book/book.move) 追踪撮合、插入或取消。读的时候把 taker 的 `OrderInfo` 和 maker 的 `Order` 分开：前者是本次执行结果，后者才是可能继续留在订单簿里的状态。

查询侧要保持 `order_id` 为字符串或 BigInt。[packages/deepbook/sources/order_query.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/order_query.move) 返回链上排序后的订单视图，前端聚合深度时只能按 price level 合并数量，不能重新用 JavaScript number 排序 u128。

## 交易实现提醒

- 用 `u128` + `BigInt` 思维处理 `order_id`、price、quantity，避免前端精度丢失。
- 先判断订单会进入撮合、直接失败、完全吃单还是剩余挂单，再设计事件监听和 UI 状态。
- 读取 `Fill` 和 `OrderInfo` 时同时记录 maker/taker 方向，避免把 base、quote 的应收应付方向写反。

## 动手检查

- order query 如何服务前端查询 依赖哪一个 `pool.move` 入口，下一跳进入哪个 `book/*` 函数？
- 成功路径会产生哪些订单事件，失败时最可能命中输入、执行策略还是余额相关校验？
- 如果前端或 indexer 展示本节数据，哪些字段必须保持链上原始整数格式？
