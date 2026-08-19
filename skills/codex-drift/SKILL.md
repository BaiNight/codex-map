---
name: codex-drift
description: >
  Read-only drift check between the current repository's code and docs/agents
  AI documentation. Use when code changed and docs may be stale, before merge,
  or when the user asks to verify documentation consistency. Compares git diff
  (or a domain sample) against api-list, cli-and-consumers, data-model, flows,
  lifecycle, variants, contracts, diagrams, and overview pages. Outputs a
  structured mismatch report only — never modifies files. For regenerating
  docs, suggest codex-map modes but do not run them unless the user explicitly
  asks. Alias: docs-auto-sync.
---

# Codex Drift

只读检查**当前仓库**代码与 `docs/agents/` 是否漂移。对话里的旧名 `docs-auto-sync` 视为本 skill。

## 硬约束

- 允许工具：Read、Grep、Glob，以及只读 git（`git diff` / `git status` / `git log`；如需 Shell，仅只读 git 子命令）
- **禁止** Write / Edit / 改任何文件（含文档与业务代码）
- **禁止** 自动执行 `codex-map`
- 无问题也要输出短报告，不得沉默
- 不要用其它仓库的文档预期来报漂移

## 对比范围

1. 优先：相对 `main`（或用户指定 base）的 `git diff` 触及路径
2. 路径 → 文档映射：
   - controllers / routes / 业务模块分组 → `docs/agents/01-domains/INDEX.md`、`docs/agents/02-surfaces/api-list.md`
   - console / commands / consumer → `docs/agents/02-surfaces/cli-and-consumers.md`
   - models → `docs/agents/03-deep-dives/data-model.md`（以 Model 为准，忽略 migrations）
   - service / handler / task / slice（及本仓库等价编排目录） → `docs/agents/03-deep-dives/flows/`、`layers.md`、`docs/agents/diagrams/flows/*.svg`
   - `*Status*` / `*State*` enums → `docs/agents/03-deep-dives/lifecycle.md`
   - strategy / adapter / chain / provider / plugin → `docs/agents/03-deep-dives/variants/`
   - HTTP 客户端 / launcher / internal 路由 → `docs/agents/03-deep-dives/contracts.md`
   - 顶层结构 / 包管理文件 / 主 config → `docs/agents/diagrams/*`、`AGENTS.md`、`docs/agents/INDEX.md`
3. 无 diff：可对用户给的 `domain=` 抽样（域名须能在本仓库 `INDEX.md` 或源码中对应），或对 `docs/agents/INDEX.md` 高优先级页抽查
4. 若 `docs/agents/` 几乎为空：报告 `coverage_gap`，建议先跑 `codex-map mode=full`，仍不执行

## 不一致类型

| 类型 | 含义 |
|---|---|
| `missing_in_docs` | 代码有入口/表/模块，文档未记载 |
| `stale_in_docs` | 文档有，代码已删/改路径/改行为 |
| `path_invalid` | 文档写的文件/类路径不存在 |
| `contract_drift` | 方法/路径/主参/表字段与文档明显不符 |
| `coverage_gap` | INDEX 标明应有但页缺失或过空 |

## 步骤

1. 确认 `docs/agents/` 是否存在可读资产
2. 确定对比基准与扫描范围
3. 按映射检查；每条不一致同时记录「代码证据」与「文档位置」（或明确文档缺失）
4. 不确定标「需确认」，不升格为确定漂移
5. 输出报告（只在对话中）

## 报告格式（必须遵守）

```markdown
# Codex Drift 报告
- 对比基准：...
- 扫描范围：...
- 结论：有 N 处不一致 / 未发现明显不一致

## 不一致清单
| ID | 类型 | 代码位置 | 文档位置 | 现象 | 建议动作 |
|----|------|----------|----------|------|----------|

## 建议的重生命令（不执行）
- codex-map mode=domains（仅补 `01-domains/INDEX.md`）
- codex-map mode=surfaces domain=<本仓库探测到的域>
- codex-map mode=flows domain=<本仓库探测到的域>（先 propose 候选，等拍板）
- codex-map mode=flows domain=<域> feature=<slug>（拍板后 deep）

## 需人工确认
- ...
```

建议动作只写「建议跑哪个 `codex-map` mode / 改哪一页」。
