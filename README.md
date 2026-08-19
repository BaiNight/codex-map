# Codex Map

给 Agent 画代码地图：探测当前仓库，生成可被 AI 直接阅读的静态文档（`docs/agents/` + `AGENTS.md`）。

这是 **Cursor / Claude Code 的 Agent Skill**，不是命令行工具。装到业务仓库后，在对话里点名使用即可。

## 两个 Skill

| Skill | 做什么 | 写文件？ |
|---|---|---|
| [`codex-map`](skills/codex-map/SKILL.md) | 生成架构图、接口与命令清单、数据模型、功能流程与流程图 | 只写 `docs/agents/**`、`AGENTS.md`（可选再写一份 `CLAUDE.md`） |
| [`codex-drift`](skills/codex-drift/SKILL.md) | 对照代码变更，报告文档是否过期 | **只读**，不改任何文件 |

`codex-map` 不修改业务源码。从代码推不出的内容会标「待确认」，不会编造。

## 示例产出（临时）

一次真实生成结果放在 [examples/](examples/README.md)，方便对照成品。看完后删除 `examples/` 即可。

## 安装

克隆本仓库后，把两个 skill 目录拷到业务仓库（或个人全局 skill 目录）。

```bash
git clone https://github.com/BaiNight/codex-map.git
```

**Cursor（项目内）：**

```bash
cp -r codex-map/skills/codex-map   <业务仓库>/.cursor/skills/
cp -r codex-map/skills/codex-drift <业务仓库>/.cursor/skills/
```

**Claude Code（项目内）：**

```bash
cp -r codex-map/skills/codex-map   <业务仓库>/.claude/skills/
cp -r codex-map/skills/codex-drift <业务仓库>/.claude/skills/
```

**个人全局（所有项目可用）：**

- Cursor：`~/.cursor/skills/`（Windows：`%USERPROFILE%\.cursor\skills\`）
- Claude Code：`~/.claude/skills/`

同样拷入 `codex-map` 与 `codex-drift` 两个目录。

## 用法

在**业务仓库**的 Agent 对话里说明：

```text
使用 codex-map mode=full
```

也可分步：`mode=diagrams` → `surfaces` → `data` → `entrypoint`。

梳理某一业务域的复杂功能时，分两步：

```text
使用 codex-map mode=flows domain=<本仓库探测到的域>
```

先给出 3～5 个候选，等你选定后再深写：

```text
使用 codex-map mode=flows domain=<域> feature=<功能名>
```

检查文档是否过期：

```text
使用 codex-drift 相对 main 检查文档漂移
```

| 参数 | 默认 | 说明 |
|---|---|---|
| `mode` | `full` | `full` \| `diagrams` \| `surfaces` \| `data` \| `flows` \| `entrypoint` |
| `system` | `auto` | 系统显示名；可省略，由仓库 README / 包名 / 目录名推断 |
| `domain` | 无 | 只处理当前仓库里探测到的某一个业务域 |
| `feature` | 无 | 指定要深写的功能；未指定时只出候选，不自动深写 |
| `dual_claude` | 关 | 为 `true` 时额外写一份与 `AGENTS.md` 内容相同的 `CLAUDE.md` |

`mode=full` 会生成全景图和清单，**不会**自动深写某个功能。

## 产出

```text
AGENTS.md
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

业务划分、入口和依赖以**当前仓库里实际存在的路径**为准。

## 非目标

- 本仓库只提供 skill 规则，不存放某个业务项目的文档成品
- `codex-drift` 只报告问题，不会自动重跑 `codex-map`

## 兼容

| 环境 | 说明 |
|---|---|
| Cursor | 项目 skill 或用户 skill |
| Claude Code | `.claude/skills/` |

## 开发

见 [CONTRIBUTING.md](CONTRIBUTING.md)。协议 [MIT](LICENSE)。版本见 [CHANGELOG.md](CHANGELOG.md)。
