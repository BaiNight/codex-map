# 对外契约（功能回写）

只展开已深挖功能上真实出现的调用。

## 本轮 deep：tiktok-profit-calculate

### 对内 MQ

| 方向 | 队列 | 字段 | 路径 |
|---|---|---|---|
| 入 | `order_expense_calculate`（routing `order_profit`）或 `order_profit_task` | `orderIds`；可选 `source`/`retry` | `OrderProfitCalculateConsumer` |
| 出（失败重试） | 延迟 `order_profit_task` | orderIds + retry++ | Consumer catch |
| 出（上游结算） | 延迟 `order_profit_task` | order id | `TiktokSettlementDropped::profitCal` |
| 出（上游付款） | 延迟 `order_profit` | order id | `OrderProfitCalculateListener` |

### 锁 / 通知

| 项 | 内容 |
|---|---|
| RedLock | `order:expense:calculate:{orderId}`，投递时 120s；抢不到则丢弃重复消息 |
| notify | 利润计算 catch → `immediatelyNotify` |

### launcher

本功能 **计算主链无 launcher**。读 mongo `tiktok_order` / `tiktok_statement_transactions` 为本仓库存储。

相关未深挖：结算拉取 `TiktokStatementsPull` 调平台 API（路径待打开）并可能 `pushToFeeCalcQueue(TIKTOK)`。

---

## 既有 deep：amazon-expense-calculate（摘要）

| 契约 | 要点 |
|---|---|
| MQ | `amazon_transaction_resolve` → `order_platform_settlement`（type=`2v2`） |
| xc_tc | 移仓 / 头程异常单 |
| xc_wms | fnsku → seller_sku |
| Redis | `OrderSettlementCache` 延迟上卷 |

## 其它 client 目录（仅索引）

| 路径 | 备注 |
|---|---|
| `common/launcher/request/` | goods / warehouse / xc_wms / xc_tms / 各平台 … |
| `backend/modules/v1/service/external/` | Wms/Tms/Goods/Basic/… |
| `common/config/external/ExternalService.php` | 组件注册 |
