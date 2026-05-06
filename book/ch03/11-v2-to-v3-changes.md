# ch03-11 DeepBookV2 到 V3 的关键变化

[返回本章](README.md)

## 先抓住结构

读“DeepBookV2 到 V3 的关键变化”时先画边界。一个真实协议最容易读乱的地方，不是函数太多，而是不知道 Pool、Book、State、Vault 和 BalanceManager 各自负责哪一段。

## 源码入口

这一节只保留必要入口，目的不是让你马上读完源码，而是建立后续定位能力：

- [packages/deepbook/sources/pool.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/pool.move)
- [packages/deepbook/sources/balance_manager.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/balance_manager.move)
- [packages/deepbook/sources/book/order_info.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/book/order_info.move)
- [packages/deepbook/sources/vault/vault.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/vault/vault.move)
- [packages/deepbook/sources/registry.move](https://github.com/MystenLabs/deepbookv3/blob/663edbf9c30d6c93100e6cd66936e1487a5dc9e0/packages/deepbook/sources/registry.move)

读源码时先确认对象、函数签名和事件名称；等正文讲到交易路径时，再回到这些入口核对。

## 读架构

从源码结构看，V3 的重点是把交易入口统一到 `Pool`，把用户资金抽象到 `BalanceManager`，并通过 `State` 把账户、历史、治理和费用组织到池内。相比只关注订单簿本身的阅读方式，V3 更强调完整交易账户：

- 订单簿不直接保管用户余额。
- 订单生命周期返回 `OrderInfo`，事件和余额计算围绕它展开。
- 池状态使用 `Versioned` 控制升级可用性。
- `Vault` 明确承担资产保管和闪电贷底层能力。

如果你迁移旧集成，优先检查 BalanceManager、TradeProof、订单类型常量、事件结构和 fee 字段，而不是只替换下单函数名。

## 阅读补充

从 V2 到 V3 的阅读重点不是“函数名变了”，而是架构职责重新拆分：用户余额通过 BalanceManager 抽象，订单生命周期集中到 OrderInfo，资产保管交给 Vault，版本控制进入 Pool/Registry 加载路径。

迁移应用时要先重画交易输入对象和事件消费逻辑。旧代码如果假设订单簿直接保管用户余额，或只监听旧事件字段，就会在 V3 中出现余额解释和订单状态回放错误。

## 工程判断

- 迁移前列出旧代码依赖的余额、订单和事件假设。
- 把 V3 的 BalanceManager 充值/授权流程纳入用户 onboarding。
- 版本控制和 Registry 检查应成为 SDK 初始化的一部分。

## 读完以后问自己

- V3 为什么引入 BalanceManager 抽象？
- OrderInfo 对事件和余额计算有什么帮助？
- 旧 V2 应用迁移时最容易漏掉哪两类输入对象？
