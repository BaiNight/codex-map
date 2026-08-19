---
name: codex-map
description: >
  Generate or regenerate AI-readable documentation under docs/agents/ and the
  root AGENTS.md entrypoint for the current repository. Use when bootstrapping
  AI docs, rebuilding architecture diagrams, API/CLI/consumer inventories,
  data-model pages, flow/lifecycle/variant/contract deep-dives, per-domain
  feature shortlists (propose then wait), deep feature docs and flowcharts after
  the user picks a feature, or regenerating AGENTS.md/INDEX.md. Supports
  mode=full|diagrams|surfaces|data|flows|entrypoint, optional domain=
  (project-discovered), optional feature= slug or natural language, optional
  system=auto|<short-name>. For mode=full (default), orchestrates parallel
  subagents after repo detection. Writes only docs/agents/** and AGENTS.md
  (optional CLAUDE.md). Alias: docs-bootstrap. For read-only drift checks use
  codex-drift, not this skill.
---

# Codex Map

为**当前仓库**生成/重生静态 AI 文档资产。栈、业务域、中间件一律按本仓库路径探测，禁止把其它仓库的模块名或域名抄过来。

对话里的旧名 `docs-bootstrap` 视为本 skill。

产出目录契约固定。

## 参数

解析用户消息（缺省如下）：

- `mode`：`full`（默认）| `diagrams` | `surfaces` | `data` | `flows` | `entrypoint`
- `system`（可选）：`auto`（默认）或任意短名（从 README / 包名 / 目录名探测，也可手写）  
  只用于 `AGENTS.md` / overview 里的系统称谓，**不改变扫描规则**
- `domain`（可选）：只框定该业务域。合法值来自本仓库探测结果
- `feature`（可选）：用户拍板的功能。可为 slug（如 `order-sync`），或自然语言。**有 `feature` 才允许深写**
- `dual_claude`（可选）：`true` 时与 `AGENTS.md` 同构双写 `CLAUDE.md`

### `mode=flows` 两步协议（硬性）

| 阶段 | 触发 | 做什么 | 停不停 |
|---|---|---|---|
| **propose** | `domain=` 有，且消息里**没有**明确选定的功能 | 扫该域，给出最复杂 **3～5** 个候选（含分数与依据）；可写/更新 `flows/{domain}.md` 候选表；**禁止**写 `{domain}-{slug}.md`、**禁止**画细流程图 | **必须停住**，等用户拍板。本轮 summary 后结束 |
| **deep** | 用户已拍板：`feature=` / 「就做 xxx」/ 「生成 xxx 详细文档与流程图」 | 只深挖拍板的那 1 个（用户明确点多个才多个）；写细节页 + 细流程图；按碰到的回写 L/V/X | 做完再 summary |

识别「已拍板」的信号（命中任一即可进 deep）：

- 显式 `feature=<slug>` 或 `feature=<自然语言>`
- 「就做 / 选 / 拍板 / 生成 … 详细文档与流程图」+ 能对应到候选表或代码里的某一功能
- 用户从上一轮候选列表里点名（序号或短名）

**禁止**：带 `domain=` 时自动挑 Top N 并直接生成细节文档/细图。  
**禁止**：propose 阶段「先做完再问你要不要改」。必须先交名单。

无 `domain=`：不走两步协议；只写 1～2 个入口最多域的骨架链（2～3 条），不出功能图。

## 硬约束

- 只写：`docs/agents/**`、`AGENTS.md`；仅当 `dual_claude=true` 时写 `CLAUDE.md`
- 禁止修改业务代码（如 `backend/`、`common/`、`console/`、`app/`、`src/` 等源码目录）
- 引用的源码路径必须在**当前仓库**真实存在；否则标「待确认」或删除臆造项
- 禁止编造 HTTP 方法、入参、表字段、模块名、状态迁移、调用链下一跳
- 禁止把其它仓库的业务域、模块名、中间件名单套到当前仓库
- **四层有则写、无则标「本仓库无」**；未命中的层禁止建页、禁止用其它系统的实例当标题
- SVG 使用 dark 主题；模块写「名字 + 一句话职责」
- `AGENTS.md` ≤ 300 行；详情只放相对链接，不粘贴长清单
- 每步结束后自检；不通过则本步重做再进入下一步
- 判断不清先做合理选择，在最终 summary 标「需人工确认」；**唯独 `flows` 的 propose 阶段必须打断等拍板**（覆盖「不要中途提问」）

## 目录契约（所有项目相同）

```text
AGENTS.md
docs/agents/
  INDEX.md
  01-domains/
  02-surfaces/api-list.md
  02-surfaces/cli-and-consumers.md
  03-deep-dives/data-model.md
  03-deep-dives/layers.md
  03-deep-dives/flows/{domain}.md              # 域索引 + 复杂度候选表（propose 可写）
  03-deep-dives/flows/{domain}-{slug}.md       # 仅 deep：步骤 + 实现细节
  03-deep-dives/lifecycle.md                   # 仅 deep 且 L 命中时按功能回写
  03-deep-dives/variants/INDEX.md
  03-deep-dives/contracts.md
  diagrams/architecture.svg
  diagrams/module-deps.svg
  diagrams/external-deps.svg
  diagrams/data-model-er.svg
  diagrams/flows/{domain}-{slug}.svg           # 仅 deep：细粒度一功能一图
```

未命中的层**不要创建对应文件**。已有过期页且本轮判定「本仓库无」→ 在 `layers.md` / `INDEX.md` 标过期，不擅自删除（summary 标需人工确认）。

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

从本仓库推导，写入 `docs/agents/01-domains/` 索引（`full` / `surfaces` / `data` / `flows` 时）：

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

### 6) 功能探测（仅 `mode=flows` 且带了 `domain=`）

「功能」= 同一业务动作的一组入口，不是一条路由一行。  
禁止用其它系统的业务外号当功能名。

#### 聚类

从该域入口归并（`api-list` / `cli-and-consumers` / 打开的 controller、console、consumer）：

| 归并信号（命中任一即可合成一个功能） |
|---|
| 同一 Controller 上的一组写操作（如 store / review / import） |
| 同一 Consumer `name` 或同一 `*StrategyProvider` / 同一 chain 类 |
| 写入同一核心 Model（`tableName()` 相同） |
| HTTP/CLI 投 MQ，由同域 Consumer 接着做 → **算同一功能**，不拆成两个 |

功能短名（`slug`）优先级：路由注释 / action 注释 → Consumer yml `name` → Controller 去 `Controller` 后缀 + action-id。只允许 `[a-z0-9-]`。禁止自创中文产品名当 slug。

#### 重要性（过滤）

| 分 | 信号（代码里真实存在才加） |
|---:|---|
| +2 | 能追到 F 编排 |
| +2 | 有 L 或 V |
| +1 | 有 X 或 MQ |
| +1 | 同时有 HTTP 与 CLI/Consumer |

重要性 = 0：进「未展开」，不进候选榜。

#### 复杂度（排序用）

只对重要性 > 0 的功能打分。数字必须来自本轮打开的代码。

| 分 | 信号 | 怎么数 |
|---:|---|---|
| +4 | V 很宽 | Provider `providers()` ≥ 3，**或** chain `apply()` / 同类列表 ≥ 5 个算子 |
| +4 | 调用深 | 入口到写库中间编排类 ≥ 4 跳（controller 不算） |
| +3 | 分支多 | 主路径上按 status/type/mode/platform 等字段的 `if`/`switch` ≥ 3 |
| +3 | 多表写入 | 不同 `tableName()` ≥ 3 |
| +2 | 有状态机 | 带 `getNext*` / 明确禁迁的 Status/State |
| +2 | 批处理 | 列表/文件行/item 循环后落库或投 MQ |
| +2 | 跨入口 | HTTP/CLI 投递，计算在 Consumer/task |
| +1 | 事务/锁/幂等 | 代码里有 `transaction` / lock / 去重键 |
| +1 | 对外 | launcher / internal / OSS 等 |

propose：按复杂度取 **Top 3～5** 进候选榜（不足则有多少列多少）。同分优先更深、V 更宽。

#### propose 产出（只推荐）

1. 更新 `flows/{domain}.md`：候选表（见下）+ 未展开索引；**不要**写各功能长步骤
2. 在对话中用同样表格请用户拍板（序号或 slug）
3. **停止**。禁止创建 `flows/{domain}-{slug}.md`、禁止画 `diagrams/flows/*.svg`、禁止因本轮去扩写 lifecycle/contracts 细节

候选表模板：

```markdown
# {domain} 功能候选（待拍板）

- 探测时间：
- 状态：**等待用户选择** — 回复「就做 N」或 `feature=<slug>` 后再深挖

| # | slug | 一句话 | 入口（HTTP/CLI/Consumer） | 重要性 | 复杂度 | 依据（打开过的路径） |
|---|------|--------|---------------------------|-------:|-------:|----------------------|
| 1 | … | … | … | … | … | V=…；跳数=…；… |

## 未展开
| 入口 | 能看到的下一跳 |
|---|---|
```

对话收尾必须包含：

```text
请拍板：回复「就做 1」或「就做 <slug>」或
使用 codex-map，mode=flows，domain=<域>，feature=<slug|自然语言>
```

#### deep 产出（拍板后才写）

每个拍板功能：

1. `flows/{domain}.md`：把该行标为「已深挖」，链到深页与图
2. `flows/{domain}-{slug}.md`：编号步骤 + **实现细节**
3. `diagrams/flows/{domain}-{slug}.svg`：细粒度流程图

用户自然语言对不上唯一候选：列出最接近的 2～3 个再问一次，**仍不深写**。

同功能 HTTP 投 MQ、Consumer 落库：一条时间线、一张图、一份细节。

##### 步骤（深页前半）

```markdown
## 功能 · {短名}

- 复杂度 / 重要性：{分} / {分}
- 一句话：
- 入口：
- 流程图：[../../diagrams/flows/{domain}-{slug}.svg](../../diagrams/flows/{domain}-{slug}.svg)

### 步骤
1. 触发：谁调用、入口文件与方法、能读到的主参
2. 校验：Abort / 权限 / 前置状态（只写代码里看得到的）
3. 编排：service / slice / task / handler 的类::方法
4. 分叉：V 目录 + 分发类；按什么字段选实现；本步无则写无
5. 写库：Model 路径 + tableName；能看到的赋值就写
6. 发出：MQ / launcher / internal / client
7. 失败：catch / Abort / 回滚 / 通知；看不到则待确认
```

每步带可打开路径。可按真实顺序增减，禁止凑步数。

##### 实现细节（深页后半）

只写本轮打开并核对过的代码。禁止粘贴整文件、禁止编造字段。

```markdown
### 实现细节

#### 调用栈
| # | 类::方法 | 路径 | 做什么（一句话） | 下一跳 |
|---|----------|------|------------------|--------|

#### 分支
| 条件（字段/枚举/字面量，抄代码） | 走向 | 路径 |
|----------------------------------|------|------|

#### 分叉展开（仅本功能有 V 时）
- 分发类、选择键
- `providers()` / `apply()` **按代码顺序列出**；超过 20 个则列前 20 + 「其余 N 个见目录」
- 每个列出的实现：类路径、一句话、写哪些 Model（看到才写）、跳过规则（看到才写）
- 另选 1 条与主路径不同的代表分支对照差异

#### 写库
| Model | tableName | 调用方法 | 能看到的赋值 | 条件 |
|-------|-----------|----------|--------------|------|

#### 消息与外部
| 方向 | 队列或 launcher/client | 能读到的字段 | 路径 |
|------|------------------------|--------------|------|

#### 失败与禁迁
| 条件 | 行为 | 路径 |

#### 事务 / 锁 / 幂等
看到的写，看不到写「本功能未见」
```

##### 细流程图（仅 deep）

- 路径：`docs/agents/diagrams/flows/{domain}-{slug}.svg`
- 主题：与 `architecture.svg` 相同 dark
- **不要**收成「入口→编排→写库」三框；画出调用栈上的真实判断和主要分叉
- 节点上限 **28**。仍超：合并纯转发步，**判断和写库不要合并**
- V 很宽：画「选择键」菱形 + 主路径一条 + 对照分支一条 + 「其它 N 套 → 见细节页」
- chain/pipeline：按 `apply()` 顺序列 4～8 个算子，其余「+N」指向细节页
- 分色：入口蓝、判断菱形黄边 `#d29922`、编排灰、写库绿、对外/MQ 橙
- 节点正文：`类::方法`、判断短句、或「写 {tableName}」。字段列表只在 md
- 边上标 是/否、通过/Abort、选择键取值
- 禁止 mermaid 替代 SVG
- 自检：图上每个判断/写库能在细节页表里对上

## mode 执行

### mode=diagrams

1. 完成「开跑前探测」；读 README、顶层目录、composer、关键 config
2. 写 `docs/agents/diagrams/architecture.svg`：按本仓库分层；核心模块「名 + 一句话」；基础设施一个方框
3. 写 `module-deps.svg`：只画本仓库模块/目录边界；循环依赖红色
4. 写 `external-deps.svg`：三类分色 — 关键语言依赖、中间件、外部 API
5. 自检路径与分层后进入 summary（若仅 diagrams）

### mode=surfaces

1. 探测域列表
2. 扫 HTTP → `docs/agents/02-surfaces/api-list.md`（方法、路径、一句话、主入参、返回要点；按模块分组；推不出标「待确认」）
3. 扫 CLI + consumers → `cli-and-consumers.md`（没有则明确写无）
4. 若带 `domain=`，只深写该域，其它域保留索引或「待补」
5. 自检：抽样打开文档中的文件路径

### mode=data

1. **只扫 Model**（当前剖面下的 Model 目录）。高价值域 = 探测域里入口最多或用户 `domain=` 指定的集合，不要默认某个业务名
2. 字段与说明来自 `rules()` / `attributeLabels()` / 属性注释（或其它栈等价物）；关系来自关系方法；表名来自 `tableName()` 或等价声明
3. 写 `docs/agents/03-deep-dives/data-model.md`（文首写明真源是 Model，不含 migration）
4. 写 `docs/agents/diagrams/data-model-er.svg`（只画 Model 声明的关系）
5. 非核心 Model 只列索引清单，不穷尽全部类
6. 禁止打开 `console/migrations`、`migrations/` 或把 migration 当对照源

### mode=flows

1. 完成开跑前探测（含第 5 节四层）
2. 写 `docs/agents/03-deep-dives/layers.md`
3. **F 本仓库无**：不写 `flows/`、不画图；summary 后结束
4. **无 `domain=`**：取入口最多的 1～2 个域，每域 2～3 条骨架链写入 `flows/{domain}.md`（模板见下），不出功能图；然后跳到更新 INDEX → Summary
5. **有 `domain=`，未拍板（propose）**：
   - 只该域，执行第 6 节聚类 + 打分
   - 写 `flows/{domain}.md` 候选表（Top 3～5）
   - 对话给出候选并请拍板
   - **停止**（不写细节页、不画细图、不深写 L/V/X）
   - Summary 标明 `phase=propose`、候选 slug 列表
6. **有 `domain=`，已拍板（deep）**：
   - 解析 `feature` / 自然语言到唯一 slug；对不上则再问，不深写
   - 写 `flows/{domain}-{slug}.md` + `diagrams/flows/{domain}-{slug}.svg`
   - 更新 `flows/{domain}.md` 该行状态为已深挖
   - **L 命中**：按本功能回写 `lifecycle.md`（其它 Status 只索引）
   - **V 命中**：更新 `variants/INDEX.md`（名 + 一句话 + 命中功能）
   - **X 命中**：按本功能回写 `contracts.md`
   - 更新 `INDEX.md` 覆盖率
   - 自检：路径可打开；图节点与细节表对得上
   - Summary 标明 `phase=deep`、slug、文件列表

骨架链模板（无 domain 时）：

```text
入口（HTTP / CLI / Consumer + 文件路径）
  → 编排类（service / handler / task / slice + 方法）
  → 分叉点（V 命中且本链用到；否则 本链无）
  → 写入的 Model（tableName + 路径；否则 待确认）
  → 对外调用（X 命中且本链用到；否则 本链无）
```

### mode=entrypoint

1. 只读已有 `docs/agents/**`（不扫全库）
2. 写/更新 `docs/agents/INDEX.md`（阅读顺序、覆盖率、缺口、本仓库探测到的 system/剖面/四层判定）
3. 写/更新根 `AGENTS.md`：项目定位、核心架构、关键模块、关键约定、怎么跑；  
   「禁区」「历史包袱」两节写「待团队补充」
4. 若 `dual_claude=true`，同构写 `CLAUDE.md`

### mode=full

默认即 `full`。工作量大，**必须**用主代理编排 + 子代理并行（见下节），禁止主代理一人顺序干完所有扫描与写文件。

逻辑顺序仍是：探测 →（并行）图 / surfaces / data / 域索引 → flows 骨架或 propose → 互校 → entrypoint → summary。  
**不要**在 full 里对某域自动 deep。若消息带了 `domain=` 且未拍板：清单类做完后对该域只做 propose 并停住。

### mode=full 子代理编排（强制）

主代理只做：探测、分发、合并 summary、互校、entrypoint、对用户说话。  
子代理用 Task（`generalPurpose` 或 `explore` 按任务选）；**同一轮消息里并行发起**互不依赖的多个 Task。

#### 写文件边界（禁止交叉写）

| 角色 | 只允许写 |
|---|---|
| 主代理 | `layers.md`（探测结果）、互校改动、`INDEX.md`、`AGENTS.md`（及可选 `CLAUDE.md`）、最终 summary |
| 子代理 A · diagrams | `docs/agents/diagrams/architecture.svg`、`module-deps.svg`、`external-deps.svg` |
| 子代理 B · surfaces | `docs/agents/02-surfaces/api-list.md`、`cli-and-consumers.md` |
| 子代理 C · data | `docs/agents/03-deep-dives/data-model.md`、`diagrams/data-model-er.svg` |
| 子代理 D · domains | `docs/agents/01-domains/**`（短索引；不写深潜） |
| 子代理 E · flows-skeleton | 仅当**无** `domain=`：`flows/{domain}.md` 骨架链（1～2 域） |
| 主代理或单子代理 · flows-propose | 仅当有 `domain=` 且未拍板：候选表；**停住等用户**，不派 deep |

禁止两个子代理写同一文件。子代理**禁止**改业务代码、禁止跑 deep、禁止改 `AGENTS.md`（留给 entrypoint）。

#### 波次

**波次 0 · 主代理（串行，必先）**

1. 开跑前探测：称谓 / 剖面 / 域列表 / 四层 F/L/V/X  
2. 写 `layers.md`  
3. 把探测摘要（system、剖面、域列表、关键路径、四层判定）写进每个子代理 prompt，子代理**不要**重做全库探测

**波次 1 · 并行（同一轮发起）**

同时派 A + B + C + D：

- A：按 mode=diagrams 写三张全景图  
- B：按 mode=surfaces 写 api-list + cli-and-consumers  
- C：按 mode=data 写 data-model + ER（高价值域用探测域里入口最多的，或用户 `domain=`）  
- D：按探测域写 `01-domains/` 短索引  

每个子代理 prompt 必须包含：仓库根路径、探测摘要、本 skill 硬约束摘要、**只写哪些文件**、完成后返回「写入文件列表 + 一句话摘要 + 需人工确认」。

**波次 2 · flows（串行，等波次 1）**

- 无 `domain=`：派 E 写骨架链（可读 api-list 选入口最多的域；若 B 未完成则按 controller 文件数）  
- 有 `domain=` 且未拍板：主代理（或单子代理）做 propose → **停住**；波次 3/4 可先做互校+entrypoint（不含 deep）  
- 有 `feature=`：本轮 full **不做 deep**；summary 提示用户单独再跑 `codex-map mode=flows` deep（避免 full 与细挖抢上下文）

**波次 3 · 主代理互校**

api/cli ↔ data-model ↔ 已有 flows 页；不一致只改文档。若仓库已有 `wiki/`、`docs/ops/` 等运维原文，用链接指出，不镜像进 agents。

**波次 4 · 主代理 entrypoint + summary**

执行 mode=entrypoint；汇总各子代理返回值写成总 summary（标明哪些子代理跑了、写了什么）。

#### 子代理失败

- 某一路失败：主代理可单路重试 1 次；仍失败则 summary 标缺口，不阻塞其它已成功文件  
- 禁止为「省事」改回主代理独自全量串行，除非环境无 Task 工具（此时 summary 写「未并行，原因：无 Task」）

#### 单 mode 是否并行

- `mode=diagrams|surfaces|data|entrypoint|flows`：默认主代理直接做，**不**强制子代理  
- 仅当用户明确要求「并行 / 用 subagent」且任务可拆（例如 surfaces 按域拆）时再拆；拆时仍遵守写文件边界

## 质量门

- 路径真实存在于当前仓库
- SVG dark；无实现级细节堆砌
- api-list 不编造契约
- data-model 以本仓库 Model 为准，不使用 migrations
- 域名称、V 页标题来自探测，不来自其它系统的记忆
- 未命中的 F/L/V/X 不建页
- **propose 不得产出** `{domain}-{slug}.md` 与细流程图
- **deep** 必须有编号步骤 + 实现细节 + SVG；图节点与细节表对得上
- 细流程图节点 ≤ 28；字段不进图
- AGENTS.md ≤ 300 行

## 降级

- 域过大：候选仍限 3～5；其余进未展开
- 配置读不全：写「已知 + 待确认」，禁止假完整
- 剖面无法判定：按 generic-php 能扫多少扫多少，summary 标需人工确认
- F 链路追断：写到最后一个可打开的类，下一跳标「待确认」
- V 过多：细节页列前 20；图上主路径 + 一条对照 + 「其它 N」
- 图将超过 28 节点：合并纯转发，保留判断与写库

## Summary 模板

```markdown
# Codex Map summary
- system / 剖面 / mode / domain / feature：
- phase：propose | deep | skeleton | n/a
- 并行：是/否；子代理列表（A diagrams / B surfaces / C data / D domains / E flows…）与各自结果：
- 探测到的业务域：
- 四层探测：F= / L= / V= / X=
- propose 时：候选 slug 列表（等拍板）：
- deep 时：深挖的 slug + md/svg 路径：
- 写入文件列表：
- 各文件一句话摘要：
- 需人工确认：
```
