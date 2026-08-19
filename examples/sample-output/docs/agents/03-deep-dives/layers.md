# 四层探测

- system / 剖面：oms（信诚-OMS）/ yii2-advanced
- 探测时间：2026-08-18
- 本轮 `domain=`：expense · `feature=` tiktok 利润计算 → slug `tiktok-profit-calculate` · **phase=deep**

| 层 | 判定 | 命中路径（真实存在） | 本轮深写 |
|---|---|---|---|
| F 编排 | 命中 | `strategy/profit/` · `task/expense/segment/` · `consumer/OrderProfitCalculateConsumer` | [flows/expense-tiktok-profit-calculate.md](flows/expense-tiktok-profit-calculate.md) |
| L 生命周期 | 命中 | `can()` 门禁 + `BuildOrderTypeEnum`；无 TikTok 专用 getNext 态机 | [lifecycle.md](lifecycle.md) |
| V 分叉 | 命中 | `OrderProfitCalculateStrategyProvider` · `TiktokAllOperatorChain` · `components/tiktok/` | [variants/INDEX.md](variants/INDEX.md) |
| X 对外契约 | 命中（弱） | 以 MQ / RedLock / notify 为主；launcher 本链无 | [contracts.md](contracts.md) |

候选与其它已深挖见 [flows/expense.md](flows/expense.md)。
