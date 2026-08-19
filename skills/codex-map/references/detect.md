# 开跑前探测

## 开跑前：探测当前仓库

在任何 mode 的扫描之前先做探测，并写入本轮 summary。  
`mode=flows` / `mode=full` 必须做完第 5 节四层探测后再写深潜页。

### 1) 系统称谓

优先级：用户 `system=`（非 auto）→ README 标题 → `composer.json` 的 `name`/`description` → 仓库目录名。  
得到短名和一句话定位，供 overview / AGENTS.md 使用。

### 2) 技术栈剖面（选一，可组合）

按**当前仓库实际存在的路径**选择，不要假设 Yii2 Advanced：

| 剖面 | 判定（路径存在即命中） | 扫描重点 |
|---|---|---|
| `yii2-advanced` | `backend/` + `common/` + `console/`，或根目录有 `yii` | modules、controllers、console、models、config |
| `yii2-basic` | `config/web.php` + `controllers/` + `commands/` | 同上但无 backend/console 拆分 |
| `generic-php` | 有 `composer.json`，但不命中上面 | `src/`/`app/`、路由、CLI 入口 |
| `mixed` | 以上都不完整 | 用「已知路径 + 待确认」，禁止假完整 |

### 3) 业务域列表

从本仓库推导域名单。**写出 `docs/agents/01-domains/INDEX.md` 的时机见「写 01-domains/INDEX.md」**；禁止只把域名顿号连写成 `INDEX.md` 里的一行。

- `yii2-advanced`：`backend/modules/*` 下的业务子目录、controller 分组、`common/` 下明显限界目录
- 其它剖面：顶层业务包、路由前缀、module 名

`domain=` 必须是探测列表中的一项；用户给了不存在的域 → 列出合法域并只做索引，不编造。

### 4) 扫描真源（按剖面填，空列标「本仓库无」）

| 产出 | 优先读取（按命中路径取交集） |
|---|---|
| 依赖与目录边界 | `composer.json`、实际存在的 `backend/` `common/` `console/` `app/` `src/` `modules/` |
| 运行配置 | `common/config/*`、`backend/config/*`、`config/*`、env、`docker/`、`supervisord/` |
| HTTP | 实际 controllers 目录 + 路由配置（Yii 常见 `backend/modules/*/controllers`、`backend/config/routes*`） |
| 异步/运维入口 | `console/controllers`、`commands/`、consumer 类、MQ 配置（没有则文档写「本仓库无 CLI/Consumer」） |
| 数据模型 | 本仓库的 Model 目录（Yii 常见 `backend/models`，其它剖面用同类路径）：`tableName()`、`rules()`、`attributeLabels()`、关系方法。**不要读、不要引用 migrations** |

架构分层按探测结果画，不要写死某一套分层（例如「调用方 → API → modules」）。有则画，无则省略并在 summary 说明。

中间件/存储只画配置里**实际出现**的（MySQL/Redis/MQ/ES/Mongo 等），不要把别的系统的中间件抄过来。

### 5) 四层探测（F / L / V / X）

与业务无关。只根据**当前仓库路径信号**判定；命中才深写。  
禁止用其它项目的业务词当层名或默认标题。

| 层 | 代号 | 含义 | 命中信号（目录或文件名存在即候选） | 必须再确认 |
|---|---|---|---|---|
| 编排 | **F** | 入口到落库谁调谁 | `service/` `handler/` `task/` `slice/`（及同义：`application/` `usecase/` `processor/`） | 至少有 1 个入口（controller / console / consumer）能追到其中一类 |
| 生命周期 | **L** | 合法状态、谁改、禁迁 | `enums/` 下 `*Status*` / `*State*`；或 Model 上成组 status 字段 + `getNext*` / 迁移方法 | 枚举或迁移方法能被打开；不要把任意 Enum 都当状态机 |
| 分叉 | **V** | 同一入口按类型走不同实现 | `strategy/` `adapter/` `chain/` `provider/` `plugin/` | 同入口或同抽象下存在 ≥2 套平行实现，或明确的 provider 分发 |
| 对外契约 | **X** | 调别人 / 被别人调 | `launcher/`、`internal` 路由、`request/` 客户端目录、`*Client.php` / `*Sdk.php` | 能指出调用方类；只列本仓库真实存在的客户端 |

探测结果写入本轮 summary，并写入 `docs/agents/03-deep-dives/layers.md`（`flows` / `full`）：

```markdown
# 四层探测

- system / 剖面：
- 探测时间：

| 层 | 判定 | 命中路径（真实存在） | 本轮深写 |
|---|---|---|---|
| F 编排 | 命中 / 本仓库无 | `...` | `flows/{domain}.md` 或 — |
| L 生命周期 | 命中 / 本仓库无 | `...` | `lifecycle.md` 或 — |
| V 分叉 | 命中 / 本仓库无 | `...` | `variants/INDEX.md` 或 — |
| X 对外契约 | 命中 / 本仓库无 | `...` | `contracts.md` 或 — |
```

规则：

- 候选目录存在但追不到入口 → F 标「本仓库无」（或「需人工确认」），不要假写链路
- V/L/X **只在 deep 阶段**、且只深写拍板功能上碰到的；其余命中路径只进索引
- 页标题用探测到的**目录名 / 类名**，不用其它系统的业务外号
- 推不出下一跳、状态迁移、外部字段 → 写「待确认」

## 写 01-domains/INDEX.md（共用）

路径固定：`docs/agents/01-domains/INDEX.md`。只做域短索引，**禁止**把 api-list 粘进来。

| 触发 | 策略 |
|---|---|
| `mode=domains` | 重生域表 |
| `mode=surfaces` 单跑 | **先写域索引，再扫 API**；api-list 完成后回填「HTTP 条数」（能数出来才写数字） |
| `mode=full` 子代理 D | 重生域表（子代理 B **不写**此文件） |
| `mode=data` / `mode=flows` | **仅当该文件不存在**时创建；已存在则只给本轮碰到的域补「深潜」链接，不改职责原文 |
| `mode=diagrams` / `entrypoint` | 不写此文件 |

重生时：职责与入口以本轮探测为准；若仓库里已有 `03-deep-dives/flows/{domain}.md`，深潜列保留相对链接。

模板（遵守「文档版式」）：

```markdown
# {system} 业务域索引

- 系统：
- 剖面：
- 探测时间：
- 真源：（controller 目录 / 路由前缀 / 模块名，必须是本仓库路径）
- 域数量：N

HTTP 见 [../02-surfaces/api-list.md](../02-surfaces/api-list.md)。CLI / Consumer 见 [../02-surfaces/cli-and-consumers.md](../02-surfaces/cli-and-consumers.md)。模型见 [../03-deep-dives/data-model.md](../03-deep-dives/data-model.md)。

## 域

| 域 | 职责（一句话） | 入口路径 | HTTP 条数 | 深潜 |
|---|---|---|---:|---|
| {slug} | … | `controllers/…` · `routes/…` | 12 或 — | [flows](../03-deep-dives/flows/{slug}.md) 或 — |

## 缺口

> 无路由的 controller、空模块、命名对不上的目录。没有则写「本轮未见」。
```

职责必须来自本轮打开的 README / 模块注释 / 控制器目录名；推不出写 `待确认`，禁止用其它仓库的外号。
