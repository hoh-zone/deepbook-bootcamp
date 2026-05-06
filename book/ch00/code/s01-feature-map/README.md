# s01-feature-map 功能地图清单

本示例不是可执行程序，而是第 00 章的配套维护清单。后续可以把它扩展成脚本，从 DeepBook 源码中自动提取模块、公开函数、事件和索引器 handler。

## 初始清单

| 功能域 | 源码路径 | 本书章节 |
| --- | --- | --- |
| DeepBookV3 spot | `packages/deepbook/sources` | ch03-ch07 |
| BalanceManager | `packages/deepbook/sources/balance_manager.move` | ch05 |
| Matching engine | `packages/deepbook/sources/book` | ch04 |
| Flashloans | `packages/deepbook/sources/pool.move`、`packages/deepbook/sources/vault/vault.move` | ch06 |
| Margin | `packages/deepbook_margin/sources` | ch08-ch09 |
| Predict | `packages/predict/sources` | ch13-ch14 |
| Indexer | `crates/indexer` | ch11-ch12 |
| Server | `crates/server` | ch12 |
| SDK and scripts | `scripts/transactions`、官方 TypeScript SDK | ch10 |

## 后续扩展

可以增加一个 `extract.ts`，输出如下 JSON：

```json
{
  "domain": "spot",
  "source": "packages/deepbook/sources/pool.move",
  "publicFunctions": [
    "place_limit_order",
    "place_market_order",
    "swap_exact_base_for_quote",
    "borrow_flashloan_base"
  ],
  "chapter": "ch03-ch07"
}
```

这个清单用于控制书稿范围：每当 DeepBook 源码升级，先更新功能地图，再决定哪些章节需要重写。
