# Codex Map Skill 渐进式披露 Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 把 `skills/codex-map/SKILL.md` 拆成 ≤150 行调度入口 + 五个 `references/` 文件，语义不变，对外文档不再出现 `docs-bootstrap`。

**Architecture:** 先在现 `SKILL.md` 仍完整时按行号剪出 references（禁止边搬边改规则），再重写薄入口；`modes-full.md` 只引用其它 references，不粘贴 inventory/flows 全文。`codex-drift` 不拆。

**Tech Stack:** Agent Skill（Markdown）；仓库 `E:\github\codex-map`；验收用 PowerShell 行数与 `rg`。

## Global Constraints

- 搬移原文，不改 `mode` 集合、propose/deep 协议、`docs/agents/` 产出契约
- 不改业务仓库已生成的 `docs/agents/` 内容；本仓只改 skill 与仓库文档
- 不把 `codex-drift` 并入 `codex-map`；drift 的 `docs-auto-sync` 别名本次不动
- 对外文件禁止出现字符串 `docs-bootstrap`（本 plan / spec 的检索词说明除外）
- YAML 不得写 `Alias: docs-bootstrap`；不得写「对话里的旧名视为本 skill」
- `SKILL.md` ≤ 150 行；未点到的 references 本轮禁止打开
- 工作目录必须是 `E:\github\codex-map`；提交信息用英文、说明 why
- 不要 push，除非用户本轮明确要求

---

## File map

| 路径 | 职责 |
|---|---|
| `skills/codex-map/references/quality.md` | 硬约束全文、版式、质量门、降级、summary 模板 |
| `skills/codex-map/references/detect.md` | 开跑前探测 1–5 节 + 写 `01-domains/INDEX.md` |
| `skills/codex-map/references/modes-inventory.md` | diagrams / domains / surfaces / data / entrypoint |
| `skills/codex-map/references/modes-flows.md` | 功能探测、propose/deep、`mode=flows` 步骤 |
| `skills/codex-map/references/modes-full.md` | full 子代理编排（引用其它文件，不复制全文） |
| `skills/codex-map/SKILL.md` | ≤150 行调度入口 |
| `skills/codex-map/examples.md` | 不改（无旧名） |
| `CONTRIBUTING.md` | 文件表加上 `references/*` |
| `CHANGELOG.md` | 增加 `0.1.2` |
| `examples/sample-output/docs/agents/INDEX.md` | 去掉 `docs-bootstrap` |

行号以实施开始时的 `skills/codex-map/SKILL.md` 为准（约 587 行）。若行号漂移，按标题切割，不要按猜的行号硬切。

---

### Task 1: 剪出 `quality.md`

**Files:**
- Create: `skills/codex-map/references/quality.md`
- Read: `skills/codex-map/SKILL.md`（仍为拆分前全文）

**Interfaces:**
- Consumes: 现 `SKILL.md` 的「硬约束」「文档版式」「质量门」「降级」「Summary 模板」
- Produces: `references/quality.md`；后续所有 mode 必读此文件

- [ ] **Step 1: 建目录**

```powershell
New-Item -ItemType Directory -Force -Path "E:\github\codex-map\skills\codex-map\references" | Out-Null
```

Expected: 无报错；目录存在。

- [ ] **Step 2: 写入 `quality.md`**

把现 `SKILL.md` 里这些块 **原文**拼进 `skills/codex-map/references/quality.md`，顺序固定：

1. 文首加一行：`# 质量与版式`
2. `## 硬约束`：现文件第 55–67 行。把最后一条「Markdown 遵守下文「文档版式」」改成「Markdown 遵守本节「文档版式」」（只改这五个字：下文→本节）
3. `## 文档版式（给人扫、给 AI 解析）`：第 69–88 行，一字不改
4. `## 质量门`：第 548–561 行
5. `## 降级`：第 563–570 行
6. `## Summary 模板`：第 572–586 行

禁止改质量门条目、禁止删 summary 字段。

- [ ] **Step 3: 确认文件存在且含 Summary 标题**

```powershell
Select-String -Path "E:\github\codex-map\skills\codex-map\references\quality.md" -Pattern "^# 质量与版式","^## Summary 模板","^## 质量门"
```

Expected: 三行都命中。

- [ ] **Step 4: Commit**

```powershell
git add skills/codex-map/references/quality.md
git commit -m "Extract quality and style rules into a skill reference file"
```

---

### Task 2: 剪出 `detect.md`

**Files:**
- Create: `skills/codex-map/references/detect.md`
- Read: `skills/codex-map/SKILL.md`

**Interfaces:**
- Consumes: `SKILL.md`「开跑前」第 1–5 节（**不含**第 6 节功能探测）+「写 01-domains/INDEX.md」
- Produces: 所有生成 mode 的探测真源

- [ ] **Step 1: 按标题切割，不要带上第 6 节**

写入 `skills/codex-map/references/detect.md`，结构：

```markdown
# 开跑前探测

```

然后 **原文**接上现 `SKILL.md`：

- 从 `## 开跑前：探测当前仓库` 起到 `### 6) 功能探测` **之前**（含第 5 节四层表与「规则：」四条）
- 空一行后接从 `## 写 01-domains/INDEX.md（共用）` 起到「职责必须来自本轮打开的 README」那一段结束（即 `## mode 执行` 之前）

**禁止**把 `### 6) 功能探测` 放进本文件。

- [ ] **Step 2: 确认不含功能探测、含 01-domains**

```powershell
Select-String -Path "E:\github\codex-map\skills\codex-map\references\detect.md" -Pattern "功能探测","写 01-domains","### 5\)"
```

Expected: `写 01-domains` 与 `### 5)` 命中；`功能探测` **零命中**。

- [ ] **Step 3: Commit**

```powershell
git add skills/codex-map/references/detect.md
git commit -m "Extract repo detection and domain-index rules into a reference file"
```

---

### Task 3: 剪出 `modes-inventory.md`

**Files:**
- Create: `skills/codex-map/references/modes-inventory.md`
- Read: `skills/codex-map/SKILL.md`

**Interfaces:**
- Consumes: `mode=diagrams|domains|surfaces|data|entrypoint` 步骤
- Produces: 清单类 mode 的执行步骤；`modes-full.md` 用「按本文件某节做」引用

- [ ] **Step 1: 写入 inventory 文件**

`skills/codex-map/references/modes-inventory.md` 开头必须是：

```markdown
# 清单类 mode

本文件覆盖 `diagrams` / `domains` / `surfaces` / `data` / `entrypoint`。
执行前已读 `detect.md` 与 `quality.md`。不要打开 `modes-flows.md` / `modes-full.md`。

```

然后 **原文**依次粘贴现 `SKILL.md` 的：

- `### mode=diagrams` 整节
- `### mode=domains` 整节
- `### mode=surfaces` 整节
- `### mode=data` 整节
- `### mode=entrypoint` 整节

**不要**粘贴 `### mode=flows` 或 `### mode=full`。

- [ ] **Step 2: 确认五个 mode 标题都在、flows/full 不在**

```powershell
Select-String -Path "E:\github\codex-map\skills\codex-map\references\modes-inventory.md" -Pattern "^### mode="
```

Expected: 恰好 5 行：diagrams、domains、surfaces、data、entrypoint。

- [ ] **Step 3: Commit**

```powershell
git add skills/codex-map/references/modes-inventory.md
git commit -m "Extract inventory-mode steps into a skill reference file"
```

---

### Task 4: 剪出 `modes-flows.md`

**Files:**
- Create: `skills/codex-map/references/modes-flows.md`
- Read: `skills/codex-map/SKILL.md`

**Interfaces:**
- Consumes: 两步协议细则（拍板信号与禁止项）、第 6 节功能探测、`mode=flows` 步骤与骨架链模板
- Produces: flows 唯一真源；薄 `SKILL.md` 只保留两步协议那张两行表

- [ ] **Step 1: 写入 flows 文件**

`skills/codex-map/references/modes-flows.md` 开头：

```markdown
# flows mode

执行前已读 `detect.md` 与 `quality.md`。不要打开 `modes-inventory.md` / `modes-full.md`。
`SKILL.md` 里的两步协议表仍有效；本文件是细则。

```

然后 **原文**粘贴：

1. 现 `SKILL.md` 中「识别「已拍板」的信号」起到「不出功能图。」（含三条禁止/无 domain 规则）
2. `### 6) 功能探测` 整节（从该标题到 `## 写 01-domains` **之前**，含 propose/deep 模板与细流程图规则）
3. `### mode=flows` 整节（含骨架链模板，到 `### mode=entrypoint` 之前）

- [ ] **Step 2: 确认关键标题**

```powershell
Select-String -Path "E:\github\codex-map\skills\codex-map\references\modes-flows.md" -Pattern "功能探测","### mode=flows","细流程图","已拍板"
```

Expected: 四处都命中。

- [ ] **Step 3: Commit**

```powershell
git add skills/codex-map/references/modes-flows.md
git commit -m "Extract flows propose/deep rules into a skill reference file"
```

---

### Task 5: 写下 `modes-full.md`（引用，不复制全文）

**Files:**
- Create: `skills/codex-map/references/modes-full.md`
- Read: `skills/codex-map/SKILL.md` 的 `mode=full` 两节（约 479–546 行）

**Interfaces:**
- Consumes: 现 full 编排语义；`detect.md` / `modes-inventory.md` / `modes-flows.md` / `quality.md` 的文件名
- Produces: full 唯一编排页；子代理 prompt 只贴该子代理允许写的文件

- [ ] **Step 1: 写入下列全文（不要再贴 inventory/flows 的步骤清单）**

创建 `skills/codex-map/references/modes-full.md`，内容必须是：

```markdown
# full mode 子代理编排

默认即 `full`。执行前已读 `detect.md` 与 `quality.md`。
工作量大，**必须**用主代理编排 + 子代理并行，禁止主代理一人顺序干完所有扫描与写文件。

逻辑顺序：探测 →（并行）图 / surfaces / data / 域索引 → flows 骨架或 propose → 互校 → entrypoint → summary。  
**不要**在 full 里对某域自动 deep。若消息带了 `domain=` 且未拍板：清单类做完后对该域只做 propose 并停住。

子代理 prompt **不要**粘贴 `modes-inventory.md` / `modes-flows.md` 全文；只写「按某文件的某节做」+ 本角色允许写的路径。

## 主代理职责

主代理只做：探测、分发、合并 summary、互校、entrypoint、对用户说话。  
子代理用 Task（`generalPurpose` 或 `explore` 按任务选）；**同一轮消息里并行发起**互不依赖的多个 Task。

## 写文件边界（禁止交叉写）

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

## 波次

**波次 0 · 主代理（串行，必先）**

1. 按 `detect.md` 做开跑前探测：称谓 / 剖面 / 域列表 / 四层 F/L/V/X  
2. 写 `layers.md`  
3. 把探测摘要（system、剖面、域列表、关键路径、四层判定）写进每个子代理 prompt，子代理**不要**重做全库探测

**波次 1 · 并行（同一轮发起）**

同时派 A + B + C + D：

- A：按 `modes-inventory.md` 的 `mode=diagrams` 写三张全景图  
- B：按 `modes-inventory.md` 的 `mode=surfaces` 写 api-list + cli-and-consumers；**跳过** surfaces 中写 `01-domains` 的步骤（由 D 写，禁止交叉）  
- C：按 `modes-inventory.md` 的 `mode=data` 写 data-model + ER（高价值域用探测域里入口最多的，或用户 `domain=`）  
- D：按 `detect.md` 的「写 01-domains/INDEX.md」写短索引  

每个子代理 prompt 必须包含：仓库根路径、探测摘要、`quality.md` 硬约束摘要、**只写哪些文件**、完成后返回「写入文件列表 + 一句话摘要 + 需人工确认」。

**波次 2 · flows（串行，等波次 1）**

- 无 `domain=`：派 E，按 `modes-flows.md` 写骨架链（可读 api-list 选入口最多的域；若 B 未完成则按 controller 文件数）  
- 有 `domain=` 且未拍板：主代理（或单子代理）按 `modes-flows.md` 做 propose → **停住**；波次 3/4 可先做互校+entrypoint（不含 deep）  
- 有 `feature=`：本轮 full **不做 deep**；summary 提示用户单独再跑 `codex-map mode=flows` deep（避免 full 与细挖抢上下文）

**波次 3 · 主代理互校**

api/cli ↔ data-model ↔ 已有 flows 页；不一致只改文档。若仓库已有 `wiki/`、`docs/ops/` 等运维原文，用链接指出，不镜像进 agents。

**波次 4 · 主代理 entrypoint + summary**

按 `modes-inventory.md` 的 `mode=entrypoint`；汇总各子代理返回值写成总 summary（标明哪些子代理跑了、写了什么）。summary 字段以 `quality.md` 模板为准。

## 子代理失败

- 某一路失败：主代理可单路重试 1 次；仍失败则 summary 标缺口，不阻塞其它已成功文件  
- 禁止为「省事」改回主代理独自全量串行，除非环境无 Task 工具（此时 summary 写「未并行，原因：无 Task」）

## 单 mode 是否并行

- `mode=diagrams|domains|surfaces|data|entrypoint|flows`：默认主代理直接做，**不**强制子代理  
- 仅当用户明确要求「并行 / 用 subagent」且任务可拆（例如 surfaces 按域拆）时再拆；拆时仍遵守写文件边界
```

- [ ] **Step 2: 确认引用了其它 references、没有 `### mode=surfaces` 步骤清单**

```powershell
Select-String -Path "E:\github\codex-map\skills\codex-map\references\modes-full.md" -Pattern "modes-inventory.md","modes-flows.md","detect.md","^### mode=surfaces"
```

Expected: 前三个命中；`^### mode=surfaces` 零命中。

- [ ] **Step 3: Commit**

```powershell
git add skills/codex-map/references/modes-full.md
git commit -m "Add full-mode orchestration reference that points at other mode files"
```

---

### Task 6: 重写薄 `SKILL.md`

**Files:**
- Modify: `skills/codex-map/SKILL.md`（整文件替换）

**Interfaces:**
- Consumes: Task 1–5 的五个 references 文件名；现 YAML description（去掉 Alias）
- Produces: ≤150 行入口；mode → 读文件表

- [ ] **Step 1: 用下面全文覆盖 `skills/codex-map/SKILL.md`**

不得追加旧段落。不得出现 `docs-bootstrap`。

````markdown
---
name: codex-map
description: >
  Generate or regenerate AI-readable documentation under docs/agents/ and the
  root AGENTS.md entrypoint for the current repository. Use when bootstrapping
  AI docs, rebuilding architecture diagrams, API/CLI/consumer inventories,
  data-model pages, flow/lifecycle/variant/contract deep-dives, per-domain
  feature shortlists (propose then wait), deep feature docs and flowcharts after
  the user picks a feature, or regenerating AGENTS.md/INDEX.md. Supports
  mode=full|diagrams|domains|surfaces|data|flows|entrypoint, optional domain=
  (project-discovered), optional feature= slug or natural language, optional
  system=auto|<short-name>. For mode=full (default), orchestrates parallel
  subagents after repo detection. Writes only docs/agents/** and AGENTS.md
  (optional CLAUDE.md). For read-only drift checks use
  codex-drift, not this skill.
---

# Codex Map

为**当前仓库**生成/重生静态 AI 文档资产。栈、业务域、中间件一律按本仓库路径探测，禁止把其它仓库的模块名或域名抄过来。

产出目录契约固定。细则在 `references/`：**未点到的文件本轮不要打开。**

## 参数

解析用户消息（缺省如下）。参数用**空格**分隔，例如 `使用 codex-map mode=flows domain=order`；逗号也能识别，但对外示例一律写空格。

- `mode`：`full`（默认）| `diagrams` | `domains` | `surfaces` | `data` | `flows` | `entrypoint`
- `system`（可选）：`auto`（默认）或任意短名（从 README / 包名 / 目录名探测，也可手写）。只用于称谓，**不改变扫描规则**
- `domain`（可选）：只框定该业务域。合法值来自本仓库探测结果
- `feature`（可选）：用户拍板的功能。slug 或自然语言。**有 `feature` 才允许深写**
- `dual_claude`（可选）：`true` 时与 `AGENTS.md` 同构双写 `CLAUDE.md`

## 硬约束

细则见 `references/quality.md`。摘要：

1. 只写 `docs/agents/**`、`AGENTS.md`；仅 `dual_claude=true` 时写 `CLAUDE.md`
2. 禁止修改业务源码目录
3. 引用路径必须在当前仓库真实存在，否则「待确认」或删除臆造项
4. 禁止编造 HTTP 方法、入参、表字段、模块名、状态迁移、调用链下一跳
5. 禁止把其它仓库的业务域、模块名、中间件名单套到当前仓库
6. 四层有则写、无则标「本仓库无」；未命中的层禁止建页
7. `AGENTS.md` ≤ 300 行；详情只放相对链接
8. `flows` 的 propose 阶段必须停住等拍板；判断不清标「需人工确认」

## 目录契约

```text
AGENTS.md
docs/agents/
  INDEX.md
  01-domains/INDEX.md
  02-surfaces/api-list.md
  02-surfaces/cli-and-consumers.md
  03-deep-dives/data-model.md
  03-deep-dives/layers.md
  03-deep-dives/flows/{domain}.md
  03-deep-dives/flows/{domain}-{slug}.md
  03-deep-dives/lifecycle.md
  03-deep-dives/variants/INDEX.md
  03-deep-dives/contracts.md
  diagrams/*.svg
```

未命中的层不要创建对应文件。过期页标过期，不擅自删除。

## 本轮读哪些文件

先读本文件（已加载），再读下表。生成类 mode 一律读 `quality.md`。

| 用户 mode | 再读 |
|---|---|
| `diagrams` / `domains` / `surfaces` / `data` / `entrypoint` | `references/detect.md` + `references/modes-inventory.md` + `references/quality.md` |
| `flows` | `references/detect.md` + `references/modes-flows.md` + `references/quality.md` |
| `full` | `references/detect.md` + `references/modes-full.md` + `references/quality.md` |

`examples.md` 仅用户问用法时打开。

## mode=flows 两步协议

| 阶段 | 触发 | 做什么 | 停不停 |
|---|---|---|---|
| **propose** | 有 `domain=`，且消息里没有明确选定的功能 | 给出最复杂 3～5 个候选；可写 `flows/{domain}.md` 候选表；禁止写 `{domain}-{slug}.md`、禁止画细流程图 | **必须停住** |
| **deep** | 用户已拍板（`feature=` / 「就做 xxx」/ 点名候选） | 只深挖拍板的那 1 个 | 做完再 summary |

禁止带 `domain=` 时自动挑 Top N 并直接深写。无 `domain=`：只写 1～2 个入口最多域的骨架链，不出功能图。细则见 `references/modes-flows.md`。

## 执行

1. 按上表打开 references，未点到的不要打开
2. 先按 `detect.md` 探测，再按对应 mode 文件做
3. 自检与 summary 以 `quality.md` 为准
```

注意：上面最外层是本 plan 的围栏；写入 SKILL.md 时，目录契约那块只保留一层 ` ```text ` 围栏。若复制后 YAML 或围栏损坏，按本 Task 的意图手工修到可被 Markdown 解析，**不要**借机加回旧段落。

- [ ] **Step 2: 行数必须 ≤ 150**

```powershell
(Get-Content "E:\github\codex-map\skills\codex-map\SKILL.md").Count
```

Expected: 数字 ≤ 150。若超了，只压缩空行与目录树注释，不删调度表。

- [ ] **Step 3: SKILL.md 不得含旧名**

```powershell
Select-String -Path "E:\github\codex-map\skills\codex-map\SKILL.md" -Pattern "docs-bootstrap"
```

Expected: 无输出。

- [ ] **Step 4: Commit**

```powershell
git add skills/codex-map/SKILL.md
git commit -m "Replace Codex Map skill entrypoint with a short mode dispatcher"
```

---

### Task 7: 清旧名并改仓库文档

**Files:**
- Modify: `examples/sample-output/docs/agents/INDEX.md`（约第 62 行）
- Modify: `CONTRIBUTING.md`（「改什么」表）
- Modify: `CHANGELOG.md`（文首加 0.1.2）

**Interfaces:**
- Consumes: spec §7 / §8
- Produces: 除 spec 外，仓库检索不到 `docs-bootstrap`

- [ ] **Step 1: 改 sample-output 那一句**

把

```markdown
- `data-model.md` 真源仍是 migrations，当前 `docs-bootstrap` skill 要求 **只扫 Model、不读 migrations**。改表前应再跑 `mode=data`，或以 `backend/models` 为准并人工核对。
```

换成：

```markdown
- `data-model.md` 真源仍是 migrations，当前 `codex-map` skill 要求 **只扫 Model、不读 migrations**。改表前应再跑 `mode=data`，或以 `backend/models` 为准并人工核对。
```

- [ ] **Step 2: 改 `CONTRIBUTING.md` 表格为**

```markdown
| 路径 | 职责 |
|---|---|
| `skills/codex-map/SKILL.md` | 生成入口与 mode 调度 |
| `skills/codex-map/references/*` | 探测、各 mode 步骤、质量门 |
| `skills/codex-map/examples.md` | 对话例句 |
| `skills/codex-drift/SKILL.md` | 只读漂移检查 |
| `README.md` | 安装与用法 |
```

- [ ] **Step 3: 在 `CHANGELOG.md` 文首、`0.1.1` 之前插入**

```markdown
## 0.1.2 — 2026-08-19

- 将 `codex-map` 拆成薄 `SKILL.md` + `references/`，按 mode 按需加载
- 对外文档不再使用历史内部 skill 名
```

禁止在 CHANGELOG 里写出 `docs-bootstrap`。

- [ ] **Step 4: README 不改结构**（安装路径仍是 `skills/codex-map/`，不要列出 references 文件名）。若 README 出现 `docs-bootstrap` 则删掉该词。

- [ ] **Step 5: Commit**

```powershell
git add examples/sample-output/docs/agents/INDEX.md CONTRIBUTING.md CHANGELOG.md README.md
git commit -m "Drop the legacy skill name from docs and record the 0.1.2 split"
```

若 README 无改动，不要 `git add README.md`。

---

### Task 8: 静态验收

**Files:**
- Test: 全仓库检索与行数；不改业务文档语义以外的文件

**Interfaces:**
- Consumes: Task 1–7 产物
- Produces: 通过 spec §9 的验收清单

- [ ] **Step 1: 五个 references 都在**

```powershell
Get-ChildItem "E:\github\codex-map\skills\codex-map\references" | Select-Object Name
```

Expected 恰好：`detect.md`、`modes-flows.md`、`modes-full.md`、`modes-inventory.md`、`quality.md`。

- [ ] **Step 2: SKILL 行数与调度表**

```powershell
(Get-Content "E:\github\codex-map\skills\codex-map\SKILL.md").Count
Select-String -Path "E:\github\codex-map\skills\codex-map\SKILL.md" -Pattern "modes-inventory.md","modes-flows.md","modes-full.md","detect.md","quality.md"
```

Expected: 行数 ≤ 150；五个文件名都在调度表里。

- [ ] **Step 3: 旧名检索（spec 允许命中 spec 自身）**

```powershell
rg -n "docs-bootstrap" E:\github\codex-map --glob "!docs/superpowers/**"
```

Expected: **零命中**。若 `rg` 不存在：

```powershell
Get-ChildItem -Path "E:\github\codex-map" -Recurse -File |
  Where-Object { $_.FullName -notmatch "docs\\superpowers\\" -and $_.FullName -notmatch "\\.git\\" } |
  Select-String -Pattern "docs-bootstrap"
```

Expected: 无输出。

- [ ] **Step 4: drift 仍独立**

```powershell
Test-Path "E:\github\codex-map\skills\codex-drift\SKILL.md"
Select-String -Path "E:\github\codex-map\skills\codex-drift\SKILL.md" -Pattern "^name: codex-drift"
```

Expected: `True`；命中 `name: codex-drift`。

- [ ] **Step 5: 确认未改 `examples/sample-output` 里除 INDEX 那一行以外的业务文档正文**（不要重跑 map、不要改 svg）。

- [ ] **Step 6: 若 Step 1–4 有失败，停在本 Task 修，不要开新语义。通过后无需再 commit（无文件变化）。若为修验收又改了文件，再 commit：**

```powershell
git add -u
git commit -m "Fix packaging split so static checks pass"
```

---

## Self-review（对照 spec）

| Spec | Task |
|---|---|
| §3 五文件目录 | 1–5 |
| §4 加载表 | 6 |
| §5 薄 SKILL、无旧名 | 6 |
| §6 边界（detect 不含第 6 节；full 不复制 inventory） | 2、5 |
| §7 清 `docs-bootstrap` | 6、7、8 |
| §8 CONTRIBUTING / CHANGELOG / examples 例句不变 | 7；`examples.md` 无改 |
| §9 验收 | 8 |
| 不拆 drift、不改产出契约 | Global + Task 8 |
| OMS 业务仓改目录名 | **不在本计划**（开源仓做完后再单独同步） |
