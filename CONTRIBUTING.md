# Contributing

感谢贡献。本仓库是 **Agent Skill**（Markdown 指令），不是可执行程序。改规则前请先在一个**非本仓**的业务仓库里跑通对应 mode。

## 改什么

| 路径 | 职责 |
|---|---|
| `skills/codex-map/SKILL.md` | 生成 / 重生 `docs/agents/**` 与 `AGENTS.md` |
| `skills/codex-drift/SKILL.md` | 只读漂移检查 |
| `README.md` | 安装与用法 |

不要往本仓提交任何业务仓库的 `docs/agents` 产物、客户名、内网路径或密钥。

## 原则

1. **探测，不预设**：域名、栈、中间件从目标仓库路径得出。
2. **禁止编造**：推不出的契约写「待确认」。
3. **不改业务代码**：skill 只允许写文档契约路径。
4. **剖面可扩展**：新增语言/框架用「路径存在即命中」加一行，不要删已有剖面。
5. **中英文**：skill 正文可中文；对外描述（YAML `description`）用英文，便于 Agent 发现。

## 提 PR

1. Fork，从 `main` 开分支。
2. 只改与本次意图相关的文件。
3. 在 PR 里写：改了哪条规则、在哪种剖面下验证过（或未验证）。
4. 不要顺手重排无关段落。

## 发布

维护者打 Git tag：`v0.x.y`，并更新 `CHANGELOG.md`。
