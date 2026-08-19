# OMS CLI 与 Consumer

## Console

入口：根目录 `yii`。命令格式 `php yii <controller-id>/<action-id>`。
选项列来自各控制器 `options()` 的返回值合集，**未按 action 逐条过滤**，具体某命令是否接收某选项待确认。

合计 **185** 个 public action，按控制器分组。

### `amazon` — 亚马逊命令行处理器

源码：`console/controllers/AmazonController.php`。控制器级选项：file, produceTimeStart, produceTimeEnd, start, end, type, key, uniqueKey, shopId, shopIds, shop, date, tag, site, recent, mode

| 命令 | 一句话 |
|---|---|
| `php yii amazon/expense-config-format` | 费用配置自动生成 -f 传入文件 |
| `php yii amazon/estimate-tidy` | 整理预估费用 |
| `php yii amazon/reality-expense-rollup` | 真实费用上卷 |
| `php yii amazon/reality-advert-migrate` | 真实广告费迁移 produceTimeStart 费用开始时间 produceTimeEnd 费用结束时间 |
| `php yii amazon/re-cal-advert` | 广告费重算 |
| `php yii amazon/re-cal-shop-settlement` | 店铺结算费用 |
| `php yii amazon/re-cal-all-settlement` | 结算费用 |
| `php yii amazon/cal-settlement` | 根据唯一key计算 |
| `php yii amazon/storage-fee-settlement` | 仓储费结算 |
| `php yii amazon/resettlement` | 重新投递结算费用 |
| `php yii amazon/problem-order-confirm` | 亚马逊FBA系统库存异常问题单自动确认 |
| `php yii amazon/find-settlement-report-without-marketplace-name` | 查找没有市场名称的结算报告: --date 2025-05-01~2025-05-31\|1D\|2W  --type 1 |
| `php yii amazon/estimated-storage-fee` | 预估仓储费 |
| `php yii amazon/all-lost-cleanup` | 全量检测遗失订单 |
| `php yii amazon/deal-compensate` | 清理deal sku 未关联 |
| `php yii amazon/flash-sale-link-sku` | 关联秒杀sku |
| `php yii amazon/pull-sp-advert-detail` | 拉取广告详情 |
| `php yii amazon/sync-refund-remark` | 同步亚马逊退款单退款描述 |
| `php yii amazon/creator-ads-cal` | 创造者联盟广告费计算 |

### `amazon-event-error-handle` — console/controllers/AmazonEventErrorHandleController.php

源码：`console/controllers/AmazonEventErrorHandleController.php`。控制器级选项：待确认（见该类 options()）

| 命令 | 一句话 |
|---|---|
| `php yii amazon-event-error-handle/events-retry` | amazon-event-error-handle/events-retry --shopId= --execTime= --eventType= |

### `amazon-finance` — amazon财务事件处理

源码：`console/controllers/AmazonFinanceController.php`。控制器级选项：startTime, endTime, groupId, shopId, recent, mode, hour, transactionType, breakdownType, uniqueKeys, lastId

| 命令 | 一句话 |
|---|---|
| `php yii amazon-finance/event-groups` | 拉取事件组列表 |
| `php yii amazon-finance/events-by-group` | 拉取事件组下事件列表 |
| `php yii amazon-finance/check-lost` | 待确认 |
| `php yii amazon-finance/financial-events` | 根据时间段拉取信息 |
| `php yii amazon-finance/group-test` | 待确认 |
| `php yii amazon-finance/event-test` | 待确认 |
| `php yii amazon-finance/event-type` | 待确认 |
| `php yii amazon-finance/financial-event-type` | 待确认 |
| `php yii amazon-finance/transaction-resolve` | 亚马逊交易数据解析 |
| `php yii amazon-finance/dispatch-transaction-item-settlement` | 派发交易数据item进入结算 --startTime\|endTime=2024-11-30T16:07:44Z --shopId=1,2,3 --transactionType=xx --breakdownType=xx |
| `php yii amazon-finance/dispatch-transaction` | 派发交易数据item进入结算 --startTime\|endTime=2024-11-30T16:07:44Z --shopId=1,2,3 --transactionType=xx --breakdownType=xx |
| `php yii amazon-finance/settlement-write-order-sn` | 根据结算时间写入订单号 |
| `php yii amazon-finance/sync-tags` | 待确认 |
| `php yii amazon-finance/fingerprints` | 为历史数据增加指纹 |

### `data-cleaning` — 数据刷新

源码：`console/controllers/DataCleaningController.php`。控制器级选项：date, pkId, shopId, platformId, mode, file, range

| 命令 | 一句话 |
|---|---|
| `php yii data-cleaning/update-spu` | 更新指定SPU分类 |
| `php yii data-cleaning/update-spu-category` | 大量更新spu分类 |
| `php yii data-cleaning/sync-sku-inv-age` | 同步sku库龄 |
| `php yii data-cleaning/sync-combined-sku-type` | 同步组合sku的类型 |
| `php yii data-cleaning/aliexpress-supply-price` | 更新速卖通供货价 |
| `php yii data-cleaning/update-refund-reason` | 更新退款原因 |

### `debug` — console/controllers/DebugController.php

源码：`console/controllers/DebugController.php`。控制器级选项：shopId, tenantId, orderId, platform, strategyId, isPrint, date, problemId, userId, orderSn, type, siteId, mode, platformOrderSn, filePath, uniqueKeys

| 命令 | 一句话 |
|---|---|
| `php yii debug/gen-order-by-mongo` | 生成Mongo存在，系统不存在的订单： -tId=1 -plat=amazon\|ebay\|otto\|walmart -date=2023-11-01 -oId= -shop=1223 |
| `php yii debug/gen-order` | 生成Mongo存在，系统不存在的订单： -tId=1 -plat=amazon\|ebay\|otto\|walmart -date=2023-11-01 -oId= -shop=1223 |
| `php yii debug/re-gen-order` | 生成系统订单: -tId=1 -plat=amazon -oId= -isPrint=true |
| `php yii debug/mark-shipped` | 标发测试： -oId=1234567890123 -plat=ebay,walmart |
| `php yii debug/exec-strategy` | 执行订单策略 -oId=122323243434 |
| `php yii debug/test-expense-gen` | 待确认 |
| `php yii debug/confirmed` | 货主不足的重新确认通过 -tId=1 -probId=17 |
| `php yii debug/re-check` | 重检 |
| `php yii debug/get-user-perm` | 获取用户权限：-tId=1 -uId=182 |
| `php yii debug/un-pay-init` | 重新初始化未付款的订单 |
| `php yii debug/amazon-return-goods` | 亚马逊退货报告投递费用计算 |
| `php yii debug/notify-by-order-shipped` | 订单已发货，通知物流：-tId=1 -oId=xxx,zzz |
| `php yii debug/re-profit` | 多件商品重新利润 |
| `php yii debug/refund-order-profit` | 退款单利润重算 |
| `php yii debug/flag-is-original` | 标记是否为原始订单 -tId=租户ID |
| `php yii debug/sync-fba-tracking-number` | 把FBA平台追踪号同步到标记追踪号 |
| `php yii debug/update-order-sales` | 更新订单销售人员 |
| `php yii debug/update-distribute-sku` | 更新配货sku |
| `php yii debug/profit-rollup` | 利润上卷 |
| `php yii debug/batch-operate` | 批量操作 |
| `php yii debug/order-inspection` | 订单巡检 |
| `php yii debug/import-expense-reprofit` | 自定义导入店铺费用重算 |
| `php yii debug/delete-transaction-item` | 待确认 |
| `php yii debug/warehouse-allocation` | 指定仓库配货 |
| `php yii debug/profit-debug` | 利润计算debug |
| `php yii debug/sync-split-merge-fields` | 同步订单拆合单字段到费用sku明细表 |
| `php yii debug/sync-shop-expense-item-times` | 同步店铺费用item维度时间字段 |
| `php yii debug/sync-reason-fields-from-file` | 从文件读取系统订单号，同步建单原因、拆合单、原单、建单类型字段到费用统计三张表 |
| `php yii debug/extract-tk-package-number` | 提取Tk原单的拆分订单包裹号     * |

### `ebay` — EbayController

源码：`console/controllers/EbayController.php`。控制器级选项：file, start, end, shopId, orderId, transactionType, uniqueKey

| 命令 | 一句话 |
|---|---|
| `php yii ebay/expense-config-format` | 费用配置自动生成 -f 传入文件 |
| `php yii ebay/reality-expense-rollup` | 真实费用上卷 |
| `php yii ebay/re-cal-shop-settlement` | 重新计算广告费 |
| `php yii ebay/re-cal-all-settlement` | 重新计算广告费 |

### `expense` — 费用控制器

源码：`console/controllers/ExpenseController.php`。控制器级选项：file, platformId, tenantId, start, end, recent, platformIds, startId, sku, shopId, type, key, orderSn

| 命令 | 一句话 |
|---|---|
| `php yii expense/product-dev-commission-task` | 产品开发提成计算任务 |
| `php yii expense/custom-import-fee-handler` | 自定义导入费用处理 |
| `php yii expense/replenish-import` | 费用补充导入 |
| `php yii expense/tidy-lost-order-expense` | 整理未匹配订单费用 [--start='2024-06-05 00:00:00'] [--end='2024-06-07 00:00:00'] [--platformId=1] [--recent=7 最近7天] |
| `php yii expense/inspect-sku-miss` | 检查sku未匹配 |
| `php yii expense/daily-shop-commission` | 每天店铺佣金比例计算 |
| `php yii expense/update-sku-categories` | 修改sku分类信息 |
| `php yii expense/settlement-recalculate` | 结算报告重算 |
| `php yii expense/multi-channel-shop-fee-check` | 多渠道发货订单店铺费用检测 |
| `php yii expense/expense-import-report` | 自定义费用导入报表 |
| `php yii expense/expense-center-charge-reset` | 费用中心计费金额重置 |
| `php yii expense/lazada-transaction-pull` | lazada结算拉取 |
| `php yii expense/expense-close-create` | 创建关账 |
| `php yii expense/expense-close-init` | 初始化费用结算数据 |
| `php yii expense/create-snapshot` | 创建快照 |

### `ling-xing` — 领星数据拉取控制台命令类

源码：`console/controllers/LingXingController.php`。控制器级选项：tenantId, date, shopId

| 命令 | 一句话 |
|---|---|
| `php yii ling-xing/receivable-report` | 拉取应收报告：-tId=1 -date=2024-11 -sId=1622 |
| `php yii ling-xing/order-item` | 拉取订单详情：-tId=1 -date=2H -sId=1622 |

### `message-send` — 消息推送控制台命令类

源码：`console/controllers/MessageSendController.php`。控制器级选项：tenantId, startDate, endDate, mode, shopId, day, uid, excludePlatform

| 命令 | 一句话 |
|---|---|
| `php yii message-send/festival` | 节日大屏全部数据 |
| `php yii message-send/festival-ranking` | 节日大屏排行【热门类目、成交榜单】： -tId=1 -sDate="2024-07-16 00:00:00" -eDate="2024-07-18 15:00:00" -mod=0,1 |
| `php yii message-send/order-stat` | 节日大屏-按订单统计：-tId=1 -sDate="2024-07-16 00:00:00" -eDate="2024-07-18 15:00:00" |
| `php yii message-send/site-country-stat` | 节日大屏-按站点国家统计：-tId=1 -sDate="2024-07-16 00:00:00" -eDate="2024-07-18 15:00:00" |
| `php yii message-send/associate-sku-stat` | 未关联sku统计并发送消息：-tId=1 -sId=1492 -mod=[1发送消息，其他不发] |
| `php yii message-send/mark-failed-notify` | 订单标发失败通知销售人员 -tId=1 -sDate= -eDate= -exPlat=3 |
| `php yii message-send/problem-order-notify` | 问题单飞书通知 |
| `php yii message-send/send-cert-fee` | 创建时间将要超过指定天数的认证费，发送给创建人 |
| `php yii message-send/auto-bind-sku-and-notify` | 自动绑定sku并发通知 |

### `order` — console/controllers/OrderController.php

源码：`console/controllers/OrderController.php`。控制器级选项：tenantId, platform, platformId, site, shop, shopId, date, day, minute, hour, start, end, type, mode, sku, file, pullStatus, isRange, isCreate, orderIds, excluded

| 命令 | 一句话 |
|---|---|
| `php yii order/pull-order-shop` | 拉单商铺列表，参数说明：-plat=amazon,ebay -shop=10,12 -site=UK,US -date=2023-07-08~2023-07-10\|1D\|1M -tId=租户ID -d=1 |
| `php yii order/amazon-sqs` | 消费亚马逊SQS消息: -tId=1 -type=stream\|notify |
| `php yii order/pull-original-order` | 拉取原始订单：-tId=1 -plat=amazon\|ebay\|tiktok -shop=123 -oId=121111,1111 |
| `php yii order/sync-shipping-type` | 拉取平台运输方式，参数：-site=US,UK -tId=1 |
| `php yii order/detection-shop` | 订单检查的商铺列表，参数说明：-plat=amazon,ebay -shop=10,12 -site=UK,US -date=2023-07-08~2023-07-10\|1D\|1M -tId=租户ID -d=1 |
| `php yii order/pull-order-fee` | 拉取订单费用，参数说明：-plat=amazon,ebay -shop=10,12 -date=2023-07-08~2023-07-10\|1D\|1M -tId=租户ID -type=1,2 |
| `php yii order/pull-amz-report` | 拉取亚马逊报告文件 -tId=1 -type=1结算报告\|2退换货报告\|3物流报告\|4仓储费\|5库存仓储超量费\|6长期仓储费 |
| `php yii order/pull-amz-advert` | 亚马逊广告拉取投递： -tId=1 -type=SP\|SB\|SD |
| `php yii order/pull-amazon-advert-info` | 拉取广告信息, 参数：-tId=1 -shop=123 -type=SP\|SB\|SD |
| `php yii order/auto-allocation` | 自动配货，参数说明： -tId=租户ID |
| `php yii order/parse-amazon-advert` | 解析亚马逊广告文件 |
| `php yii order/timed-mark-shipment` | 定时补偿标发: -tId=1 -plat=amazon,ebay |
| `php yii order/re-mark-shipped` | 当追踪号有变更时，需系统自动将最新的追踪号上传到后台 |
| `php yii order/sync-shop` | 同步平台、店铺、站点 参数说明： -tId=租户ID |
| `php yii order/sync-shop-company` | 同步店铺公司信息 |
| `php yii order/create-ad-report` | 订单检查的商铺列表，参数说明：-shop=1451 -date=2023-07-08\|1 -tId=租户ID -type=SB\|SD\|SP -plat=amazon\|amazonvc -mode=[0：不输出日志] |
| `php yii order/order-inspection` | 订单巡检任务 |
| `php yii order/get-ebay-refund-status` | 获取Ebay退款单的状态 参数：-tId=1 |
| `php yii order/exec-order-strategy` | 定时执行订单排程策略 参数：-tId=1 |
| `php yii order/detection-unconfirmed-order` | 订单检测待确认的订单 参数：-tId=1 |
| `php yii order/sub-notification` | 定时更新亚马逊店铺订阅ID， 参数：-tId=1 -type=notify\|stream -shop=123 |
| `php yii order/auto-gen-tracking-number` | 自动生成追踪号， 参数：-tId=1 -oId=xxx,xxx |
| `php yii order/sync-system-order` | 同步源订单存在，系统订单不存在的情况 |
| `php yii order/sync-order-status` | 同步订单状态，参数说明：-platform=walmart -shop=10,12 -site=UK,US -date=2023-07-08~2023-07-10\|1D\|1M -tId=租户ID -d=1 |
| `php yii order/unmark-ship-notify` | 未标发的订单通知 |
| `php yii order/settlement-profit-cal` | 平台结算后的利润计算 |
| `php yii order/refund-sku-detection` | 退款单sku未空检测 |
| `php yii order/problem-order-profit` | 问题单利润计算 |
| `php yii order/create-traffic-report` | 创建流量报告 |
| `php yii order/refund-problem-order` | 处理全部退款的问题订单 |
| `php yii order/amz-fba-sku-replace` | 亚马逊FBA订单SKU替换: -tId=1 -oId=2222,1111 |
| `php yii order/order-refund-es-reinspect` | 退款单巡检 |
| `php yii order/latest-delivery-notify` | 延迟发货订单通知 |
| `php yii order/pull-tiktok-warehouse` | 拉取tiktok平台仓库 |
| `php yii order/manual-get-tracking-number` | 手动调用预生成追踪号接口 |
| `php yii order/send-email-of-order` | 发送订单邮件 |
| `php yii order/pull-amazon-invoice` | 拉取亚马逊广告发票：-tId=1 -shop=1234 |
| `php yii order/fetch-fba-tracking-number` | FBA追踪号获取 |

### `otto-commission` — otto佣金命令行

源码：`console/controllers/OttoCommissionController.php`。控制器级选项：file

| 命令 | 一句话 |
|---|---|
| `php yii otto-commission/import` | 导入otto 佣金配置 -f otto_commission_rate.xlsx |

### `otto` — otto命令行

源码：`console/controllers/OttoController.php`。控制器级选项：start, end, shopId, orderSn

| 命令 | 一句话 |
|---|---|
| `php yii otto/re-cal-all-settlement` | 结算费用 --shopId --start --end |
| `php yii otto/payment-sync` | 支付信息同步 |
| `php yii otto/recreate-order` | 重新生成订单 |

### `sales-stat` — 销售数据统计

源码：`console/controllers/SalesStatController.php`。控制器级选项：tenantId, date, saleId, day, deptId

| 命令 | 一句话 |
|---|---|
| `php yii sales-stat/sales-kpi` | 统计销售业绩: -tId=1 -date=2D\|2025-01-10~2025-01-15 -sId=363,461 |
| `php yii sales-stat/sales-m2m` | 销售的环比数据统计: -tId=1 -date=2024-09 -sId=363,461 |
| `php yii sales-stat/department-kpi` | 统计部门业绩: -tId=1 -date=2D\|2025-01-10~2025-01-15 -dId=98 |
| `php yii sales-stat/department-m2m` | 部门的环比数据统计: -tId=1 -date=2024-09 -dId=363,461 |

### `shadow-knife` — 影刀数据文件解析

源码：`console/controllers/ShadowKnifeController.php`。控制器级选项：tenantId, filepath, date, platform, shopId

| 命令 | 一句话 |
|---|---|
| `php yii shadow-knife/parse-deferred-order` | 解析影刀延迟订单文件 |
| `php yii shadow-knife/parse-all-files` | 解析文件 |
| `php yii shadow-knife/parse-by-file` | 按文件路径导入 |

### `shein` — shein命令行

源码：`console/controllers/SheinController.php`。控制器级选项：start, end, shopId, orderNo, tenantId, date

| 命令 | 一句话 |
|---|---|
| `php yii shein/pull-order` | 通过订单号拉取 |
| `php yii shein/recreate-order` | 重新生成订单 |
| `php yii shein/re-cal-all-settlement` | 结算费用 --shopId --start --end |
| `php yii shein/pull-return-order` | 拉取退货单 |
| `php yii shein/regenerate-return-order` | 重新生成退货单 |
| `php yii shein/asin-url-update` | asin 跳转链接更新 |

### `statistics` — 定时统计

源码：`console/controllers/StatisticsController.php`。控制器级选项：tenantId, date, lastDay, saiHe, fromOrder, saiHePlatform, orderPlatform

| 命令 | 一句话 |
|---|---|
| `php yii statistics/sku-sales-volume` | sku销量统计 |
| `php yii statistics/distribute-sku-stat` | 配货sku销量统计 |

### `sync-data-to-es` — console/controllers/SyncDataToEsController.php

源码：`console/controllers/SyncDataToEsController.php`。控制器级选项：tenantId

| 命令 | 一句话 |
|---|---|
| `php yii sync-data-to-es/preferred` | 批量同步 物流优选数据 到ES |
| `php yii sync-data-to-es/order-strategy` | 同步订单排程策略到ES |
| `php yii sync-data-to-es/blacklist` | 批量同步黑名单到ES, 参数说明: -tId=租户ID |

### `temu` — temu命令行

源码：`console/controllers/TemuController.php`。控制器级选项：start, end, shop, shopId, orderSn, tenantId, status, date, uniqueKeys, mark, recent, file, type, productId

| 命令 | 一句话 |
|---|---|
| `php yii temu/order-pull` | 拉单 |
| `php yii temu/recreate-order` | 重新生成订单 |
| `php yii temu/update-email` | 邮箱更新 |
| `php yii temu/reinspection-order` | 重检FBA且发货仓库为空的订单 |
| `php yii temu/wms-debug` | 获取仓库信息 |
| `php yii temu/tracking-number-update` | 追踪号更新 |
| `php yii temu/zip-code` | 邮编更新 |
| `php yii temu/ying-dao-temu-settlement-resolver` | 解析影刀temu费用 |
| `php yii temu/ying-dao-temu-settlement-file-resolver` | 解析影刀temu费用 |
| `php yii temu/settlement-dispatch` | 结算数据派发 |
| `php yii temu/expense-config-format` | 费用配置自动生成 -f 传入文件 |
| `php yii temu/canceled-shipped-order-notify` | 已发货但平台取消订单飞书通知 |
| `php yii temu/batch-order-num-update` | 批量单号更新 |
| `php yii temu/auto-create-refund-order` | 自动生成temu半托管退款单 |
| `php yii temu/ads-fee-push` | 手动投递Temu广告费 |
| `php yii temu/pull-amount-info` | 按订单号拉取平台金额信息 |

### `tiktok` — tiktok

源码：`console/controllers/TiktokController.php`。控制器级选项：待确认（见该类 options()）

| 命令 | 一句话 |
|---|---|
| `php yii tiktok/resettlement` | 重新投递结算 |
| `php yii tiktok/statement-pull` | 根据结算id拉取 |
| `php yii tiktok/shipping-fee-tax-update` | 运费税费更新 |
| `php yii tiktok/asin-update` | 待确认 |
| `php yii tiktok/reinspection-order` | 订单发货仓库为空重检订单 |
| `php yii tiktok/ads-fee-push` | 手动投递广告费 |

### `walmart` — 沃尔玛命令行

源码：`console/controllers/WalmartController.php`。控制器级选项：start, end, shop, key, orderSn

| 命令 | 一句话 |
|---|---|
| `php yii walmart/re-cal-all-settlement` | 结算费用 --shop --start --end |
| `php yii walmart/recreate-order` | 重新生成订单 |

### `xingpan` — 星盘独立站命令行

源码：`console/controllers/XingpanController.php`。控制器级选项：start, end, shopId, orderNo, tenantId, date

| 命令 | 一句话 |
|---|---|
| `php yii xingpan/pull-order` | 通过订单号拉取 |
| `php yii xingpan/recreate-order` | 重新生成订单 |


README 里写的 `order/re-cal-all-profit` 在 `console/controllers` **未找到**对应 action，以本表为准。

## Consumer

真源：`common/config/app/rabbitmq/consumer.yml`。
队列名默认与 consumer `name` 相同（见 `queue.yml`）；exchange/binding 见同目录 `exchange.yml` / `binding.yml`，本表不展开。

配置登记 **87** 个。

| name | 职责 | 类文件 |
|---|---|---|
| `es_order` | 订单同步es消费 | `backend/modules/v1/consumer/EsOrderConsumer.php` |
| `es_order_refund` | 退款单es同步 | `backend/modules/v1/consumer/EsOrderRefundConsumer.php` |
| `base_shop_sync_oms` | 商铺基础信息变更 | `backend/modules/v1/consumer/BaseShopChangedConsumer.php` |
| `goods_channel_sku_changed` | 店铺与渠道sku关系发生变更 | `backend/modules/v1/consumer/ShopSkuChangedConsumer.php` |
| `dispatcher_order_update` | 待确认 | `backend/modules/v1/consumer/DispatcherOrderUpdateConsumer.php` |
| `spu_change_sync_oms` | SPU发生变更 | `backend/modules/v1/consumer/SpuChangedConsumer.php` |
| `original_order` | 订单同步消费者 | `backend/modules/v1/consumer/SyncOrderConsumer.php` |
| `order_refund_generator` | 待确认 | `backend/modules/v1/consumer/OrderRefundGeneratorConsumer.php` |
| `order_platform_settlement` | 订单费用消费 | `backend/modules/v1/consumer/OrderExpenseSettlementConsumer.php` |
| `amazon_order_item` | 消费队列数据，根据订单号，拉取订单完整信息 | `backend/modules/v1/consumer/AmazonOrderItemConsumer.php` |
| `batch_operation` | 订单批量处理消费者 | `backend/modules/v1/consumer/OrderBatchOperationConsumer.php` |
| `amazon_order_detection` | 消费需要订单检测的店铺 | `backend/modules/v1/consumer/AmazonOrderDetectionConsumer.php` |
| `ebay_order_detection` | 消费需要订单检测的商铺 | `backend/modules/v1/consumer/EbayOrderDetectionConsumer.php` |
| `ebay_order_fee_pull` | 订单费用拉取 | `backend/modules/v1/consumer/EbayOrderFeePullConsumer.php` |
| `walmart_order_fee_pull` | walmart 订单费用报告拉取 | `backend/modules/v1/consumer/WalmartOrderFeePullConsumer.php` |
| `amazon_order_fee_pull` | 订单费用拉取 | `backend/modules/v1/consumer/AmazonOrderFeePullConsumer.php` |
| `amazon_report_doc` | 亚马逊订单费用报告文档拉取 | `backend/modules/v1/consumer/AmazonReportDocConsumer.php` |
| `amazon_shipment_report_after` | 亚马逊 物流报告后续处理 | `backend/modules/v1/consumer/AmazonShipmentReportAfterConsumer.php` |
| `mark_shipment` | 订单标记发货 | `backend/modules/v1/consumer/OrderMarkShipmentConsumer.php` |
| `order_attr_change` | 订单属性发生变化处理 | `backend/modules/v1/consumer/OrderAttrChangedConsumer.php` |
| `amazon_shop` | 待确认 | `backend/modules/v1/consumer/AmazonOrderPullConsumer.php` |
| `ebay_shop` | 待确认 | `backend/modules/v1/consumer/EbayOrderPullConsumer.php` |
| `walmart_shop` | 沃尔玛订单拉取消费者 | `backend/modules/v1/consumer/WalmartOrderPullConsumer.php` |
| `temusemi_shop` | temu订单拉取消费者 | `backend/modules/v1/consumer/TemuOrderPullConsumer.php` |
| `temusemi_order_detection` | temu 订单检测 | `backend/modules/v1/consumer/TemuOrderDetectionConsumer.php` |
| `shein_shop` | shein拉单消费者 | `backend/modules/v1/consumer/SheinOrderPullConsumer.php` |
| `shein_order_detection` | SHEIN 订单检测 | `backend/modules/v1/consumer/SheinOrderDetectionConsumer.php` |
| `kaufland_shop` | 待确认 | `backend/modules/v1/consumer/KauflandOrderPullConsumer.php` |
| `kaufland_order_detection` | 待确认 | `backend/modules/v1/consumer/KauflandOrderDetectionConsumer.php` |
| `order_expense_calculate` | 订单利润计算 | `backend/modules/v1/consumer/OrderProfitCalculateConsumer.php` |
| `import_tracking_number` | 导入追踪号 | `backend/modules/v1/consumer/ImportTrackingNumberConsumer.php` |
| `amazon_create_advert` | 创建亚马逊广告 | `backend/modules/v1/consumer/AmazonCreateAdvertConsumer.php` |
| `amazon_advert` | 亚马逊店铺广告费用 | `backend/modules/v1/consumer/AmazonShopAdConsumer.php` |
| `amazon_advert_info` | 亚马逊店铺广告Asin | `backend/modules/v1/consumer/AmazonAdvertInfoConsumer.php` |
| `amazon_sb_advert_detail` | 亚马逊店铺Sb类型广告费用详情 | `backend/modules/v1/consumer/AmazonSbAdvertConsumer.php` |
| `amazon_advert_handler` | 亚马逊广告收入消费者 | `backend/modules/v1/consumer/AmazonAdvertiseConsumer.php` |
| `tiktok_advert_fee` | TikTok广告费用消费者 | `backend/modules/v1/consumer/TiktokAdsAdvertiseConsumer.php` |
| `temu_ads_fee` | Temu广告费用消费者 | `backend/modules/v1/consumer/TemuAdsAdvertiseConsumer.php` |
| `amazon_creator_ads_fee` | 亚马逊创造者联盟广告费用消费者 | `backend/modules/v1/consumer/AmazonCreatorConnectionsAdsConsumer.php` |
| `order_index_inspection` | 订单巡检 | `backend/modules/v1/consumer/ElasticOrderInspectionConsumer.php` |
| `sub_order_info` | 待确认 | `backend/modules/v1/consumer/OrderInfoConsumer.php` |
| `amazon_refund_detection` | 亚马逊订单退款检测 | `backend/modules/v1/consumer/AmazonRefundDetectionConsumer.php` |
| `reality_expense_rollup` | 结算上卷消费 | `backend/modules/v1/consumer/RealityExpenseRollupConsumer.php` |
| `order_refund` | 订单退款 | `backend/modules/v1/consumer/OrderRefundConsumer.php` |
| `walmart_order_detection` | walmart 订单检测 | `backend/modules/v1/consumer/WalmartOrderDetectionConsumer.php` |
| `user_auth_transfer` | 订单用户权限转移消费者 | `backend/modules/v1/consumer/OrderUserAuthTransferConsumer.php` |
| `sku_sales_volume` | 订单sku销量统计 | `backend/modules/v1/consumer/OrderSkuSalesVolumeConsumer.php` |
| `return_goods_expense` | 退货订单返还费用计算 | `backend/modules/v1/consumer/OrderReturnGoodsExpenseConsumer.php` |
| `walmart_sync_order_status` | 同步walmart订单状态 | `backend/modules/v1/consumer/WalmartSyncOrderStatusConsumer.php` |
| `otto_shop` | OTTO拉单消费者 | `backend/modules/v1/consumer/OttoOrderPullConsumer.php` |
| `otto_order_detection` | otto 订单检测 | `backend/modules/v1/consumer/OttoOrderDetectionConsumer.php` |
| `order_profit_rollup` | 利润上卷 | `backend/modules/v1/consumer/OrderProfitRollupConsumer.php` |
| `otto_order_fee_pull` | otto 订单费用报告拉取 | `backend/modules/v1/consumer/OttoOrderFeePullConsumer.php` |
| `order_profit_task` | 订单利润计算 | `backend/modules/v1/consumer/OrderProfitCalculateConsumer.php` |
| `storage_fee_settlement` | 亚马逊仓储费 | `backend/modules/v1/consumer/AmazonStorageFeeConsumer.php` |
| `import_expense` | 自定义导入费用 | `backend/modules/v1/consumer/ImportExpenseCustomizeConsumer.php` |
| `amazon_finance_event_detect` | 亚马逊结算事件检测 | `backend/modules/v1/consumer/AmazonFinanceEventsDetectConsumer.php` |
| `shein_return_order_pull` | SHEIN 退货单拉取 | `backend/modules/v1/consumer/SheinReturnOrderPullConsumer.php` |
| `amazon_get_transaction` | 亚马逊0619新接口 | `backend/modules/v1/consumer/AmazonGetTransactionConsumer.php` |
| `amazon_transaction_resolve` | 解析亚马逊交易数据 | `backend/modules/v1/consumer/AmazonTransactionResolveConsumer.php` |
| `saihe_refund_fee_calculate` | 赛盒退款单费用计算 | `backend/modules/v1/consumer/SaiheOrderRefundCalculateConsumer.php` |
| `order_init_expense_calculate` | 订单初始费用计算消费者 | `backend/modules/v1/consumer/ExpenseInitializeCalculateConsumer.php` |
| `tiktok_shop` | 待确认 | `backend/modules/v1/consumer/TiktokOrderPullConsumer.php` |
| `tiktok_order_detection` | Tiktok 订单检测 | `backend/modules/v1/consumer/TiktokOrderDetectionConsumer.php` |
| `tiktok_order_fee_pull` | Tiktok结算报告拉取 | `backend/modules/v1/consumer/TiktokStatementTransactionsPullConsumer.php` |
| `aliexpress_shop` | 待确认 | `backend/modules/v1/consumer/AliexpressOrderPullConsumer.php` |
| `aliexpress_order_detection` | Aliexpress 订单检测 | `backend/modules/v1/consumer/AliexpressOrderDetectionConsumer.php` |
| `aliexpress_order_fee_pull` | 速卖通结算报告拉取 | `backend/modules/v1/consumer/AliexpressSettlementReportPullConsumer.php` |
| `xingpan_shop` | 独立站拉单消费者 | `backend/modules/v1/consumer/XingpanOrderPullConsumer.php` |
| `xingpan_order_detection` | 星盘独立站 订单检测 | `backend/modules/v1/consumer/XingpanOrderDetectionConsumer.php` |
| `expense_refund_dispatch` | 退款费用计算派发 | `backend/modules/v1/consumer/ExpenseRefundDispatchConsumer.php` |
| `shopee_shop` | 待确认 | `backend/modules/v1/consumer/ShopeeOrderPullConsumer.php` |
| `shopee_order_detection` | Shopee 订单检测 | `backend/modules/v1/consumer/ShopeeOrderDetectionConsumer.php` |
| `expense_customize_import_report` | 待确认 | `backend/modules/v1/consumer/ExpenseCustomizeImportReportConsumer.php` |
| `expense_sku_snapshot` | SKU费用快照定存 | `backend/modules/v1/consumer/ExpenseSkuSnapshotConsumer.php` |
| `order_item_financial` | 订单商品收支明细 | `backend/modules/v1/consumer/OrderItemFinancialConsumer.php` |
| `shopee_order_fee_pull` | shopee结算报告拉取消费者 | `backend/modules/v1/consumer/ShopeeSettlementPullConsumer.php` |
| `es_order_refund_inspect` | 退款单es同步 | `backend/modules/v1/consumer/EsOrderRefundConsumer.php` |
| `lazada_shop` | Lazada订单拉取消费者 | `backend/modules/v1/consumer/LazadaOrderPullConsumer.php` |
| `lazada_order_detection` | Lazada 订单检测 | `backend/modules/v1/consumer/LazadaOrderDetectionConsumer.php` |
| `lazada_order_fee_pull` | Lazada 订单费用报告拉取 | `backend/modules/v1/consumer/LazadaOrderFeePullConsumer.php` |
| `amazon_purchase_order` | 亚马逊采购订单生成队列 | `backend/modules/v1/consumer/AmazonPurchaseOrderConsumer.php` |
| `user_auth_transfer_execute` | 权限转移执行消费者 | `backend/modules/v1/consumer/UserAuthTransferExecuteConsumer.php` |
| `amazon_invoice` | 待确认 | `backend/modules/v1/consumer/AmazonInvoiceConsumer.php` |
| `amazon_invoice_detail` | 待确认 | `backend/modules/v1/consumer/AmazonInvoiceDetailConsumer.php` |
| `allegro_shop` | 待确认 | `backend/modules/v1/consumer/AllegroOrderPullConsumer.php` |
| `allegro_order_detection` | Allegro 订单检测 | `backend/modules/v1/consumer/AllegroOrderDetectionConsumer.php` |