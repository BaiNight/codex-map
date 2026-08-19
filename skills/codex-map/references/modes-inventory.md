# 清单类 mode

本文件覆盖 `diagrams` / `domains` / `surfaces` / `data` / `entrypoint`。
执行前已读 `detect.md` 与 `quality.md`。`diagrams` / `data` 还必须读 `diagrams.md` 再画 SVG。不要打开 `modes-flows.md` / `modes-full.md`。

### mode=diagrams

1. 完成「开跑前探测」；读 README、顶层目录、composer、关键 config
2. 按 `diagrams.md` **企业分层风**写 `docs/agents/diagrams/architecture.svg`：4～5 层带 + 左轨层名 + 层色实心卡片（标题/职责/技术点最多 3 行）+ 底栏一行图例；每行 ≤ 4 框；禁止深色主题、禁止右侧说明书墙
3. 按 `diagrams.md` 写 `module-deps.svg`：同样分层实心卡；只画本仓库模块/目录边界；≤ 12 框；循环用红实心卡，说明写 md 不写进图
4. 按 `diagrams.md` 写 `external-deps.svg`：三列层带分色 — 关键语言依赖、中间件、外部 API；每列 ≤ 7 项小卡
5. 自检：路径真实 + `diagrams.md` 落盘清单。仅 diagrams 则进入 summary

### mode=domains

只写域短索引，不扫全量 API。用于分步跑漏了 `01-domains`，或只想刷新域表。

1. 按 `detect.md` 定包并查官网，用「官网 ∩ 仓库」出域名单；官网失败才读对应离线节
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
2. **只扫 Model**（本轮官网 ∩ 仓库后的 Model 目录）。高价值域 = 探测域里入口最多或用户 `domain=` 指定的集合，不要默认某个业务名
3. 字段与表名以本轮官网 Model 规范为准（求交后的真实类），不要套其它框架 API
4. 写 `docs/agents/03-deep-dives/data-model.md`（文首写明真源是 Model，不含 migration）
5. 按 `diagrams.md` **企业分层风**写 `docs/agents/diagrams/data-model-er.svg`（白底实体 + 顶部域色条；只画 Model 声明的关系；≤ 12 实体，每表 ≤ 3 个关系字段）
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
