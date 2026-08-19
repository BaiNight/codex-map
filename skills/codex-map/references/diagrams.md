# SVG 画图规范

写任何 `docs/agents/diagrams/**/*.svg` 前读本文件。目标：**一眼读完**。细节在 md，图只放结构。

## 原则（按优先级）

1. 留白优于塞满。宁可少画，禁止靠缩小字号或压缩间距硬塞
2. 每个节点最多 **2 行字**；第三行起删掉或写进 md
3. 禁止侧栏/底栏说明书，禁止节点里写代码、字段清单、异常堆栈、文件路径作文
4. 必须复用下方 token，禁止另起颜色或字体

## 必须粘贴的 defs

每张图 `<defs>` 原样使用（可按需加减 marker id，颜色不要改）：

```svg
<defs>
  <style>
    .bg { fill: #0d1117; }
    .title { fill: #e6edf3; font: 700 20px "Segoe UI","PingFang SC","Microsoft YaHei",sans-serif; }
    .sub { fill: #8b949e; font: 12px "Segoe UI","PingFang SC","Microsoft YaHei",sans-serif; }
    .sec { fill: #8b949e; font: 600 11px "Segoe UI","PingFang SC","Microsoft YaHei",sans-serif; letter-spacing: .06em; }
    .n { fill: #e6edf3; font: 600 13px "Segoe UI","PingFang SC","Microsoft YaHei",sans-serif; }
    .d { fill: #8b949e; font: 11px "Segoe UI","PingFang SC","Microsoft YaHei",sans-serif; }
    .lane { fill: #161b22; stroke: #30363d; stroke-width: 1; }
    .arr { stroke: #484f58; stroke-width: 1.4; fill: none; marker-end: url(#ah); }
  </style>
  <marker id="ah" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto">
    <path d="M 0 0 L 10 5 L 0 10 z" fill="#484f58"/>
  </marker>
</defs>
```

节点填色（框 `rx="8"`）：

| 角色 | fill | stroke |
|---|---|---|
| 入口 / HTTP / CLI / MQ | `#12233a` | `#388bfd` |
| 编排 / 模块 | `#1f2a37` | `#79c0ff` |
| 判断菱形 | `#161b22` | `#d29922` |
| 写库 | `#12261c` | `#3fb950` |
| 对外 / 平台 API | `#2a1f14` | `#f0883e` |
| 公共库 | `#241c33` | `#a371f7` |
| 循环依赖 | `#2a1616` | `#f85149` |
| 待确认 / 未引用 | 同上 fill | `#6e7681` 虚线 `4 3` |

## 画布与网格

| 图种 | 建议 viewBox | 硬上限 | 节点/实体上限 |
|---|---|---|---|
| architecture / module-deps / external-deps | `1200×780` | `1280×900` | 见分图 |
| flow | 单列 `720×H`；一条旁路时 `1000×900` | `1100×1000` | **16** |
| ER | `1200×720` | `1280×900` | **12** 实体 |

- 页边 40。标题 `(40,36)`，副标题 `(40,56)` 一句 ≤ 48 字
- 第一泳道顶 `y=80`。泳道 `rx="10"`，内边距 ≥ 16
- 列间距 ≥ 28，行间距 ≥ 32
- 矩形节点默认 `220×56`；菱形外接约 `136×72`
- 字号禁止小于 11px。文本距框左 14px、距顶约 22px（标题）/ 40px（副行）
- 超长截断，不要换第三行：类名取短名 + `::方法`；中文 ≤ 12 字，超出用 `…`

## 节点文案

- 标题只准四种：模块名、`短类名::方法`、判断短句、`写 {table}`
- 副行：一句话职责。禁止 `$var`、禁止参数列表、禁止路径、禁止 `catch` 代码
- 边标签：`是` / `否` / 取值，≤ 6 字
- 图例最多一行，放标题右侧；不要另开说明墙

## 连线

- 只走正交折线，线宽 1.4
- 箭头只画终点；同层禁止交叉。旁路走泳道外侧 16px 走廊
- 禁止斜线穿过文字或节点
- 多个写库动作汇成 **1 个**「写库」节点，不要每个表一根线

## 禁止

- 右侧或底部长文注释墙
- 流程图超过两列（主路径 + 至多 1 条旁路）
- 一个框 ≥ 3 行字
- 把 viewBox 拉到 1600+ 再往里塞
- mermaid 替代 SVG；浅色主题；emoji；外链字体
- 为塞字把字号收到 10px 以下

## 分图配方

### architecture.svg

- 自上而下 **4～5** 条泳道：调用方 → 入口 → 核心模块 → 公共/存储 → 外部
- 每行 ≤ **4** 框
- 核心模块只画入口最多的 6～8 个，其余一个「其它 N」
- 日志 / 追踪 / 进程 / 认证 **不要**竖栏；需要时并进「运行时」一框
- 存储按类合并：`MySQL ×N`、`Redis`、`MQ`，不要十几个小方块

### module-deps.svg

- 按目录分层，≤ **12** 框
- 循环：红边 + 标题旁「红 = 循环」。use 例子只写 md
- 禁止半页「循环说明」面板

### external-deps.svg

- 三列等宽：语言依赖 / 中间件 / 外部 API
- 每列 ≤ **7** 项，其余 `+N 见 md`
- 列宽约 360，项高 48，项间距 12

### data-model-er.svg

- ≤ 12 张核心表（入口最多的域，或用户 `domain=`）
- 每表：表名 + ≤ 3 个关系字段；不列 `created_at` / `updated_at` / `tenant_id` 等审计列
- 关系正交，边上标 `1` / `N`
- 其余表只在 `data-model.md` 做索引

### flows/*.svg

- **主路径单列自上而下**；最多再开 1 条右侧旁路（失败或对照 V）
- 不要收成「入口 → 编排 → 写库」三框；判断和写库要保留
- 仍超 16：合并纯转发。pipeline 只画 4～6 个算子 +「+N 见细节页」
- 失败：一个红框「失败：补偿 / ACK」，不要抄 `catch` 正文
- 图是细节页的**子集**：图上每个判断/写库能在表里对上；表可以更细

## 落盘前自检（全过才写文件）

- defs 与本文件 token 一致
- 无 3 行字节点、无 `$` 代码、无字段清单、无侧栏墙
- 相邻节点间距 ≥ 28；文字没有出框
- 流程图 ≤ 16 节点；ER ≤ 12 实体；全景图 ≤ 5 泳道且每行 ≤ 4 框
- 连线正交且不压字
- viewBox 未超过本文件硬上限
