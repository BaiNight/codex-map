# Codex Map Skill 渐进式披露

日期：2026-08-19  
状态：已确认  
仓库：`codex-map`（开源 skill，非业务仓）

把 `skills/codex-map/SKILL.md`（约 600 行）拆成薄入口 + 按 mode 加载的 `references/`，对齐 Anthropic Agent Skills 规范。生成物契约、`mode=` 语义、`codex-drift` 职责不变。

## 1. 背景

当前全部探测规则、各 mode 步骤、flows 两步、full 子代理、版式与质量门都写在一份 `SKILL.md` 里。代理每次触发都会整份进上下文。

业界同类（DeepWiki / zread / agent-ready / GSD map-codebase / GitNexus）不能替换本 skill 的清单 + 深潜能力。可借鉴的只有一条：**入口短、细则按需读。**

已确认：

- 拆分粒度：一个 skill，薄 `SKILL.md` 调度，五个 `references/` 文件
- 不拆成多个 skill，不加 CLI，不改 `docs/agents/` 产出
- 对外文档与 skill 正文 **禁止出现 OMS 时期的内部旧名**（见 §7）

## 2. 非目标

- 不改 `mode` 集合与 propose/deep 协议
- 不改目录契约 `docs/agents/**`、`AGENTS.md`
- 不把 `codex-drift` 并入 `codex-map`
- 不重跑、不改业务仓库里已生成的 `docs/agents/` 内容
- 不引入图谱 MCP、不补 conventions/testing 覆盖面（其它调研项，不在本次）

## 3. 目录

```text
skills/codex-map/
  SKILL.md                      # 入口，≤ 150 行
  examples.md                   # 对话示例，保留
  references/
    detect.md                   # 开跑前探测 + F/L/V/X + 写 01-domains
    modes-inventory.md          # diagrams / domains / surfaces / data / entrypoint
    modes-flows.md              # propose → 停 → deep
    modes-full.md               # 子代理编排
    quality.md                  # 版式 + 质量门 + 降级 + summary 模板
```

`skills/codex-drift/` 结构本次不拆。

## 4. 加载协议

运行时已加载 `SKILL.md`。`SKILL.md` 用表写死 mode → 文件。**未点到的 references 本轮禁止打开。** 生成类 mode 一律再读 `quality.md`（SKILL 里只有 8 条硬约束摘要，细则以 `quality.md` 为准）。

| 用户 mode | 本轮再读 |
|---|---|
| `diagrams` / `domains` / `surfaces` / `data` / `entrypoint` | `detect.md` + `modes-inventory.md` + `quality.md` |
| `flows` | `detect.md` + `modes-flows.md` + `quality.md` |
| `full` | `detect.md` + `modes-full.md` + `quality.md` |

`full` 的子代理 prompt 只贴该子代理允许写的文件与对应 mode 步骤，不把另外四个 references 全文塞进每个子代理。

`examples.md` 仅人读 / 用户问用法时打开，生成时不强制读。

## 5. SKILL.md 正文

YAML `description`（英文）保持可发现：生成/重生、各 mode 名、`domain=` / `feature=`、`codex-drift` 分流。**删除任何历史别名。**

正文 ≤ 150 行，只留：

1. 一句话定位；只写 `docs/agents/**`、`AGENTS.md`（`dual_claude=true` 才写 `CLAUDE.md`）
2. 参数表（空格分隔；逗号也能认，示例只写空格）
3. 硬约束 8 条摘要
4. 目录契约树（不展开写法）
5. mode → 读哪个文件
6. flows 两步协议一张表（propose 必须停；有 `feature=` 才 deep）
7. 「未点到的 references 本轮不要打开」

不进 SKILL.md：探测细则、各 mode 步骤、01-domains 模板、版式细则、质量门全文、降级、summary 模板全文、子代理波次。

## 6. references 边界

从现有 `SKILL.md` **搬移原文，不改语义**。

| 文件 | 迁入 | 禁止写入 |
|---|---|---|
| `detect.md` | 剖面、域名单、扫描真源、F/L/V/X、写 `01-domains/INDEX.md` 的时机与模板 | 扫 API、画全景 SVG、propose/deep 步骤 |
| `modes-inventory.md` | `diagrams` / `domains` / `surfaces` / `data` / `entrypoint` 步骤 | flows 两步、full 子代理 |
| `modes-flows.md` | 功能聚类、打分、propose/deep 产出、骨架链、细流程图节点规则 | 全景三图、api-list |
| `modes-full.md` | 写文件边界、波次、B 不写 01-domains、失败重试 | 重复粘贴 inventory/flows 全文；只写「按某文件做」 |
| `quality.md` | 文档版式、质量门、降级、summary 模板 | 业务探测、mode 步骤 |

## 7. 命名与对外文档

- skill 对外只称 `codex-map` / `codex-drift`
- README、SKILL.md、examples.md、CHANGELOG、CONTRIBUTING、sample-output **禁止出现** 字符串 `docs-bootstrap`
- YAML 不得写 `Alias: docs-bootstrap`
- 不得写「对话里的旧名视为本 skill」
- 实现时对仓库 `grep docs-bootstrap`，命中则删或改成 `codex-map`
- 同步到业务仓时，skill 目录名必须是 `codex-map` / `codex-drift`，不要保留旧目录名，也不要在任何 md 里解释旧名

`codex-drift` 是否去掉它自己的历史别名：本次 **只处理 map 的 OMS 旧名**；drift 原文不动，除非同一文件为删 map 旧名而碰到。

## 8. 其它文档

- `README.md`：安装路径仍是 `skills/codex-map/`；不需要列出 references 文件名（那是代理加载细节）
- `CONTRIBUTING.md`：改什么表加上 `skills/codex-map/references/*`
- `CHANGELOG.md`：记 0.1.2 拆分，不写旧内部名
- `examples.md`：例句仍用 `使用 codex-map mode=…`

## 9. 验收

- `SKILL.md` ≤ 150 行；五个 references 都存在且从现 skill 搬出、语义不丢
- 每个 mode 在 SKILL 调度表里能对应到要读的文件
- 除本 spec §7 的检索词外，仓库内其它文件检索不到 `docs-bootstrap`
- `codex-drift` 仍可单独使用
- 已有 `docs/agents/` 产物不被本变更改写

## 10. 实现顺序（spec 级，非计划细节）

1. 从现 `SKILL.md` 剪出五个 references（先搬后删，禁止边搬边改规则）
2. 重写薄 `SKILL.md` + 调度表
3. 清旧名、改 CONTRIBUTING / CHANGELOG
4. 静态验收：行数、调度表、全文检索旧名
