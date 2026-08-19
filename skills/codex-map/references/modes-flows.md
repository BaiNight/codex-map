# flows mode

执行前已读 `detect.md` 与 `quality.md`。不要打开 `modes-inventory.md` / `modes-full.md`。
`SKILL.md` 里的两步协议表仍有效；本文件是细则。

识别「已拍板」的信号（命中任一即可进 deep）：

- 显式 `feature=<slug>` 或 `feature=<自然语言>`
- 「就做 / 选 / 拍板 / 生成 … 详细文档与流程图」+ 能对应到候选表或代码里的某一功能
- 用户从上一轮候选列表里点名（序号或短名）

**禁止**：带 `domain=` 时自动挑 Top N 并直接生成细节文档/细图。  
**禁止**：propose 阶段「先做完再问你要不要改」。必须先交名单。

无 `domain=`：不走两步协议；只写 1～2 个入口最多域的骨架链（2～3 条），不出功能图。

### 6) 功能探测（仅 `mode=flows` 且带了 `domain=`）

「功能」= 同一业务动作的一组入口，不是一条路由一行。  
禁止用其它系统的业务外号当功能名。

#### 聚类

从该域入口归并（`api-list` / `cli-and-consumers` / 打开的 controller、console、consumer）：

| 归并信号（命中任一即可合成一个功能） |
|---|
| 同一 Controller 上的一组写操作（如 store / review / import） |
| 同一 Consumer `name` 或同一 `*StrategyProvider` / 同一 chain 类 |
| 写入同一核心 Model（`tableName()` 相同） |
| HTTP/CLI 投 MQ，由同域 Consumer 接着做 → **算同一功能**，不拆成两个 |

功能短名（`slug`）优先级：路由注释 / action 注释 → Consumer yml `name` → Controller 去 `Controller` 后缀 + action-id。只允许 `[a-z0-9-]`。禁止自创中文产品名当 slug。

#### 重要性（过滤）

| 分 | 信号（代码里真实存在才加） |
|---:|---|
| +2 | 能追到 F 编排 |
| +2 | 有 L 或 V |
| +1 | 有 X 或 MQ |
| +1 | 同时有 HTTP 与 CLI/Consumer |

重要性 = 0：进「未展开」，不进候选榜。

#### 复杂度（排序用）

只对重要性 > 0 的功能打分。数字必须来自本轮打开的代码。

| 分 | 信号 | 怎么数 |
|---:|---|---|
| +4 | V 很宽 | Provider `providers()` ≥ 3，**或** chain `apply()` / 同类列表 ≥ 5 个算子 |
| +4 | 调用深 | 入口到写库中间编排类 ≥ 4 跳（controller 不算） |
| +3 | 分支多 | 主路径上按 status/type/mode/platform 等字段的 `if`/`switch` ≥ 3 |
| +3 | 多表写入 | 不同 `tableName()` ≥ 3 |
| +2 | 有状态机 | 带 `getNext*` / 明确禁迁的 Status/State |
| +2 | 批处理 | 列表/文件行/item 循环后落库或投 MQ |
| +2 | 跨入口 | HTTP/CLI 投递，计算在 Consumer/task |
| +1 | 事务/锁/幂等 | 代码里有 `transaction` / lock / 去重键 |
| +1 | 对外 | launcher / internal / OSS 等 |

propose：按复杂度取 **Top 3～5** 进候选榜（不足则有多少列多少）。同分优先更深、V 更宽。

#### propose 产出（只推荐）

1. 更新 `flows/{domain}.md`：候选表（见下）+ 未展开索引；**不要**写各功能长步骤
2. 在对话中用同样表格请用户拍板（序号或 slug）
3. **停止**。禁止创建 `flows/{domain}-{slug}.md`、禁止画 `diagrams/flows/*.svg`、禁止因本轮去扩写 lifecycle/contracts 细节

候选表模板：

```markdown
# {domain} 功能候选（待拍板）

- 探测时间：
- 状态：**等待用户选择** — 回复「就做 N」或 `feature=<slug>` 后再深挖

| # | slug | 一句话 | 入口（HTTP/CLI/Consumer） | 重要性 | 复杂度 | 依据（打开过的路径） |
|---|------|--------|---------------------------|-------:|-------:|----------------------|
| 1 | … | … | … | … | … | V=…；跳数=…；… |

## 未展开
| 入口 | 能看到的下一跳 |
|---|---|
```

对话收尾必须包含：

```text
请拍板：回复「就做 1」或「就做 <slug>」或
使用 codex-map mode=flows domain=<域> feature=<slug|自然语言>
```

#### deep 产出（拍板后才写）

每个拍板功能：

1. `flows/{domain}.md`：把该行标为「已深挖」，链到深页与图
2. `flows/{domain}-{slug}.md`：编号步骤 + **实现细节**
3. `diagrams/flows/{domain}-{slug}.svg`：细粒度流程图

用户自然语言对不上唯一候选：列出最接近的 2～3 个再问一次，**仍不深写**。

同功能 HTTP 投 MQ、Consumer 落库：一条时间线、一张图、一份细节。

##### 步骤（深页前半）

```markdown
## 功能 · {短名}

- 复杂度 / 重要性：{分} / {分}
- 一句话：
- 入口：
- 流程图：[../../diagrams/flows/{domain}-{slug}.svg](../../diagrams/flows/{domain}-{slug}.svg)

### 步骤
1. 触发：谁调用、入口文件与方法、能读到的主参
2. 校验：Abort / 权限 / 前置状态（只写代码里看得到的）
3. 编排：service / slice / task / handler 的类::方法
4. 分叉：V 目录 + 分发类；按什么字段选实现；本步无则写无
5. 写库：Model 路径 + tableName；能看到的赋值就写
6. 发出：MQ / launcher / internal / client
7. 失败：catch / Abort / 回滚 / 通知；看不到则待确认
```

每步带可打开路径。可按真实顺序增减，禁止凑步数。

##### 实现细节（深页后半）

只写本轮打开并核对过的代码。禁止粘贴整文件、禁止编造字段。

```markdown
### 实现细节

#### 调用栈
| # | 类::方法 | 路径 | 做什么（一句话） | 下一跳 |
|---|----------|------|------------------|--------|

#### 分支
| 条件（字段/枚举/字面量，抄代码） | 走向 | 路径 |
|----------------------------------|------|------|

#### 分叉展开（仅本功能有 V 时）
- 分发类、选择键
- `providers()` / `apply()` **按代码顺序列出**；超过 20 个则列前 20 + 「其余 N 个见目录」
- 每个列出的实现：类路径、一句话、写哪些 Model（看到才写）、跳过规则（看到才写）
- 另选 1 条与主路径不同的代表分支对照差异

#### 写库
| Model | tableName | 调用方法 | 能看到的赋值 | 条件 |
|-------|-----------|----------|--------------|------|

#### 消息与外部
| 方向 | 队列或 launcher/client | 能读到的字段 | 路径 |
|------|------------------------|--------------|------|

#### 失败与禁迁
| 条件 | 行为 | 路径 |

#### 事务 / 锁 / 幂等
看到的写，看不到写「本功能未见」
```

##### 细流程图（仅 deep）

- 路径：`docs/agents/diagrams/flows/{domain}-{slug}.svg`
- 主题：与 `architecture.svg` 相同 dark
- **不要**收成「入口→编排→写库」三框；画出调用栈上的真实判断和主要分叉
- 节点上限 **28**。仍超：合并纯转发步，**判断和写库不要合并**
- V 很宽：画「选择键」菱形 + 主路径一条 + 对照分支一条 + 「其它 N 套 → 见细节页」
- chain/pipeline：按 `apply()` 顺序列 4～8 个算子，其余「+N」指向细节页
- 分色：入口蓝、判断菱形黄边 `#d29922`、编排灰、写库绿、对外/MQ 橙
- 节点正文：`类::方法`、判断短句、或「写 {tableName}」。字段列表只在 md
- 边上标 是/否、通过/Abort、选择键取值
- 禁止 mermaid 替代 SVG
- 自检：图上每个判断/写库能在细节页表里对上

### mode=flows

1. 完成开跑前探测（含第 5 节四层）
2. 若 `docs/agents/01-domains/INDEX.md` **不存在**：按共用节补写；已存在则只给本轮域补「深潜」链接
3. 写 `docs/agents/03-deep-dives/layers.md`
4. **F 本仓库无**：不写 `flows/`、不画图；summary 后结束
5. **无 `domain=`**：取入口最多的 1～2 个域，每域 2～3 条骨架链写入 `flows/{domain}.md`（模板见下），不出功能图；然后跳到更新 INDEX → Summary
6. **有 `domain=`，未拍板（propose）**：
   - 只该域，执行第 6 节聚类 + 打分
   - 写 `flows/{domain}.md` 候选表（Top 3～5）
   - 对话给出候选并请拍板
   - **停止**（不写细节页、不画细图、不深写 L/V/X）
   - Summary 标明 `phase=propose`、候选 slug 列表
7. **有 `domain=`，已拍板（deep）**：
   - 解析 `feature` / 自然语言到唯一 slug；对不上则再问，不深写
   - 写 `flows/{domain}-{slug}.md` + `diagrams/flows/{domain}-{slug}.svg`
   - 更新 `flows/{domain}.md` 该行状态为已深挖
   - **L 命中**：按本功能回写 `lifecycle.md`（其它 Status 只索引）
   - **V 命中**：更新 `variants/INDEX.md`（名 + 一句话 + 命中功能）
   - **X 命中**：按本功能回写 `contracts.md`
   - 更新 `INDEX.md` 覆盖率
   - 自检：路径可打开；图节点与细节表对得上
   - Summary 标明 `phase=deep`、slug、文件列表

骨架链模板（无 domain 时）：

```text
入口（HTTP / CLI / Consumer + 文件路径）
  → 编排类（service / handler / task / slice + 方法）
  → 分叉点（V 命中且本链用到；否则 本链无）
  → 写入的 Model（tableName + 路径；否则 待确认）
  → 对外调用（X 命中且本链用到；否则 本链无）
```
