# OMS AI 文档索引

- **system**：oms（信诚-OMS）
- **剖面**：yii2-advanced（`backend/` + `common/` + `console/`）
- **探测域**：base、order、expense、internal、refund、export、report、saihe、exchange、subscribe、log  
  来源：[01-domains/INDEX.md](01-domains/INDEX.md)
- **本页职责**：阅读顺序、覆盖率、缺口。项目定位与约定见根目录 [AGENTS.md](../../AGENTS.md)。

## 阅读顺序

新会话按这次序读，细节只跟链接，不要整页粘贴进上下文。

1. 根 [AGENTS.md](../../AGENTS.md) — 定位、分层、约定、怎么跑
2. [diagrams/architecture.svg](diagrams/architecture.svg) — 调用方 → runtime → v1 模块 → common → 存储 → 外部
3. [01-domains/INDEX.md](01-domains/INDEX.md) — 11 个域的入口路径
4. 按任务选表面：
   - 改 HTTP：[02-surfaces/api-list.md](02-surfaces/api-list.md)
   - 改 CLI / MQ：[02-surfaces/cli-and-consumers.md](02-surfaces/cli-and-consumers.md)
   - 改表 / Model：[03-deep-dives/data-model.md](03-deep-dives/data-model.md) + [diagrams/data-model-er.svg](diagrams/data-model-er.svg)
   - 追调用链：[03-deep-dives/layers.md](03-deep-dives/layers.md) → [03-deep-dives/flows/expense.md](03-deep-dives/flows/expense.md) → 已深挖 [tiktok-profit-calculate](03-deep-dives/flows/expense-tiktok-profit-calculate.md) / [amazon-expense-calculate](03-deep-dives/flows/expense-amazon-expense-calculate.md) + [diagrams/flows/](diagrams/flows/)
5. 需要边界时再看 [diagrams/module-deps.svg](diagrams/module-deps.svg)、[diagrams/external-deps.svg](diagrams/external-deps.svg)
6. 运维命令原文：[wiki/](../../wiki/)、[document/](../../document/)（不镜像到 agents）

## 覆盖率

| 契约路径 | 状态 | 摘要（来自该文件自身） |
|---|---|---|
| `01-domains/INDEX.md` | 有 | 11 域 + 3 条入口缺口 |
| `02-surfaces/api-list.md` | 有 | 301 条显式路由；主入参大量「待确认」 |
| `02-surfaces/cli-and-consumers.md` | 有 | 185 个 `php yii` action + 87 个 yml Consumer |
| `03-deep-dives/data-model.md` | 有 | 文首仍写 migrations 优先；与当前 skill「只扫 Model」不一致 |
| `diagrams/architecture.svg` | 有 | Yii2 Advanced · backend API + console Worker |
| `diagrams/module-deps.svg` | 有 | 本仓库目录边界；红色=循环依赖 |
| `diagrams/external-deps.svg` | 有 | PHP 包 / 中间件 / 外部 API |
| `diagrams/data-model-er.svg` | 有 | 随 data-model 生成，含 migration 对照信息 |
| `03-deep-dives/layers.md` | 本轮写入 | flows deep：tiktok-profit-calculate；F/L/V/X 命中 |
| `03-deep-dives/flows/expense.md` | 本轮写入 | 候选表；tiktok + amazon 已深挖 |
| `03-deep-dives/flows/expense-tiktok-profit-calculate.md` | 本轮 deep | 仅 `order_expense_calculate` Consumer 主链；精确条件、双分支前 12 算子、独立写库节点与失败路径 |
| `diagrams/flows/expense-tiktok-profit-calculate.svg` | 本轮 deep | Consumer→SYSTEM 前 12 算子→六个独立存储节点；异常补偿分支 |
| `diagrams/flows/expense-tiktok-profit-calculate-branches.svg` | 本轮 deep | SYSTEM/MANUAL 实际返回数组前 12 项逐节点对照 |
| `03-deep-dives/flows/expense-amazon-expense-calculate.md` | 已有 deep | Amazon V2 交易结算 |
| `diagrams/flows/expense-amazon-expense-calculate.svg` | 已有 deep | Amazon 结算细图 |
| `03-deep-dives/lifecycle.md` | 本轮回写 | can() 门禁 + BuildOrderType；Amazon 摘要保留 |
| `03-deep-dives/variants/INDEX.md` | 本轮回写 | profit Provider + TiktokAllOperatorChain |
| `03-deep-dives/contracts.md` | 本轮回写 | 利润 MQ / RedLock / notify |
| 根 `AGENTS.md` | 已有 | ≤300 行入口（本轮未重写） |

HTTP 条数（摘自 api-list / data-model）：base 103、order 52、expense 43、internal 42、refund 21、export 18、report 8、saihe 6、exchange 6、subscribe 1、log 1。

## 缺口

来自已有 `docs/agents/**`，本 mode **未扫全库**。

### 资产缺失

- 未双写 `CLAUDE.md`（未传 `dual_claude=true`）。
- 运维文保留在 `wiki/`、`document/`，不建 `99-ops/`。
- `flows/` deep：`expense-tiktok-profit-calculate`、`expense-amazon-expense-calculate`；其它域无链页。

### 文档自洽

- `data-model.md` 真源仍是 migrations，当前 `codex-map` skill 要求 **只扫 Model、不读 migrations**。改表前应再跑 `mode=data`，或以 `backend/models` 为准并人工核对。
- 上一轮用户要求「Model 优先、migration 仅对照」**尚未落地**到 `data-model.md`。

### 已有页内缺口（原文档已标）

- `OrderNotifyController`、`OrderSkuSalesStatisticsController` 无显式路由；`v2` 无 HTTP 入口。见 [01-domains/INDEX.md](01-domains/INDEX.md)。
- 很多 action 整包 `post()`，api-list 主入参为「待确认」。
- CLI `options()` 是控制器级合集，未按命令过滤。
- README 的 `order/re-cal-all-profit` 在 console **没有**对应 action。见 [02-surfaces/cli-and-consumers.md](02-surfaces/cli-and-consumers.md)。
- 部分 Consumer 职责为「待确认」。
- data-model：有表无 Model、表名不一致、费用中心大量仅 Model、Mongo/ES 未展开。见该页「需人工确认」。
- architecture：ClickHouse / TiDB 仅配置，业务引用待确认。

## 禁止

- 不要把 WMS/TMS/CMS 的域名套进本仓库。
- 不要编造 HTTP 方法、入参、表字段。
- 不要把 `docs/prompts/` 当产出真源（那是旧 prompt 包）。
