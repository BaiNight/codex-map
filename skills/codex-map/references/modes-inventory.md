# 清单类 mode

本文件覆盖 `diagrams` / `domains` / `surfaces` / `data` / `entrypoint`。
执行前已读 `detect.md` 与 `quality.md`。不要打开 `modes-flows.md` / `modes-full.md`。

### mode=diagrams

1. 完成「开跑前探测」；读 README、顶层目录、composer、关键 config
2. 写 `docs/agents/diagrams/architecture.svg`：按本仓库分层；核心模块「名 + 一句话」；基础设施一个方框
3. 写 `module-deps.svg`：只画本仓库模块/目录边界；循环依赖红色
4. 写 `external-deps.svg`：三类分色 — 关键语言依赖、中间件、外部 API
5. 自检路径与分层后进入 summary（若仅 diagrams）

### mode=domains

只写域短索引，不扫全量 API。用于分步跑漏了 `01-domains`，或只想刷新域表。

1. 完成开跑前探测第 1～3 节
2. 按「写 01-domains/INDEX.md」重生域表（条数暂 `—`，除非本轮已打开路由文件能直接数出来）
3. 若已有 `docs/agents/INDEX.md`：阅读顺序在架构图之后、surfaces 之前插入 `01-domains/INDEX.md`；覆盖率行改为有
4. 不改 api-list / data-model / flows / AGENTS.md
5. summary

### mode=surfaces

1. 探测域列表
2. **立刻**按「写 01-domains/INDEX.md」写出域短索引（不要等 API 扫完；条数可先 `—`）
3. 扫 HTTP → `docs/agents/02-surfaces/api-list.md`（方法、路径、一句话、主入参、返回要点；按模块分组；推不出标「待确认」；长清单先放分组锚点索引表）
4. 扫 CLI + consumers → `cli-and-consumers.md`（没有则明确写无）
5. 回填 `01-domains/INDEX.md` 的「HTTP 条数」
6. 若带 `domain=`，只深写该域的 surfaces，其它域在域索引中保留行或标「待补」
7. 自检：抽样打开文档中的文件路径；确认 `01-domains/INDEX.md` 已存在且域数量 ≥ 1

### mode=data

1. 若 `docs/agents/01-domains/INDEX.md` **不存在**：按共用节补写；已存在则不动职责原文
2. **只扫 Model**（当前剖面下的 Model 目录）。高价值域 = 探测域里入口最多或用户 `domain=` 指定的集合，不要默认某个业务名
3. 字段与说明来自 `rules()` / `attributeLabels()` / 属性注释（或其它栈等价物）；关系来自关系方法；表名来自 `tableName()` 或等价声明
4. 写 `docs/agents/03-deep-dives/data-model.md`（文首写明真源是 Model，不含 migration）
5. 写 `docs/agents/diagrams/data-model-er.svg`（只画 Model 声明的关系）
6. 非核心 Model 只列索引清单，不穷尽全部类
7. 禁止打开 `console/migrations`、`migrations/` 或把 migration 当对照源

### mode=entrypoint

1. 只读已有 `docs/agents/**`（不扫全库）
2. 写/更新 `docs/agents/INDEX.md`：阅读顺序、覆盖率、缺口、本仓库探测到的 system/剖面/四层判定。  
   **若 `01-domains/INDEX.md` 存在**：阅读顺序插在架构图之后、surfaces 之前，用链接指向它；**禁止**把域名单只写成一行顿号列表来代替该页。  
   **若该页不存在**但 `02-surfaces` 已有：覆盖率标缺口，summary 提示跑 `mode=domains` 或再跑 `mode=surfaces`
3. 写/更新根 `AGENTS.md`：项目定位、核心架构、关键模块、关键约定、怎么跑；  
   「禁区」「历史包袱」两节写「待团队补充」；业务域一节链到 `docs/agents/01-domains/INDEX.md`（文件存在时）
4. 若 `dual_claude=true`，同构写 `CLAUDE.md`
