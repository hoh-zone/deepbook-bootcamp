# ch08-15 开多和开空资金路径

[返回本章](README.md)

## 先看风险边界

读“开多和开空资金路径”时先问：这一步会让账户风险变大还是变小，谁有权继续执行，失败时应该归因到价格、债务、池配置还是对象权限。

## 源码入口

重点阅读：

- [packages/deepbook_margin/sources/margin_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/margin_manager.move)
- [packages/deepbook_margin/sources/pool_proxy.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/pool_proxy.move)
- [packages/deepbook_margin/sources/margin_pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook_margin/sources/margin_pool.move)

> **源码旁白**：先定位结构体、入口函数和事件，再回到本节的资金路径或应用流程。不要从 helper 函数开始读。

## 读风险控制面

开多通常是“抵押 quote，借 quote，买 base”。路径是：用户存入 USDC 到 `MarginManager<SUI, USDC>`，调用 `borrow_quote` 从 `MarginPool<USDC>` 借 USDC，借款进入 manager，然后通过 `pool_proxy.place_market_order` 买入 SUI。此时资产端有 SUI 和可能剩余 USDC，债务端是 USDC borrow shares。

开空通常是“抵押 quote 或 base，借 base，卖 base”。路径是：用户调用 `borrow_base` 从 `MarginPool<SUI>` 借 SUI，借款进入 manager，然后通过 `pool_proxy.place_market_order` 卖出 SUI 得到 USDC。此时资产端主要是 USDC，债务端是 SUI borrow shares。

两条路径都依赖同一个风险率公式，只是 debt unit 不同。开多的风险通常对 base 下跌敏感；开空的风险通常对 base 上涨敏感。

## 工程旁白

开多通常借 quote，例如借 USDC 买 SUI；债务是 quote，资产侧多了 base，base 下跌会让资产折算价值下降。开空通常借 base，例如借 SUI 卖成 USDC；债务是 base，base 上涨会让用 quote 折算后的偿债压力上升。

无论开多还是开空，借来的 coin 会进入同一个 `MarginManager` 的 DeepBook BalanceManager，再由 `pool_proxy` 下单成交。它不是闪电贷，因为债务不会在同一笔交易末尾自动关闭，后续还款需要显式调用 `repay_base` 或 `repay_quote`。

## 风控判断

- 交易预览必须显示 debt asset，否则用户无法理解价格方向对风险率的影响。
- 开仓 PTB 完成后立即结算或刷新 settled balances，避免 UI 漏算资产。
- 组合借款加下单时，先检查借款后风险率，再检查成交后风险率。

## 动手检查

- 借 quote 买 base 后，哪种价格变化会降低风险率？
- 借 base 卖出后，还款需要回购哪种资产？
- 如果只完成借款不下单，manager 的风险率和余额会如何显示？
