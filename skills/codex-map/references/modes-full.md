# full mode 子代理编排

默认即 `full`。执行前已读 `detect.md` 与 `quality.md`。
工作量大，**必须**用主代理编排 + 子代理并行，禁止主代理一人顺序干完所有扫描与写文件。

逻辑顺序：探测 →（并行）图 / surfaces / data / 域索引 → flows 骨架或 propose → 互校 → entrypoint → summary。  
**不要**在 full 里对某域自动 deep。若消息带了 `domain=` 且未拍板：清单类做完后对该域只做 propose 并停住。

子代理 prompt **不要**粘贴 `modes-inventory.md` / `modes-flows.md` 全文；只写「按某文件的某节做」+ 本角色允许写的路径。

## 主代理职责

主代理只做：探测、分发、合并 summary、互校、entrypoint、对用户说话。  
子代理用 Task（`generalPurpose` 或 `explore` 按任务选）；**同一轮消息里并行发起**互不依赖的多个 Task。

## 写文件边界（禁止交叉写）

| 角色 | 只允许写 |
|---|---|
| 主代理 | `layers.md`（探测结果）、互校改动、`INDEX.md`、`AGENTS.md`（及可选 `CLAUDE.md`）、最终 summary |
| 子代理 A · diagrams | `docs/agents/diagrams/architecture.svg`、`module-deps.svg`、`external-deps.svg` |
| 子代理 B · surfaces | `docs/agents/02-surfaces/api-list.md`、`cli-and-consumers.md` |
| 子代理 C · data | `docs/agents/03-deep-dives/data-model.md`、`diagrams/data-model-er.svg` |
| 子代理 D · domains | `docs/agents/01-domains/**`（短索引；不写深潜） |
| 子代理 E · flows-skeleton | 仅当**无** `domain=`：`flows/{domain}.md` 骨架链（1～2 域） |
| 主代理或单子代理 · flows-propose | 仅当有 `domain=` 且未拍板：候选表；**停住等用户**，不派 deep |

禁止两个子代理写同一文件。子代理**禁止**改业务代码、禁止跑 deep、禁止改 `AGENTS.md`（留给 entrypoint）。

## 波次

**波次 0 · 主代理（串行，必先）**

1. 按 `detect.md` 做开跑前探测：称谓 / 剖面 / 域列表 / 四层 F/L/V/X  
2. 写 `layers.md`  
3. 把探测摘要（system、剖面、域列表、关键路径、四层判定）写进每个子代理 prompt，子代理**不要**重做全库探测

**波次 1 · 并行（同一轮发起）**

同时派 A + B + C + D：

- A：按 `modes-inventory.md` 的 `mode=diagrams` 写三张全景图  
- B：按 `modes-inventory.md` 的 `mode=surfaces` 写 api-list + cli-and-consumers；**跳过** surfaces 中写 `01-domains` 的步骤（由 D 写，禁止交叉）  
- C：按 `modes-inventory.md` 的 `mode=data` 写 data-model + ER（高价值域用探测域里入口最多的，或用户 `domain=`）  
- D：按 `detect.md` 的「写 01-domains/INDEX.md」写短索引  

每个子代理 prompt 必须包含：仓库根路径、探测摘要、`quality.md` 硬约束摘要、**只写哪些文件**、完成后返回「写入文件列表 + 一句话摘要 + 需人工确认」。

**波次 2 · flows（串行，等波次 1）**

- 无 `domain=`：派 E，按 `modes-flows.md` 写骨架链（可读 api-list 选入口最多的域；若 B 未完成则按 controller 文件数）  
- 有 `domain=` 且未拍板：主代理（或单子代理）按 `modes-flows.md` 做 propose → **停住**；波次 3/4 可先做互校+entrypoint（不含 deep）  
- 有 `feature=`：本轮 full **不做 deep**；summary 提示用户单独再跑 `codex-map mode=flows` deep（避免 full 与细挖抢上下文）

**波次 3 · 主代理互校**

api/cli ↔ data-model ↔ 已有 flows 页；不一致只改文档。若仓库已有 `wiki/`、`docs/ops/` 等运维原文，用链接指出，不镜像进 agents。

**波次 4 · 主代理 entrypoint + summary**

按 `modes-inventory.md` 的 `mode=entrypoint`；汇总各子代理返回值写成总 summary（标明哪些子代理跑了、写了什么）。summary 字段以 `quality.md` 模板为准。

## 子代理失败

- 某一路失败：主代理可单路重试 1 次；仍失败则 summary 标缺口，不阻塞其它已成功文件  
- 禁止为「省事」改回主代理独自全量串行，除非环境无 Task 工具（此时 summary 写「未并行，原因：无 Task」）

## 单 mode 是否并行

- `mode=diagrams|domains|surfaces|data|entrypoint|flows`：默认主代理直接做，**不**强制子代理  
- 仅当用户明确要求「并行 / 用 subagent」且任务可拆（例如 surfaces 按域拆）时再拆；拆时仍遵守写文件边界
