# Changelog

## 0.1.3 — 2026-08-19

- 新增 `references/diagrams.md`：统一 SVG token、网格与留白；图只放结构，细节留在 md
- 全景图禁止侧栏说明书；流程图节点上限 16，ER 实体上限 12

## 0.1.2 — 2026-08-19

- 将 `codex-map` 拆成薄 `SKILL.md` + `references/`，按 mode 按需加载
- 对外文档不再使用历史内部 skill 名

## 0.1.1 — 2026-08-19

- 分步跑 `mode=surfaces` 时必写 `docs/agents/01-domains/INDEX.md`；`data` / `flows` 在该页缺失时补写
- 新增 `mode=domains`：只补域短索引，不重扫 API
- 文档版式：标题 + 表格，禁止 emoji / HTML / `<details>`，避免装饰干扰 Agent 解析

## 0.1.0 — 2026-08-19

- 首次开源：`codex-map`（生成）与 `codex-drift`（只读对账）
- 目录契约：`docs/agents/**` + 根目录 `AGENTS.md`
- `mode=flows` 两步协议：propose 候选 → 用户拍板 → deep
- `mode=full` 由主代理探测后并行子代理写图 / surfaces / data / 域索引
