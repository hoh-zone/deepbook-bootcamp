# ch09-08 通过 DeepBook Pool 执行杠杆卖出

[返回本章](README.md)

## 本节目标

- 实现借 base 后通过 DeepBook Pool 卖出 base 的杠杆做空流程。
- 能展示 base debt 的风险方向、reduce-only 降风险入口和成交后状态。
- 能避免用户把 quote 余额增长误判为无风险盈利。

## 源码关联

重点阅读：

- [packages/deepbook_margin/sources/pool_proxy.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/pool_proxy.move)
- [packages/deepbook_margin/sources/margin_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/margin_manager.move)
- [packages/deepbook_margin/sources/margin_pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/margin_pool.move)
- `book/ch09/code/s03-borrow-and-trade/README.md`

阅读时先从这些文件定位结构体、入口函数和事件，再回到正文中的资金路径或应用流程。

## 正文

杠杆卖出常见路径：

1. 用户存入 base 或 quote 抵押。
2. 调 `borrow_base` 借入 base。
3. 调 `pool_proxy.place_market_order` 卖出 base 得到 quote。
4. 结算成交资产。
5. 展示 debt asset 为 base 的风险率。

开空的风险方向和开多相反。base 价格上涨会增加用 quote 折算的 base 债务价值，风险率下降。风控面板要明确显示 debt asset，避免用户只看 quote 余额误判安全。

当 Pool 被禁用常规 Margin 交易时，应用应改用 reduce-only 入口帮助用户降风险：

- `place_reduce_only_limit_order`
- `place_reduce_only_market_order`

这些入口只允许减少净债务方向的交易，不能扩大仓位。

补充说明：

杠杆卖出后，用户通常持有 quote 资产但欠 base。base 价格上涨会让还款成本变高，风险率下降；仅显示 quote 余额会掩盖真实风险。

当 Pool 禁用常规 Margin 交易时，reduce-only 卖出/买回入口是用户自救路径。应用应根据当前 debt asset 和订单方向判断是否可 reduce-only，而不是简单隐藏交易模块。

## 开发要点

- 做空确认页把 debt asset 用醒目字段展示，并显示 base 上涨时的清算距离。
- reduce-only 入口单独构造，不复用普通下单参数后端静默切换。
- 平空时通常需要买回 base 后 `repay_base`，流程要和卖出开仓分开。

## 检查问题

- 卖出后用户欠的是哪种资产，价格上涨还是下跌更危险？
- 当前订单是否真的减少 base debt 风险，而不是扩大空头？
- quote 余额足够时，是否还需要先买回 base 才能还款？
