# ch03-04 Pool 是入口，Book 是撮合核心

[返回本章](README.md)

## 先抓住结构

先把“Pool 是入口，Book 是撮合核心”放进 DeepBook 的对象图里。这里不是罗列模块，而是建立阅读顺序：入口在哪里，状态放在哪里，资金最终在哪里结算。

## 源码入口

这一节只保留必要入口，目的不是让你马上读完源码，而是建立后续定位能力：

- [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)
- [packages/deepbook/sources/book/book.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/book/book.move)
- [packages/deepbook/sources/book/order.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/book/order.move)
- [packages/deepbook/sources/book/order_info.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/book/order_info.move)
- [packages/deepbook/sources/book/fill.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/book/fill.move)

读源码时先确认对象、函数签名和事件名称；等正文讲到交易路径时，再回到这些入口核对。

## 读架构

Pool 对外暴露 `place_limit_order`、`place_market_order`、`swap_exact_*`、`modify_order`、`cancel_order`、`withdraw_settled_amounts` 等接口。Book 不直接暴露给外部调用者；它是 `PoolInner` 的内部字段。

一次限价单的核心调用链是：

```text
pool::place_limit_order
  -> pool::place_order_int
    -> order_info::new
    -> book::create_order
      -> order_info::validate_inputs
      -> utils::encode_order_id
      -> book::match_against_book
        -> order_info::match_maker
          -> order::generate_fill
      -> book::inject_limit_order
    -> state::process_create
    -> vault::settle_balance_manager
    -> order_info events
```

`Book` 只决定“能不能成交、成交多少、是否插入订单簿”。账户开放订单、历史费率、maker/taker 费用和资金扣划都不在 Book 中完成，而是在 `State` 和 `Vault` 中完成。

## 阅读补充

`Pool` 负责把外部交易参数转换成协议内部可执行流程：加载版本、校验参数、构造 `OrderInfo`、调用 `Book`，最后协调 State 和 Vault。`Book` 不应该承担权限、费用和资产保管，它的核心职责是维护 bids/asks、匹配价格和生成 fill。

阅读调用链时，从 `pool.move` 的 public entry/public 函数进入，再跳到 `book.move` 看撮合循环，最后回到 `order_info.move` 和 `fill.move` 看成交如何记录。这个来回跳转能避免误以为某个单文件包含完整交易语义。

## 工程判断

- 入口函数说明外部 API，Book 函数说明撮合规则，两者不要混写。
- 调试撮合异常时记录 order id、side、price、quantity 和 order type。
- 费用和余额问题要继续追到 State/Vault，而不是停在 Book。

## 读完以后问自己

- Pool 在下单流程中承担哪些 Book 以外的职责？
- Book 生成 fill 后，为什么还需要 OrderInfo？
- 撮合成功但余额异常时应该继续读哪个模块？
