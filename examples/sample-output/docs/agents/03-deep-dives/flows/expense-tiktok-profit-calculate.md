# expense · tiktok-profit-calculate

- 范围：**只展开** RabbitMQ `order_expense_calculate` → `OrderProfitCalculateConsumer::consume` 主链。
- 不展开：付款/结算等上游投递、`order_profit_task` 正常消费链、广告费、结算报告拉取、订单初始化费用。
- 主链图：[../../diagrams/flows/expense-tiktok-profit-calculate.svg](../../diagrams/flows/expense-tiktok-profit-calculate.svg)
- 分支对照图：[../../diagrams/flows/expense-tiktok-profit-calculate-branches.svg](../../diagrams/flows/expense-tiktok-profit-calculate-branches.svg)
- 队列绑定：`order_expense_calculate` → exchange `order.calculate` → routing key `order_profit`。
- TikTok 选择值：`PlatformEnum::TIKTOK = 53`。

> “前 12 个算子”按 `TiktokAllOperatorChain::apply()` **实际返回数组的前 12 项**计算。由于 `build_order_type` 决定数组前缀，SYSTEM 与 MANUAL 的前 12 项不同，本文分别逐项列出。

## 主链步骤

1. `OrderProfitCalculateConsumer::consume()` 读取消息体；`empty($body)` 或 JSON 解码后 `empty($data)` 时直接 ACK。
2. `Oms::$service->order->findByIds($data->orderIds)` 批量读取订单，逐单执行 Strategy。
3. `$this->context->has($order->platform_id)` 命中 TikTok Strategy；TikTok 的 `type()` 返回 `PlatformEnum::TIKTOK`（53）。
4. `TiktokOrderProfitCalculateStrategy::handler()` 用 `associateOrder($payload)` 找同平台单号关联订单，逐单创建 `TiktokAllOperatorChain`。
5. `ExpenseSegmentManager::run()` 构造 `ProfitPayload`，执行关账限制、`can()` 门禁、`before()` 清理和 Pipeline。
6. `TiktokAllOperatorChain::apply()` 按 `build_order_type` 组装算子数组；分支图完整画出两条分支各自前 12 项。
7. 算子通过 `createOrUpdate*` 写 `order_expense_item_stats`；Pipeline 完成后触发 `OrderExpenseChangeEvent`。
8. `OrderExpenseCalculateListener` 启动 `ExpenseOrderAndSkuRollupTask`，依次上卷并独立写 `order_expense_sku_stats`、`order_expense_stats`、`order`、`order_item`。
9. 正常或失败最终都由 Consumer `finally` 返回 `ConsumerInterface::MSG_ACK`。

## 调用栈

| # | 类::方法 | 源码路径 | 下一跳 |
|---|---|---|---|
| 1 | `OrderProfitCalculateConsumer::__construct` | `backend/modules/v1/consumer/OrderProfitCalculateConsumer.php` | `StrategyContext::init(new OrderProfitCalculateStrategyProvider())` |
| 2 | `OrderProfitCalculateConsumer::consume` | 同上 | `findByIds` → Strategy |
| 3 | `OrderProfitCalculateStrategyProvider::providers` | `backend/modules/v1/strategy/profit/OrderProfitCalculateStrategyProvider.php` | `TiktokOrderProfitCalculateStrategy` |
| 4 | `TiktokOrderProfitCalculateStrategy::handler` | `backend/modules/v1/strategy/profit/TiktokOrderProfitCalculateStrategy.php` | `associateOrder` → `ExpenseSegmentManager::run` |
| 5 | `AbstractOrderChain::payload` | `backend/modules/v1/task/expense/segment/chain/AbstractOrderChain.php` | 组装 order/items/recipient/logistics/头程 |
| 6 | `ExpenseSegmentManager::run` | `backend/modules/v1/task/expense/segment/ExpenseSegmentManager.php` | restriction → can → before → Pipeline |
| 7 | `TiktokAllOperatorChain::apply` | `backend/modules/v1/task/expense/segment/chain/TiktokAllOperatorChain.php` | 算子数组 |
| 8 | `ExpenseSegmentManager::profit` | `backend/modules/v1/task/expense/segment/ExpenseSegmentManager.php` | `OrderExpenseChangeEvent` |
| 9 | `OrderExpenseCalculateListener::handler` | `backend/modules/v1/listener/expense/OrderExpenseCalculateListener.php` | `ExpenseOrderAndSkuRollupTask::run` |
| 10 | `RollupSkuAndOrderExpenseChain::apply` | `backend/modules/v1/task/expense/rollup/chain/RollupSkuAndOrderExpenseChain.php` | SKU/订单预估与真实上卷 |
| 11 | `ExpenseOrderAndSkuRollupTask::orderProfit` | `backend/modules/v1/task/expense/rollup/ExpenseOrderAndSkuRollupTask.php` | 写订单利润与 SKU 收入 |

## 关键判断条件（原样抄代码）

### Consumer 与 Strategy

| 代码条件 | 结果 | 源码 |
|---|---|---|
| `if (empty($body))` | 直接 ACK | `OrderProfitCalculateConsumer.php:46` |
| `if (empty($data))` | 直接 ACK | `OrderProfitCalculateConsumer.php:55` |
| `if (!empty($data->source) && $data->source == 'debugReprofit')` | 写 `debugReprofit:{order_id}` 缓存 10 秒 | `OrderProfitCalculateConsumer.php:70` |
| `if ($this->context->has($order->platform_id))` | 执行平台 Strategy | `OrderProfitCalculateConsumer.php:75` |
| `if ($this->context->existsSubstitute())` | 未命中平台时执行 substitute | `OrderProfitCalculateConsumer.php:80` |
| `return PlatformEnum::TIKTOK;` | TikTok Strategy 类型为 53 | `TiktokOrderProfitCalculateStrategy.php:48` |

### Payload、关账与 Pipeline

| 代码条件 | 结果 | 源码 |
|---|---|---|
| `if (!$payload->hasOrderExclude)` | 才调用 WMS 头程配货 | `AbstractOrderChain.php:67` |
| `if ($order->warehouse_id <= 0 || $order->pay_status == BoolEnum::NO || empty($recipient->recipient_country))` | 头程数据返回空数组 | `AbstractOrderChain.php:88` |
| `if (!$item->sku)` | 头程数据返回空数组 | `AbstractOrderChain.php:92` |
| `if ($item->quantity == 0)` | 当前 SKU 不发给 WMS | `AbstractOrderChain.php:168` |
| `if ($source != "refundRouting" && !Oms::$service->orderExpenseItemStats->isOnlyCalItemStat($payload->order ?? null))` | 执行关账限制 | `ExpenseSegmentManager.php:53` |
| `if (empty($payload->order) || empty($payload->order->settlement_time))` | 不触发关账改道 | `ExpenseSegmentManager.php:131` |
| `if (empty($latestSettlementAt))` | 不触发关账改道 | `ExpenseSegmentManager.php:139` |
| `if ($orderSettlementAt->timestamp < $limitSettlementAt->timestamp && $latestSettlementAt->timestamp > $limitSettlementAt->timestamp)` | 投 `order_profit_rollup`，本次不跑算子 | `ExpenseSegmentManager.php:148` |
| `if ($latestSettlementAt->timestamp < $limitSettlementAt->timestamp)` | 记日志并结束本次计算 | `ExpenseSegmentManager.php:153` |
| `if (!$chain->can())` | 释放 `order:expense:calculate:{id}` 锁并结束 | `ExpenseSegmentManager.php:63` |
| `if (Oms::$service->orderExpenseItemStats->isOnlyCalItemStat($this->order))` | `before()` 不清旧费用 | `AbstractAllOperatorChain.php:36` |
| `if ($order && Yii::$app->cache->exists('debugReprofit:' . $order->id))` | `isOnlyCalItemStat()` 返回 true，仅重算 item 明细 | `OrderExpenseItemStatsService.php:1108` |

### `AbstractAllOperatorChain::can()`

| 代码条件 | 结果 |
|---|---|
| `if (empty($order->original_order_currency))` | `return false` |
| `if (empty($order->exchange_rate))` | `return false` |
| `if ($order->pay_status != BoolEnum::YES)` | `return false`；`BoolEnum::YES = 1` |
| `if (empty($quantity))` | `return false` |
| `if (!$this->checkItems())` | `return false` |
| `if (empty($items))` | `checkItems()` 返回 false |
| `if (empty($item->sku))` | `checkItems()` 返回 false |
| `if ($order->build_order_type == BuildOrderTypeEnum::MANUAL && $order->order_status == OrderStatusEnum::CANCEL_VAL)` | 清费用后 false；`MANUAL = 1`，`CANCEL_VAL = 9` |
| `if ((count($splitMarks) > 0 || count($mergeMarks) > 0) && $order->build_order_type == BuildOrderTypeEnum::SYSTEM)` | 进入拆合单检查；`SYSTEM = 0` |
| `if (count($cancelStatus) === count($orders))` | 清费用后 false |
| `if ($receiptType != ExpenseItemReceiptTypeEnum::ORIGINAL && $order->order_status == OrderStatusEnum::CANCEL_VAL)` | 清费用后 false；`ORIGINAL = 1` |
| `if ($order->shipping_type == ShippingTypeEnum::FBA && empty($order->warehouse_id))` | 清费用后 false；`FBA = 1` |
| `if ($order->shipping_type == ShippingTypeEnum::FBM && (empty($order->warehouse_id) || empty($order->shipping_method_id)))` | 进入 FBM 缺配置检查；`FBM = 2` |
| `if ($order->is_original_order)` | 上一条命中时直接 true |

### 分支与通用跳过规则

| 代码条件 | 结果 |
|---|---|
| `if ($this->order->build_order_type == BuildOrderTypeEnum::MANUAL)` | MANUAL 前缀；`MANUAL = 1` |
| `else if ($this->order->build_order_type == BuildOrderTypeEnum::SYSTEM)` | SYSTEM 前缀；`SYSTEM = 0` |
| `if ($this->isSkip($payload))` | 当前算子直接 `$next($payload)` |
| `return $this->payload->order->build_order_type != BuildOrderTypeEnum::SYSTEM || $this->payload->receiptType != ExpenseItemReceiptTypeEnum::ORIGINAL;` | `PlatformOrderSkipRule`；非系统单或非原单跳过 |
| `switch ($receiptType)`：`REISSUE/RESEND/OTHER`、`ORIGINAL`、`SPLIT`、`MERGE` | `ComplexSkipRule` 选择对应规则；值分别为 4/5/7、1、2、3 |

## SYSTEM 分支：返回数组前 12 个算子

共享前置：第 1、2、5～12 项受 `PlatformOrderSkipRule` 控制；第 3、4 项覆盖了 `isSkip()`，改走平台取消且已发货的专用门禁。第 2～12 项继承 `AbstractTiktokInitialOrderExpenseOperator`，会执行 `if (!$payload->originalOrder)`、`if (!$originalOrder)`、SKU 映射判断和 `if ($fee == null)`；保存时调用 `saveRealityWithEstimateItem()`，分别以 `ExpenseStageEnum::REALITY = 2`、`ESTIMATE = 1` 写两条费用项。

| # | 算子节点 | 节点内关键判断/动作（代码原样） |
|---:|---|---|
| 1 | `TiktokFullSettlementExpense` | `if (Oms::$service->orderExpenseItemStats->isOnlyCalItemStat($payload->order))` 跳过结算；`if (empty($transactions))` 下一算子；`if ($transaction['type'] == 'ORDER')` 走 SKU 费用，否则店铺费用 |
| 2 | `TiktokSalesRevenue` | `if ($siteConfig && $salesAmount > 0 && in_array($order['site'], $siteConfig->custom_conf))` 扣产品税；写 `SALES_REVENUE` |
| 3 | `TiktokPlatformCancelRefundAmount` | `if (empty($shopConf))`、`if (!in_array($order['shop_id'], $southeastShopId))`、`if (empty($settlement))`、`if (!empty($refundTransactions))` 均返回 null；`return $payload->order->platform_order_status != PlatformOrderStatusEnum::TIKTOK_CANCELLED || $payload->order->shipping_status != ShippingStatusEnum::YES;`；枚举值 54、1 |
| 4 | `TiktokPlatformCancelRefundSellerDiscountPrice` | 继承第 3 项门禁；`if (is_null($refundAmount))` 返回 null；写退款卖家优惠 |
| 5 | `TiktokTaxRevenue` | `if ($siteConfig && in_array($order['site'], $siteConfig->custom_conf))` 取产品税，否则遍历 `item_tax` |
| 6 | `TiktokShippingFeeRevenue` | `if ($this->isSkipSettlement($order))` 返回 null；当前基类实现固定 `return false;`；EU 站扣 `payment.shipping_fee_tax` |
| 7 | `TiktokShippingFeeSellerDiscount` | `if ($this->isSkipSettlement($order))` 返回 null；当前固定 false；写负的卖家运费优惠 |
| 8 | `TiktokDeductionTaxRevenue` | `if ($sstTax)` 时累加并把 source 改为 `PLATFORM_EXPENSE_SETTLEMENT_REPORT`；最终返回负税费 |
| 9 | `TiktokPlatformDiscountPrice` | 无额外分支；`return -ArrayHelper::getValue($item, 'platform_discount');` |
| 10 | `TiktokPlatformOtherExpenseDiscount` | 无额外分支；`return ArrayHelper::getValue($item, 'platform_discount');` |
| 11 | `TiktokSellerDiscountPrice` | 无额外分支；`return -ArrayHelper::getValue($item, 'seller_discount');` |
| 12 | `TiktokRetailDeliveryOtherRevenue` | 无额外分支；`return ArrayHelper::getValue($item, 'retail_delivery_fee');` |

## MANUAL 分支：返回数组前 12 个算子

MANUAL 前缀只有 6 项，因此第 7～12 项来自紧随其后的公共 `array_push()`，不是省略项。

| # | 算子节点 | 节点内关键判断/动作（代码原样） |
|---:|---|---|
| 1 | `ShippedSettlement` | `if (empty($payload->order))` 跳过；`if ($payload->order->shipping_status == BoolEnum::YES)` 更新结算态 |
| 2 | `ManualSalesRevenue` | `if ($order->build_order_type != BuildOrderTypeEnum::MANUAL && in_array($order->platform_id, $this->accessPlatforms))` 跳过；写手工销售收入 |
| 3 | `ManualFreight` | `if ($order->original_order_amount_cny == 0 || $order->shipping_revenue == 0)` 跳过 |
| 4 | `ManualDiscountedPrice` | 无额外 process 门禁；金额为 `-abs($discountAmount)` |
| 5 | `SystemVatDeductCustomerTax` | `if ($order->original_order_amount == 0)`、`if (empty($setting))`、`if ($tax == 0)` 跳过；已有费用：`if (!empty($itemStats) && !empty($itemStats->fee))` |
| 6 | `RefundAmount` | `if ($this->shouldSkipBySettlementRefundExpense($order))` 跳过；`return $refundExpense && $refundExpense[0]->mark != ExpenseIndexFieldEnum::REFUND_REVENUE;`；`if ($refundAmount == 0)` 不写当前退款 |
| 7 | `TikTokShippedSettlement` | `if ($orgOrder && $orgOrder['is_sample_order'])` 返回 false（不跳过），否则 true；随后仍要求 `shipping_status == BoolEnum::YES` |
| 8 | `MultiChannelSettlementExpense` | `if (empty($multiOrder))` 跳过；`if (!$logistic || !$logistic->overseas_wms_out_number)` 跳过；币种不同时换算 |
| 9 | `AmazonMultiChannelOrderV2Operator` | `return $payload->order->platform_id != PlatformEnum::AMAZON;`；TikTok=53，因此该节点直接跳过 |
| 10 | `TiktokEstimateHistoryFBAOperate` | `if ($payload->stage != ExpenseStageEnum::ESTIMATE || empty($payload->order->platform_order_type) || !in_array(PlatformOrderTypeEnum::TK_TIKTOK, $payload->order->platform_order_type))` 跳过 |
| 11 | `InitialPurchaseCost` | `if ($payload->hasOrderExclude)` 跳过；`if ($order->exchange_rate == 0)` 跳过当前 SKU |
| 12 | `TradeCommission` | `if ($order->order_amount == 0)` 跳过；`if (in_array($order->platform_id, [PlatformEnum::ALIEXPRESS, PlatformEnum::TIKTOK, PlatformEnum::LAZADA]) && $order->settlement_status && $order->build_order_type == BuildOrderTypeEnum::SYSTEM)` 跳过；MANUAL 不满足最后一项 |

## 写库节点（每个存储独立）

| 独立节点 | Model 真源 | 写入/清理调用 | 可确认字段或作用 |
|---|---|---|---|
| `order_expense_item_stats` | `backend/models/mongo/OrderExpenseItemStats.php` | `createOrUpdate`、`createOrUpdateShop`、`save`、`clearByOrderId` | `fee/cny_fee/type/financial_type/stage/source/mark/receipt_type/arithmetic/...`；算子明细 |
| `order_expense_shop_stats` | `backend/models/mongo/OrderExpenseShopStats.php` | `ExpenseShopTask::store()` → `save($shopStats)`；重建前 `clearShopFeeBy` | `OrderExpenseItemStatsService::save()` 在 `ascription == SHOP` 时同步联动；写 `stage/fee/cny_fee` 及费用项、SKU 属性 |
| `order_expense_sku_stats` | `backend/models/mongo/OrderExpenseSkuStats.php` | `updateByEntity`、`clearByOrderId` | 上卷 `profit/income/expenditure` 及各费用字段 |
| `order_expense_stats` | `backend/models/mongo/OrderExpenseStats.php` | `updateByEntity`、`clearByOrderId` | 订单维度 `profit/income/expenditure` 及各费用字段 |
| `order` | `backend/models/order/Order.php`，`tableName() = 'order'` | `updateByEntity` | 清理时归零；上卷写 `order_profit/income/order_profitability/total_order_profit/total_order_profit_rate/total_order_profit_time`；结算算子还可能写结算状态 |
| `order_item` | `backend/models/order/OrderItem.php`，`tableName() = 'order_item'` | `updateSkuIncome($order)` → `updateByEntity($item)` | 按 `sku-platform_sku-order_item_id` 匹配 `order_expense_sku_stats`，汇总后写 `$item->income` |

写入顺序不是一个事务：算子先写 `order_expense_item_stats`；事件上卷再写 SKU 聚合、订单聚合、`order`、`order_item`。源码未见覆盖整条主链的 DB/Mongo transaction。

## 上卷关键判断

| 代码条件 | 结果 |
|---|---|
| `if (Oms::$service->orderExpenseItemStats->isOnlyCalItemStat($order))` | 删除 only-item 缓存后不做上卷 |
| `if ($order->platform_id == PlatformEnum::AMAZON)` | Amazon 使用 TAG2；TikTok 走空 tag |
| `if (empty($data))` | 当前 SKU/订单聚合节点不写 |
| `if (empty($orderStat))` / `if (empty($skuStats))` | 初始化聚合文档，否则覆盖已有文档 |
| `if ($settlementTime && $orderSettlementAt->timestamp < $limitSettlementAt->timestamp)` | payment rollup 返回空算子数组 |
| `return $payload->order->settlement_status != BoolEnum::YES;` | `RealitySkuExpense` 与 `RealityOrderExpense` 在未结算时跳过 |
| `if ($orderIncome > 0)` | 才写 `order_profitability` |
| `if ($income != 0 && abs($income) != 0.000001)` | 才计算 `total_order_profit_rate` |
| `if ($order->order_status == OrderStatusEnum::CANCEL_VAL)` | 写完总利润后不触发利润检测 |
| `if ($allDetected && (empty($order->problem_cause_id) || !in_array(OrderQuestionEnum::PROFIT_LOW, $order->problem_cause_id)))` | 不重复触发利润检测 |

## 失败路径

1. Consumer 内任意 Strategy、算子或未被内部吞掉的 Listener 异常进入 `catch (\Throwable $throwable)`。
2. 记录 `stdoutErr`，调用 `Oms::$service->notify->immediatelyNotify(...)`。
3. `$retry = $data->retry ?? 1`，随后：
   `deliveryDelay($orderIds, 'OrderProfitCalculateConsumer', 'order_profit_task', random_int(120, 300), ++$retry)`。
4. `finally` 固定 `return ConsumerInterface::MSG_ACK`，原消息不会 NACK/requeue。

其它非异常失败/短路：

| 条件 | 行为 |
|---|---|
| `empty($body)` / `empty($data)` | 静默 ACK |
| Strategy 不存在且 `existsSubstitute()` 为 false | 当前订单静默跳过，最终 ACK |
| `can() === false` | 释放计算锁，不进入 Pipeline，最终 ACK |
| `AbstractOrderChain::payload()` 头程请求抛异常 | catch 后只输出错误，继续主链 |
| `if (!$lock->lock())`（`OrderExpenseItemStatsService::createOrUpdate`） | 当前费用项静默不写，算子链继续 |
| `AbstractOrderExpense::handle()` 上卷订单维度抛 `Throwable` | 内部 `Yii::error` 后吞掉异常，Pipeline 继续；Consumer 不会因此重试 |
| `OrderExpenseCalculateListener` 抛 `Exception` | 包装后重新抛出，进入 Consumer 重试分支 |

## 事务、锁与一致性

- `can() === false` 和 Pipeline 正常完成都会释放 `order:expense:calculate:{id}`。
- SKU 上卷锁：`sku_expense_rollup:{order_id}_{stage}`，2 秒。
- 订单上卷锁：`order_expense_rollup:{order_id}_{stage}`，2 秒。
- `before()` 默认触发 `OrderExpenseRestEvent($order, '利润计算', 1)`；Listener 按当前订单清三个 Mongo 聚合集合并归零 `order` 利润字段。
- 全链无统一事务；Consumer 又始终 ACK，因此应以“清理后重算 + 延迟补偿”理解其一致性，不是原子提交。

## 枚举值速查

- `PlatformEnum::TIKTOK = 53`
- `BuildOrderTypeEnum::SYSTEM = 0`，`MANUAL = 1`
- `BoolEnum::NO = 0`，`YES = 1`
- `ShippingTypeEnum::FBA = 1`，`FBM = 2`
- `OrderStatusEnum::CANCEL_VAL = 9`
- `ExpenseItemReceiptTypeEnum::ORIGINAL/SPLIT/MERGE/REISSUE/RESEND/REFUND/OTHER = 1/2/3/4/5/6/7`
- `ExpenseStageEnum::ESTIMATE = 1`，`REALITY = 2`
- `PlatformOrderStatusEnum::TIKTOK_CANCELLED = 54`

## 需人工确认

- `OrderExpenseItemStatsService::createOrUpdate*` 的唯一键冲突策略未在本页继续下钻；本页只确认调用点与 Model 字段。
