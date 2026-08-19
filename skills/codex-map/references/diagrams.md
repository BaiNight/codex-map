# SVG 画图规范

写任何 `docs/agents/diagrams/**/*.svg` 前读本文件。**两套外观，禁止混用。**

| 图种 | 外观 | 一句话 |
|---|---|---|
| `architecture.svg` / `module-deps.svg` / `external-deps.svg` / `data-model-er.svg` | **企业分层风** | 浅灰底、层色实心卡片、左轨层名、底栏一行图例 |
| `diagrams/flows/**` | **ProcessOn 风** | 白底、细描边、橙菱形、蓝步骤；不要改 |

细节仍在 md。图只放结构，禁止右侧说明书墙。

## 原则（按优先级）

1. 留白优于塞满。宁可少画，禁止靠缩小字号或压缩间距硬塞
2. **全景图**每个卡片最多 **3 行**（标题 / 职责 / 技术点）；**流程图**最多 **2 行**（`S*` + 短句）
3. 禁止右侧/半页说明书；禁止节点里写代码、字段清单、异常堆栈、路径作文
4. 必须复用本文件 token，禁止另起颜色、禁止 GitHub 深色主题（`#0d1117`）
5. 全景图**必须**用层色实心卡片（对标企业架构图）；流程图**禁止**大色块填满

---

## 架构图外观（企业分层风）

对标常见企业架构图：浅灰画布 + 横向色带 + 实心圆角卡片 + 左轨层名 + 底栏图例。

- 画布浅灰 `#f4f6f8`，**不要**纯白、**不要**深色
- **层带**：横向圆角底（`rx="12"`），淡色铺底；左侧 **52px 色轨**写层名（可 `-90°` 竖排）
- **主卡片**：层色**实心填充** + **白字**，`rx="10"`
- **外部系统**：白底 + 紫描边 2px + 深字
- **旁路存储 / 待确认**：浅黄底 + 橙虚线边 + 深字
- **循环依赖**（仅 module-deps）：红实心 `#c0392b` + 白字
- **主箭头**：实线深灰 `#4a5568`（数据/控制流，自上而下）
- **辅线**：虚线浅灰 `#94a3b8`（依赖、读取配置、落盘）
- **底栏图例一行**：色块=层角色；实线=数据流；虚线=依赖/存储。不要第二行说明文
- 必须手写 SVG。**禁止** mermaid 渲这四张图（渲不出实心卡片 + 左轨）

### 层色（必须用这张表，禁止另起）

| 角色 | 卡片 fill | 层带 fill | 左轨 fill | 用于 |
|---|---|---|---|---|
| 接入 | `#5b4b8a` | `#eee8f7` | `#d5c8ec` | 调用方、HTTP、CLI、定时 |
| 引擎 | `#e67e22` | `#fff4e5` | `#ffe0b2` | 运行时、统一入口、编排中枢 |
| 核心 | `#c0392b` | 同引擎带 | 同引擎轨 | 每图最多 1～2 个最关键框 |
| 能力 | `#1a9b7c` | `#e6f6f2` | `#b2dfdb` | 业务模块、核心能力 |
| 公共 | `#6d4c9f` | `#f3eef8` | `#d7c6ef` | 公共库、跨模块封装 |
| 基础 | `#1e6bb8` | `#e8f1fa` | `#bbdefb` | 存储、配置、会话、infra |
| 外部 | `#ffffff` 描边 `#7c6bb0` 2px | 可挂在能力带右侧 | — | 外部 API / 兄弟系统；字用深色类 |
| 旁路 | `#fff3cd` 描边 `#f5a623` 虚线 `5 3` | 可挂在基础带右侧 | — | 文件系统、仅配置未引用；字用深色类 |
| 循环 | `#c0392b` | — | — | 仅 module-deps 的环节点 |

白字卡片用 `.cn` / `.cs` / `.cd`；白底/浅黄卡片用 `.cn-d` / `.cs-d`。

### 必须粘贴的 defs（全景 / ER）

```svg
<defs>
  <style>
    .bg { fill: #f4f6f8; }
    .title { fill: #1a1d26; font: 700 22px "Segoe UI","PingFang SC","Microsoft YaHei",sans-serif; }
    .sub { fill: #5c6570; font: 12px "Segoe UI","PingFang SC","Microsoft YaHei",sans-serif; }
    .rail { fill: #3d4654; font: 700 13px "Segoe UI","PingFang SC","Microsoft YaHei",sans-serif; }
    .cn { fill: #ffffff; font: 700 14px "Segoe UI","PingFang SC","Microsoft YaHei",sans-serif; }
    .cs { fill: #ffffff; fill-opacity: .84; font: 11px "Segoe UI","PingFang SC","Microsoft YaHei",sans-serif; }
    .cd { fill: #ffffff; fill-opacity: .7; font: 10px "Segoe UI","PingFang SC","Microsoft YaHei",sans-serif; }
    .cn-d { fill: #1a1d26; font: 700 14px "Segoe UI","PingFang SC","Microsoft YaHei",sans-serif; }
    .cs-d { fill: #5c6570; font: 11px "Segoe UI","PingFang SC","Microsoft YaHei",sans-serif; }
    .leg { fill: #5c6570; font: 11px "Segoe UI","PingFang SC","Microsoft YaHei",sans-serif; }
    .arr { stroke: #4a5568; stroke-width: 1.8; fill: none; marker-end: url(#ah); }
    .arr-dash { stroke: #94a3b8; stroke-width: 1.4; fill: none; stroke-dasharray: 6 4; marker-end: url(#ahg); }
  </style>
  <marker id="ah" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto">
    <path d="M 0 0 L 10 5 L 0 10 z" fill="#4a5568"/>
  </marker>
  <marker id="ahg" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto">
    <path d="M 0 0 L 10 5 L 0 10 z" fill="#94a3b8"/>
  </marker>
  <marker id="ahr" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto">
    <path d="M 0 0 L 10 5 L 0 10 z" fill="#c0392b"/>
  </marker>
</defs>
```

### 卡片骨架（复制后改坐标与层色）

```svg
<g>
  <rect x="96" y="120" width="220" height="86" rx="10" fill="#5b4b8a"/>
  <text class="cn" x="112" y="146">HTTP API</text>
  <text class="cs" x="112" y="166">对外 REST 入口</text>
  <text class="cd" x="112" y="184">真实目录或技术点</text>
</g>
```

第三行没有就删掉，卡片高度改 `68`。禁止第四行。

### 层带 + 左轨骨架

```svg
<rect x="24" y="88" width="1392" height="156" rx="12" fill="#eee8f7"/>
<rect x="24" y="88" width="52" height="156" rx="12" fill="#d5c8ec"/>
<rect x="48" y="88" width="28" height="156" fill="#d5c8ec"/>
<text class="rail" transform="rotate(-90,50,166)" x="50" y="166">接入层</text>
```

层名用本仓库真实分层（探测出来的叫法），不要抄「Oryx / Agent」这类无关词。典型映射：调用方→接入层，运行时/编排→引擎层，业务模块→能力层，存储/配置→基础层。

### 底栏图例（必须有，只准一行）

画在 `height - 48` 附近：6～8 个 12×12 色块 + 短标签，再加「实线 数据流 / 虚线 依赖」。不要另开说明墙。

---

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
- 仍然禁止右侧说明书、头像、水印墙

### 流程图 defs（仅 flows）

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

---

## 画布与网格

| 图种 | 建议 viewBox | 硬上限 | 节点/实体上限 |
|---|---|---|---|
| architecture / module-deps / external-deps | `1440×960` | `1600×1100` | 见分图 |
| flow L1 overview | 单列 `720×H` | `900×900` | **9** |
| flow L2 / L3 | 单列 `720×H`；一条旁路时 `1000×900` | `1100×1000` | L2 **14** / L3 **8** |
| ER | `1440×860` | `1600×1000` | **12** 实体 |

全景图：

- 页边 24。标题 `(32,36)`，副标题 `(32,56)` 一句 ≤ 56 字
- 第一层带顶 `y=80`。层带之间间距 16；层带 `rx="12"`
- 左轨宽 52；卡片区从 `x=92` 起
- 卡片默认 `220×86`（2 行高改 `68`）；列间距 ≥ 20，行间距 ≥ 16
- 字号禁止小于 10px（仅第三行技术点可用 10）；标题 14、职责 11
- 超长截断：中文 ≤ 14 字，超出用 `…`

流程图：

- 页边 40。标题 `(40,36)`
- 列间距 ≥ 28，行间距 ≥ 32
- 矩形节点默认 `220×56`；菱形外接约 `136×72`
- 字号禁止小于 11px

## 节点文案

**全景图卡片**

- 第 1 行：模块/入口/存储短名
- 第 2 行：一句话职责
- 第 3 行（可选）：真实技术点，如 `Nginx + PHP-FPM`、`MySQL ×2`、`modules/v1`
- 禁止 `$var`、禁止参数列表、禁止 `catch`、禁止把路径写成作文

**流程图**

- 标题只准：`S1` / `S3.1` + 模块名或 `短类名::方法` 或判断短句或 `写 {table}`
- 副行：一句话。边标签：`是` / `否` / 取值，≤ 6 字

## 连线

- 只走正交折线
- 全景图主箭头线宽 1.8，辅线 1.4 虚线；流程图主箭头 1.6
- 箭头只画终点；同层禁止交叉
- 禁止斜线穿过文字或节点
- 多个写库动作汇成 **1 个**「写库」节点，不要每个表一根线

## 禁止

- 右侧或底部长文注释墙（底栏一行图例除外）
- GitHub 深色画布（`#0d1117` / `#161b22` / `theme: dark`）
- 全景图画成白底浅描边（那是流程图，不是架构图）
- 流程图用层色实心大卡片（那是架构图，不是流程图）
- 流程图超过两列（主路径 + 至多 1 条旁路）
- 全景卡片 ≥ 4 行字；流程节点 ≥ 3 行字
- 用 mermaid 代替这四张全景 SVG
- emoji；外链字体
- 为塞字把字号收到 10px 以下（流程图 11px 以下）

---

## 分图配方

### architecture.svg

自上而下 **4～5** 条层带（名称按本仓库探测，不要虚构）：

1. 接入层：调用方 / HTTP / CLI / 定时（紫卡，每行 ≤ 4）
2. 引擎层：运行时入口 + 编排中枢（橙卡；最关键 1 个可用红「核心」）
3. 能力层：入口最多的 6～8 个业务模块（绿卡）+ 右侧可挂「外部系统」白底紫边
4. 基础层：存储按类合并（`MySQL ×N`、`Redis`、`MQ`）蓝卡；仅配置未引用的用旁路黄虚线卡
5. 若公共库与能力明显分层，可在能力与基础之间插「公共层」紫卡

- 日志 / 追踪 / 进程 / 认证 **不要**竖栏；需要时并进基础层「运行时」一卡
- 层与层之间画 **1 条**向下主箭头（可对中），不要每个卡片一根线
- 能力层内部不要画成蜘蛛网
- 标题：`{系统名} 整体架构`

### module-deps.svg

- 按目录分层，同样用层带 + 实心卡片，≤ **12** 框
- 单向依赖：实线深灰箭头
- 循环：红实心卡 + 红箭头；标题旁或图例写「红 = 循环」
- use 例子、改代码注意 **只写 md**，禁止半页「循环说明」面板

### external-deps.svg

- 三列等宽层带：语言依赖（紫卡）/ 中间件（蓝或绿卡）/ 外部 API（橙卡或白底紫边）
- 列头用该列主色实心条 + 白字
- 每列 ≤ **7** 项，其余 `+N 见 md`；项是小卡 `高 64`（标题+一句）
- 列宽约 440，列间距 16
- 待确认项用旁路黄虚线卡，不要另写说明墙

### data-model-er.svg

- ≤ 12 张核心表（入口最多的域，或用户 `domain=`）
- 实体：白底卡片 + **顶部 28px 色条**写表名（白字）；色条色按域：能力绿 / 接入紫 / 基础蓝 / 退款橙 / 费用公共紫
- 色条下方 ≤ 3 个关系字段（深字 `.cs-d`）；不列 `created_at` / `updated_at` / `tenant_id`
- 关系正交，边上标 `1` / `N`
- 底栏一行图例：色条=域，箭头=逻辑关联
- 其余表只在 `data-model.md` 做索引

### flows/*.svg（分层，禁止一张超长总图）

长业务按 `modes-flows.md`：**L1 全貌 + 按阶段 L2**。图上节点必须以 `S1` / `S3.1` 开头，与深页步骤表同一套编号。

目录：`docs/agents/diagrams/flows/{domain}/{slug}/`。文件名 `{nn}-{kind}.svg`，`01` 起连续编号。

| 层 | 文件名 | 画什么 | 节点 |
|---|---|---|---|
| L1 | `01-overview.svg` | 主步骤，阶段不展开 | 5～9 |
| L2 | `{nn}-{phase}.svg` | 只展开一个阶段 | 8～14 |
| L3 | `{nn}-{phase}-detail.svg` | 仅当 L2 仍超（全功能 ≤ 1 张） | 4～8 |

禁止把 SVG 平铺在 `diagrams/flows/` 根目录。

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

---

## 落盘前自检（全过才写文件）

- **全景 / ER**：浅灰底 `#f4f6f8`；层色实心卡片 + 白字；有左轨层名；有且仅有一行底栏图例；defs 用「架构图」那份
- **全景 / ER 禁**：`#0d1117` 深底、白底浅描边小框、右侧说明书墙
- **流程图**：白底、橙边菱形、蓝边矩形、蓝箭头；没有大色块、没有黑底
- 无 4 行字全景卡、无 3 行字流程节点、无 `$` 代码、无字段清单
- 相邻全景卡片间距 ≥ 20；流程节点间距 ≥ 28；文字没有出框
- L1 ≤ 9、L2 ≤ 14、L3 ≤ 8；ER ≤ 12 实体；架构图 ≤ 5 层带且每行 ≤ 4 框
- 流程图节点以 `S*` 开头；无一张覆盖全链路的细图
- 连线正交且不压字（手写时）
- viewBox 未超过本文件硬上限
