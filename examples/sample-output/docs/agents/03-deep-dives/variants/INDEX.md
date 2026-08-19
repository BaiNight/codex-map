# 分叉目录索引

页标题用目录名。已深挖功能标「deep」。

## 本轮 deep（tiktok-profit-calculate）

| 目录 / 分发类 | 一句话 | 命中功能 |
|---|---|---|
| `backend/modules/v1/strategy/profit/` | 按 `platform_id` 选订单利润 Strategy | `tiktok-profit-calculate` |
| `OrderProfitCalculateStrategyProvider` | 含 `TiktokOrderProfitCalculateStrategy`；substitute=Other | 同上 |
| `backend/modules/v1/task/expense/segment/chain/` | `TiktokAllOperatorChain`（MANUAL/SYSTEM + 共用算子） | 同上 |
| `ExpenseSegmentManager` | can/before/Pipeline/profit → ChangeEvent | 同上 |
| `backend/modules/v1/task/expense/segment/components/tiktok/` | TikTok 专用收入/税/运费/退款等算子 | 同上 |

上游相关（未单独 deep）：`strategy/expense/settlement/TiktokSettlementExpenseStrategy` + `TiktokSettlementDropped` 写结算费后投利润队列。

## 既有 deep（amazon-expense-calculate）

| 目录 / 分发类 | 一句话 |
|---|---|
| `strategy/expense/settlement/` + `AmazonTransactionExpenseStrategy` | type=`2v2` |
| `strategy/expense/amazon/transaction/` | 店铺 BoolStrategy×6 |
| `AmazonTransactionItemHandlerOperatorChain` | → `AmazonTransactionDropped` |

## 命中但未深写（只列目录）

| 目录 | 一句话 |
|---|---|
| `strategy/expense/initialize/` | 含 `TiktokExpenseInitializeStrategy` |
| `strategy/expense/delivery/` | 含 `TiktokAdvertDeliveryStrategy` |
| `strategy/expense/center/` | 费用中心 |
| `strategy/expense/rollup/` | 店铺上卷 |
| `strategy/expense/amazon/finance/` | Amazon finance |
| `strategy/expense/temu/` | Temu |
| `strategy/expense/refund/` | 退货 |
| `adapter/order/` | 拉单适配 |

不穷尽各类下全部 Strategy。
