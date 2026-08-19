# SVG 画图规范

写任何 `docs/agents/diagrams/**/*.svg` 前读本文件。目标：**一眼读完**。细节在 md，图只放结构。

## 原则（按优先级）

1. 留白优于塞满。宁可少画，禁止靠缩小字号或压缩间距硬塞
2. 每个节点最多 **2 行字**；第三行起删掉或写进 md
3. 禁止侧栏/底栏说明书，禁止节点里写代码、字段清单、异常堆栈、文件路径作文
4. 必须复用下方 token，禁止另起颜色
5. **禁止黑底白框**；流程图学 ProcessOn：**白底、细描边、少填色**

## 流程图外观（必须长这样）

对标常见业务流程图（白画布 + 橙菱形 + 蓝步骤），不要大色块填满：

- 画布纯白 `#ffffff`，四周留白，不要灰底、不要水印
- **开始/结束**：圆角胶囊，浅灰底 `#f3f4f6`，灰边 `#9ca3af`，黑字
- **步骤矩形**：白底（或极浅蓝 `#f8fbff`），**细蓝边** `#4a90d9`（线宽 1.6），字蓝 `#2563eb` 或黑
- **判断菱形**：白底，**细橙边** `#f5a623`，字橙 `#d97706` 或黑
- **失败**：白底，细红边 `#e85d4c`，字红 `#dc2626`
- **主箭头**：中蓝 `#4a90d9`；回边/旁路用更细的灰 `#9ca3af`
- 边上「是 / 否」小字、灰色，贴在菱形出口旁
- 角色靠**边框色**区分，不要蓝/黄/绿大色块
- 仍然禁止右侧说明书、头像、水印墙（参考图里那种备注盒不要画）

## 必须粘贴的 defs（手写 SVG / 全景图）

全景 / ER 可用极浅填，方便扫模块；**流程图按上一节描边风**，不要套这张重填色表。

```svg
<defs>
  <style>
    .bg { fill: #ffffff; }
    .title { fill: #1f2328; font: 700 20px "Segoe UI","PingFang SC","Microsoft YaHei",sans-serif; }
    .sub { fill: #656d76; font: 12px "Segoe UI","PingFang SC","Microsoft YaHei",sans-serif; }
    .sec { fill: #656d76; font: 600 11px "Segoe UI","PingFang SC","Microsoft YaHei",sans-serif; letter-spacing: .06em; }
    .n { fill: #1f2328; font: 600 13px "Segoe UI","PingFang SC","Microsoft YaHei",sans-serif; }
    .d { fill: #656d76; font: 11px "Segoe UI","PingFang SC","Microsoft YaHei",sans-serif; }
    .lane { fill: #ffffff; stroke: #e5e7eb; stroke-width: 1; }
    .arr { stroke: #4a90d9; stroke-width: 1.6; fill: none; marker-end: url(#ah); }
    .arr-side { stroke: #9ca3af; stroke-width: 1.2; fill: none; marker-end: url(#ahg); }
  </style>
  <marker id="ah" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto">
    <path d="M 0 0 L 10 5 L 0 10 z" fill="#4a90d9"/>
  </marker>
  <marker id="ahg" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto">
    <path d="M 0 0 L 10 5 L 0 10 z" fill="#9ca3af"/>
  </marker>
</defs>
```

全景 / ER 节点（流程图不要用这列重填）：

| 角色 | fill | stroke |
|---|---|---|
| 入口 / 模块 | `#f8fbff` | `#4a90d9` |
| 公共库 | `#faf5ff` | `#7c3aed` |
| 写库 / 存储 | `#f0fdf4` | `#16a34a` |
| 对外 | `#fff7ed` | `#ea580c` |
| 循环依赖 | `#fff1f2` | `#e85d4c` |
| 待确认 | `#ffffff` | `#9ca3af` 虚线 `4 3` |

## 画布与网格

| 图种 | 建议 viewBox | 硬上限 | 节点/实体上限 |
|---|---|---|---|
| architecture / module-deps / external-deps | `1200×780` | `1280×900` | 见分图 |
| flow L1 overview | 单列 `720×H` | `900×900` | **9** |
| flow L2 / L3 | 单列 `720×H`；一条旁路时 `1000×900` | `1100×1000` | L2 **14** / L3 **8** |
| ER | `1200×720` | `1280×900` | **12** 实体 |

- 页边 40。标题 `(40,36)`，副标题 `(40,56)` 一句 ≤ 48 字
- 第一泳道顶 `y=80`。泳道 `rx="10"`，内边距 ≥ 16
- 列间距 ≥ 28，行间距 ≥ 32
- 矩形节点默认 `220×56`；菱形外接约 `136×72`
- 字号禁止小于 11px。文本距框左 14px、距顶约 22px（标题）/ 40px（副行）
- 超长截断，不要换第三行：类名取短名 + `::方法`；中文 ≤ 12 字，超出用 `…`

## 节点文案

- 标题只准四种：模块名、`短类名::方法`、判断短句、`写 {table}`。流程图必须在前面加 `S1` / `S3.1`
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
- 用 mermaid 围栏代替已规定的 SVG 成品；emoji；外链字体
- 黑/深色画布（`#0d1117` / `theme: dark` / `theme: forest` 深底）
- 白框画在深底上，或深字画在深蓝框上（看不清）
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

### flows/*.svg（分层，禁止一张超长总图）

长业务按 `modes-flows.md`：**L1 全貌 + 按阶段 L2**。图上节点必须以 `S1` / `S3.1` 开头，与深页步骤表同一套编号。

| 层 | 文件名 | 画什么 | 节点 |
|---|---|---|---|
| L1 | `{domain}-{slug}-overview.svg` | 主步骤，阶段不展开 | 5～9 |
| L2 | `{domain}-{slug}-{phase}.svg` | 只展开一个阶段 | 8～14 |
| L3 | `{domain}-{slug}-{phase}-detail.svg` | 仅当 L2 仍超（全功能 ≤ 1 张） | 4～8 |

- **主路径单列**；每张图最多 1 条旁路
- L1 不要收成「入口 → 编排 → 写库」三框，但也不要画出算子/每张表
- L2 判断和写库保留；纯转发合并。pipeline 4～6 个算子 +「+N」
- 失败在 L1 是一个红框「S{n} 失败」；细节只在 `fail` 的 L2
- L2 禁止连到其它阶段内部；跨阶段只走 L1
- 图是步骤表的**子集**

#### 出图方式（流程图优先自动排版）

1. 先写同名 `{stem}.mmd`，**必须**用下面这份头（禁止 `theme: dark`，禁止大色块 fill）：

```text
%%{init: {"theme": "base", "themeVariables": {
  "primaryColor": "#ffffff", "primaryTextColor": "#1f2328",
  "primaryBorderColor": "#4a90d9", "lineColor": "#4a90d9",
  "secondaryColor": "#ffffff", "tertiaryColor": "#ffffff",
  "background": "#ffffff"
}}}%%
flowchart TD
  classDef start fill:#f3f4f6,stroke:#9ca3af,color:#1f2328
  classDef orch fill:#ffffff,stroke:#4a90d9,color:#2563eb
  classDef decide fill:#ffffff,stroke:#f5a623,color:#d97706
  classDef write fill:#ffffff,stroke:#4a90d9,color:#2563eb
  classDef ext fill:#ffffff,stroke:#4a90d9,color:#2563eb
  classDef fail fill:#ffffff,stroke:#e85d4c,color:#dc2626
```

2. 开始/结束用 `((开始))` / `((结束))` 并挂 `:::start`；步骤 `:::orch`；判断 `{是否…?}:::decide`
3. 节点 ID 用 `S1` / `S3_1`，标签显示 `S3.1 …`
4. 渲染：Mermaid MCP 用 `theme=default`、`background=#ffffff` / `mmdc` / Kroki。**禁止** `theme=dark`
5. 渲完自检：白底；判断是白心橙边；步骤是白心蓝边；箭头是蓝的。出现大黄/大绿/大蓝色块或黑底 → 改 `.mmd` 再渲
6. 没有渲染器：手写 SVG 按「流程图外观」
7. 深页只链 `.svg`

## 落盘前自检（全过才写文件）

- 流程图：白底、橙边菱形、蓝边矩形、蓝箭头；没有大色块、没有黑底
- 全景 / ER：白底 + 极浅填；defs 与本文件一致
- 无 3 行字节点、无 `$` 代码、无字段清单、无侧栏墙
- 相邻节点间距 ≥ 28；文字没有出框
- L1 ≤ 9、L2 ≤ 14、L3 ≤ 8；ER ≤ 12 实体；全景图 ≤ 5 泳道且每行 ≤ 4 框
- 流程图节点以 `S*` 开头；无一张覆盖全链路的细图
- 连线正交且不压字（手写时）
- viewBox 未超过本文件硬上限
