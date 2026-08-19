# OMS 业务域索引

探测自 `backend/modules/v1/controllers` 分组 + `backend/modules/v1/routes`。
HTTP 详情见 [../02-surfaces/api-list.md](../02-surfaces/api-list.md)，CLI/Consumer 见 [../02-surfaces/cli-and-consumers.md](../02-surfaces/cli-and-consumers.md)。
数据模型见 [../03-deep-dives/data-model.md](../03-deep-dives/data-model.md)，ER 见 [../diagrams/data-model-er.svg](../diagrams/data-model-er.svg)。
expense 候选与已深挖见 [../03-deep-dives/flows/expense.md](../03-deep-dives/flows/expense.md)；TikTok 利润 [../03-deep-dives/flows/expense-tiktok-profit-calculate.md](../03-deep-dives/flows/expense-tiktok-profit-calculate.md)；Amazon 费用 [../03-deep-dives/flows/expense-amazon-expense-calculate.md](../03-deep-dives/flows/expense-amazon-expense-calculate.md)。

| 域 | 职责 | 入口 |
|---|---|---|
| order（订单） | 拉单后的系统订单、拆合、标发、批量、首页统计、Amazon PO | `backend/modules/v1/controllers/order/` · `routes/order/route.php` |
| expense（费用） | 自定义费用、费用中心、结算/预估、关账、Amazon 发票 | `backend/modules/v1/controllers/expense/` · `routes/expense/route.php` |
| refund（退款） | 退款单与退款原因 | `backend/modules/v1/controllers/refund/` · `routes/refund/route.php` |
| exchange（换货） | 换货单列表/审核 | `backend/modules/v1/controllers/exchange/` · `routes/exchange/route.php` |
| export（导出） | 利润/SKU/退款等异步导出，多走 internal | `backend/modules/v1/controllers/export/` · `routes/export/route.php` |
| report（报表） | 利润趋势、业绩达成、物流统计 | `backend/modules/v1/controllers/report/` · `routes/report/route.php` |
| internal（内部回调） | 供 WMS/TMS/客服等调用的 internal/* | `backend/modules/v1/controllers/internal/` · `routes/internal/route.php` |
| subscribe（订阅） | 订单状态订阅 | `backend/modules/v1/controllers/subscribe/` · `routes/subscribe/route.php` |
| saihe（赛盒） | 赛盒订单/退款（路由写在 order/route.php） | `backend/modules/v1/controllers/saihe/` |
| base（基础配置） | 策略、拆单、物流优选、KPI、黑名单、渠道、VAT | `backend/modules/v1/controllers/base/` · `routes/base/route.php` |
| log（业务日志） | GET /log/business | `backend/modules/v1/controllers/BusinessLoggerController.php` · `routes/log/route.php` |

## 缺口

- `OrderNotifyController`（`controllers/order`）无路由条目，`actionShip` 空实现。
- `OrderSkuSalesStatisticsController`（`controllers/report`）无路由条目。
- `backend/modules/v2` 仅 Module 类，无 HTTP 入口。
