---
name: codex-map
description: >
  Generate or regenerate AI-readable docs for the current repository: root
  AGENTS.md and docs/agents inventories, architecture SVGs, and one collapsible
  flow.html per deep-dived feature. Use when the user says codex-map or
  使用 codex-map, or asks to bootstrap, refresh, or map agent docs, domains,
  APIs, CLI, consumers, data models, critical paths, or a business flow.
  Modes: full, diagrams, domains, surfaces, data, flows, entrypoint. Optional
  domain, feature, refresh=off|cited|force. Propose flow candidates and wait
  before writing a deep page. Deep pages cite path:line. Do not rewrite
  existing inventory pages unless refresh=force. Do not invent HTTP methods,
  fields, or call edges. Writes only docs/agents and AGENTS.md. Drift checks
  use codex-drift, not this skill.
compatibility: >
  Agent Skills hosts such as Cursor, Claude Code, and Codex. Reads the
  repository. May fetch framework docs during stack detection. Does not
  install packages or start the app.
metadata:
  version: "1.2"
---

# Codex Map

为**当前仓库**生成/重生静态 AI 文档资产。栈、业务域、中间件一律按本仓库路径探测，禁止把其它仓库的模块名或域名抄过来。

产出目录契约固定。细则在 `references/`：**未点到的文件本轮不要打开。** `README.md` 只给人看，执行时不要读。

## 参数

解析用户消息（缺省如下）。参数用**空格**分隔，例如 `使用 codex-map mode=flows domain=order`；逗号也能识别，但对外示例一律写空格。

- `mode`：`full`（默认）| `diagrams` | `domains` | `surfaces` | `data` | `flows` | `entrypoint`
- `system`（可选）：`auto`（默认）或任意短名（从 README / 包名 / 目录名探测，也可手写）。只用于称谓，**不改变扫描规则**
- `domain`（可选）：只框定该业务域。合法值来自本仓库探测结果
- `feature`（可选）：用户拍板的功能。slug 或自然语言。**有 `feature` 才允许深写**
- `refresh`（可选）：`off`（默认）| `cited` | `force`。细则见 `references/quality.md`「已有页不整页重写」
  - 没说刷新 / 重生 / 重做 / 重扫 → `off`
  - 「核对引用」「标过期」「深挖还准不准」→ `cited`
  - 「刷新」「重生」「重做」「重扫」或写了 `refresh=force` → `force`
  - `flows` 的「刷新候选」只重算未深挖候选，**不**把整库提升成 `force`
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
9. 「禁区」「历史包袱」禁止编造实文；资产互校未过不得宣称完整
10. 怎么跑只抄仓库已有命令；DTO/Request 不进 data-model；清单超上限必须砍
11. 已有清单、全景图、深页默认不整页重写。只有 `refresh=force`（或用户说了刷新 / 重生 / 重做 / 重扫）才覆盖该 mode 允许的文件。点名 `feature=` 只重写那一个功能
12. 深页步骤、分支、调用栈必须有本轮打开过的 `路径:行号`。没有行号的判断不能画成菱形
13. `AGENTS.md` 必须有「先查哪里」。调用关系优先写本轮对上的 codebase-memory 项目名；其次才是仓库里真实有的 CodeGraph / GitNexus；都没有就写探测到的 `rg` 命令。接口表、模型表不是调用图
14. 扫描结构时若会话里有 `user-codebase-memory-mcp`，按 `references/detect.md`「知识图谱」用它。图只提供入口和调用线索。HTTP 契约、表名、`路径:行号` 仍以打开的源码为准。没有这个 MCP 就跳过，不中断扫描

## 本技能的文件

根目录只有 `SKILL.md`（本文件）和 `README.md`。细则都在 `references/`，从这里直接链接，不要再套一层索引。

| 文件 | 何时读 |
|---|---|
| `references/quality.md` | 任何会写文件的 mode |
| `references/detect.md` | 探测栈、域、四层 |
| `references/modes-full.md` | `mode=full` |
| `references/modes-inventory.md` | `diagrams` / `domains` / `surfaces` / `data` / `entrypoint` |
| `references/modes-flows.md` | `mode=flows` |
| `references/diagrams.md` | 画全景 SVG，或 deep 的 `flow.html` |
| `references/examples.md` | 用户问怎么调用 |

## 目录契约

```text
AGENTS.md
docs/agents/
  INDEX.md
  01-domains/INDEX.md
  02-surfaces/api-list.md
  02-surfaces/cli-and-consumers.md
  03-deep-dives/data-model.md
  03-deep-dives/critical-paths.md
  03-deep-dives/forbidden-candidates.md
  03-deep-dives/risk-zones.md
  03-deep-dives/layers.md
  03-deep-dives/flows/{domain}.md
  03-deep-dives/flows/{domain}-{slug}.md
  03-deep-dives/lifecycle.md
  03-deep-dives/variants/INDEX.md
  03-deep-dives/contracts.md
  diagrams/*.svg
  diagrams/flows/{domain}/{slug}/flow.html
```

未命中的层不要创建对应文件。过期页标过期，不擅自删除。

## 本轮读哪些文件

先读本文件（已加载），再读下表。生成类 mode 一律读 `quality.md`。  
`detect.md`：先看 `composer.json` 定包，再 **WebFetch 官网结构/Model 页**，与本仓库路径求交。不要为新框架写死目录。官网失败才读文末离线兜底。

| 用户 mode | 再读 |
|---|---|
| `diagrams` / `data` | `references/detect.md` + `references/modes-inventory.md` + `references/quality.md` + `references/diagrams.md` |
| `domains` / `surfaces` / `entrypoint` | `references/detect.md` + `references/modes-inventory.md` + `references/quality.md` |
| `flows` | `references/detect.md` + `references/modes-flows.md` + `references/quality.md`；**deep 再读** `references/diagrams.md` |
| `full` | `references/detect.md` + `references/modes-full.md` + `references/quality.md`（画图子代理再读 `diagrams.md`） |

`references/examples.md` 仅用户问用法时打开。

## mode=flows 两步协议

| 阶段 | 触发 | 做什么 | 停不停 |
|---|---|---|---|
| **propose** | 有 `domain=`，且消息里没有明确选定的功能 | 无可用候选页才扫描打分；已有且仍等待选择则只复述名单。禁止写 `{domain}-{slug}.md`、禁止画细流程图 | **必须停住** |
| **deep** | 用户已拍板（`feature=` / 「就做 xxx」/ 点名候选） | 只深挖拍板的那 1 个 | 做完再 summary |

禁止带 `domain=` 时自动挑 Top N 并直接深写。无 `domain=`：**不写** `flows/`，提示下一步 `mode=flows domain=<域>`。细则见 `references/modes-flows.md`。

## 执行

1. 按上表打开 references，未点到的不要打开
2. 先按 `detect.md` 探测，再按对应 mode 文件做
3. 自检与 summary 以 `quality.md` 为准
