# ch00-08 闪电贷在源码中的位置

[返回本章](README.md)

DeepBookV3 当前锁定源码中存在闪电贷能力。它不在 `deepbook_margin` 包里，而是在现货核心的 `Pool` 和 `Vault` 路径中。

公开入口位于 [pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)：`borrow_flashloan_base`、`borrow_flashloan_quote`、`return_flashloan_base` 和 `return_flashloan_quote`。这四个函数按资产方向拆开：base 和 quote 都可以借出，也都必须按对应类型归还。

具体实现位于 [vault.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/vault/vault.move)。`Vault` 中定义了 `FlashLoan` hot potato 结构和 `FlashLoanBorrowed` 事件。borrow 函数会检查借款数量大于 0、vault 中有足够余额，然后把资产拆出为 `Coin`，同时返回一个 `FlashLoan` 证明。return 函数会校验 pool id、资产 type name 和归还数量，最后销毁 `FlashLoan`。

理解这个设计要抓住 hot potato 模式：调用者拿到借出的 `Coin` 后，也拿到一个必须被处理掉的 `FlashLoan` 对象。Sui Move 的资源语义会迫使交易在同一个 PTB 中完成借出和归还，否则无法合法结束。这就是链上闪电贷的关键约束：不是靠后台追债，而是靠类型和资源生命周期保证原子性。

闪电贷和 Margin 借贷不同。Margin 的借款会形成持续债务、风险率、利息和清算路径；DeepBookV3 spot 的闪电贷是同交易内借出和归还，不形成长期 loan shares。读源码时不要因为都出现 borrow、loan 这些词就混在一起。

应用场景包括原子套利、组合清算、跨协议再平衡和无需预先持仓的复杂 PTB。但生产使用前必须验证池子流动性、归还数量、费用外部性、交易失败时的用户提示和潜在 MEV 风险。

## 本节检查

- [ ] 能指出闪电贷的公开入口和实现文件。
- [ ] 能解释 `FlashLoan` hot potato 的意义。
- [ ] 能区分现货闪电贷和 Margin 持续借贷。
