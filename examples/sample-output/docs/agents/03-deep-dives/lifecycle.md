# 生命周期（功能回写）

只展开已深挖功能上出现的 Status/State。其它枚举只索引。

## 本轮 deep：tiktok-profit-calculate

### 计算准入（非独立 *Status* 枚举，但是硬门禁）

文件：`backend/modules/v1/task/expense/segment/chain/AbstractAllOperatorChain.php` `can()`  
用在：`TiktokAllOperatorChain`（及同基类其它平台 All 链）。

| 条件 | 行为 |
|---|---|
| 无 `original_order_currency` / 无 `exchange_rate` | 不计算 |
| `pay_status != YES` | 不计算 |
| 无 orderItem 或 sku 未绑定 | 不计算 |
| MANUAL 且 `order_status == CANCEL` | Rest 清费用后不计算 |
| 拆合单全部 CANCEL | Rest 后不计算 |
| 非原单且 CANCEL | Rest 后不计算 |
| FBA 无 `warehouse_id`；FBM 无仓/运方式（原单例外） | Rest 后不计算 |

谁改：本功能 **不改** 这些前置字段；只读判断。清费用走 `OrderExpenseRestEvent`。

### BuildOrderTypeEnum（建单类型）

文件：`backend/modules/v1/enums/order/BuildOrderTypeEnum.php`

| 值 | 含义 | 本功能走向 |
|---:|---|---|
| 0 | SYSTEM 系统建单 | TikTok 专用算子前缀 + 共用段 |
| 1 | MANUAL 手动建单 | Manual* 前缀 + 共用段 |

### SettlementStatusEnum（订单结算态，Bool）

文件：`backend/modules/v1/enums/order/SettlementStatusEnum.php`（继承 BoolEnum）  
本功能算子可读 `order.settlement_status`；**主利润链是否写该字段：待确认**（结算上游更可能写）。

---

## 既有 deep：amazon-expense-calculate（摘要）

### AmazonTransactionStatusEnum

| 值 | 本功能用法 |
|---|---|
| `RELEASED` | 仅此态进入结算计算 |
| `DEFERRED_RELEASED` | 常量；计算侧未见单独分支 |

详见上一轮深页。订单 `settlement_status` / `withdrawal_status` 可在 Amazon 结算路径被改。

## 其它 Status（仅索引）

| 文件 | 备注 |
|---|---|
| `ExpenseCustomizeImportStatusEnum` | 自定义导入审核 |
| `ExpenseCustomizeImportExecStatusEnum` | 文件导入执行态 |
| `ExpenseCloseStatusEnum` | 关账 |
| `ExpenseProcessTypeEnum::SKIP` | field_map 跳过 |
| `OrderStatusEnum` | can() 用到 CANCEL |
| `BoolEnum` pay_status | can() 用到 |
