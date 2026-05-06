# ch00-04 现货交易能力

[返回本章](README.md)

DeepBookV3 spot 的公开入口集中在 [pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)。从应用开发角度看，现货能力可以分为下单、兑换、订单管理、结算、治理和查询。

下单能力包括 `place_limit_order` 和 `place_market_order`。限价单表达指定价格和数量，可能成交、部分成交，也可能进入订单簿等待后续撮合。市价单在源码中被转换成极端价格的 IOC 订单，用已有流动性尽量成交，未成交部分取消。订单参数会涉及方向、价格、数量、self match 选项、order type、过期时间和是否用 DEEP 支付费用。

兑换能力包括 `swap_exact_base_for_quote`、`swap_exact_quote_for_base` 以及带 manager 的版本。直接 swap 对协议集成很重要，因为调用者可以用 `Coin` 输入和输出，不必自己先管理完整 `BalanceManager` 生命周期。源码内部仍会走撮合路径，只是包装了临时资金管理和输出检查。

订单管理包括 `modify_order`、`cancel_order`、`cancel_orders`、`cancel_live_order`、`cancel_live_orders` 和 `cancel_all_orders`。这些函数服务交易端、做市端和风控后台。做市商通常关心批量撤单和批量重挂，普通用户更关心取消某一笔订单，清算或风控逻辑则需要理解 open orders 对余额和风险的影响。

结算能力包括 `withdraw_settled_amounts` 和 `withdraw_settled_amounts_permissionless`。撮合完成后，用户可提取已结算资产和应得金额。DeepBook 不应被理解成“成交后余额立刻出现在钱包里”的模型；很多路径会先进入内部状态，再通过结算函数进入 `BalanceManager`。

查询能力包括报价输出估算、mid price、level2 order book、账户 open orders、单个订单、订单详情、locked balance、pool params 等。这些链上查询函数和 Server 的 REST 查询互补：链上查询适合交易前验证，Indexer/Server 适合页面展示和历史数据。

## 本节检查

- [ ] 能列出现货交易的六类能力。
- [ ] 能解释 swap 与订单簿撮合之间的关系。
- [ ] 能说明为什么成交和结算要分开理解。
