# OMS HTTP 入口清单

真源：`backend/modules/v1/routes/**/*.php`（由 `backend/config/route.php` 汇总进 `urlManager`）。

## 约定

- 路径即路由键，**没有** URL 前缀 `/v1`（`v1` 只在内部 route target）。
- `urlManager.enablePrettyUrl=true`，`showScriptName=false`。
- 面：路径含 `internal` 标「内部」，其余标「对外」。
- 通用入参（`backend/controllers/BaseController.php`）：`pageNum`、`pageSize`、`search_after`；Header `X-Tenant-Id` / `X-User-Id`。
- 通用返回：`{status, code, message, data, total, timestamp, version}`。
- 主入参/返回从对应 action 推断；推不出标「待确认」，**未编造字段**。

合计 **301** 条显式路由。

## order

| 面 | 方法 | 路径 | 一句话 | 主入参 | 返回要点 | 控制器 |
|---|---|---|---|---|---|---|
| 对外 | `POST` | `/order/create` | 添加订单 | order, payment, products, recipient, sender | data=业务对象 | `backend/modules/v1/controllers/order/OrderController.php` |
| 对外 | `POST` | `/order/update` | 修改订单 | order | data=业务对象 | `backend/modules/v1/controllers/order/OrderController.php` |
| 对外 | `GET` | `/order/detail` | 订单详情 | id, order_sn, platform_order_sn, shop_id | data=业务对象 | `backend/modules/v1/controllers/order/OrderController.php` |
| 对外 | `POST` | `/order/list` | 订单列表 | 待确认 | 分页：data + total | `backend/modules/v1/controllers/order/OrderController.php` |
| 对外 | `POST` | `/order/associateSku` | 关联sku | 待确认 | data=业务对象 | `backend/modules/v1/controllers/order/OrderController.php` |
| 对外 | `GET` | `/order/invoice` | 获取订单发票 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/order/OrderController.php` |
| 对外 | `GET` | `/order/createDefault` | 新增订单时给前端默认填入的数据 | new_order_reason | data=业务对象 | `backend/modules/v1/controllers/order/OrderController.php` |
| 内部 | `POST` | `/internal/order/batchQueryShippingTime` | 批量查询发货时间 | order_sn | data=业务对象 | `backend/modules/v1/controllers/order/OrderController.php` |
| 对外 | `POST` | `/order/shopSaleList` | 订单店铺销售数据列表 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/order/OrderController.php` |
| 对外 | `POST` | `/order/updateRecipient` | 只修改订单收件相关信息 | recipient | data=业务对象 | `backend/modules/v1/controllers/order/OrderController.php` |
| 对外 | `GET` | `/order/attachment-tpl` | 订单附件模板列表 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/order/OrderController.php` |
| 对外 | `POST` | `/order/attachment` | 设置附件 | 请求体/查询串（字段待确认） | data=业务对象 | `backend/modules/v1/controllers/order/OrderController.php` |
| 对外 | `GET` | `/order/expense/order` | 获取订单关联费用 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/order/OrderExpenseController.php` |
| 对外 | `GET` | `/order/expense/sku` | sku 维度利润 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/order/OrderExpenseController.php` |
| 对外 | `POST` | `/order/expense/againCalculate` | 重新计算利润 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/order/OrderExpenseController.php` |
| 对外 | `POST` | `/order/expense/updatePurchasePrice` | 更新采购价 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/order/OrderExpenseController.php` |
| 对外 | `POST` | `/order/canMerged/index` | 获取可合并订单列表 | 待确认 | 分页：data + total | `backend/modules/v1/controllers/order/OrderCanMergedController.php` |
| 对外 | `POST` | `/order/canMerged/merge` | 合并订单 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/order/OrderCanMergedController.php` |
| 对外 | `POST` | `/order/canMerged/notMerged` | 不合并 | ids | data=业务对象 | `backend/modules/v1/controllers/order/OrderCanMergedController.php` |
| 对外 | `POST` | `/order/merged/index` | 已合并列表 | 待确认 | 分页：data + total | `backend/modules/v1/controllers/order/OrderMergedController.php` |
| 对外 | `POST` | `/order/merged/cancel` | 取消合并 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/order/OrderMergedController.php` |
| 对外 | `POST` | `/order/batch/cancel` | 批量取消 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/order/OrderBatchController.php` |
| 对外 | `POST` | `/order/batch/split` | 拆分订单 | 请求体/查询串（字段待确认） | data=业务对象 | `backend/modules/v1/controllers/order/OrderBatchController.php` |
| 对外 | `POST` | `/order/batch/init` | 初始化订单 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/order/OrderBatchController.php` |
| 对外 | `POST` | `/order/batch/replaceSku` | 批量替换sku | 请求体/查询串（字段待确认） | data=业务对象 | `backend/modules/v1/controllers/order/OrderBatchController.php` |
| 对外 | `POST` | `/order/batch/updateOrderRemark` | 批量修改订单备注 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/order/OrderBatchController.php` |
| 对外 | `POST` | `/order/batch/uploadTraceNumber` | 批量上传追踪号 | tracing_number | data=业务对象 | `backend/modules/v1/controllers/order/OrderUploadTraceNumberController.php` |
| 对外 | `POST` | `/order/batch/translate-recipient-info` | 翻译收件人信息为英文 | recipient_info | data=业务对象 | `backend/modules/v1/controllers/order/OrderBatchController.php` |
| 对外 | `GET` | `/order/uploadTraceNumber/history` | 待确认 | pageSize | data=业务对象 | `backend/modules/v1/controllers/order/OrderUploadTraceNumberController.php` |
| 对外 | `POST` | `/order/batch/operation` | 执行批量操作 | 请求体/查询串（字段待确认） | data=业务对象 | `backend/modules/v1/controllers/order/BatchOperationController.php` |
| 对外 | `GET` | `/order/batch/results` | 获取批量操作结果 | batch_number | data=业务对象 | `backend/modules/v1/controllers/order/BatchOperationController.php` |
| 对外 | `POST` | `/order/batch/overseaDistributeRecord` | 获取海外仓配货报文 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/order/OrderBatchController.php` |
| 对外 | `POST` | `/order/batch/downloadAttachment` | 批量下载订单附件 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/order/OrderBatchController.php` |
| 对外 | `POST` | `/homepage/stat` | 订单统计，包含订单量、销售额 | 请求体/查询串（字段待确认） | data=业务对象 | `backend/modules/v1/controllers/order/OrderHomepageController.php` |
| 对外 | `GET` | `/homepage/to-be-delivered` | 未发货订单统计 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/order/OrderHomepageController.php` |
| 对外 | `GET` | `/homepage/to-be-handle` | 待处理订单统计 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/order/OrderHomepageController.php` |
| 对外 | `GET` | `/homepage/goods-ranking` | 商品排行 | cycle | data=业务对象 | `backend/modules/v1/controllers/order/OrderHomepageController.php` |
| 对外 | `GET` | `/homepage/shop-ranking` | 获取店铺排名 | cycle | data=业务对象 | `backend/modules/v1/controllers/order/OrderHomepageController.php` |
| 对外 | `GET` | `/homepage/sales-ranking` | 销售排行 | cycle | data=业务对象 | `backend/modules/v1/controllers/order/OrderHomepageController.php` |
| 对外 | `POST` | `/homepage/platform-sales-details` | 平台销售明细 | 请求体/查询串（字段待确认） | data=业务对象 | `backend/modules/v1/controllers/order/OrderHomepageController.php` |
| 对外 | `POST` | `/homepage/site-sales-details` | 国家销售明细 | 请求体/查询串（字段待确认） | data=业务对象 | `backend/modules/v1/controllers/order/OrderHomepageController.php` |
| 对外 | `POST` | `/homepage/dept-sales-details` | 事业部销售明细 | 请求体/查询串（字段待确认） | data=业务对象 | `backend/modules/v1/controllers/order/OrderHomepageController.php` |
| 对外 | `POST` | `/order/batch/import` | 导入订单 | oss_url | data=业务对象 | `backend/modules/v1/controllers/order/OrderBatchImportController.php` |
| 对外 | `GET` | `/order/wait-statistic` | 订单待办统计 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/order/OrderController.php` |
| 对外 | `GET` | `/upload/out-number` | 当前人导入的海外仓出库单号列表 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/order/OrderUploadController.php` |
| 对外 | `POST` | `/upload/out-number` | 批量上传海外仓出库单号 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/order/OrderUploadController.php` |
| 对外 | `POST` | `/order/multi-channel/ship` | 订单多渠道发货 | 请求体/查询串（字段待确认） | data=业务对象 | `backend/modules/v1/controllers/order/OrderMultiChannelController.php` |
| 对外 | `POST` | `/order/multi-channel/batch-ship` | 批量多渠道发货 | 请求体/查询串（字段待确认） | data=业务对象 | `backend/modules/v1/controllers/order/OrderMultiChannelController.php` |
| 对外 | `POST` | `/amazon-po/list` | 采购订单列表 | 请求体/查询串（字段待确认） | 分页：data + total | `backend/modules/v1/controllers/order/AmazonPurchaseOrderController.php` |
| 对外 | `GET` | `/amazon-po/detail` | 采购订单详情 | 请求体/查询串（字段待确认） | data=业务对象 | `backend/modules/v1/controllers/order/AmazonPurchaseOrderController.php` |
| 对外 | `POST` | `/amazon-po/bind-sku` | 绑定SKU | 请求体/查询串（字段待确认） | data=业务对象 | `backend/modules/v1/controllers/order/AmazonPurchaseOrderController.php` |
| 对外 | `POST` | `/amazon-po/modify-warehouse` | 修改发货仓库 | 请求体/查询串（字段待确认） | data=业务对象 | `backend/modules/v1/controllers/order/AmazonPurchaseOrderController.php` |

## saihe

| 面 | 方法 | 路径 | 一句话 | 主入参 | 返回要点 | 控制器 |
|---|---|---|---|---|---|---|
| 对外 | `POST` | `/saihe/order/list` | 赛盒订单列表 | 请求体/查询串（字段待确认） | 分页：data + total | `backend/modules/v1/controllers/saihe/SaiheOrderController.php` |
| 对外 | `GET` | `/saihe/order/detail` | 赛盒订单详情 | 请求体/查询串（字段待确认） | data=业务对象 | `backend/modules/v1/controllers/saihe/SaiheOrderController.php` |
| 对外 | `GET` | `/saihe/order/log` | 日志 | order_code | data=业务对象 | `backend/modules/v1/controllers/saihe/SaiheOrderController.php` |
| 对外 | `POST` | `/saihe/apply-refund` | 申请赛盒退款单 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/saihe/SaiheOrderController.php` |
| 对外 | `POST` | `/saihe/audit-refund` | 审核退款单 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/saihe/SaiheOrderController.php` |
| 对外 | `POST` | `/saihe/refund/list` | 赛盒退款单列表 | 待确认 | 分页：data + total | `backend/modules/v1/controllers/saihe/SaiheOrderRefundController.php` |

## expense

| 面 | 方法 | 路径 | 一句话 | 主入参 | 返回要点 | 控制器 |
|---|---|---|---|---|---|---|
| 对外 | `POST` | `/expense/custom/import` | 导入自定义费用 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/expense/CustomExpenseController.php` |
| 对外 | `POST` | `/expense/custom/index` | 列表 | 待确认 | 分页：data + total | `backend/modules/v1/controllers/expense/CustomExpenseController.php` |
| 对外 | `POST` | `/expense/custom/review` | 审核 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/expense/CustomExpenseController.php` |
| 对外 | `GET` | `/expense/custom/download-template` | 获取下载模板 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/expense/CustomExpenseController.php` |
| 对外 | `GET` | `/expense/custom/download-expense-file` | 获取下载模板 | id | data=业务对象 | `backend/modules/v1/controllers/expense/CustomExpenseController.php` |
| 对外 | `GET` | `/expense/product-dev-commission-setting/list` | 列表 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/expense/ProductDevCommissionSettingController.php` |
| 对外 | `POST` | `/expense/product-dev-commission-setting/store` | 添加 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/expense/ProductDevCommissionSettingController.php` |
| 对外 | `POST` | `/expense/product-dev-commission-setting/update` | 修改 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/expense/ProductDevCommissionSettingController.php` |
| 对外 | `POST` | `/expense/product-dev-commission-setting/delete` | 删除 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/expense/ProductDevCommissionSettingController.php` |
| 对外 | `POST` | `/expense/product-dev-commission-setting/export` | 导出 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/expense/ProductDevCommissionSettingController.php` |
| 对外 | `POST` | `/expense/product-dev-commission-setting/switchState` | 切换状态 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/expense/ProductDevCommissionSettingController.php` |
| 对外 | `POST` | `/expense/custom-expense/expense-import` | 费用导入 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/expense/CustomExpenseController.php` |
| 对外 | `GET` | `/expense/custom-expense/expense-import-detail/<id>` | 费用导入详情 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/expense/CustomExpenseController.php` |
| 对外 | `GET` | `/expense/custom-expense/expense-item` | 获取费用项 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/expense/CustomExpenseController.php` |
| 对外 | `GET` | `/expense/custom-expense/download-frame-template` | 获取下载模板 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/expense/CustomExpenseController.php` |
| 对外 | `POST` | `/expense/custom-expense/financial-close-date-update` | 关账时间更新 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/expense/CustomExpenseController.php` |
| 对外 | `POST` | `/expense/custom-expense/import-report` | 导入报表 | 待确认 | 分页：data + total | `backend/modules/v1/controllers/expense/CustomExpenseController.php` |
| 对外 | `POST` | `/expense/custom-expense/import-report-amount` | 报表总金额 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/expense/CustomExpenseController.php` |
| 对外 | `POST` | `/expense/expense-center/store` | 待确认 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/expense/ExpenseCenterController.php` |
| 对外 | `POST` | `/expense/expense-center/update` | 待确认 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/expense/ExpenseCenterController.php` |
| 对外 | `POST` | `/expense/expense-center/delete` | 待确认 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/expense/ExpenseCenterController.php` |
| 对外 | `POST` | `/expense/expense-center/list` | 列表 | 待确认 | 分页：data + total | `backend/modules/v1/controllers/expense/ExpenseCenterController.php` |
| 对外 | `GET` | `/expense/expense-center/detail/<id>` | 待确认 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/expense/ExpenseCenterController.php` |
| 对外 | `GET` | `/expense/expense-center/strategy-detail/<id>` | 策略详情 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/expense/ExpenseCenterController.php` |
| 对外 | `POST` | `/expense/expense-center/switch-state` | 切换状态 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/expense/ExpenseCenterController.php` |
| 对外 | `POST` | `/expense/expense-center/executed-expense-details` | 执行费用明细列表 | 待确认 | 分页：data + total | `backend/modules/v1/controllers/expense/ExpenseCenterController.php` |
| 对外 | `GET` | `/expense/expense-center/expense-types` | 费用项 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/expense/ExpenseCenterController.php` |
| 对外 | `POST` | `/expense/expense-order-profit-detail/settlement-import-store` | 导入 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/expense/ExpenseOrderProfitDetailController.php` |
| 对外 | `POST` | `/expense/expense-order-profit-detail/settlement-import-update` | 修改 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/expense/ExpenseOrderProfitDetailController.php` |
| 对外 | `POST` | `/expense/expense-order-profit-detail/import-record` | 结算费用导入 | 待确认 | 分页：data + total | `backend/modules/v1/controllers/expense/ExpenseOrderProfitDetailController.php` |
| 对外 | `POST` | `/expense/expense-order-profit-detail/import-review` | 审核 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/expense/ExpenseOrderProfitDetailController.php` |
| 对外 | `GET` | `/expense/expense-close/list` | 关账列表 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/expense/ExpenseCloseController.php` |
| 对外 | `POST` | `/expense/expense-close/update-status` | 修改关账状态 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/expense/ExpenseCloseController.php` |
| 对外 | `POST` | `/expense/expense-order-profit-detail/import-condition` | 导入前置条件 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/expense/ExpenseOrderProfitDetailController.php` |
| 对外 | `GET` | `/expense/expense-order-profit-detail/import-items` | 导入条件 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/expense/ExpenseOrderProfitDetailController.php` |
| 对外 | `GET` | `/expense/expense-order-profit-detail/import-template` | 导入模板 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/expense/ExpenseOrderProfitDetailController.php` |
| 对外 | `POST` | `/expense/expense-order-profit-detail/settlements` | 结算详情列表 | 待确认 | 分页：data + total | `backend/modules/v1/controllers/expense/ExpenseOrderProfitDetailController.php` |
| 对外 | `POST` | `/expense/expense-order-profit-detail/settlements-export` | 结算详情导出 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/expense/ExpenseOrderProfitDetailController.php` |
| 对外 | `POST` | `/expense/expense-order-profit-detail/estimates` | 预估详情列表 | 待确认 | 分页：data + total | `backend/modules/v1/controllers/expense/ExpenseOrderProfitDetailController.php` |
| 对外 | `POST` | `/expense/expense-order-profit-detail/estimates-export` | 预估详情导出 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/expense/ExpenseOrderProfitDetailController.php` |
| 对外 | `POST` | `/expense/expense-center/executed-expense-details-export` | 执行费用明细导出 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/expense/ExpenseCenterController.php` |
| 对外 | `POST` | `/expense/amazon-invoice/index` | 发票列表 | 待确认 | 分页：data + total | `backend/modules/v1/controllers/expense/AmazonInvoiceController.php` |
| 内部 | `POST` | `/internal/expense/amazon-invoice/export` | 发票列表导出 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/expense/AmazonInvoiceController.php` |

## refund

| 面 | 方法 | 路径 | 一句话 | 主入参 | 返回要点 | 控制器 |
|---|---|---|---|---|---|---|
| 对外 | `GET` | `/order/refund/reason/list` | 获取退款原因管理 | 待确认 | 分页：data + total | `backend/modules/v1/controllers/refund/OrderRefundReasonController.php` |
| 对外 | `POST` | `/order/refund/reason/create` | 新增退款原因 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/refund/OrderRefundReasonController.php` |
| 对外 | `POST` | `/order/refund/reason/switchStatus` | 切换状态 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/refund/OrderRefundReasonController.php` |
| 对外 | `POST` | `/order/refund/reason/update` | 修改退款原因 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/refund/OrderRefundReasonController.php` |
| 对外 | `POST` | `/order/refund/reason/destroy` | 删除退款原因 | id | data=业务对象 | `backend/modules/v1/controllers/refund/OrderRefundReasonController.php` |
| 对外 | `GET` | `/order/refund/reason/options` | 获取选项 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/refund/OrderRefundReasonController.php` |
| 对外 | `POST` | `/order/refund/reason/predict` | 退款原因预测 | order_sn, refund_reason | data=业务对象 | `backend/modules/v1/controllers/refund/OrderRefundReasonController.php` |
| 对外 | `POST` | `/order/refund/list` | 订单退款列表 | 待确认 | 分页：data + total | `backend/modules/v1/controllers/refund/OrderRefundController.php` |
| 对外 | `GET` | `/order/refund/detail` | 订单退款单详情 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/refund/OrderRefundController.php` |
| 对外 | `POST` | `/order/refund/create` | 订单新增退款 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/refund/OrderRefundController.php` |
| 对外 | `POST` | `/order/refund/update` | 订单修改退款 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/refund/OrderRefundController.php` |
| 对外 | `POST` | `/order/refund/destroy` | 待确认 | 待确认 | 待确认 | `backend/modules/v1/controllers/refund/OrderRefundController.php` |
| 对外 | `POST` | `/order/refund/review` | 审核 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/refund/OrderRefundController.php` |
| 对外 | `POST` | `/order/refund/reject` | 退款驳回 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/refund/OrderRefundController.php` |
| 对外 | `GET` | `/order/refund/overflow` | 订单退款信息概览 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/refund/OrderRefundController.php` |
| 对外 | `POST` | `/order/refund/request` | Api退款请求 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/refund/OrderRefundController.php` |
| 对外 | `POST` | `/order/refund/saveAttachment` | 保存退款附件 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/refund/OrderRefundController.php` |
| 对外 | `POST` | `/order/refund/import` | 退款单导入前检查预览 | oss_url | data=业务对象 | `backend/modules/v1/controllers/refund/OrderRefundImportController.php` |
| 对外 | `POST` | `/order/refund/importPreview` | 退款单导入前检查预览 | oss_url | data=业务对象 | `backend/modules/v1/controllers/refund/OrderRefundImportController.php` |
| 对外 | `POST` | `/order/refund/edit-remark` | 编辑退款备注 | 请求体/查询串（字段待确认） | data=业务对象 | `backend/modules/v1/controllers/refund/OrderRefundController.php` |
| 对外 | `POST` | `/order/refund/edit-desc` | 编辑退款描述 | 请求体/查询串（字段待确认） | data=业务对象 | `backend/modules/v1/controllers/refund/OrderRefundController.php` |

## exchange

| 面 | 方法 | 路径 | 一句话 | 主入参 | 返回要点 | 控制器 |
|---|---|---|---|---|---|---|
| 对外 | `POST` | `/order/exchange/list` | 首页 | 待确认 | 分页：data + total | `backend/modules/v1/controllers/exchange/OrderExchangeController.php` |
| 对外 | `GET` | `/order/exchange/detail` | 详情 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/exchange/OrderExchangeController.php` |
| 对外 | `POST` | `/order/exchange/create` | 订单退换货新增 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/exchange/OrderExchangeController.php` |
| 对外 | `POST` | `/order/exchange/update` | 订单退换货修改 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/exchange/OrderExchangeController.php` |
| 对外 | `POST` | `/order/exchange/review` | 审核 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/exchange/OrderExchangeController.php` |
| 对外 | `POST` | `/order/exchange/statusStats` | 状态统计 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/exchange/OrderExchangeController.php` |

## export

| 面 | 方法 | 路径 | 一句话 | 主入参 | 返回要点 | 控制器 |
|---|---|---|---|---|---|---|
| 内部 | `POST` | `/internal/report/profit/detail` | 订单维度费用导出 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/export/OrderProfitDetailController.php` |
| 对外 | `POST` | `/report/profit/detail` | 订单维度费用导出 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/export/OrderProfitDetailController.php` |
| 内部 | `POST` | `/internal/report/profit/sku` | sku维度费用导出 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/export/OrderProfitSkuController.php` |
| 内部 | `POST` | `/internal/report/profit/snapshot-performance/export` | 导出 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/export/OrderProfitSnapshotPerformanceController.php` |
| 内部 | `POST` | `/internal/report/order/shop/expenses` | sku维度费用导出 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/export/OrderShopExpenseController.php` |
| 对外 | `POST` | `/report/order/shop/expenses` | sku维度费用导出 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/export/OrderShopExpenseController.php` |
| 内部 | `POST` | `/internal/report/order/sku` | 订单sku导出 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/export/OrderSkuController.php` |
| 内部 | `POST` | `/internal/report/order/sku/flatten` | 订单sku平铺导出 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/export/OrderSkuController.php` |
| 内部 | `POST` | `/internal/order/result/export` | 订单导入结果导出 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/export/OrderImportResultController.php` |
| 内部 | `POST` | `/internal/order/refund/sku/export` | 订单退款sku维度导出 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/export/OrderRefundController.php` |
| 内部 | `POST` | `/internal/order/refund/export` | 订单退款订单维度导出 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/export/OrderRefundController.php` |
| 内部 | `POST` | `/internal/expense/product-dev-commission-result/export` | 导出回调 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/export/ProductDevCommissionResultController.php` |
| 内部 | `POST` | `/internal/order/refundResult/export` | 订单退款导入结果导出 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/export/OrderRefundImportResultController.php` |
| 内部 | `POST` | `/internal/export/order/distribution-sku` | 配货sku导出 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/export/OrderSkuController.php` |
| 内部 | `POST` | `/internal/export/order-return-goods/sku` | sku维度导出 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/export/OrderReturnGoodsController.php` |
| 内部 | `POST` | `/internal/export/order-return-goods/order` | 订单维度导出 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/export/OrderReturnGoodsController.php` |
| 内部 | `POST` | `/internal/expense/expense-customize/import-report` | 自定义费用报表导出 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/export/ExpenseCustomizeReportController.php` |
| 内部 | `POST` | `/internal/export/order-source-channel/export` | 订单来源渠道导出 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/export/OrderSourceChannelController.php` |

## report

| 面 | 方法 | 路径 | 一句话 | 主入参 | 返回要点 | 控制器 |
|---|---|---|---|---|---|---|
| 对外 | `POST` | `/report/profit/sales/trend` | 订单销售趋势 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/report/OrderProfitReportController.php` |
| 对外 | `POST` | `/report/profit/index/overflow` | 订单费用指标概览 | 请求体/查询串（字段待确认） | data=业务对象 | `backend/modules/v1/controllers/report/OrderProfitReportController.php` |
| 对外 | `POST` | `/report/profit/composite` | 获取综合指标 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/report/OrderProfitReportController.php` |
| 对外 | `GET` | `/report/profit/snapshot-versions` | 获取已存在的费用快照版本 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/report/OrderProfitReportController.php` |
| 对外 | `POST` | `/report/profit/snapshot-performance` | 订单利润快照业绩负责人报表 | 请求体/查询串（字段待确认） | 分页：data + total | `backend/modules/v1/controllers/report/OrderProfitReportController.php` |
| 对外 | `GET` | `/report/performance/option` | 业绩报表下拉 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/report/PerformanceReportController.php` |
| 对外 | `POST` | `/report/performance/index` | 业绩报表 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/report/PerformanceReportController.php` |
| 内部 | `GET` | `/internal/report/logistics/channelStatistics` | 物流渠道统计 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/report/TmsLogisticsReportController.php` |

## internal

| 面 | 方法 | 路径 | 一句话 | 主入参 | 返回要点 | 控制器 |
|---|---|---|---|---|---|---|
| 内部 | `GET` | `/internal/shop-shipping-address` | 店铺发货地址 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/internal/ShopAddressController.php` |
| 内部 | `POST` | `/internal/shop-shipping-addresses` | 批量获取店铺地址 | address_type, shop_ids | data=业务对象 | `backend/modules/v1/controllers/internal/ShopAddressController.php` |
| 内部 | `GET` | `/internal/order/recipient` | 获取订单收件人信息 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/internal/OrderController.php` |
| 内部 | `POST` | `/internal/order/recipients` | 批量获取订单收件人信息 | order_sns | data=业务对象 | `backend/modules/v1/controllers/internal/OrderController.php` |
| 内部 | `GET` | `/internal/order/detail` | 内部获取订单详情 | with_related_order_sn | data=业务对象 | `backend/modules/v1/controllers/order/OrderController.php` |
| 内部 | `POST` | `/internal/order/list` | 提供给外部订单列表 | 请求体/查询串（字段待确认） | 分页：data + total | `backend/modules/v1/controllers/internal/OrderController.php` |
| 内部 | `POST` | `/internal/order/easy-list` | 提供给外部订单简单列表 | order_sn | data=业务对象 | `backend/modules/v1/controllers/internal/OrderController.php` |
| 内部 | `POST` | `/internal/subscribe/order-info` | 订阅其他系统推送的订单信息 | 请求体/查询串（字段待确认） | data=业务对象 | `backend/modules/v1/controllers/internal/SubscribeController.php` |
| 内部 | `POST` | `/internal/subscribe/customer-system` | 订阅客服系统数据变更 | 请求体/查询串（字段待确认） | data=业务对象 | `backend/modules/v1/controllers/internal/SubscribeController.php` |
| 内部 | `GET` | `/internal/order/skuSalesVolume` | 获取sku销量 | 待确认 | 分页：data + total | `backend/modules/v1/controllers/internal/OrderController.php` |
| 内部 | `POST` | `/internal/order/distribute-sku-stat` | 获取配货sku销量 | 待确认 | 分页：data + total | `backend/modules/v1/controllers/internal/OrderController.php` |
| 内部 | `POST` | `/internal/order/user-auth-transfer` | 订单用户权限转移 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/internal/OrderController.php` |
| 内部 | `POST` | `/internal/order/platform-site-link` | 获取平台站点链接 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/internal/OrderController.php` |
| 内部 | `POST` | `/internal/order-refund/refund` | 订单退款单信息 | 待确认 | 分页：data + total | `backend/modules/v1/controllers/internal/OrderRefundController.php` |
| 内部 | `POST` | `/internal/order-refund/exchange` | 订单退货单信息 | 待确认 | 分页：data + total | `backend/modules/v1/controllers/internal/OrderRefundController.php` |
| 内部 | `GET` | `/internal/order/order-source-channel` | 获取订单来源渠道 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/internal/OrderController.php` |
| 内部 | `GET` | `/internal/order/auth-shop` | 获取有权限的店铺 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/internal/OrderController.php` |
| 内部 | `POST` | `/internal/order/marked-logistic-carrier` | 获取标记物流商 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/internal/OrderController.php` |
| 内部 | `POST` | `/internal/order/temu-logistic-carrier` | temu物流商 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/internal/OrderController.php` |
| 内部 | `GET` | `/internal/festival/stat` | 订单统计，包含订单量、销售额、客单价 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/order/OrderFestivalController.php` |
| 内部 | `GET` | `/internal/festival/stat-site` | 按站点统计销售额，订单量 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/order/OrderFestivalController.php` |
| 内部 | `GET` | `/internal/festival/sales-ranking` | 销量排行 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/order/OrderFestivalController.php` |
| 内部 | `POST` | `/internal/kpi/export` | 导出 | 请求体/查询串（字段待确认） | data=业务对象 | `backend/modules/v1/controllers/base/OrderPerformanceIndicatorController.php` |
| 内部 | `POST` | `/internal/kpi/index-member` | 指标列表导出 | 请求体/查询串（字段待确认） | data=业务对象 | `backend/modules/v1/controllers/base/OrderPerformanceIndicatorController.php` |
| 内部 | `POST` | `/internal/order-sku-commission/export` | 导出 | 请求体/查询串（字段待确认） | data=业务对象 | `backend/modules/v1/controllers/base/OrderChannelSkuCommissionController.php` |
| 内部 | `GET` | `/internal/base/config-setting/detail` | 获取配置值 | key | data=业务对象 | `backend/modules/v1/controllers/base/ConfigSettingController.php` |
| 内部 | `POST` | `/internal/order/problem-sku-list` | 问题单sku列表 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/internal/OrderController.php` |
| 内部 | `POST` | `/internal/olap/sku-gross-metrics/index` | sku产毛指标 | 待确认 | 分页：data + total | `backend/modules/v1/controllers/internal/olap/SkuGrossMetricsController.php` |
| 内部 | `POST` | `/internal/olap/sales-days-stat` | sku时间段销售统计 | 待确认 | 分页：data + total | `backend/modules/v1/controllers/internal/olap/SkuGrossMetricsController.php` |
| 内部 | `POST` | `/internal/olap/finance-latest-stats/index` | 获取SKU30-90天费用统计列表 | 待确认 | 分页：data + total | `backend/modules/v1/controllers/internal/olap/FinanceLatestStatsController.php` |
| 对外 | `POST` | `/olap/finance-latest-stats/index` | 获取SKU30-90天费用统计列表 | 待确认 | 分页：data + total | `backend/modules/v1/controllers/internal/olap/FinanceLatestStatsController.php` |
| 内部 | `POST` | `/internal/custom-expense/import` | 自定义费用导入 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/internal/CustomExpenseController.php` |
| 内部 | `POST` | `/internal/order-stat/order-count` | 订单量统计 | 请求体/查询串（字段待确认） | data=业务对象 | `backend/modules/v1/controllers/internal/OrderStatController.php` |
| 内部 | `POST` | `/internal/shop/operating-costs` | 获取店铺运营成本 | 请求体/查询串（字段待确认） | data=业务对象 | `backend/modules/v1/controllers/internal/OrderSourceChannelController.php` |
| 内部 | `POST` | `/internal/expense/expense-center/store` | 费用中心创建 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/expense/ExpenseCenterController.php` |
| 内部 | `POST` | `/internal/expense/expense-center/update` | 费用中心修改 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/expense/ExpenseCenterController.php` |
| 内部 | `POST` | `/internal/expense/expense-center/switch-state` | 切换状态 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/expense/ExpenseCenterController.php` |
| 内部 | `GET` | `/internal/expense/expense-center/strategy-detail/<id>` | 策略详情 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/expense/ExpenseCenterController.php` |
| 内部 | `POST` | `/internal/expense/expense-center/list` | 导出费用中心 | 请求体/查询串（字段待确认） | data=业务对象 | `backend/modules/v1/controllers/expense/ExpenseCenterController.php` |
| 内部 | `POST` | `/internal/wms-return-goods/return-goods` | 仓库退件入库通知 | 请求体/查询串（字段待确认） | data=业务对象 | `backend/modules/v1/controllers/internal/WmsReturnGoodsController.php` |
| 内部 | `POST` | `/internal/expense-order-profit-detail/estimates` | 预估详情列表 | 待确认 | 分页：data + total | `backend/modules/v1/controllers/expense/ExpenseOrderProfitDetailController.php` |
| 内部 | `POST` | `/internal/expense-order-profit-detail/settlements` | 结算详情列表 | 待确认 | 分页：data + total | `backend/modules/v1/controllers/expense/ExpenseOrderProfitDetailController.php` |

## subscribe

| 面 | 方法 | 路径 | 一句话 | 主入参 | 返回要点 | 控制器 |
|---|---|---|---|---|---|---|
| 内部 | `POST` | `/internal/subscribe/order/state` | 订单状态 | expand, order_sn, source, state | data=业务对象 | `backend/modules/v1/controllers/subscribe/OrderStateSubscribeController.php` |

## base

| 面 | 方法 | 路径 | 一句话 | 主入参 | 返回要点 | 控制器 |
|---|---|---|---|---|---|---|
| 对外 | `GET,POST` | `/options` | 枚举值列表 | 请求体/查询串（字段待确认） | data=业务对象 | `backend/modules/v1/controllers/base/OptionsController.php` |
| 内部 | `GET` | `/internal/swagger/doc` | 待确认 | 待确认 | 待确认 | `backend/modules/v1/controllers/base/SwaggerDocController.php` |
| 对外 | `GET` | `/file/signUrl` | 上传文件 | dir | data=业务对象 | `backend/modules/v1/controllers/base/FileController.php` |
| 对外 | `GET` | `/import/tpl` | 获取导入模板 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/base/CommonController.php` |
| 对外 | `GET` | `/import/filepath` | 获取导入文件链接 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/base/CommonController.php` |
| 对外 | `POST` | `/query/params/create` | 保存查询参数 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/QueryParamsManagerController.php` |
| 对外 | `GET` | `/query/params/index` | 获取查询参数 | scope, user_id | data=业务对象 | `backend/modules/v1/controllers/QueryParamsManagerController.php` |
| 对外 | `POST` | `/query/params/delete` | 删除查询参数 | id | data=业务对象 | `backend/modules/v1/controllers/QueryParamsManagerController.php` |
| 对外 | `POST` | `/query/params/sort` | 查询条件排序 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/QueryParamsManagerController.php` |
| 对外 | `POST` | `/base/profit-setting/index` | 利润检测设置列表 | 请求体/查询串（字段待确认） | 分页：data + total | `backend/modules/v1/controllers/base/ProfitSettingController.php` |
| 对外 | `GET` | `/base/profit-setting/detail` | 详情 | id | data=业务对象 | `backend/modules/v1/controllers/base/ProfitSettingController.php` |
| 对外 | `POST` | `/base/profit-setting/create` | 创建 | 请求体/查询串（字段待确认） | data=业务对象 | `backend/modules/v1/controllers/base/ProfitSettingController.php` |
| 对外 | `POST` | `/base/profit-setting/update` | 更新 | 请求体/查询串（字段待确认） | data=业务对象 | `backend/modules/v1/controllers/base/ProfitSettingController.php` |
| 对外 | `POST` | `/base/profit-setting/delete` | 删除 | id | data=业务对象 | `backend/modules/v1/controllers/base/ProfitSettingController.php` |
| 对外 | `POST` | `/base/profit-setting/status` | 启用禁用 | id, status | data=业务对象 | `backend/modules/v1/controllers/base/ProfitSettingController.php` |
| 对外 | `POST` | `/base/order-commission/index` | 平台佣金设置列表 | 请求体/查询串（字段待确认） | 分页：data + total | `backend/modules/v1/controllers/base/OrderCommissionController.php` |
| 对外 | `GET` | `/base/order-commission/detail` | 详情 | id | data=业务对象 | `backend/modules/v1/controllers/base/OrderCommissionController.php` |
| 对外 | `POST` | `/base/order-commission/create` | 新增 | 请求体/查询串（字段待确认） | data=业务对象 | `backend/modules/v1/controllers/base/OrderCommissionController.php` |
| 对外 | `POST` | `/base/order-commission/update` | 更新 | 请求体/查询串（字段待确认） | data=业务对象 | `backend/modules/v1/controllers/base/OrderCommissionController.php` |
| 对外 | `POST` | `/base/order-commission/delete` | 删除 | id | data=业务对象 | `backend/modules/v1/controllers/base/OrderCommissionController.php` |
| 对外 | `POST` | `/base/order-commission/status` | 更新状态 | id, status | data=业务对象 | `backend/modules/v1/controllers/base/OrderCommissionController.php` |
| 对外 | `POST` | `/base/order-sku-commission/index` | 平台sku佣金设置列表 | 请求体/查询串（字段待确认） | 分页：data + total | `backend/modules/v1/controllers/base/OrderChannelSkuCommissionController.php` |
| 对外 | `GET` | `/base/order-sku-commission/detail` | 详情 | id | data=业务对象 | `backend/modules/v1/controllers/base/OrderChannelSkuCommissionController.php` |
| 对外 | `POST` | `/base/order-sku-commission/create` | 新增 | 请求体/查询串（字段待确认） | data=业务对象 | `backend/modules/v1/controllers/base/OrderChannelSkuCommissionController.php` |
| 对外 | `POST` | `/base/order-sku-commission/update` | 更新 | 请求体/查询串（字段待确认） | data=业务对象 | `backend/modules/v1/controllers/base/OrderChannelSkuCommissionController.php` |
| 对外 | `POST` | `/base/order-sku-commission/delete` | 删除 | id | data=业务对象 | `backend/modules/v1/controllers/base/OrderChannelSkuCommissionController.php` |
| 对外 | `POST` | `/base/order-sku-commission/status` | 状态变更 | id, status | data=业务对象 | `backend/modules/v1/controllers/base/OrderChannelSkuCommissionController.php` |
| 对外 | `POST` | `/base/order-sku-commission/rate` | 批量佣金比例修改 | commission_rate, id, remark | data=业务对象 | `backend/modules/v1/controllers/base/OrderChannelSkuCommissionController.php` |
| 对外 | `POST` | `/base/vat-setting/index` | 列表 | 请求体/查询串（字段待确认） | 分页：data + total | `backend/modules/v1/controllers/base/VatSettingController.php` |
| 对外 | `GET` | `/base/vat-setting/detail` | 详情 | id | data=业务对象 | `backend/modules/v1/controllers/base/VatSettingController.php` |
| 对外 | `POST` | `/base/vat-setting/create` | 创建 | 请求体/查询串（字段待确认） | data=业务对象 | `backend/modules/v1/controllers/base/VatSettingController.php` |
| 对外 | `POST` | `/base/vat-setting/update` | 更新 | 请求体/查询串（字段待确认） | data=业务对象 | `backend/modules/v1/controllers/base/VatSettingController.php` |
| 对外 | `POST` | `/base/vat-setting/delete` | 删除 | id | data=业务对象 | `backend/modules/v1/controllers/base/VatSettingController.php` |
| 对外 | `POST` | `/base/vat-setting/status` | 批量修改状态 | id, status | data=业务对象 | `backend/modules/v1/controllers/base/VatSettingController.php` |
| 对外 | `POST` | `/base/order-source/index` | 列表 | 请求体/查询串（字段待确认） | 分页：data + total | `backend/modules/v1/controllers/base/OrderSourceController.php` |
| 对外 | `POST` | `/base/order-source/create` | 新增 | 请求体/查询串（字段待确认） | data=业务对象 | `backend/modules/v1/controllers/base/OrderSourceController.php` |
| 对外 | `POST` | `/base/order-source/batch-update` | 批量操作 | 请求体/查询串（字段待确认） | data=业务对象 | `backend/modules/v1/controllers/base/OrderSourceController.php` |
| 对外 | `POST` | `/base/order-source/delete` | 删除店铺地址 | 请求体/查询串（字段待确认） | data=业务对象 | `backend/modules/v1/controllers/base/OrderSourceController.php` |
| 对外 | `GET` | `/base/order-source/detail` | 详情 | id | data=业务对象 | `backend/modules/v1/controllers/base/OrderSourceController.php` |
| 对外 | `GET` | `/base/shop-shipping-address` | 店铺发货地址 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/internal/ShopAddressController.php` |
| 对外 | `POST` | `/base/order-strategy/index` | 排程策略列表 | 请求体/查询串（字段待确认） | 分页：data + total | `backend/modules/v1/controllers/base/OrderStrategyController.php` |
| 对外 | `POST` | `/base/order-strategy/create` | 创建策略 | 请求体/查询串（字段待确认） | data=业务对象 | `backend/modules/v1/controllers/base/OrderStrategyController.php` |
| 对外 | `GET` | `/base/order-strategy/<id:\d+>` | 单个详情 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/base/OrderStrategyController.php` |
| 对外 | `POST` | `/base/order-strategy/<id:\d+>/edit` | 编辑策略 | 请求体/查询串（字段待确认） | data=业务对象 | `backend/modules/v1/controllers/base/OrderStrategyController.php` |
| 对外 | `POST` | `/base/order-strategy/<id:\d+>/del` | 删除 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/base/OrderStrategyController.php` |
| 对外 | `POST` | `/base/order-strategy/modify-state` | 修改状态 | ids, status | data=业务对象 | `backend/modules/v1/controllers/base/OrderStrategyController.php` |
| 对外 | `POST` | `/base/order-strategy/exec-index` | 策略执行日志列表 | 请求体/查询串（字段待确认） | data=业务对象 | `backend/modules/v1/controllers/base/OrderStrategyController.php` |
| 对外 | `GET` | `/base/order-strategy/rule-name` | 获取物流优选规则名称列表 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/base/OrderStrategyController.php` |
| 对外 | `GET` | `/base/order-strategy/name` | 获取策略名称列表 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/base/OrderStrategyController.php` |
| 对外 | `POST` | `/base/custom-cost-item/index` | 列表 | 请求体/查询串（字段待确认） | 分页：data + total | `backend/modules/v1/controllers/base/CustomCostItemController.php` |
| 对外 | `POST` | `/base/custom-cost-item/create` | 添加 | 请求体/查询串（字段待确认） | data=业务对象 | `backend/modules/v1/controllers/base/CustomCostItemController.php` |
| 对外 | `POST` | `/base/custom-cost-item/update` | 编辑 | 请求体/查询串（字段待确认） | data=业务对象 | `backend/modules/v1/controllers/base/CustomCostItemController.php` |
| 对外 | `POST` | `/base/custom-cost-item/batch-update` | 批量修改状态 | ids, status | data=业务对象 | `backend/modules/v1/controllers/base/CustomCostItemController.php` |
| 对外 | `GET` | `/base/custom-cost-item/detail` | 详情 | id | data=业务对象 | `backend/modules/v1/controllers/base/CustomCostItemController.php` |
| 对外 | `GET` | `/base/custom-cost-item/fee-type` | 费用类型 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/base/CustomCostItemController.php` |
| 对外 | `POST` | `/base/config-setting/index` | 列表 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/base/ConfigSettingController.php` |
| 对外 | `POST` | `/base/config-setting/create` | 系统参数配置新增 | 待确认 | 待确认 | `backend/modules/v1/controllers/base/ConfigSettingController.php` |
| 对外 | `POST` | `/base/config-setting/update` | 系统参数配置编辑 | 请求体/查询串（字段待确认） | data=业务对象 | `backend/modules/v1/controllers/base/ConfigSettingController.php` |
| 对外 | `POST` | `/base/config-setting/batch-update` | 更新 | 请求体/查询串（字段待确认） | data=业务对象 | `backend/modules/v1/controllers/base/ConfigSettingController.php` |
| 对外 | `GET` | `/base/config-setting/detail` | 获取配置值 | key | data=业务对象 | `backend/modules/v1/controllers/base/ConfigSettingController.php` |
| 对外 | `POST` | `/base/blacklist/index` | 黑名单列表 | 请求体/查询串（字段待确认） | 分页：data + total | `backend/modules/v1/controllers/base/OrderBlacklistController.php` |
| 对外 | `GET` | `/base/blacklist/<id:\d+>` | 详情 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/base/OrderBlacklistController.php` |
| 对外 | `POST` | `/base/blacklist/create` | 新增 | 请求体/查询串（字段待确认） | data=业务对象 | `backend/modules/v1/controllers/base/OrderBlacklistController.php` |
| 对外 | `POST` | `/base/blacklist/del` | 删除 | ids | data=业务对象 | `backend/modules/v1/controllers/base/OrderBlacklistController.php` |
| 对外 | `POST` | `/base/blacklist/<id:\d+>/edit` | 编辑 | 请求体/查询串（字段待确认） | data=业务对象 | `backend/modules/v1/controllers/base/OrderBlacklistController.php` |
| 对外 | `POST` | `/base/blacklist/status` | 修改状态 | ids, status | data=业务对象 | `backend/modules/v1/controllers/base/OrderBlacklistController.php` |
| 对外 | `POST` | `/base/blacklist/import` | 导入黑名单 | oss_url | data=业务对象 | `backend/modules/v1/controllers/base/OrderBlacklistController.php` |
| 对外 | `GET` | `/base/blacklist/import-tpl` | 获取导入模板 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/base/OrderBlacklistController.php` |
| 对外 | `POST` | `/base/wms-preferred/index` | 列表 | 请求体/查询串（字段待确认） | 分页：data + total | `backend/modules/v1/controllers/base/OrderWmsPreferredController.php` |
| 对外 | `POST` | `/base/wms-preferred/create` | 新增 | 请求体/查询串（字段待确认） | data=业务对象 | `backend/modules/v1/controllers/base/OrderWmsPreferredController.php` |
| 对外 | `GET` | `/base/wms-preferred/<id:\d+>` | 详情 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/base/OrderWmsPreferredController.php` |
| 对外 | `POST` | `/base/wms-preferred/<id:\d+>/del` | 删除 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/base/OrderWmsPreferredController.php` |
| 对外 | `POST` | `/base/wms-preferred/<id:\d+>/edit` | 编辑 | 请求体/查询串（字段待确认） | data=业务对象 | `backend/modules/v1/controllers/base/OrderWmsPreferredController.php` |
| 对外 | `POST` | `/base/wms-preferred/status` | 修改状态 | ids, status | data=业务对象 | `backend/modules/v1/controllers/base/OrderWmsPreferredController.php` |
| 对外 | `POST` | `/base/kpi/index` | 业绩指标列表 | 请求体/查询串（字段待确认） | 分页：data + total | `backend/modules/v1/controllers/base/OrderPerformanceIndicatorController.php` |
| 对外 | `POST` | `/base/kpi/create` | 新增业绩指标 | 请求体/查询串（字段待确认） | data=业务对象 | `backend/modules/v1/controllers/base/OrderPerformanceIndicatorController.php` |
| 对外 | `GET` | `/base/kpi/<id:\d+>` | 指标详情 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/base/OrderPerformanceIndicatorController.php` |
| 对外 | `POST` | `/base/kpi/<id:\d+>/edit` | 编辑业绩指标 | 请求体/查询串（字段待确认） | data=业务对象 | `backend/modules/v1/controllers/base/OrderPerformanceIndicatorController.php` |
| 对外 | `POST` | `/base/kpi/status` | 状态：启用\|禁用 | 请求体/查询串（字段待确认） | data=业务对象 | `backend/modules/v1/controllers/base/OrderPerformanceIndicatorController.php` |
| 对外 | `POST` | `/base/kpi/<id:\d+>/del` | 删除 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/base/OrderPerformanceIndicatorController.php` |
| 对外 | `POST` | `/base/kpi/import-personal` | 导入个人指标 | oss_url | data=业务对象 | `backend/modules/v1/controllers/base/OrderPerformanceIndicatorController.php` |
| 对外 | `POST` | `/base/kpi/import-dept` | 导入目标指标 | oss_url | data=业务对象 | `backend/modules/v1/controllers/base/OrderPerformanceIndicatorController.php` |
| 对外 | `POST` | `/base/kpi/<id:\d+>/finish` | 业绩完成情况 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/base/OrderPerformanceIndicatorController.php` |
| 对外 | `GET` | `/base/kpi/<id:\d+>/member` | 获取指标下的人员、以及所属部门 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/base/OrderPerformanceIndicatorController.php` |
| 对外 | `GET` | `/base/kpi/user-center` | 个人中心 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/base/OrderPerformanceIndicatorController.php` |
| 对外 | `POST` | `/base/kpi/member` | 指标成员列表 | 请求体/查询串（字段待确认） | 分页：data + total | `backend/modules/v1/controllers/base/OrderPerformanceIndicatorController.php` |
| 对外 | `GET` | `/base/kpi/display-field` | 业绩达成报表显示字段 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/base/OrderPerformanceIndicatorController.php` |
| 对外 | `POST` | `/base/custom-choose-shipping` | 平台运输方式列表 | 请求体/查询串（字段待确认） | 分页：data + total | `backend/modules/v1/controllers/base/ShippingTypeController.php` |
| 对外 | `GET` | `/base/shipping-service` | 查找所有运输方式名称 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/base/ShippingTypeController.php` |
| 对外 | `POST` | `/base/mark-strategy/index` | 列表 | 请求体/查询串（字段待确认） | 分页：data + total | `backend/modules/v1/controllers/base/PlatformLabelStrategyController.php` |
| 对外 | `POST` | `/base/mark-strategy/create` | 新增 | 请求体/查询串（字段待确认） | data=业务对象 | `backend/modules/v1/controllers/base/PlatformLabelStrategyController.php` |
| 对外 | `POST` | `/base/mark-strategy/edit` | 编辑 | 请求体/查询串（字段待确认） | data=业务对象 | `backend/modules/v1/controllers/base/PlatformLabelStrategyController.php` |
| 对外 | `POST` | `/base/mark-strategy/del` | 删除 | id | data=业务对象 | `backend/modules/v1/controllers/base/PlatformLabelStrategyController.php` |
| 对外 | `POST` | `/base/mark-strategy/status` | 修改状态 | ids, status | data=业务对象 | `backend/modules/v1/controllers/base/PlatformLabelStrategyController.php` |
| 对外 | `GET` | `/base/mark-strategy/detail` | 详情 | id | data=业务对象 | `backend/modules/v1/controllers/base/PlatformLabelStrategyController.php` |
| 对外 | `POST` | `/base/split-rule/index` | 列表 | 请求体/查询串（字段待确认） | 分页：data + total | `backend/modules/v1/controllers/base/OrderSplitRuleController.php` |
| 对外 | `POST` | `/base/split-rule/create` | 新增 | 请求体/查询串（字段待确认） | data=业务对象 | `backend/modules/v1/controllers/base/OrderSplitRuleController.php` |
| 对外 | `GET` | `/base/split-rule/<id:\d+>` | 详情 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/base/OrderSplitRuleController.php` |
| 对外 | `POST` | `/base/split-rule/<id:\d+>/del` | 删除 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/base/OrderSplitRuleController.php` |
| 对外 | `POST` | `/base/split-rule/<id:\d+>/edit` | 编辑 | 请求体/查询串（字段待确认） | data=业务对象 | `backend/modules/v1/controllers/base/OrderSplitRuleController.php` |
| 对外 | `POST` | `/base/split-rule/status` | 修改状态 | ids, status | data=业务对象 | `backend/modules/v1/controllers/base/OrderSplitRuleController.php` |
| 对外 | `POST` | `/base/platform-warehouse` | 平台仓列表 | name, platform_id, shop_id | data=业务对象 | `backend/modules/v1/controllers/base/PlatformController.php` |
| 对外 | `GET` | `/base/platform-warehouse-group` | 平台发货仓库分组列表 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/base/PlatformController.php` |

## log

| 面 | 方法 | 路径 | 一句话 | 主入参 | 返回要点 | 控制器 |
|---|---|---|---|---|---|---|
| 对外 | `GET` | `/log/business` | 业务日志列表 | 待确认 | data=业务对象 | `backend/modules/v1/controllers/BusinessLoggerController.php` |

## 有 Controller、无显式路由

| 类 | 说明 |
|---|---|
| `backend/modules/v1/controllers/order/OrderNotifyController.php` | `actionShip` 空；routes 未登记 |
| `backend/modules/v1/controllers/report/OrderSkuSalesStatisticsController.php` | `actionIndex`；routes 未登记 |
