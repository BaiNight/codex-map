# Codex Map

给 Agent 画代码地图：探测当前仓库，生成可被 AI 直接阅读的静态文档（`docs/agents/` + `AGENTS.md`）。

这是 **Cursor / Claude Code 的 Agent Skill**，不是 CLI、也不是 Web Wiki。Agent 读 `SKILL.md` 后按规则扫描你的业务仓库并写文档。

旧名：`docs-bootstrap`（生成）、`docs-auto-sync`（对账）。对话里用旧名仍应视为本套 skill。

## 两个 Skill

| Skill | 做什么 | 写文件？ |
|---|---|---|
| [`codex-map`](skills/codex-map/SKILL.md) | 生成 / 重生架构图、API/CLI 清单、数据模型、主链路与细流程图 | 只写 `docs/agents/**`、`AGENTS.md`（可选 `CLAUDE.md`） |
| [`codex-drift`](skills/codex-drift/SKILL.md) | 对照 git diff 或抽样，报告文档漂移 | **只读**，不改文件 |

`codex-map` **不会**改业务源码。推不出的入参/字段必须标「待确认」，禁止编造。

## 安装

Skill 不会随 `git clone` 自动进编辑器。拷到目标业务仓库，或拷到个人全局目录。

**项目内（推荐，跟仓库走）：**

```bash
git clone https://github.com/BaiNight/codex-map.git
```

Cursor：

```text
<业务仓库>/.cursor/skills/codex-map/SKILL.md
<业务仓库>/.cursor/skills/codex-drift/SKILL.md
```

Claude Code：

```text
<业务仓库>/.claude/skills/codex-map/SKILL.md
<业务仓库>/.claude/skills/codex-drift/SKILL.md
```

把本仓 `skills/codex-map`、`skills/codex-drift` 两个目录拷过去即可。

**个人全局（所有项目可用）：**

- Cursor：`~/.cursor/skills/codex-map/`
- Claude Code：`~/.claude/skills/codex-map/`

Windows 示例：`%USERPROFILE%\.cursor\skills\codex-map\`

## 用法

在**业务仓库**的 Agent 对话里说明（不要在本 skill 仓里对空气跑）：

```text
使用 codex-map，mode=full
```

或分步：`mode=diagrams` → `surfaces` → `data` → `entrypoint`。

主链路两步（**先推荐，等你拍板，再深挖**）：

```text
使用 codex-map，mode=flows，domain=<本仓库探测到的域>
```

Agent 给出最复杂 3～5 个候选后**必须停住**。你再点名：

```text
使用 codex-map，mode=flows，domain=<域>，feature=<slug 或自然语言>
```

对账（只出报告）：

```text
使用 codex-drift，相对 main 检查文档漂移
```

| 参数 | 默认 | 说明 |
|---|---|---|
| `mode` | `full` | `full` \| `diagrams` \| `surfaces` \| `data` \| `flows` \| `entrypoint` |
| `system` | `auto` | 只影响称呼，从 README / 包名 / 目录名探测；可手动短名 |
| `domain` | 无 | 必须是当前仓库探测到的域 |
| `feature` | 无 | 有才允许 `flows` 深挖 |
| `dual_claude` | 关 | `true` 时与 `AGENTS.md` 同构再写一份 `CLAUDE.md` |

`mode=full` 时主代理先探测，再并行子代理写图 / 清单 / 模型；**不会**自动深挖某个功能。

## 产出

```text
AGENTS.md                          # Agent 入口（≤300 行）
docs/agents/
  INDEX.md
  01-domains/
  02-surfaces/api-list.md
  02-surfaces/cli-and-consumers.md
  03-deep-dives/data-model.md
  03-deep-dives/layers.md
  03-deep-dives/flows/
  diagrams/*.svg
```

栈、业务域、中间件都按**目标仓库真实路径**探测。PHP Yii2 Advanced（`backend/` + `common/` + `console/`）是其中一个剖面；对不上则走 `generic-php` 或 `mixed`，禁止把别的项目的模块名抄过来。

## 非目标

- 不提供 DeepWiki / RAG 服务
- 不以本仓替代业务仓库里的 `docs/agents`（本仓只有规则）
- `codex-drift` 不自动执行 `codex-map`

## 兼容

| 环境 | 说明 |
|---|---|
| Cursor | 项目或用户 skill |
| Claude Code | `.claude/skills/` |
| 语言 | 探测规则目前偏 PHP / Yii 目录习惯；其它栈会降级并标「需人工确认」 |

## 开发

见 [CONTRIBUTING.md](CONTRIBUTING.md)。协议 [MIT](LICENSE)。

## 版本

见 [CHANGELOG.md](CHANGELOG.md)。
