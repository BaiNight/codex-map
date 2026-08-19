# Changelog

## 0.1.1 — 2026-08-19

- 分步跑 `mode=surfaces` 时必写 `docs/agents/01-domains/INDEX.md`；`data` / `flows` 在该页缺失时补写
- 新增 `mode=domains`：只补域短索引，不重扫 API
- 文档版式：标题 + 表格，禁止 emoji / HTML / `<details>`，避免装饰干扰 Agent 解析

## 0.1.0 — 2026-08-19

- 首次开源：`codex-map`（生成）与 `codex-drift`（只读对账）
- 目录契约：`docs/agents/**` + 根目录 `AGENTS.md`
- `mode=flows` 两步协议：propose 候选 → 用户拍板 → deep
- `mode=full` 由主代理探测后并行子代理写图 / surfaces / data / 域索引
