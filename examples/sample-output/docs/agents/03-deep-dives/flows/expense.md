# expense 功能候选

- 探测时间：2026-08-18
- 本轮 deep：`tiktok-profit-calculate`（自然语言「tiktok 利润计算」）
- 上次 deep：`amazon-expense-calculate`

| # | slug | 状态 | 一句话 | 入口 | 重要性 | 复杂度 | 依据 |
|---|------|------|--------|------|-------:|-------:|------|
| 1 | `tiktok-profit-calculate` | **已深挖** → [细节](expense-tiktok-profit-calculate.md) · [主链图](../../diagrams/flows/expense-tiktok-profit-calculate.svg) · [分支对照图](../../diagrams/flows/expense-tiktok-profit-calculate-branches.svg) | TikTok 订单利润：AllOperatorChain 重算费用并上卷写 order | Consumer `order_expense_calculate`（本轮只展开该入口） | 6 | ~19 | SYSTEM/MANUAL 各前 12 算子；can 多分支；六个独立存储节点 |
| 2 | `amazon-expense-calculate` | **已深挖** → [expense-amazon-expense-calculate.md](expense-amazon-expense-calculate.md) · [图](../../diagrams/flows/expense-amazon-expense-calculate.svg) | Amazon V2 交易平铺后结算写费用 + 延迟上卷 | `amazon_transaction_resolve` / `order_platform_settlement` | 6 | ~20 | V=店铺 Bool×6 |
| 3 | `custom-review` | 候选 | 自定义费用审核 → import_expense | HTTP review + Consumer | 5 | 高 | L 审核态机 |
| 4 | `custom-import` | 候选 | 自定义费用导入 | HTTP + CLI | 5 | 中高 | L 导入态 |
| 5 | `order-init-expense-calculate` | 候选 | 订单初始费用 | Consumer `order_init_expense_calculate` | 4 | 中高 | V=initialize |

## 未展开

| 入口 | 能看到的下一跳 |
|---|---|
| Consumer `order_platform_settlement` type=TIKTOK | `TiktokSettlementExpenseStrategy` → `TiktokSettlementDropped`（不在本轮展开范围） |
| `tiktok_order_fee_pull` / `TiktokStatementsPull` | 拉结算写 `tiktok_statement_transactions` |
| `tiktok_advert_fee` | `TiktokAdsAdvertiseConsumer` |
| `TiktokExpenseInitializeStrategy` | `TiktokPlatformOrderCreateOperatorChain` |
| `POST /expense/expense-center/*` | DispatchSlice |

## 需人工确认

- 「tiktok 利润计算」映射为 **订单利润 Consumer + TiktokAllOperatorChain**；若要深挖 **结算拉取/广告费**，再 `feature=` 指定。
