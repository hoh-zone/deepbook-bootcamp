# s01-architecture-map

目标：从 DeepBookV3 Move 源码中提取模块依赖关系，生成 ch03 架构图的输入数据。

建议读取：

- [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)
- [packages/deepbook/sources/book/book.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/book/book.move)
- [packages/deepbook/sources/state/state.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/state/state.move)
- [packages/deepbook/sources/vault/vault.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/vault/vault.move)
- [packages/deepbook/sources/balance_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/balance_manager.move)
- [packages/deepbook/sources/registry.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/registry.move)

可扩展实现：

```bash
pnpm init
pnpm add -D tsx typescript
pnpm tsx index.ts
```

`index.ts` 可以扫描 `use deepbook::{...}` 和 `use deepbook::module`，输出 Mermaid：

```text
pool --> book
pool --> state
pool --> vault
pool --> balance_manager
pool --> registry
state --> account
state --> governance
state --> history
book --> order
book --> order_info
order_info --> fill
```

生产注意事项：源码依赖图只能说明编译期依赖，不能替代交易调用链。下单路径还需要结合 `pool::place_order_int` 和 `state::process_create` 手工标注。
