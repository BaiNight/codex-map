# expense · amazon-expense-calculate

- 复杂度 / 重要性：约 20 / 6（V 宽 + 调用深 + 多分支 + 多表 + 跨入口 + 对外）
- 一句话：Amazon 交易（V2 / TAG2）平铺后进结算队列，按归属 SKU/ORDER/SHOP 写入费用明细，并延迟上卷利润
- 入口：拉单侧 `AmazonGetTransaction` → Consumer `amazon_transaction_resolve`；CLI `php yii amazon-finance/transaction-resolve` / `dispatch-transaction-item-settlement`；主计算 Consumer `order_platform_settlement`
- 流程图：[../../diagrams/flows/expense-amazon-expense-calculate.svg](../../diagrams/flows/expense-amazon-expense-calculate.svg)
- 选择键：结算 `type = PlatformEnum::AMAZON . ExpenseTag::TAG2`（即 `2` + `v2`）；店铺分叉走 `AmazonTransactionStrategyProvider` 的 `correspond()`

> 不含：订单初始费用（`order_init_expense_calculate` + `AmazonExpenseInitializeStrategy`）；v1 `AmazonSettlementExpenseStrategy`（Provider 里已注释）。二者另立功能。

### 步骤

1. 触发：
   - 拉数：`common/platform/amazon/AmazonGetTransaction.php` 写原始交易后投 `amazon_transaction_resolve`，body `{"id": mongo_id}`。
   - 平铺：`AmazonTransactionResolveConsumer::consume` → `AmazonTransactionResolveTask::handler`。`backend/modules/v1/consumer/AmazonTransactionResolveConsumer.php`
   - 重跑：`php yii amazon-finance/transaction-resolve` 按条件捞 id 再投同一队列；`dispatch-transaction-item-settlement` 直接投 `order_platform_settlement`。`console/controllers/AmazonFinanceController.php`
   - 计算：`OrderExpenseSettlementConsumer::consume`，消息 `type` + `uniqueKeys`。`backend/modules/v1/consumer/OrderExpenseSettlementConsumer.php`
2. 校验：
   - Resolve：`id` 空或原交易不存在 → stdout 后 ACK。
   - 计算：`transactionStatus != RELEASED` → 跳过；`order_expense_field_map` 找不到 → mismatch + notify；`process_type == SKIP` → 跳过并 `return`（整批提前结束）。
3. 编排：
   - Resolve：`AmazonTransactionResolveTask::handler` 平铺 item/breakdown → `amazonTransaction->createOrUpdateItem` → `MqHelper::publish(order_platform_settlement, type=2v2)`。
   - 计算：`StrategyContext` + `ExpenseSettlementStrategyProvider` → `AmazonTransactionExpenseStrategy::handler` → `ExpenseSegmentManager::run(AmazonTransactionItemHandlerOperatorChain)` → `AmazonTransactionDropped::process` → `handleUniqueKeys` / `handleTransactions`。
4. 分叉：
   - 平台结算：`type` = `AMAZON`+`TAG2` 选 `AmazonTransactionExpenseStrategy`（v1 Settlement 策略已注释）。
   - 归属：`getAscription` — `Shipment`/`Refund` 且有 sku → SKU；无 sku → ORDER；其它 transactionType → SHOP。
   - 店铺：`executeCorrespond(AmazonTransactionStrategyProvider)` 按 `correspond()` 顺序匹配（见实现细节）。
5. 写库：
   - 平铺：mongo `amazon_transaction_item`。
   - 费用：mongo `order_expense_item_stats`（`createOrUpdate` / `createOrUpdateShop`）；店铺前先 `destroyShopExpense` 删 item+shop stats。
   - 订单：`updateSettlement` / `updateWithdrawalStatus` 可能改 `order.settlement_*` / `withdrawal_status`。
   - 失配：mongo `platform_expense_mismatch`。
6. 发出：
   - Resolve → MQ `order_platform_settlement`（routing `order_expense_settlement`）。
   - 有订单号：`OrderSettlementCache::setOrderId` 延迟上卷；定时 `php yii order/settlement-profit-cal` → producer `order_expense_calculate`。
   - 店铺分支可能调 `xc_tc`（移仓/头程）、`xc_wms`（fnsku→seller_sku）。
   - 失败重投：`Common::pushToFeeCalcQueue($type, $uniqueKeys[0], delay)`。
7. 失败：Settlement Consumer catch → notify + 延迟重入队，仍 ACK。Resolve publish 失败只打日志。fieldMap 缺失走 `misMatchRecord` + `fastNotifyObserver`。

---

### 实现细节

#### 调用栈

| # | 类::方法 | 路径 | 做什么 | 下一跳 |
|---|----------|------|--------|--------|
| 1 | `AmazonGetTransaction` 拉批 | `common/platform/amazon/AmazonGetTransaction.php` | 写原始交易，投平铺队列 | `amazon_transaction_resolve` |
| 2 | `AmazonTransactionResolveConsumer::consume` | `backend/modules/v1/consumer/AmazonTransactionResolveConsumer.php` | 按 id 取原交易 | ResolveTask |
| 3 | `AmazonTransactionResolveTask::handler` | `backend/modules/v1/task/expense/process/AmazonTransactionResolveTask.php` | 平铺 breakdown，只收集 RELEASED 的 uniqueKey | createOrUpdateItem + MQ settlement |
| 4 | `OrderExpenseSettlementConsumer::consume` | `backend/modules/v1/consumer/OrderExpenseSettlementConsumer.php` | 按 `type` 选平台策略 | StrategyContext::execute |
| 5 | `AmazonTransactionExpenseStrategy::handler` | `backend/modules/v1/strategy/expense/settlement/AmazonTransactionExpenseStrategy.php` | Amazon V2 入口 | ExpenseSegmentManager::run |
| 6 | `ExpenseSegmentManager::run` | `backend/modules/v1/task/expense/segment/ExpenseSegmentManager.php` | Pipeline + 关账限制 | Chain.apply |
| 7 | `AmazonTransactionItemHandlerOperatorChain::apply` | `…/chain/AmazonTransactionItemHandlerOperatorChain.php` | 仅挂 `AmazonTransactionDropped` | Operator |
| 8 | `AmazonTransactionDropped` → `handleTransactions` | `…/components/amazon/AmazonTransactionDropped.php` · `AbstractAmazonTransactionOperator.php` | 按条过滤、分归属、写费用、延迟上卷 | SKU/ORDER/SHOP |

#### 分支

| 条件（抄代码） | 走向 | 路径 |
|---|---|---|
| `transactionStatus != RELEASED` | 跳过该条 | `AbstractAmazonTransactionOperator::handleTransactions` |
| `findByUniqueKeyAndPlatform` 空 | `misMatchRecord` + continue | 同上 |
| `fieldMap->process_type == SKIP` | stdout 后 **return**（结束当前 handleTransactions） | 同上 |
| `transactionType in {Shipment,Refund}` 且有 sku | `skuExpenses` | `getAscription` |
| 同上但无 sku | `orderExpenses` | 同上 |
| 其它 transactionType | `shopExpenses` | 同上 |
| ORDER/SKU 找不到订单或 item | 降级 `shopExpenses(..., lost=true)` | `orderExpenses` / `skuExpenses` |
| sku 且 `orderId` 以 `S0` 开头 | 多渠道：物流查单 → `emitMultiChannelOrder` | `skuExpenses` |
| Shipment + 有 postedDate + 非 FBA 操作费 | `updateSettlement` 写订单结算态 | `updateSettlement` |

#### 分叉展开（V）

**A. 结算 Provider** — `ExpenseSettlementStrategyProvider`  
选择键：`$data->type`。Amazon 现用 `AmazonTransactionExpenseStrategy`，`type()` = `PlatformEnum::AMAZON . ExpenseTag::TAG2`。`AmazonSettlementExpenseStrategy`（纯 AMAZON）在 providers 里已注释。

**B. 店铺 Bool Provider** — `AmazonTransactionStrategyProvider`（`executeCorrespond` **按数组顺序**取第一个 `correspond==true`）

| 顺序 | 类 | correspond | 一句话 |
|---:|---|---|---|
| 1 | `AmazonDealRelatedSkuStrategy` | `strategyRules`（field_map.strategy == 类短名） | 按 DealSku 销量占比拆店铺费 |
| 2 | `AmazonTransactionCouponStrategy` | `strategyRules` | 按优惠券关联 AmazonOrderItem 数量占比拆 |
| 3 | `AmazonTransactionOrderIdStrategy` | 有 orderId 且 asin/sku 皆空 | 常规订单 / FBA 头程 / 移仓 三路分摊 |
| 4 | `AmazonTransactionOrderIdWithSkuStrategy` | 有 orderId 且 (asin 或 sku) | 订单+SKU 对齐；对不上则 default 店铺费 |
| 5 | `AmazonTransactionShipmentIdStrategy` | 有 shipmentId | 移仓单按 quantity 分摊 |
| 6 | `AmazonTransactionOrdinaryStrategy` | 恒 true | 兜底：单条 default 店铺费 |

对照分支（相对「有 sku 的 Shipment 主路径」）：无 sku 的 Shipment → `orderExpenses` 按订单金额比例拆到各 OrderItem → `saveExpenseItem`；找不到订单则店铺 `lost=true`。

#### 写库

| Model | tableName / collection | 调用 | 能看到的赋值 | 条件 |
|---|---|---|---|---|
| `AmazonTransactionItem` | `amazon_transaction_item` | `createOrUpdateItem` | 平铺字段 + `uniqueKey` md5 | Resolve |
| `OrderExpenseFieldMap` | `order_expense_field_map` | 只读 `findByUniqueKeyAndPlatform` | — | 映射键 `transactionType.breakdownType`（或 config 下用 description） |
| `OrderExpenseItemStats` | `order_expense_item_stats` | `createOrUpdate` / `createOrUpdateShop` | fee、tag=v2、source=平台结算报告、unique_key 等 | SKU/ORDER 或 SHOP |
| `OrderExpenseShopStats` | `order_expense_shop_stats` | `deleteAll`（destroy） | — | 写店铺费前清 |
| `Order` | `order` | `updateByEntity` | `settlement_status/time`、`withdrawal_status` | Shipment 结算 / 有 settlementId |
| `PlatformExpenseMismatch` | `platform_expense_mismatch` | `record("amazon", …)` | — | 无 fieldMap |
| `AmazonDealSku` | `amazon_deal_sku`（table） | 只读 | — | Deal 策略 |
| mongo `AmazonOrderItem` | 待确认 collection | 只读 PreferenceIds | — | Coupon 策略 |

#### 消息与外部

| 方向 | 队列 / launcher | 字段 | 路径 |
|---|---|---|---|
| 出 | MQ `amazon_transaction_resolve` | `id` | AmazonGetTransaction / CLI |
| 出 | MQ `order_platform_settlement` | `type`=`2v2`，`uniqueKeys`[] | ResolveTask；Common::pushToFeeCalcQueue |
| 出 | Redis `order_settlement:{H}` → 后 MQ `order_expense_calculate` | order id | delayedRollup；`SettlementProfitCalculateTask` |
| 出 | `xc_tc` POST `/internal/api/order/removal-order-info` | `order_no` | `GetRemovalOrderInfoRequest` |
| 出 | `xc_tc` AbnormalOrderInfo | `shipment_id` 等 | `AbnormalOrderInfoRequest`（头程） |
| 出 | `xc_wms` POST `/internal/api/list-fba-inv` | `fnsku`、`shop_id` | `GetFbaInventoryRequest` |
| 内 | notify | mismatch / settlement catch | `NotifyBo` / `immediatelyNotify` |

#### 失败与禁迁

| 条件 | 行为 | 路径 |
|---|---|---|
| 非 RELEASED | 跳过计算 | handleTransactions |
| process_type SKIP | return 结束本批处理 | 同上 |
| Settlement 抛错 | notify + 延迟 pushToFeeCalcQueue；ACK | OrderExpenseSettlementConsumer |
| Resolve 投 settlement 失败 | stdoutErr，不抛给上层 | ResolveTask |
| 无 fieldMap | mismatch 表 + 限频 notify | misMatchRecord |

#### 事务 / 锁 / 幂等

- 本功能未见 DB `transaction` 包住整条结算。
- 幂等：item `uniqueKey`；写店铺前 `destroyShopExpense(uniqueKey)`；`createOrUpdate` / `createOrUpdateItem`。
- 延迟上卷：`OrderSettlementCache`（Redis），非分布式锁。
- `ExpenseSegmentManager` 关账限制 / RedLock 仍可能命中（通用路径），本链 AmazonTransactionDropped 以 delayedRollup 为主。

## 需人工确认

- Coupon 策略查询的 `AmazonOrderItem` collectionName 本轮未展开。
- `field->component` 配置表里具体插件列表未打开 `params['expense']['component']`。
- v1 结算链（`AmazonSettlementItemHandlerOperatorChain`）是否仍有历史消息依赖。
- 拉原始交易写入的 `amazon_transaction`（非 item）字段集未全文展开。
