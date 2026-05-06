# 关键定义地图：结构体、方法和事件怎么读

本页解决一个核心问题：读者不应该只看到 GitHub 链接。真正有效的源码讲解，必须把关键 `struct`、方法签名、字段含义、状态变化和常见误读直接放进书里。

本页代码摘录自 [MystenLabs/deepbookv3@663edbf9c30d6c93100e6cd66936e1487a5dc9e0](https://github.com/MystenLabs/deepbookv3/tree/663edbf9c30d6c93100e6cd66936e1487a5dc9e0)。链接只用于复核版本，正文解释以书内定义卡片为主。

本书后续所有源码讲解都按“定义卡片”写：

1. **关键定义**：摘出最小必要的 `struct`、`public fun` 或 handler。
2. **字段解释**：说明每个字段在协议状态中代表什么。
3. **调用入口**：列出最重要的方法，不堆完整 API。
4. **状态变化**：说明对象、资金、订单或事件如何变化。
5. **常见误读**：指出读者最容易混淆的边界。

## 01 Spot 入口：Pool 和 PoolInner

```move
public struct Pool<phantom BaseAsset, phantom QuoteAsset> has key {
    id: UID,
    inner: Versioned,
}

public struct PoolInner<phantom BaseAsset, phantom QuoteAsset> has store {
    allowed_versions: VecSet<u64>,
    pool_id: ID,
    book: Book,
    state: State,
    vault: Vault<BaseAsset, QuoteAsset>,
    deep_price: DeepPrice,
    registered_pool: bool,
}
```

`Pool` 是外部应用传入 PTB 的共享对象。它很薄，只有对象身份和 `Versioned` 包装。真正业务状态在 `PoolInner`：`book` 管订单簿，`state` 管账户、费用、历史和治理，`vault` 管 base/quote/DEEP 资产，`deep_price` 服务 DEEP 费用换算，`allowed_versions` 决定当前包版本是否还能操作这个池。

常见误读是把 `Pool` 当成“订单簿对象”。更准确的说法是：`Pool` 是一个交易入口对象，订单簿只是它内部的一部分。

## 02 Spot 写交易入口

```move
public fun place_limit_order<BaseAsset, QuoteAsset>(
    self: &mut Pool<BaseAsset, QuoteAsset>,
    balance_manager: &mut BalanceManager,
    trade_proof: &TradeProof,
    client_order_id: u64,
    order_type: u8,
    self_matching_option: u8,
    price: u64,
    quantity: u64,
    is_bid: bool,
    pay_with_deep: bool,
    expire_timestamp: u64,
    clock: &Clock,
    ctx: &TxContext,
): OrderInfo
```

这个签名已经解释了大半个交易模型：

- `self: &mut Pool`：会改变池内订单簿、账户状态或 vault。
- `balance_manager: &mut BalanceManager`：资金不直接从钱包扣，而从交易账户结算。
- `trade_proof: &TradeProof`：证明当前调用者能代表这个 manager 交易。
- `price`、`quantity`、`is_bid`：订单意图。
- `order_type`、`self_matching_option`、`expire_timestamp`：执行策略和失败边界。
- `clock`：订单过期、epoch、事件时间都依赖它。
- 返回 `OrderInfo`：本次交易生命周期结果，而不是长期挂单对象。

读任何 Spot SDK 封装时，都应该反向映射到这个签名，检查对象、类型参数和整数单位是否完整。

## 03 账户抽象：BalanceManager、Cap、TradeProof

```move
public struct BalanceManager has key, store {
    id: UID,
    owner: address,
    balances: Bag,
    allow_listed: VecSet<ID>,
}

public struct TradeCap has key, store {
    id: UID,
    balance_manager_id: ID,
}

public struct DepositCap has key, store {
    id: UID,
    balance_manager_id: ID,
}

public struct WithdrawCap has key, store {
    id: UID,
    balance_manager_id: ID,
}

public struct TradeProof has drop {
    balance_manager_id: ID,
    trader: address,
}
```

`BalanceManager` 是 DeepBook 的交易账户，不是钱包。`balances` 保存内部资产余额，`owner` 能管理权限和资金，`allow_listed` 保存被授权 cap 的 ID。`TradeCap`、`DepositCap`、`WithdrawCap` 把交易、存入、取出权限拆开。`TradeProof` 是交易内临时权限证明，只有 `drop`，不会长期保存。

关键方法：

```move
public fun new(ctx: &mut TxContext): BalanceManager
public fun deposit<T>(balance_manager: &mut BalanceManager, coin: Coin<T>, ctx: &mut TxContext)
public fun withdraw<T>(balance_manager: &mut BalanceManager, withdraw_amount: u64, ctx: &mut TxContext): Coin<T>
public fun generate_proof_as_owner(balance_manager: &BalanceManager, ctx: &TxContext): TradeProof
public fun generate_proof_as_trader(balance_manager: &BalanceManager, trade_cap: &TradeCap, ctx: &TxContext): TradeProof
```

常见误读是“钱包里有 coin 就能下单”。DeepBook 非 swap 下单路径通常先要求资产进入 `BalanceManager`，然后交易通过 `TradeProof` 结算。

## 04 撮合核心：Order、OrderInfo、Fill

```move
public struct Order has drop, store {
    balance_manager_id: ID,
    order_id: u128,
    client_order_id: u64,
    quantity: u64,
    filled_quantity: u64,
    fee_is_deep: bool,
    order_deep_price: OrderDeepPrice,
    epoch: u64,
    status: u8,
    expire_timestamp: u64,
}

public struct OrderInfo has copy, drop, store {
    pool_id: ID,
    order_id: u128,
    balance_manager_id: ID,
    client_order_id: u64,
    trader: address,
    order_type: u8,
    self_matching_option: u8,
    price: u64,
    is_bid: bool,
    original_quantity: u64,
    executed_quantity: u64,
    cumulative_quote_quantity: u64,
    fills: vector<Fill>,
    paid_fees: u64,
    maker_fees: u64,
    status: u8,
    market_order: bool,
    order_inserted: bool,
    timestamp: u64,
}

public struct Fill has copy, drop, store {
    maker_order_id: u128,
    maker_client_order_id: u64,
    execution_price: u64,
    balance_manager_id: ID,
    expired: bool,
    completed: bool,
    base_quantity: u64,
    quote_quantity: u64,
    taker_is_bid: bool,
    taker_fee: u64,
    maker_fee: u64,
}
```

三个对象的职责不同：

- `Order`：会留在订单簿中的 maker 状态。
- `OrderInfo`：本次 taker 或新订单的执行结果，函数返回它，事件也从它产生。
- `Fill`：一次 taker 与 maker 的成交切片。

不要把 `OrderInfo` 当成订单簿里的长期订单。完全吃单的市价单可能只有 `OrderInfo` 和 `Fill`，不会变成 `Order`。

## 05 资产保管和闪电贷：Vault、FlashLoan

```move
public struct Vault<phantom BaseAsset, phantom QuoteAsset> has store {
    base_balance: Balance<BaseAsset>,
    quote_balance: Balance<QuoteAsset>,
    deep_balance: Balance<DEEP>,
}

public struct FlashLoan {
    pool_id: ID,
    borrow_quantity: u64,
    type_name: TypeName,
}
```

`Vault` 是池内资产保管层，撮合模块不直接转 coin。`FlashLoan` 是 hot potato：借出时返回 `Coin<T>` 和 `FlashLoan`，归还时必须把同一个 `FlashLoan` 消费掉。

关键入口：

```move
public fun borrow_flashloan_base<BaseAsset, QuoteAsset>(
    self: &mut Pool<BaseAsset, QuoteAsset>,
    base_amount: u64,
    ctx: &mut TxContext,
): (Coin<BaseAsset>, FlashLoan)

public fun return_flashloan_base<BaseAsset, QuoteAsset>(
    self: &mut Pool<BaseAsset, QuoteAsset>,
    coin: Coin<BaseAsset>,
    flash_loan: FlashLoan,
)
```

归还时校验三件事：`pool_id` 匹配，资产 `type_name` 匹配，归还数量等于 `borrow_quantity`。这不是 Margin 借贷，不存在跨交易债务。

## 06 Margin 核心：MarginManager、MarginPool、MarginRegistry

```move
public struct MarginManager<phantom BaseAsset, phantom QuoteAsset> has key {
    id: UID,
    owner: address,
    deepbook_pool: ID,
    margin_pool_id: Option<ID>,
    balance_manager: BalanceManager,
    deposit_cap: DepositCap,
    withdraw_cap: WithdrawCap,
    trade_cap: TradeCap,
    borrowed_base_shares: u64,
    borrowed_quote_shares: u64,
    take_profit_stop_loss: TakeProfitStopLoss,
    extra_fields: VecMap<String, u64>,
}
```

`MarginManager` 是用户在单个 DeepBook Pool 上的保证金账户。它内部持有一个 DeepBook `BalanceManager` 和三种 cap，因此 Margin 交易仍然复用 Spot 的账户模型，只是在外层增加借贷、oracle、风险率和清算约束。

```move
public struct MarginPool<phantom Asset> has key, store {
    id: UID,
    vault: Balance<Asset>,
    state: State,
    config: ProtocolConfig,
    protocol_fees: ProtocolFees,
    positions: PositionManager,
    allowed_deepbook_pools: VecSet<ID>,
    rate_limiter: RateLimiter,
    extra_fields: VecMap<String, u64>,
}
```

`MarginPool` 是某个资产的借贷池。`vault` 保存供应资产，`state` 管 shares、利息和借贷比例，`allowed_deepbook_pools` 决定哪些 DeepBook 池允许用这个资产做 Margin 交易。

## 07 Predict 核心：Predict 和交易入口

```move
public struct Predict has key {
    id: UID,
    vault: Vault,
    fee_reserve: FeeReserve,
    treasury_cap: TreasuryCap<PLP>,
    pricing_config: PricingConfig,
    risk_config: RiskConfig,
    treasury_config: TreasuryConfig,
    oracle_config: OracleConfig,
    withdrawal_limiter: RateLimiter,
    trading_paused: bool,
}
```

`Predict` 是预测市场的共享对象。它不是订单簿；它围绕 oracle、价格区间、vault 风险、LP 份额和费用储备组织状态。

核心入口：

```move
public fun mint<Quote>(
    predict: &mut Predict,
    manager: &mut PredictManager,
    oracle: &OracleSVI,
    key: RangeKey,
    quantity: u64,
    clock: &Clock,
    ctx: &mut TxContext,
)

public fun redeem<Quote>(
    predict: &mut Predict,
    manager: &mut PredictManager,
    oracle: &OracleSVI,
    key: RangeKey,
    quantity: u64,
    clock: &Clock,
    ctx: &mut TxContext,
)
```

`mint` 买入一个区间头寸，`redeem` 赎回活跃或已结算头寸。这里的难点不是撮合，而是 oracle 是否结算、vault 是否能承担风险、费用是否进入 fee reserve、LP 份额如何变化。

## 08 Indexer handler：事件如何变成表

```rust
define_handler! {
    name: OrderFillHandler,
    processor_name: "order_fill",
    event_type: OrderFilled,
    db_model: OrderFill,
    table: order_fills,
    map_event: |event, meta| OrderFill {
        event_digest: meta.event_digest(),
        digest: meta.digest(),
        checkpoint: meta.checkpoint(),
        pool_id: event.pool_id.to_string(),
        maker_order_id: event.maker_order_id.to_string(),
        taker_order_id: event.taker_order_id.to_string(),
        price: event.price as i64,
        base_quantity: event.base_quantity as i64,
        quote_quantity: event.quote_quantity as i64,
        onchain_timestamp: event.timestamp as i64,
    }
}
```

Indexer 不是“再查一次链上对象”。它把 Move 事件转成可查询的读模型。`event_digest` 通常由交易 digest 和 event index 组成，用来保证一笔交易中多个事件也能唯一落库。应用页面中的成交历史、K 线和账户流水，都要从这些事件语义推导。

## 后续写作要求

读者进入任何源码章节，都应该先看到本章相关的定义卡片，再看到调用链和链接。链接用于复核，不应该承担解释职责。
