# OMS 数据模型

真源优先级：`console/migrations/**` > `backend/models/**`。
migrations 只有建表、几乎无后续 alter；线上字段可能已漂移，与 model 不一致时以「待确认」标出。
本仓库 **没有** DB 级外键（Yii ActiveRecord 逻辑关联）。

## 探测与范围

| 项 | 结果 |
|---|---|
| 系统 / 剖面 | oms（信诚-OMS）/ yii2-advanced |
| MySQL 建表 migration | 39 个（含 dwh 1 个） |
| 有 tableName() 的 Model | 91 个 |
| 高价值域判定 | 按 `api-list.md` 显式路由条数，不是按「订单/费用」默认 |
| 非核心 | 只列索引，不穷尽字段 |

HTTP 入口条数（来自 [../02-surfaces/api-list.md](../02-surfaces/api-list.md)）：

| 域 | 路由条数 | 本页深度 |
|---|---:|---|
| base | 103 | 核心对象 |
| order | 52 | 核心对象 |
| expense | 43 | 核心对象 |
| internal | 42 | 核心对象 |
| refund | 21 | 核心对象 |
| export | 18 | 索引 |
| report | 8 | 索引（有 migration） |
| saihe | 6 | 索引 |
| exchange | 6 | 核心对象 |
| subscribe | 1 | 核心对象 |
| log | 1 | 索引 |

约定公共列（多数 MySQL 表）：`id` PK、`tenant_id`、`create_time`/`update_time`、`create_user`/`update_user`。下文「关键列」省略这组。

存储分层（配置真实存在，见 `common/config/app/components/`）：

| 引擎 | 组件 | 用途（已知） |
|---|---|---|
| MySQL | `db` | 业务主库（本文主体） |
| MongoDB | `mongodb` | 平台原始单/广告/费用流水，见文末索引 |
| Elasticsearch | `elasticsearch` | 订单/策略检索，见 `backend/models/es/` |
| ClickHouse | `clickhouse` | OLAP 统计，见 `backend/models/olap/` |
| Redis | `cache` | 缓存，无业务表 |

## 逻辑关联（无 DB FK）

从 migration 列名 + 已声明的 ActiveRecord 关系抽出，未在代码看到的不写。

| 从 | 到 | 依据 |
|---|---|---|
| `order` 1—N `order_item` | `order_item.order_id` | migration + `Order::getOrderItems()` |
| `order` 1—N `order_item_assemble` | `order_item_assemble.order_id`；列 `order_item_id` comment 是「数据原平台id」，**不是** item.id | migration 列 |
| `order` 1—1/N `order_payment` / `order_logistics` / `order_recipient` / `order_sender` | 各表 `order_id` | migration 列 |
| `order` 1—N `order_expense_report` | `order_expense_report.order_id` | migration 列 |
| `order` 1—N `order_refund` | `order_refund.order_id` | migration 列 |
| `order_refund` 1—N `order_refund_products` | `order_refund_products.order_refund_id` + `order_id` | migration 列 |
| `order` 1—N `order_exchange` | `order_exchange.order_id` | migration 列 |
| `order_exchange` 1—N `order_exchange_products` | `order_exchange_products.order_exchange_id` | migration 列 |
| `order` N—1 `order_strategy` | `order.order_strategy_id` | migration 列 |
| `order` → 分仓/物流优选 | `splitting_strategy_id` 等，落点待确认（现存 `order_wms_preferred` 与旧表 `logistics_preferred`） | migration 列名 |
| `expense_customize_import` 1—N `expense_customize_detail` | `expense_customize_detail.import_id` | migration 列 |
| `order_source_channel` 1—N `order_source_agent` / `order_source_shop_address` | `source_channel_id` | migration 列 |
| `expense_rule` 1—1 `expense_rule_strategy` | `ExpenseRule` / `ExpenseRuleStrategy` hasOne | **无** 对应 migration，表由 model 声明 |
| `expense_settlement_fee_import_batch` 1—N `expense_settlement_fee_import` | hasMany `batch_id` | **无** 对应 migration |

## base

| 表 | 一句话 | 关键列（migration） | Model | migration |
|---|---|---|---|---|
| `profit_setting` | 利润检测设置表；另有配对 `*_log` | `status`（状态(0：禁用；1：启用)）, `rule_type`（规则类型(1: 系统促销sku设置; 2: 按订单销售额区间设置)）, `rule_name`（规则名称）, `order_minimum`（订单利润率最低值%）, `start_sell_price`（起始订单金额）, `close_sell_price`（结束订单金额）, `platform_id`（平台id）, `site`（国家/站点）, `shop_id`（店铺id） | `backend/models/base/ProfitSetting.php` | `console/migrations/m230601_015028_profit_setting.php` |
| `order_commission` | 平台佣金设置表；另有配对 `*_log` | `status`（状态(0：禁用；1：启用)）, `lowest_selling_price`（最低销售价(CNY)）, `highest_selling_price`（最高销售价(CNY)）, `commission_rate`（佣金比例%）, `buy_it_now_commission`（一口价佣金(CNY)）, `minimum_commission`（最低佣金(CNY)）, `additional_commission`（附加佣金(CNY)）, `platform_id`（平台id）, `site`（国家/站点）, `shop_id`（店铺id） | `backend/models/base/OrderCommission.php` | `console/migrations/m230601_015038_order_commission.php` |
| `vat_setting` | 销售VAT设置表；另有配对 `*_log` | `status`（状态(0：禁用；1：启用)）, `vat_type`（VAT类型(1: 按运输方式; 2: 按发货国家)）, `formula`（计算公式{1: 订单总额x税率; 2: 订单总额/(1+税率)* 税率}）, `platform_id`（平台id）, `site`（国家/站点）, `shop_id`（店铺id） | `backend/models/base/VatSetting.php` | `console/migrations/m230601_015045_vat_setting.php` |
| `config_setting` | 系统参数设置表；另有配对 `*_log` | `name`（配置名称）, `key`（键名）, `value`（值）, `desc`（备注信息）, `group`（分组:0=未分组;1=订单配置;2=退款/退货单据设置;）, `custom_conf`（自定义配置）, `index`（排序）, `type` | `backend/models/base/ConfigSetting.php` | `console/migrations/m230601_073446_create_config_setting.php` |
| `logistics_preferred` | 物流优选；另有配对 `*_log` | `status`（启用状态：0=禁用；1=启用）, `warehouse_type`（仓库类型）, `rule_name`（规则名称）, `priority_level`（优先级别）, `platform`（平台）, `sites`（站点）, `is_manual_bin_selection`（手动选仓）, `warehouse_list`（仓库列表）, `is_verify_inventory`（是否验证仓库类型：0=否;1=是）, `shipping_method_list`（运输方式）, `shipping_method_ids`（运输方式id）, `shipping_method_day`（运输方式规则多少天） | 待确认（本仓库无对应 Model） | `console/migrations/m230605_110738_create_logistics_preferred.php` |
| `order_strategy` | 订单操作策略表；另有配对 `*_log` | `order_status`（订单状态）, `product_status`（商品状态）, `change_order_status`（改变订单状态(订单新状态)）, `mark_shipping_status`（平台标记发货状态）, `status`（状态(0：禁用；1：启用)）, `platform_order_type`（平台订单类型）, `transport_type`（渠道类型运输方式(海陆空等)）, `strategy_name`（策略名称）, `priority`（策略优先级(数字越大优先级越高)）, `strategy_description`（策略说明）, `strategy_start_time`（策略开始时间）, `strategy_end_time`（策略结束时间） | `backend/models/base/OrderStrategy.php` | `console/migrations/m230606_011416_order_strategy.php` |
| `platform_label_strategy` | 平台标发策略；另有配对 `*_log` | `order_status`（订单状态）, `status`（启用状态）, `strategy_name`（平台标发策略名称）, `priority_level`（平台标发等级）, `platform`（平台）, `sites`（站点）, `shop`（店铺）, `country`（国家）, `shipping_warehouse`（发货仓库）, `shipping_method`（运输方式）, `product_sku`（产品sku）, `order_amount_currency`（订单金额币种） | `backend/models/base/PlatformLabelStrategy.php` | `console/migrations/m230606_085603_create_platform_label_strategy.php` |
| `pre_generation_track_num` | 预生成追踪号策略；另有配对 `*_log` | `status`（启用状态）, `time_type`（执行时间类型1=添加时间；2=支付时间；3=剩余发货时间）, `rule_name`（规则名称）, `priority_level`（平台标发等级）, `platform`（平台）, `sites`（站点）, `shop`（店铺）, `shipping_warehouse_list`（仓库）, `shipping_method_list`（运输渠道）, `is_in_stock`（是否必须有库存）, `is_execution_time`（执行时间）, `execution_time`（添加时间；执行时间；最后发货时间n小时后预报） | 待确认（本仓库无对应 Model） | `console/migrations/m230607_012242_create_pre_generation_track_num.php` |
| `order_source_channel` | 订单来源渠道表；另有配对 `*_log` | `status`（状态：0=禁用；1=启用；）, `seller_id`（店铺sellerId）, `sander_tax_id`（寄件人税号（IOSS））, `invoice_template_id`（订单发票模板）, `platform`（平台）, `platform_id`（平台id）, `site`（站点/国家）, `site_id`（站点id）, `shop`（店铺代号）, `shop_id`（店铺id）, `real_shop_name`（店铺真实名称）, `shipping_time`（发货时效（天）） | `backend/models/base/OrderSourceChannel.php` | `console/migrations/m230612_035406_order_source_channel.php` |
| `order_source_agent` | 订单渠道来源-代理信息表 | `source_channel_id`（订单渠道来源主表id(order_source_channels_id)）, `country_id`（国家id）, `custom_tags_id`（自定义标签id）, `tag_type`（标签类型:0=殴代;1=英代）, `company_name`（公司名称）, `country`（国家）, `province`（省份）, `city`（城市）, `region`（地区）, `zip_code`（邮编）, `detailed_address`（详细地址）, `is_default`（是否默认标签0=默认；1=自定义标签） | `backend/models/base/OrderSourceAgent.php` | `console/migrations/m230612_035427_order_source_agent.php` |
| `order_source_shop_address` | 订单来源渠道-店铺发货-发票地址 | `source_channel_id`（订单渠道来源主表id(order_source_channels_id)）, `source_shop_id`（店铺地址来源店铺id）, `country_id`（国家id）, `address_type`（地址类型:0=店铺发货地址 1=店铺发票地址）, `shipping_type`（运输类型0=未选；1=空运；2=海运；3=陆运；4=快递）, `name`（姓名）, `preferred_address`（街道地址1）, `alternate_address`（街道地址2）, `city`（城市）, `district`（区/县）, `province`（省份）, `country`（国家） | `backend/models/base/OrderSourceShopAddress.php` | `console/migrations/m230612_035456_order_source_shop_address.php` |
| `order_blacklist` | 订单黑名单 | `order_sn`（来源订单）, `status`（状态: 1=启用；2=禁用）, `recipient_country_id`（收件人国家id）, `recipient_name`（收件人姓名）, `recipient_email`（收件人邮箱）, `recipient_phone`（收件人电话号码）, `recipient_province`（收件人省/州）, `recipient_city`（收件人城市）, `recipient_door_num`（门牌号）, `recipient_address`（收件人详细地址）, `reason`（拉黑原因）, `platform_id`（平台id） | `backend/models/base/OrderBlacklist.php` | `console/migrations/m230804_083246_create_order_blacklist.php` |
| `order_wms_preferred` | 订单分仓物流优选表 | `status`（状态: 1=启用；2=禁用）, `choose_type`（物流优选类型：1按价格匹配 2按时效最快匹配）, `rule_name`（规则名称）, `priority`（优先级，数字越大优先级越高）, `warehouse`（发货仓库）, `is_verify_inventory`（是否验证库存：0否 1验证）, `shipping`（运输方式）, `limitation_days`（时效天数限制）, `platform_id`（平台id）, `site`（国家/站点） | `backend/models/base/OrderWmsPreferred.php` | `console/migrations/m230816_093941_create_order_wms_preferred.php` |

## order

| 表 | 一句话 | 关键列（migration） | Model | migration |
|---|---|---|---|---|
| `order` | 订单主表 | `parent_order_id`（父级订单id）, `parent_order_sn`（系统父级订单号）, `order_sn`（系统订单号）, `platform_order_sn`（平台订单号）, `order_status`（订单状态）, `pay_status`（付款状态；0=未付款；1=已付款）, `shipping_status`（发货状态：0=未发货；1=已发货）, `settlement_status`（结算状态:  0=未结算；1=已结算）, `refund_status`（退款状态:  0=未退款;1=部分退款；2=全部退款）, `mark_shipment_status`（1 未标记 2标记中 3已标记 4标记失败 5 不标记）, `original_id`（平台订单id）, `platform_shipping_method_id`（平台运输方式） | `backend/models/order/Order.php` | `console/migrations/m230616_025436_create_order_table.php` |
| `order_logistics` | 订单物流信息表 | `order_id`（订单id）, `bill_status`（对账状态 1 未对账 2 已对账）, `ship_warehouse_id`（发货仓库id）, `shipping_method_id`（发货运输方式id）, `logistics_id`（物流商ID）, `logistics_type`（发货类型【海陆空】）, `ship_warehouse_name`（发货仓库名称）, `transport_name`（运输方式名称）, `parcel_no`（包裹单号）, `overseas_wms_out_number`（海外仓出库单号）, `real_track_no`（真实追踪号）, `logistics_bill_num`（物流商单号） | `backend/models/order/OrderLogistics.php` | `console/migrations/m230616_034710_create_order_logistics_table.php` |
| `order_payment` | 订单支付信息 | `order_id`（订单id）, `pay_status`（付款状态；0=未付款；1=已付款）, `settlement_status`（结算状态:  0=未结算；1=已结算）, `payment_type`（订单支付类型）, `transaction_no`（交易号）, `beneficiary_bank`（支付银行卡）, `payer`（付款人）, `payment_mode`（订单支付方式）, `paid_at`（付款时间）, `pay_amount`, `into_amount` | `backend/models/order/OrderPayment.php` | `console/migrations/m230616_055739_create_order_payment_table.php` |
| `black_list` | 黑名单 | `status`（状态:0=禁用；1=启用）, `platform`（平台 (为空表示所有平台通用)）, `name`（平台 (为空表示所有平台通用)）, `email`（邮箱）, `phone`（电话号码）, `country`（国家）, `province`（省/州）, `city`（城市）, `door_num`（门牌号）, `full_address`（详细地址）, `order_source`（订单来源）, `black_reason`（加入黑名单原因） | `backend/models/order/BlackList.php` | `console/migrations/m230616_060552_create_black_list_table.php` |
| `order_item` | 订单sku | `order_id`（订单主表id(order_id)）, `sales_status`（销售状态）, `order_item_id`（数据原平台id）, `brand_type`（侵权类型）, `spu`（系统spu）, `sku`（系统sku）, `custom_sku`（自定义sku）, `i_robot_box_sku`（赛盒sku）, `platform_sku`（渠道sku）, `asin`（ASIN）, `main_img`（主图）, `sku_cn_title`（sku中文标题） | `backend/models/order/OrderItem.php` | `console/migrations/m230616_061326_create_order_item_table.php` |
| `order_recipient` | 收件人表 | `order_id`（订单主表id(order_id)）, `order_sn`（订单号）, `clientele_id`（客户id）, `recipient_country_id`（收件人国家id）, `recipient_tax_id`（收件人税号）, `buyer_email`（买家邮箱）, `buyer_name`（买家姓名）, `buyer_country`（买家国家）, `recipient_name`（收件人姓名）, `recipient_last_name`（收件人姓氏）, `recipient_email`（收件人邮箱）, `recipient_phone`（收件人电话号码） | `backend/models/order/OrderRecipient.php` | `console/migrations/m230616_063301_create_order_recipient_table.php` |
| `order_sender` | 寄件人表 | `order_id`（订单主表id(order_id)）, `sender_country_id`（寄件人国家）, `sender_tax_id`（寄件人税号）, `sender_name`（寄件人姓名）, `sender_last_name`（寄件人姓氏）, `sender_email`（寄件人邮箱）, `sender_phone`（寄件人电话号码）, `sender_company`（寄件单位）, `sender_country`（寄件人国家）, `sender_province`（寄件人省/州）, `sender_city`（寄件人城市）, `sender_region`（寄件人区/县） | `backend/models/order/OrderSender.php` | `console/migrations/m230616_063318_create_order_sender_table.php` |
| `state_machines_history` | 状态机历史记录 | `business_id`（租户编号）, `name`（状态名称）, `old_state`（旧状态）, `old_state_name`（旧状态名称）, `new_state`（新状态）, `new_state_name`（新状态名称） | `backend/models/order/StateMachinesHistory.php` | `console/migrations/m230705_075143_create_state_machines_history_table.php` |
| `order_item_assemble` | 订单sku组合 | `order_id`（订单主表id(order_id)）, `product_status`（商品状态）, `order_item_id`（数据原平台id）, `spu`（系统spu）, `sku`（系统sku）, `i_robot_box_sku`（赛盒sku）, `custom_sku`（自定义sku）, `platform_sku`（渠道sku）, `asin`（ASIN）, `main_img`（主图）, `sku_cn_title`（sku中文标题）, `sku_en_title`（sku英文标题） | `backend/models/order/OrderItemAssemble.php` | `console/migrations/m230828_035748_create_order_item_assemble_table.php` |
| `platform_shipping_type` | 平台运输方式 | `shipping_service_id`（运输服务ID）, `site`（站点）, `description`（运输类型介绍）, `shipping_carrier`（发货承运人）, `shipping_service`（运输服务）, `international_service`（国际服务）, `platform_id`（平台id） | `backend/models/order/ShippingType.php` | `console/migrations/m230921_071731_create_platform_shipping_type_table.php` |
| `order_mark_shipment_record` | 标发记录；`$table` 未声明，从表名推断 | `order_sn`（系统订单号）, `status`（状态）, `platform`（订单平台）, `marking_method`（标发方式）, `marking_params`（标发参数） | `backend/models/order/OrderMarkShipmentRecord.php` | `console/migrations/m231025_031727_order_mark_shipment_record.php` |

## expense

| 表 | 一句话 | 关键列（migration） | Model | migration |
|---|---|---|---|---|
| `custom_cost_item` | 自定义费用项主表；另有配对 `*_log` | `status`（状态: 0=禁用：1=启用）, `shipping_warehouse_id`（发货仓库id）, `fee_type`（费用类型）, `order_shipping_type`（订单发货类型 0=请选择; 1=本地直发包裹（客户订单）; 2=海外仓直发订单（客户订单）; 3=平台仓直发订单（客户订单））, `shipping_type`（发货类型 0=按仓库名称1=按仓库类型）, `warehouse_type`（仓库类型）, `fee_description`（费用说明）, `fee_type_uniq`（订单费用映射表.unique_key）, `priority`（优先级）, `shipping_warehouse`（发货仓库名称）, `platform_id`（平台id）, `site_id`（站点） | `backend/models/base/CustomCostItem.php` | `console/migrations/m230612_030435_custom_cost_item.php` |
| `order_expense_report` | 订单费用 | `order_id`（订单主表id(order_id)）, `order_sn`（系统订单号）, `platform_order_sn`（平台订单号）, `original_order_id`（原平台订单id）, `order_item_id`（数据原平台id）, `logical_type`（订单逻辑类型）, `platform`（平台）, `platform_id`（平台id）, `site`（站点）, `site_id`（站点id）, `shop`（店铺）, `shop_id`（店铺id） | `backend/models/order/OrderExpenseReport.php` | `console/migrations/m230616_062342_create_order_expense_report_table.php` |
| `expense_customize_detail` | 费用自定义详情 | `import_id`（导入id）, `order_sn`（系统订单号）, `unique_key`（唯一标识）, `financial_type`（财务类型）, `platform`（平台）, `platform_id`（平台id）, `site`（站点）, `site_id`（站点id）, `shop`（店铺）, `shop_id`（店铺id）, `sales_id`（销售id）, `type`（费用类型） | `backend/models/expense/ExpenseCustomizeDetail.php` | `console/migrations/m230724_080730_create_expense_customize_detail_table.php` |
| `expense_customize_import` | 费用自定义导入 | `status`（状态）, `file_name`（导入文件名）, `file_url`（文件路径）, `remark`（备注）, `review_at`（审核时间）, `reviewer_id`（审核人） | `backend/models/expense/ExpenseCustomizeImport.php` | `console/migrations/m230724_080922_create_expense_customize_import_table.php` |
| `order_expense_field_map` | 订单费用映射 | `expense_type`（费用类型）, `financial_type`（财务类型 1 收入 2 支出）, `unique_key`（费用唯一标识）, `process_type`（处理类型）, `platform_id`（平台id）, `assemble`（费用大项）, `assemble_code`（费用大项编码）, `segmentation`（费用细分）, `segmentation_code`（费用细分编码）, `extremity`（费用末端细分）, `extremity_code`（费用末端细分编码）, `discern`（特殊识别码） | `backend/models/expense/OrderExpenseFieldMap.php` | `console/migrations/m230816_061142_create_order_expense_field_map_table.php` |
| `order_expense_stats` | 订单费用统计；库 `xc_oms_dwh` | `order_sn`（系统订单号）, `platform_order_sn`（平台订单号）, `order_status`（订单状态）, `shipping_warehouse_id`（发货仓库id）, `platform`（平台）, `platform_id`（平台id）, `site`（站点）, `site_id`（站点id）, `shop`（店铺）, `shop_id`（店铺id）, `sales_id`（销售id）, `order_currency`（币种） | 待确认（本仓库无对应 Model） | `console/migrations/xc_oms_dwh/m230802_054303_create_order_expense_stats_table.php` |

## internal

本域 **没有** 独立 migration 表。

WMS/TMS/客服回调读写 `order` / `order_item` / `order_logistics` 等订单聚合，以及费用相关表。详见 [../02-surfaces/api-list.md](../02-surfaces/api-list.md) internal 节。

## refund

| 表 | 一句话 | 关键列（migration） | Model | migration |
|---|---|---|---|---|
| `order_refund_reason` | 订单退款原因管理 | `parent_id`（父级退款原因id）, `platform_id`（平台id）, `reason`（退款原因）, `platform`（所属平台）, `is_enable`（是否启用 0 未启用 1 启用） | `backend/models/refund/OrderRefundReason.php` | `console/migrations/m230710_084527_create_order_refund_reason_table.php` |
| `order_refund` | 订单退款 | `order_id`（订单id）, `exchange_sn`（退换货单号）, `refund_status`（退款状态）, `api_refund_status`（api退款状态）, `primary_cause_id`（一级退款原因）, `secondary_cause_id`（二级退款原因）, `return_warehouse_id`（退件仓库）, `refund_type`（退款类型）, `dispute_type`（纠纷类型）, `platform`（平台）, `platform_id`（平台id）, `site`（站点） | `backend/models/refund/OrderRefund.php` | `console/migrations/m230711_024618_create_order_refund_table.php` |
| `order_refund_products` | 订单退款商品信息 | `order_id`（订单id）, `order_refund_id`（退款单id）, `refund_status`（退款状态）, `order_item_id`（数据原平台id）, `sku`（系统sku）, `platform_sku`（渠道sku）, `spu`（系统spu）, `asin`（ASIN）, `custom_sku`（自定义sku）, `sku_cn_title`（sku中文标题）, `main_img`（主图）, `quantity`（数量） | `backend/models/refund/OrderRefundProducts.php` | `console/migrations/m230711_024630_create_order_refund_products_table.php` |

## export

本域 **没有** 独立 migration 表。

异步导出读利润/SKU/退款等已有表，无独立业务表。

## report

| 表 | 一句话 | 关键列（migration） | Model | migration |
|---|---|---|---|---|
| `performance_indicator` | 业绩指标；另有配对 `*_log`；model 表名是 `order_performance_indicator`，与 migration 不一致，待确认 | `status`（状态(0：禁用；1：启用)）, `currency_id`（币种ID）, `department_user_id`（部门人员id）, `indicator_type`（指标类型:0=请选择；1=销售额；2=产品毛利；3=销售毛利；4=产品毛利率；5=销售毛利率；6=已售商品成本）, `currency`（币种）, `start_time`（时间范围-开始时间）, `end_time`（时间范围-结束时间）, `department_name`（部门名称）, `desired_target` | `backend/models/base/OrderPerformanceIndicator.php` | `console/migrations/m230701_032931_create_performance_indicator.php` |
| `performance_statement` | 业绩报表 | `performance_indicator_id`（业绩指标主表id）, `department_user_id`（部门人员id）, `user_id`（人员id）, `indicator_type`（指标类型:0=请选择；1=销售额；2=产品毛利；3=销售毛利；4=产品毛利率；5=销售毛利率；6=已售商品成本）, `currency`（币种）, `start_time`（时间范围-开始时间）, `end_time`（时间范围-结束时间）, `department_name`（部门名称）, `individual_target`, `individual_achievement`, `achievement_rate`, `average_daily_target` | 待确认（本仓库无对应 Model） | `console/migrations/m230701_035241_create_performance_statement.php` |

## saihe

本域 **没有** 独立 migration 表。

赛盒订单/退款走 `backend/modules/v1/controllers/saihe/`，历史数据走 shdb（外部库，本仓库无 migration）。Mongo 有 `SaiheWarehouse` 等，见文末。

## exchange

| 表 | 一句话 | 关键列（migration） | Model | migration |
|---|---|---|---|---|
| `order_exchange` | 订单退换货 | `order_id`（订单id）, `exchange_sn`（退换货单号）, `replacement_order_sn`（换货订单号）, `storage_sn`（入库单号）, `exchange_status`（退换货状态）, `storage_status`（入库状态）, `replacement_order_id`（换货订单）, `exchange_warehouse_id`（退回仓库id）, `exchange_type`（售后类型）, `platform`（平台）, `platform_id`（平台id）, `site`（站点） | `backend/models/exchange/OrderExchange.php` | `console/migrations/m230714_022039_create_order_exchange_table.php` |
| `order_exchange_products` | 订单退换货产品 | `order_exchange_id`（退换货单id）, `order_item_id`（订单item id）, `problem_type`（问题类型）, `spu`（系统spu）, `sku`（sku）, `custom_sku`（自定义sku）, `platform_sku`（渠道sku）, `sku_cn_title`（sku中文标题）, `asin`（ASIN）, `quantity`（退换数量）, `problem_description`（问题描述）, `situation`（产品情况） | `backend/models/exchange/OrderExchangeProducts.php` | `console/migrations/m230714_022051_create_order_exchange_products_table.php` |

## subscribe

| 表 | 一句话 | 关键列（migration） | Model | migration |
|---|---|---|---|---|
| `data_arrival` | 数据到达记录 | `unique_key`（唯一keY）, `name`（数据名称）, `body`（数据内容） | `backend/models/base/DataArrival.php` | `console/migrations/m230901_035603_create_data_arrival_table.php` |
| `data_subscribe` | 数据订阅 | `name`（数据名称）, `event`（数据触发事件） | `backend/models/base/DataSubscribe.php` | `console/migrations/m230901_040009_create_data_subscribe_table.php` |

## log

本域 **没有** 独立 migration 表。

业务日志走 `BusinessLogRecorder` / Trait，部分主表有配对 `*_log`（见 BaseMigration::createLogTable）。HTTP 仅 `GET /log/business`。

## 有 Model、无本仓库 migration 的表/集合（索引）

这些对象**存在** `tableName()`，但 `console/migrations` 里没有建表文件。
可能来自未入库的 SQL、其它库、或 Mongo/CH 集合。**禁止把下列字段当契约。**

### folder `amazon`（5）

| tableName | Model |
|---|---|
| `amazon_deal_sku` | `backend/models/amazon/AmazonDealSku.php` |
| `amazon_financial_node_seek` | `backend/models/amazon/AmazonFinancialNodeSeek.php` |
| `amazon_purchase_order` | `backend/models/amazon/AmazonPurchaseOrder.php` |
| `amazon_purchase_order_item` | `backend/models/amazon/AmazonPurchaseOrderItem.php` |
| `amazon_storage_fee_rule` | `backend/models/amazon/AmazonStorageFeeRule.php` |

### folder `base`（10）

| tableName | Model |
|---|---|
| `api_call_log` | `backend/models/base/ApiCallLog.php` |
| `kpi_achievement_report` | `backend/models/base/KpiAchievementReport.php` |
| `order_channel_sku_commission` | `backend/models/base/OrderChannelSkuCommission.php` |
| `order_performance_indicator` | `backend/models/base/OrderPerformanceIndicator.php` |
| `order_performance_indicator_detail` | `backend/models/base/OrderPerformanceIndicatorDetail.php` |
| `order_split_rule` | `backend/models/base/OrderSplitRule.php` |
| `order_strategy_log` | `backend/models/base/OrderStrategyLog.php` |
| `order_strategy_match_record` | `backend/models/base/OrderStrategyMatchRecord.php` |
| `platform_site_link` | `backend/models/base/PlatformSiteLink.php` |
| `platform_site_time_diff` | `backend/models/base/PlatformSiteTimeDiff.php` |

### folder `exchange`（1）

| tableName | Model |
|---|---|
| `order_exchange_auth` | `backend/models/exchange/OrderExchangeAuth.php` |

### folder `expense`（24）

| tableName | Model |
|---|---|
| `amazon_storage_fee_estimate` | `backend/models/expense/AmazonStorageFeeEstimate.php` |
| `expense_calculate_strategy_detail` | `backend/models/expense/ExpenseCalculateStrategyDetail.php` |
| `expense_close` | `backend/models/expense/ExpenseClose.php` |
| `expense_customize_import_report` | `backend/models/expense/ExpenseCustomizeImportReport.php` |
| `expense_dept_charge` | `backend/models/expense/ExpenseDeptCharge.php` |
| `expense_general_charge` | `backend/models/expense/ExpenseGeneralCharge.php` |
| `expense_replenish` | `backend/models/expense/ExpenseReplenish.php` |
| `expense_rule` | `backend/models/expense/ExpenseRule.php` |
| `expense_rule_condition` | `backend/models/expense/ExpenseRuleCondition.php` |
| `expense_rule_strategy` | `backend/models/expense/ExpenseRuleStrategy.php` |
| `expense_settlement_fee_import` | `backend/models/expense/ExpenseSettlementFeeImport.php` |
| `expense_settlement_fee_import_batch` | `backend/models/expense/ExpenseSettlementFeeImportBatch.php` |
| `expense_sku_charge` | `backend/models/expense/ExpenseSkuCharge.php` |
| `expense_warehouse_charge` | `backend/models/expense/ExpenseWarehouseCharge.php` |
| `product_dev_commission_result` | `backend/models/expense/ProductDevCommissionResult.php` |
| `product_dev_commission_setting` | `backend/models/expense/ProductDevCommissionSetting.php` |
| `product_dev_commission_task` | `backend/models/expense/ProductDevCommissionTask.php` |
| `temu_buyer_refusal` | `backend/models/expense/TemuBuyerRefusal.php` |
| `temu_income` | `backend/models/expense/TemuIncome.php` |
| `temu_refund` | `backend/models/expense/TemuRefund.php` |
| `temu_shipping_income` | `backend/models/expense/TemuShippingIncome.php` |
| `temu_shipping_refund` | `backend/models/expense/TemuShippingRefund.php` |
| `temu_shipping_refund_order` | `backend/models/expense/TemuShippingRefundOrder.php` |
| `temu_violation` | `backend/models/expense/TemuViolation.php` |

### folder `irb`（2）

| tableName | Model |
|---|---|
| `irb_order_balance_record` | `backend/models/irb/IrbOrderBalanceRecord.php` |
| `irb_sales_account` | `backend/models/irb/IrbSalesAccount.php` |

### folder `olap`（7）

| tableName | Model |
|---|---|
| `ads_finance_shipping_time_amazon_last_30_days_expense_stats_d` | `backend/models/olap/AdsFinanceShippingTimeAmazonLast30DaysExpenseStatsD.php` |
| `ads_finance_shop_sku_refund_rate_stats_d` | `backend/models/olap/AdsFinanceShopSkuRefundRateStatsD.php` |
| `ads_finance_sku_3m_expense_stats_d` | `backend/models/olap/AdsFinanceSkuExpenseStats.php` |
| `ads_finance_sku_expense_stats_split_sku_h` | `backend/models/olap/SkuGrossMarginMetrics.php` |
| `ads_finance_wms_shop_sku_30_90_days_stats_d` | `backend/models/olap/AdsFinanceWmsShopSku3090DaysStatsD.php` |
| `ads_order_order_time_sales_statistics_5m_view` | `backend/models/olap/SalesStatistics.php` |
| `ads_supplychain_warehouse_all_process_inventories_d` | `backend/models/olap/WarehouseSnapshot.php` |

### folder `order`（7）

| tableName | Model |
|---|---|
| `order_email_notice` | `backend/models/order/OrderEmailNotice.php` |
| `order_exclude` | `backend/models/order/OrderExclude.php` |
| `order_item_financial` | `backend/models/order/OrderItemFinancial.php` |
| `order_multi_channel_ship` | `backend/models/order/OrderMultiChannelShip.php` |
| `order_return_reason` | `backend/models/order/OrderReturnReason.php` |
| `order_upload_out_number` | `backend/models/order/OrderUploadOutNumber.php` |
| `order_upload_trace_number` | `backend/models/order/OrderUploadTraceNumber.php` |

### folder `refund`（1）

| tableName | Model |
|---|---|
| `order_refund_auth` | `backend/models/refund/OrderRefundAuth.php` |

## Mongo collection（索引，非 MySQL）

来自 `backend/models/mongo/*::collectionName()`，共 85 个。平台原始单/广告/仓储费/流水，**无本仓库 migration。**

| collection | Model |
|---|---|
| `aliexpress_order` | `backend/models/mongo/AliexpressOrder.php` |
| `aliexpress_order_detail` | `backend/models/mongo/AliexpressOrderDetail.php` |
| `aliexpress_transactions` | `backend/models/mongo/AliexpressTransactions.php` |
| `allegro_order` | `backend/models/mongo/AllegroOrder.php` |
| `amazon_advert_record` | `backend/models/mongo/AmazonAdvertRecord.php` |
| `amazon_creator_ads` | `backend/models/mongo/AmazonCreatorAds.php` |
| `amazon_doc_record` | `backend/models/mongo/AmazonDocRecord.php` |
| `amazon_finance` | `backend/models/mongo/AmazonFinance.php` |
| `amazon_invoice` | `backend/models/mongo/AmazonInvoice.php` |
| `amazon_invoice_detail` | `backend/models/mongo/AmazonInvoiceDetail.php` |
| `amazon_order` | `backend/models/mongo/AmazonOrder.php` |
| `amazon_order_item` | `backend/models/mongo/AmazonOrderItem.php` |
| `amazon_purchase_order` | `backend/models/mongo/AmazonPurchaseOrder.php` |
| `amazon_purchase_order_status` | `backend/models/mongo/AmazonPurchaseOrderStatus.php` |
| `amazon_report_create_record` | `backend/models/mongo/AmazonReportCreateRecord.php` |
| `amazon_return_the_goods` | `backend/models/mongo/AmazonReturnTheGoods.php` |
| `amazon_sb_advert` | `backend/models/mongo/AmazonSbAdvert.php` |
| `amazon_sb_advert_detail` | `backend/models/mongo/AmazonSbAdvertDetail.php` |
| `amazon_sb_traffic` | `backend/models/mongo/AmazonSbTraffic.php` |
| `amazon_sd_advert` | `backend/models/mongo/AmazonSdAdvert.php` |
| `amazon_sd_traffic` | `backend/models/mongo/AmazonSdTraffic.php` |
| `amazon_shipment` | `backend/models/mongo/AmazonShipment.php` |
| `amazon_sp_advert` | `backend/models/mongo/AmazonSpAdvert.php` |
| `amazon_sp_traffic` | `backend/models/mongo/AmazonSpTraffic.php` |
| `amazon_sqs` | `backend/models/mongo/AmazonSqs.php` |
| `amazon_storage_fee` | `backend/models/mongo/AmazonStorageFee.php` |
| `amazon_storage_long_fee` | `backend/models/mongo/AmazonStorageLongFee.php` |
| `amazon_storage_overage_fee` | `backend/models/mongo/AmazonStorageOverageFee.php` |
| `amazon_traffic_report` | `backend/models/mongo/AmazonTrafficReport.php` |
| `amazon_transaction_item` | `backend/models/amazon/AmazonTransactionItem.php` |
| `amazon_vendor_sales` | `backend/models/mongo/AmazonVendorSalesReport.php` |
| `daily_department_data` | `backend/models/mongo/DailyDepartmentData.php` |
| `daily_department_stat_data` | `backend/models/mongo/DailyDepartmentStatData.php` |
| `daily_sales_data` | `backend/models/mongo/DailySalesData.php` |
| `daily_sales_stat_data` | `backend/models/mongo/DailySalesStatData.php` |
| `ebay_order` | `backend/models/mongo/EbayOrder.php` |
| `ebay_order_supplement` | `backend/models/mongo/EbayOrderSupplement.php` |
| `ebay_transaction` | `backend/models/mongo/EbayTransaction.php` |
| `excel_column_snapshot` | `backend/models/mongo/ExcelColumnSnapshot.php` |
| `file_parse_record` | `backend/models/mongo/FileParseRecord.php` |
| `kaufland_order` | `backend/models/mongo/KauflandOrder.php` |
| `lazada_order` | `backend/models/mongo/LazadaOrder.php` |
| `lazada_order_items` | `backend/models/mongo/LazadaOrderItems.php` |
| `lazada_transactions` | `backend/models/mongo/LazadaTransactions.php` |
| `lingxing_amazon_order` | `backend/models/mongo/LingXingAmazonOrder.php` |
| `lingxing_amazon_order_item` | `backend/models/mongo/LingXingAmazonOrderItem.php` |
| `lingxing_amazon_receivable_report` | `backend/models/mongo/LingXingAmazonReceivableReport.php` |
| `order_distribute_sku_stat` | `backend/models/mongo/OrderDistributeSkuStat.php` |
| `order_expense_item_stats` | `backend/models/mongo/OrderExpenseItemStats.php` |
| `order_expense_item_stats` | `backend/models/mongo/SyncShopTag.php` |
| `order_expense_shop_stats` | `backend/models/mongo/OrderExpenseShopStats.php` |
| `order_expense_sku_stats` | `backend/models/mongo/OrderExpenseSkuStats.php` |
| `order_expense_stats` | `backend/models/mongo/OrderExpenseStats.php` |
| `order_expense_stats_test` | `backend/models/mongo/OrderExpenseStatsTest.php` |
| `order_import` | `backend/models/mongo/OrderImport.php` |
| `order_refund_import` | `backend/models/mongo/OrderRefundImport.php` |
| `order_sku_sales_volume_statistics` | `backend/models/mongo/OrderSkuSalesVolumeStatistics.php` |
| `otto_order` | `backend/models/mongo/OttoOrder.php` |
| `otto_receipts` | `backend/models/mongo/OttoReceipts.php` |
| `platform_expense_mismatch` | `backend/models/mongo/PlatformExpenseMismatch.php` |
| `saihe_order` | `backend/models/mongo/SaiheOrder.php` |
| `saihe_order_log` | `backend/models/mongo/SaiheOrderLog.php` |
| `saihe_order_refund` | `backend/models/mongo/SaiheOrderRefund.php` |
| `saihe_order_source_type` | `backend/models/mongo/SaiheOrderSourceType.php` |
| `saihe_warehouse` | `backend/models/mongo/SaiheWarehouse.php` |
| `shein_order` | `backend/models/mongo/SheinOrder.php` |
| `shein_order_detail` | `backend/models/mongo/SheinOrderDetail.php` |
| `shein_return_orders` | `backend/models/mongo/SheinReturnOrders.php` |
| `shop_commission_ratio` | `backend/models/mongo/ShopCommissionRatio.php` |
| `shopee_escrow_detail` | `backend/models/mongo/ShopeeEscrowDetail.php` |
| `shopee_escrow_list` | `backend/models/mongo/ShopeeEscrowList.php` |
| `shopee_order` | `backend/models/mongo/ShopeeOrder.php` |
| `temu_amount_list` | `backend/models/mongo/TemuAmountList.php` |
| `temu_order` | `backend/models/mongo/TemuOrder.php` |
| `temu_transaction` | `backend/models/mongo/TemuTransaction.php` |
| `tiktok_ads` | `backend/models/mongo/TiktokAds.php` |
| `tiktok_order` | `backend/models/mongo/TiktokOrder.php` |
| `tiktok_order_statement_transactions` | `backend/models/mongo/TiktokOrderStatementTransactions.php` |
| `tiktok_statement_transactions` | `backend/models/mongo/TiktokStatementTransactions.php` |
| `tiktok_warehouse` | `backend/models/mongo/TiktokWarehouse.php` |
| `unique_order_ids` | `backend/models/mongo/UniqueOrderIds.php` |
| `walmart_order` | `backend/models/mongo/WalmartOrder.php` |
| `walmart_order_temp` | `backend/models/mongo/WalmartOrderTemp.php` |
| `walmart_report` | `backend/models/mongo/WalmartReport.php` |
| `xingpan_order` | `backend/models/mongo/XingpanOrder.php` |

## Elasticsearch index（索引）

| index() | Model |
|---|---|
| `order` | `backend/models/es/OrderDocument.php` |
| `order_blacklist` | `backend/models/es/OrderBlacklistEs.php` |
| `order_refund` | `backend/models/es/OrderRefundEs.php` |
| `order_strategy` | `backend/models/es/OrderStrategyEs.php` |
| `order_wms_preferred` | `backend/models/es/OrderWmsPreferredEs.php` |

## 重要枚举（只写 migration comment 里能读到的）

| 表.列 | 取值（来自 comment） |
|---|---|
| `profit_setting.status` | 状态(0：禁用；1：启用) |
| `order_commission.status` | 状态(0：禁用；1：启用) |
| `vat_setting.status` | 状态(0：禁用；1：启用) |
| `config_setting.group` | 分组:0=未分组;1=订单配置;2=退款/退货单据设置; |
| `logistics_preferred.is_verify_inventory` | 是否验证仓库类型：0=否;1=是 |
| `logistics_preferred.is_lowest_price_match` | 最低价格匹配：0=否;1=是 |
| `logistics_preferred.is_fastest_time` | 最低时效匹配：0=否;1=是 |
| `logistics_preferred.status` | 启用状态：0=禁用；1=启用 |
| `order_strategy.status` | 状态(0：禁用；1：启用) |
| `platform_label_strategy.execution_condition` | 执行条件:0=未选择执行条件;1=在订单最晚发货时间 %s 小时标记发货; 2=真实发货后立刻标记发货;3=在订单的支付时间加 %s 小时标记发货 3=在订单的拉单时间加 %s 小时标记发货 |
| `platform_label_strategy.marking_method` | 标发方式 0=请选择标发方式;1=只上传真实追踪号;2=可上传本地追踪号;3=空追踪号标记发货;4=自行标记发货_ERP不操作;5=只上传本地追踪号; |
| `pre_generation_track_num.time_type` | 执行时间类型1=添加时间；2=支付时间；3=剩余发货时间 |
| `custom_cost_item.order_shipping_type` | 订单发货类型 0=请选择; 1=本地直发包裹（客户订单）; 2=海外仓直发订单（客户订单）; 3=平台仓直发订单（客户订单） |
| `custom_cost_item.shipping_type` | 发货类型 0=按仓库名称1=按仓库类型 |
| `custom_cost_item.sharing_rule` | 分摊规则 0=未选择；1=按产品体积分摊；2=按产品数量分摊；3=按产品毛重分摊 |
| `custom_cost_item.billing_method` | 计费方式 0=按产品个数；1=按包裹个数（订单个量） |
| `custom_cost_item.status` | 状态: 0=禁用：1=启用 |
| `order_source_channel.is_marge_orders_address` | 是否合并相同地址订单包裹0=不合并；1=合并 |
| `order_source_channel.status` | 状态：0=禁用；1=启用； |
| `order_source_agent.tag_type` | 标签类型:0=殴代;1=英代 |
| `order_source_agent.is_default` | 是否默认标签0=默认；1=自定义标签 |
| `order_source_shop_address.address_type` | 地址类型:0=店铺发货地址 1=店铺发票地址 |
| `order_source_shop_address.shipping_type` | 运输类型0=未选；1=空运；2=海运；3=陆运；4=快递 |
| `order_source_shop_address.is_default_address` | 是否默认地址：0=不是；1=是 |
| `order_source_shop_address.is_delete` | 是否删除(0: 否；1：是) |
| `order.order_type` | 订单类型:0=单品单件；1=单品多件；2多品多件 |
| `order.build_order_type` | 系统建单类型:0=系统建单；1=手动建单 |
| `order.new_order_reason` | 新建订单原因:0=补发订单；1=重发订单；2=其它 |
| `order.pay_status` | 付款状态；0=未付款；1=已付款 |
| `order.shipping_status` | 发货状态：0=未发货；1=已发货 |
| `order.settlement_status` | 结算状态:  0=未结算；1=已结算 |
| `order.refund_status` | 退款状态:  0=未退款;1=部分退款；2=全部退款 |
| `order.is_consolidate_order` | 是否合并订单：0不是：1=是合并订单 |
| `order.is_split_order` | 是否拆单:0=不是；1=是 |
| `order.is_logistics_reconciliation` | 是否物流对账:0=未对账；1=已对账 |
| `order_payment.pay_status` | 付款状态；0=未付款；1=已付款 |
| `order_payment.settlement_status` | 结算状态:  0=未结算；1=已结算 |
| `black_list.status` | 状态:0=禁用；1=启用 |
| `black_list.is_delete` | 是否删除 0=未删除；1=已删除 |
| `performance_indicator.indicator_type` | 指标类型:0=请选择；1=销售额；2=产品毛利；3=销售毛利；4=产品毛利率；5=销售毛利率；6=已售商品成本 |
| `performance_indicator.status` | 状态(0：禁用；1：启用) |
| `performance_statement.indicator_type` | 指标类型:0=请选择；1=销售额；2=产品毛利；3=销售毛利；4=产品毛利率；5=销售毛利率；6=已售商品成本 |
| `order_blacklist.status` | 状态: 1=启用；2=禁用 |
| `order_wms_preferred.status` | 状态: 1=启用；2=禁用 |

## 需人工确认

- migrations 之后的 alter **不在本仓库**，model 属性可能比 migration 新（`Order` 已有 `withdrawal_status` 等，migration 无）。
- `m231025_031727_order_mark_shipment_record` 未声明 `$table` / `$comment`，表名按文件名+Model 推断。
- `performance_indicator`（migration）vs `order_performance_indicator`（Model）表名不一致。
- `logistics_preferred`、`pre_generation_track_num`、`performance_statement` 有 migration、本仓库无 Model。
- `logistics_preferred` 与后来的 `order_wms_preferred` 是否并存/替代，待确认。
- `order_refund_reason` 的 `$modelDir` 写成 `order`，路由域是 refund。
- 费用中心大量表（关账、规则、结算导入、Temu* 等）只有 Model，没有 migration。
- IRB（`backend/models/irb/`）与 ClickHouse OLAP 表无 migration。
- Mongo / ES 只列集合名，结构未展开。

