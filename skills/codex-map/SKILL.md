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
  diagrams/flows/{domain}/{slug}/01-overview.svg
  diagrams/flows/{domain}/{slug}/{nn}-{phase}.svg
```

未命中的层不要创建对应文件。过期页标过期，不擅自删除。

## 本轮读哪些文件

先读本文件（已加载），再读下表。生成类 mode 一律读 `quality.md`。

| 用户 mode | 再读 |
|---|---|
| `diagrams` / `data` | `references/detect.md` + `references/modes-inventory.md` + `references/quality.md` + `references/diagrams.md` |
| `domains` / `surfaces` / `entrypoint` | `references/detect.md` + `references/modes-inventory.md` + `references/quality.md` |
| `flows` | `references/detect.md` + `references/modes-flows.md` + `references/quality.md`；**deep 再读** `references/diagrams.md` |
| `full` | `references/detect.md` + `references/modes-full.md` + `references/quality.md`（画图子代理再读 `diagrams.md`） |

`examples.md` 仅用户问用法时打开。

## mode=flows 两步协议

| 阶段 | 触发 | 做什么 | 停不停 |
|---|---|---|---|
| **propose** | 有 `domain=`，且消息里没有明确选定的功能 | 无可用候选页才扫描打分；已有且仍等待选择则只复述名单。禁止写 `{domain}-{slug}.md`、禁止画细流程图 | **必须停住** |
| **deep** | 用户已拍板（`feature=` / 「就做 xxx」/ 点名候选） | 只深挖拍板的那 1 个 | 做完再 summary |

禁止带 `domain=` 时自动挑 Top N 并直接深写。无 `domain=`：只写 1～2 个入口最多域的骨架链，不出功能图。细则见 `references/modes-flows.md`。

## 执行

1. 按上表打开 references，未点到的不要打开
2. 先按 `detect.md` 探测，再按对应 mode 文件做
3. 自检与 summary 以 `quality.md` 为准
