# Examples

在**业务仓库**对话中使用（不要在本 skill 仓库空跑）。

## 全量地图

```text
使用 codex-map mode=full
```

## 只重生图

```text
使用 codex-map mode=diagrams
```

## 只补域索引

```text
使用 codex-map mode=domains
```

`mode=surfaces` 单跑时也会写 `01-domains/INDEX.md`，不必先跑 `domains`。

## 主链路：先候选

```text
使用 codex-map mode=flows domain=order
```

停住，等拍板。新 session 再跑同一句：若 `flows/order.md` 仍是等待选择，只复述名单，不重扫。要重算加「刷新候选」。

## 主链路：拍板后深挖

```text
使用 codex-map mode=flows domain=order feature=order-sync
```

或自然语言：`请生成 订单同步 详细文档与流程图`。

深挖后看 `flows/{domain}-{slug}.md` 的 `S*` 步骤表，图在 `diagrams/flows/{domain}/{slug}/`：`01-overview.svg` 起按序编号。

## 入口文件

```text
使用 codex-map mode=entrypoint
```

需要 Claude Code 双写时加 `dual_claude=true`。

## 对账

```text
使用 codex-drift 相对 main 检查文档漂移
```
