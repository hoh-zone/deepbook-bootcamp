# s01 闪电贷 Move wrapper

目标：编写一个 Move wrapper，在同一函数中调用借出和归还，验证 hot potato 模式。

核心调用：

```move
let (coin, loan) = pool::borrow_flashloan_base<Base, Quote>(pool, amount, ctx);
pool::return_flashloan_base<Base, Quote>(pool, coin, loan);
```

构建与测试：

```bash
sui move build
sui move test
```

测试用例建议：

- `amount = 0`，期望 `EInvalidLoanQuantity = 3`。
- 借 base 后归还 base，成功。
- 借 quote 后归还 quote，成功。
- 借 base 后尝试归还 quote，期望 `EIncorrectTypeReturned = 5`。

注意事项：

- wrapper 不能保存 `FlashLoan`，它没有 `store`。
- wrapper 不能丢弃 `FlashLoan`，它没有 `drop`。
- 利润必须和本金拆开，只归还精确本金。
