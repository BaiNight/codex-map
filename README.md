# codex-map

为当前仓库生成给 Agent 读的静态文档：根上的 `AGENTS.md`，以及 `docs/agents/` 里的域索引、接口清单、全景图和按功能折叠的流程图。

人看本文件。Agent 执行时只读 `SKILL.md`，再按需打开 `references/`。不要把本文件当操作手册喂给 Agent。

规范：[Agent Skills](https://agentskills.io/specification)。`name` 与目录名同为 `codex-map`。

## 目录

```text
codex-map/
├── README.md                 # 人
├── SKILL.md                  # Agent 入口
└── references/
    ├── detect.md             # 探测栈、域、四层
    ├── quality.md            # 硬约束、行号、增量更新
    ├── modes-full.md         # mode=full 的编排
    ├── modes-inventory.md    # 清单类 mode
    ├── modes-flows.md        # 先候选，再深挖
    ├── diagrams.md           # 全景 SVG 与 flow.html
    └── examples.md           # 调用例句
```

没有 `scripts/` 和 `assets/`。这个技能不靠脚本生成文档。

## 怎么用

在业务仓库里说：

```text
使用 codex-map mode=full
使用 codex-map mode=flows domain=order
使用 codex-map mode=flows domain=order feature=order-sync
```

更多例句在 `references/examples.md`。

| 参数 | 缺省 | 作用 |
|---|---|---|
| `mode` | `full` | `full` `diagrams` `domains` `surfaces` `data` `flows` `entrypoint` |
| `domain` | 无 | 只框定一个业务域。值来自本仓库探测，不抄别的项目 |
| `feature` | 无 | 拍板后才深挖这一个功能 |
| `refresh` | `off` | `off` 不覆盖已有清单；`cited` 只核对行号并标过期；`force` 才整页重写 |
| `system` | `auto` | 只改文档里的称谓 |
| `dual_claude` | 否 | `true` 时再写一份同构的 `CLAUDE.md` |

`mode=flows` 有 `domain`、没有 `feature` 时必须停下来等拍板，不能自己挑一个功能写完。

## 写出什么

```text
AGENTS.md
docs/agents/
  INDEX.md
  01-domains/INDEX.md
  02-surfaces/api-list.md
  02-surfaces/cli-and-consumers.md
  03-deep-dives/...
  diagrams/*.svg
  diagrams/flows/{domain}/{slug}/flow.html
```

仓库里没有的层不建页。深挖一个功能只出一份 `flow.html`：打开只看主步骤，点 `+` 展开子流程。

## 边界

- 只写 `docs/agents/**` 和 `AGENTS.md`。不改业务源码，不装依赖，不启动应用。
- 接口表和模型表是扫描快照。`AGENTS.md`「先查哪里」写调用关系该去哪查。
- 扫描「谁调用谁」时，会话里有 codebase-memory 就先查它。HTTP 方法、表名和行号仍以打开的源码为准。没有这个 MCP 就退回读文件，不中断。
- 深挖页的步骤要带 `路径:行号`。没有行号的判断不能画成菱形。
- 清单和全景图已经存在时，默认不整页重写。要重生就加 `refresh=force`。
- 只读的文档漂移检查用 `codex-drift`，不是这个技能。
