# Examples

在**业务仓库**对话中使用（不要在本 skill 仓库空跑）。

## 全量地图

```text
使用 codex-map mode=full
```

第一次跑、清单还不存在时：不写 `flows/`，对齐接口与模型（Request/DTO 留在 api-list，不进模型页），写出 ≤8 条核心链路和 ≤10 条待确认风险地带，从 README/compose 抄怎么跑。`AGENTS.md` 禁区两节留空（或保留人写）。清单已经在时，这条命令不再覆盖它们，只补「先查哪里」。要对某域深挖，再跑 `mode=flows domain=...`。

## 只重生图

```text
使用 codex-map mode=diagrams refresh=force
```

全景图是企业分层风（浅灰底、层色实心卡片）。流程图不要动。没写 `refresh=force` 且图已在：不重画。

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

深挖后看 `flows/{domain}-{slug}.md` 的 `S*` 步骤表，每步有 `路径:行号`。图只有一份 `diagrams/flows/{domain}/{slug}/flow.html`，打开只看主步骤，点 `+` 展开。再跑同一个 `feature=` 只重写这一份，不重写清单。

## 已有文档时不要整库重生

```text
使用 codex-map mode=full
```

清单和全景图已经在时，默认不覆盖，只给 `AGENTS.md` 补「先查哪里」。要整页重生才写 `refresh=force`。

## 只核对深挖还准不准

```text
使用 codex-map mode=flows refresh=cited
```

打开深页里的 `路径:行号`。对不上的行标 `已过期`，不重画 `flow.html`。

## 入口文件

```text
使用 codex-map mode=entrypoint
```

「怎么跑」只从 README / compose / 已有脚本抄命令和端口，不装依赖、不启动应用。需要 Claude Code 双写时加 `dual_claude=true`。

## 对账

```text
使用 codex-drift 相对 main 检查文档漂移
```
