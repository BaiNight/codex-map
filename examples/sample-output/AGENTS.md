# 信诚 OMS — Agent 入口

信诚订单管理系统（oms）。Yii2 Advanced：`backend` HTTP、`console` CLI/Worker、`common` 共享。  
读图与清单请从 [docs/agents/INDEX.md](docs/agents/INDEX.md) 按顺序进入，**不要**把长清单贴进对话。

## 项目定位

拉单之后的系统订单、费用/利润、退款换货、标发与平台结算；对内给 WMS/TMS/客服回调，对外接多家电商平台与赛盒 IRB。

- 业务域（11）：[docs/agents/01-domains/INDEX.md](docs/agents/01-domains/INDEX.md)
- HTTP 路由最多的域是 **base**（配置），不是默认的「订单/费用」。条数见 INDEX。

## 核心架构

分层以 [docs/agents/diagrams/architecture.svg](docs/agents/diagrams/architecture.svg) 为准。

```text
调用方（ERP 前端 / 内部系统 / 赛盒 / SQS / 定时）
  → Nginx + PHP-FPM（docker: nginx-oms / php-oms）
  → backend API（app-backend，modules/v1；v2 已注册无 controllers）
  → controllers → service / consumer / strategy / task / slice → repository → models
  → common（platform / launcher / tools·es·db·state·aliyun）
  → 存储 + 外部平台 / 内部 ERP
并行：console + worker-oms（php yii + RabbitMQ Consumer）
```

存储（配置已声明）：MySQL `db`（主库）、`shdb`（赛盒 IRB）、`apdb`、`dwdb`；Mongo；ES；Redis×2；RabbitMQ；OSS。ClickHouse / TiDB 仅配置，引用待确认。

内部目录依赖与循环（红色）：[docs/agents/diagrams/module-deps.svg](docs/agents/diagrams/module-deps.svg)  
语言包 / 中间件 / 外部 API：[docs/agents/diagrams/external-deps.svg](docs/agents/diagrams/external-deps.svg)

## 关键模块

编排都在 `backend/modules/v1/`。职责一句话：

| 模块 | 职责 |
|---|---|
| order | 订单拉取、拆合、标发、状态 |
| expense | 费用分段、结算、自定义费用 |
| refund / exchange | 退款单、换货单 |
| export / report | 异步导出、利润与经营报表 |
| internal / subscribe | WMS/TMS 等回调；订单状态订阅 |
| saihe | 赛盒订单/退款（路由写在 order） |
| base | 策略、拆单、物流优选、KPI、渠道、VAT |
| log | 业务日志 `GET /log/business` |

`common/platform`：各平台拉单/检测/标发。`common/launcher`：HTTP/Nacos 调 xc_wms、xc_tms、xc_goods、xc_auth 等。

## 关键约定

- **路由**：键即路径，**没有** `/v1` 前缀；`enablePrettyUrl=true`，`showScriptName=false`。清单：[docs/agents/02-surfaces/api-list.md](docs/agents/02-surfaces/api-list.md)（301 条）。
- **面**：路径含 `internal` 为内部，其余对外。
- **通用入参**：`pageNum`、`pageSize`、`search_after`；Header `X-Tenant-Id` / `X-User-Id`。
- **通用返回**：`{status, code, message, data, total, timestamp, version}`。
- **推不出的入参/字段标「待确认」**，禁止编造。
- **CLI**：`php yii <controller-id>/<action-id>`，入口根目录 `yii`。185 个 action；`options()` 是控制器级合集。[docs/agents/02-surfaces/cli-and-consumers.md](docs/agents/02-surfaces/cli-and-consumers.md)
- **Consumer**：`common/config/app/rabbitmq/consumer.yml` 登记 87 个；队列名默认等于 consumer `name`。
- **数据**：现有 [docs/agents/03-deep-dives/data-model.md](docs/agents/03-deep-dives/data-model.md) 文首仍写 migrations 优先。当前 skill 要求以 `backend/models`（`tableName` / `rules` / `attributeLabels` / 关系方法）为准。改表前先对 Model，不要只信 migration 列清单。ER：[docs/agents/diagrams/data-model-er.svg](docs/agents/diagrams/data-model-er.svg)
- **无 DB 外键**，关联是 ActiveRecord 逻辑关联。多数 MySQL 表有 `id`、`tenant_id`、时间与用户列。
- **不要**把 OMS 域名单套到别的 ERP 仓库。

## 怎么跑

本套文档只记录入口形态，不收录完整安装。

| 入口 | 怎么跑 | 详见 |
|---|---|---|
| HTTP API | docker `nginx-oms` + `php-oms` → `backend` / v1 | architecture.svg |
| 定时 / 重算 | 仓库根 `php yii …`，Supervisor Worker | cli-and-consumers.md |
| MQ | `worker-oms` 消费 yml 里的 name | cli-and-consumers.md |

README 里的 `order/re-cal-all-profit` **没有**对应 console action，以 cli-and-consumers 为准。

运维原文：`wiki/`、`document/`（费用命令、拉单、ES 等）。

## 禁区

待团队补充。

## 历史包袱

待团队补充。

已知文档缺口（不是禁区，改代码前要看见）：

- 无路由：`OrderNotifyController`（`actionShip` 空）、`OrderSkuSalesStatisticsController`；`v2` 无 HTTP。
- 费用中心大量表只有 Model；部分 migration 无 Model；`performance_indicator` 与 Model 表名不一致。
- 运维命令原文仍在 `wiki/`、`document/`，不另建 agents 运维目录。
