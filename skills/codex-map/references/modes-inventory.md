# 清单类 mode

本文件覆盖 `diagrams` / `domains` / `surfaces` / `data` / `entrypoint`。
执行前已读 `detect.md` 与 `quality.md`。`diagrams` / `data` 还必须读 `diagrams.md` 再画 SVG。不要打开 `modes-flows.md` / `modes-full.md`。

### mode=diagrams

1. 完成「开跑前探测」；读 README、顶层目录、composer、关键 config
2. 按 `diagrams.md` **企业分层风**写 `docs/agents/diagrams/architecture.svg`：4～5 层带 + 左轨层名 + 层色实心卡片（标题/职责/技术点最多 3 行）+ 底栏一行图例；每行 ≤ 4 框；禁止深色主题、禁止右侧说明书墙
3. 按 `diagrams.md` 写 `module-deps.svg`：同样分层实心卡；只画本仓库模块/目录边界；≤ 12 框；循环用红实心卡，说明写 md 不写进图
4. 按 `diagrams.md` 写 `external-deps.svg`：三列层带分色 — 关键语言依赖、中间件、外部 API；每列 ≤ 7 项小卡
5. **内容 review**（`diagrams.md`「内容 review」）：不过则本步重画，禁止带错图进入 summary
6. 自检：路径真实 + `diagrams.md` 落盘清单 + 内容 review。仅 diagrams 则进入 summary

### mode=domains

只写域短索引，不扫全量 API。用于分步跑漏了 `01-domains`，或只想刷新域表。

1. 按 `detect.md` 定包并查官网，用「官网 ∩ 仓库」出域名单；官网失败才读对应离线节
2. 按「写 01-domains/INDEX.md」重生域表（条数暂 `—`，除非本轮已打开路由文件能直接数出来）
3. 若已有 `docs/agents/INDEX.md`：阅读顺序在架构图之后、surfaces 之前插入 `01-domains/INDEX.md`；覆盖率行改为有
4. 不改 api-list / data-model / flows / critical-paths / AGENTS.md
5. summary

### mode=surfaces

1. 探测域列表
2. **立刻**按「写 01-domains/INDEX.md」写出域短索引（不要等 API 扫完；条数可先 `—`）
3. 扫 HTTP → `docs/agents/02-surfaces/api-list.md`  
   - 每个接口：方法、路径、一句话、主入参、返回要点、**暴露**（`对外` / `内部`）  
   - **对外**：路由文件登记、给前端或第三方的 HTTP  
   - **内部**：仅内部 RPC / 仅 SDK / 未进路由的 controller 方法  
   - 按模块分组；推不出标「待确认」；长清单先放分组锚点索引表  
   - H1 元信息之后、分组清单之前，写「冒烟候选（只列不跑）」：合计 **≤5**；有登录/鉴权则必占 1 条；其余覆盖主业务**写路径**；禁止只挑最简单的 GET；本 skill **不 curl、不启动应用**
4. 扫 CLI + consumers → `cli-and-consumers.md`（没有则明确写无）
5. 回填 `01-domains/INDEX.md` 的「HTTP 条数」
6. 若带 `domain=`，只深写该域的 surfaces，其它域在域索引中保留行或标「待补」
7. 若 `data-model.md` 已存在：按 `quality.md`「资产互校」做接口↔模型对照；缺口写 api-list 文末引用块，不改业务代码
8. 自检：抽样打开文档中的文件路径；确认 `01-domains/INDEX.md` 已存在且域数量 ≥ 1；冒烟候选都能在清单里打开

### mode=data

1. 若 `docs/agents/01-domains/INDEX.md` **不存在**：按共用节补写；已存在则不动职责原文
2. **只扫 Model**（本轮官网 ∩ 仓库后的 Model 目录）。高价值域 = 探测域里入口最多或用户 `domain=` 指定的集合，不要默认某个业务名
3. 字段与表名以本轮官网 Model 规范为准（求交后的真实类），不要套其它框架 API
4. 写 `docs/agents/03-deep-dives/data-model.md`（文首写明真源是 Model，不含 migration）
5. 按 `diagrams.md` **企业分层风**写 `docs/agents/diagrams/data-model-er.svg`（白底实体 + 顶部域色条；只画 Model 声明的关系；≤ 12 实体，每表 ≤ 3 个关系字段）
6. 非核心 Model 只列索引清单，不穷尽全部类
7. 禁止打开 `console/migrations`、`migrations/` 或把 migration 当对照源
8. 若 `api-list.md` 已存在：按 `quality.md`「资产互校」做模型↔接口对照；缺口写 data-model 文末引用块

### mode=entrypoint

1. 只读已有 `docs/agents/**`（不扫全库 API/Model）。允许：按 `quality.md` 对禁区候选做定点 Grep / 可选 git log（合计 ≤8 条）
2. 写/更新 `docs/agents/INDEX.md`：阅读顺序、覆盖率、缺口、本仓库探测到的 system/剖面/四层判定。  
   **若 `01-domains/INDEX.md` 存在**：阅读顺序插在架构图之后、surfaces 之前，用链接指向它；**禁止**把域名单只写成一行顿号列表来代替该页。  
   **若 `critical-paths.md` 存在**：阅读顺序插在 surfaces 之后、data-model 之前。  
   **若 `critical-paths.md` 不存在**但已有 `api-list.md`：按 `quality.md`「核心链路」补写（≤8），再链接。已存在则只链接，不重写（用户说「刷新核心链路」除外）。  
   **若 `01-domains/INDEX.md` 不存在**但 `02-surfaces` 已有：覆盖率标缺口，summary 提示跑 `mode=domains` 或再跑 `mode=surfaces`
3. 写/更新根 `AGENTS.md`：项目定位、核心架构、关键模块、关键约定、怎么跑；详情用相对链接，不复制清单。  
   「禁区」「历史包袱」遵守 `quality.md`「禁区与历史包袱」（占位或保留人写，禁止编造）。  
   若有 `forbidden-candidates.md`，在禁区节后加一句链接「候选线索（待确认）」，不要把候选写成禁区正文。  
   业务域一节链到 `docs/agents/01-domains/INDEX.md`（文件存在时）
4. 若 `dual_claude=true`，同构写 `CLAUDE.md`（禁区两节同样纪律）
5. 互校表：若本轮能读到 api-list 与 data-model，把 `quality.md` 互校结果写入 INDEX 缺口与 summary

## 核心链路页模板（`mode=full` / entrypoint 补写时）

```markdown
# {system} 核心链路

- 系统：
- 剖面：
- 探测时间：
- 真源：（api-list / data-model / 域索引，必须是本仓库已有页）
- 条数：N（≤8）

改造时容易出问题的链路，不是覆盖率清单。不写测试、不跑应用。

| # | 链路 | 起点（方法+路径或 CLI） | 关键节点 | 成功终点 | 依据 |
|---|------|-------------------------|----------|----------|------|
```

账号详情、纯列表分页这类不要进表。推不出成功终点就写 `待确认`，禁止编造。
