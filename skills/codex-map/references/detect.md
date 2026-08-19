# 开跑前探测

在任何 mode 的扫描之前先做本节 1～4。  
**换框架不必新写一节目录。** 定包后去官网拉开发规范，再和本仓库路径求交。  
`mode=flows` / `mode=full` 必须做完四层判定后再写深潜页。`full` 无 `domain=` 时只写 `layers.md`，不写 `flows/`。

## 1. 系统称谓

优先级：用户 `system=`（非 auto）→ README 标题 → `composer.json` 的 `name`/`description` → 仓库目录名。  
得到短名和一句话定位，供 overview / AGENTS.md 使用。

## 2. 定包（只看 composer.json）

打开仓库根 `composer.json` 的 `require` 与 `require-dev`。**目录名不参与框架判定。** 没打开该文件不准定框架。

按表**从上到下，命中即停**。表只给包名和官网入口；**不要**在本文件为每个框架写死 `app/Http`、`backend/modules`。

| `composer.json` 含此包 | 剖面 | 必打开的官网（结构 / Model） |
|---|---|---|
| `laravel/framework` | `laravel` | https://laravel.com/docs/structure · https://laravel.com/docs/eloquent |
| `topthink/framework` 或 `topthink/think` | `thinkphp` | https://doc.thinkphp.cn/v8_0/directory.html · https://doc.thinkphp.cn/v8_0/model.html |
| `yiisoft/yii2` | `yii2` | https://www.yiiframework.com/doc/guide/2.0/en/structure-overview · https://www.yiiframework.com/doc/guide/2.0/en/db-active-record |
| `codeigniter4/framework` | `codeigniter` | https://codeigniter.com/user_guide/concepts/structure.html · https://codeigniter.com/user_guide/models/model.html |
| `symfony/framework-bundle` | `symfony` | https://symfony.com/doc/current/best_practices.html · https://symfony.com/doc/current/doctrine.html |
| 以上都没有 | `generic` | Packagist 该包首页 / `composer.json` 的 `homepage`；仍无则不查官网 |

新框架：只在上表加一行「包名 + 官网结构页 + Model 页」，禁止再复制一整节目录清单。

禁止：用 `backend/` + `common/`、`artisan`、`spark`、`think`、`yii` 猜框架。  
禁止：用另一个框架的记忆填路径（Laravel 仓库禁止出现 `tableName()` / `launcher/`）。

## 3. 四层行为（无路径）

与框架无关。路径和 API **只许**来自「本轮官网规范 ∩ 本仓库真实存在的路径」。

| 层 | 含义 | 算命中的条件（行为） |
|---|---|---|
| **F** 编排 | 入口到落库谁调谁 | 至少 1 个 HTTP/CLI/Job/Consumer 入口能追到编排类，再到 Model 或写库 |
| **L** 生命周期 | 合法状态、谁改、禁迁 | 能打开状态枚举，或某 Model 上的 status + 迁移方法。禁止把普通 Model 列表当 L |
| **V** 分叉 | 同一入口按类型走不同实现 | 同入口或同抽象下 ≥2 套平行实现。禁止因缺少某个目录名就判「本仓库无」 |
| **X** 对外契约 | 调别人 / 被别人调 | 能指出本仓库真实存在的客户端/RPC/SDK 类 |

探测结果写入 summary，并写入 `docs/agents/03-deep-dives/layers.md`（`flows` / `full`）：

```markdown
# 四层探测

- system / 剖面 / 官网来源：
- 探测时间：

| 层 | 判定 | 命中路径（真实存在） | 本轮深写 |
|---|---|---|---|
| F 编排 | 命中 / 本仓库无 | `...` | `flows/{domain}.md` 或 — |
| L 生命周期 | 命中 / 本仓库无 | `...` | `lifecycle.md` 或 — |
| V 分叉 | 命中 / 本仓库无 | `...` | `variants/INDEX.md` 或 — |
| X 对外契约 | 命中 / 本仓库无 | `...` | `contracts.md` 或 — |
```

规则：

- 「命中路径」必须在本仓库能打开，且能在本轮官网摘录里对上（或标「仓库变体，官网无」）
- 候选目录在、但追不到入口 → F 标「本仓库无」或「需人工确认」
- V/L/X **只在 deep 阶段**、且只深写拍板功能上碰到的；其余命中路径只进索引
- 页标题用探测到的目录名 / 类名，不用其它系统的业务外号

## 4. 查官网，再扫仓库

定包后**必须**按这个顺序做，不要先翻文末离线兜底。

1. **拉规范**：用 WebFetch / 浏览器打开上表官网（结构页 + Model 页）。摘出：路由文件、Controller 目录、Model 目录、表名/字段 API、命令/Job 目录。只摘官方写明的，禁止脑补
2. **求交**：在本仓库 glob 这些路径，只保留真实存在的。官网写了但仓库没有 → 「本仓库无」
3. **变体**：官网若写了 multi-app / modules，或仓库里明显有平行子应用（同级都有 controller/model），按**本仓库目录名**列入域。禁止用其它项目的域名
4. **四层**：用第 3 节行为，在求交后的路径上确认 F/L/V/X
5. **域 / HTTP / CLI / Model**：定义来自官网摘录 + 求交结果，不要读其它框架的离线节
6. 然后按文末「写 01-domains/INDEX.md」决定是否写域索引

官网打不开、超时、或摘不出目录时：

- 剖面是 laravel / thinkphp / yii2 → 才允许读文末对应离线节
- 其它剖面（含 codeigniter / symfony / generic）→ 走 [8. generic](#8-generic)，summary 写「官网未取到」
- **禁止**官网失败就改用另一个框架的离线节

---

文末第 5～8 节是**离线兜底**，不是主路径。官网已取到时不要打开。

---

## 5. laravel（仅官网失败）

仅当剖面是 `laravel` **且**第 4 步官网失败。路径必须在当前仓库真实存在；没有的行标「本仓库无」。

工程实际（官方默认 + 常见变体，**有哪个用哪个**）：

- 官方默认：`routes/web.php` `routes/api.php` `routes/console.php`，`app/Http/Controllers`，`app/Models`，`app/Console/Commands`，`app/Jobs`
- 多应用 / 多模块：`app/{子应用}/`（如 `api` `admin` `common`），或 `Modules/{Name}/`（nwidart）。子应用名以目录为准，禁止套其它仓库的域名
- 不要假设一定有 `app/Services`、`app/Enums`、`app/Actions`；存在再扫

### 域

从本仓库推导，不要抄其它系统：

1. `routes/*.php` 里的 `prefix` / 文件名（`api.php` 不算一个业务域）
2. `app/Http/Controllers/{分组}`
3. 多应用时：`app/` 下带 `Http`、`Controllers` 或自己 `routes` 的子目录

`domain=` 必须是这张名单里的一项。

### HTTP

- 路由：`routes/*.php`；多应用再加该子应用下真实存在的 `routes/`
- Controller：`app/Http/Controllers`，以及 `app/*/Http/Controllers`、`app/*/Controllers`、`Modules/*/Http/Controllers`
- 方法以路由文件登记为准；推不出的入参标「待确认」

### 异步

- Artisan：`app/Console/Commands`、`app/*/Console`
- Job / 队列：`app/Jobs`、`app/*/Jobs`；再核对 `composer.json` 是否有队列包
- 没有则文档写「本仓库无 CLI/Consumer」

### Model

**不要读 `database/migrations`。**

- 目录：`app/Models`、`app/*/Models`、`app/*/Model`（以实有为准）
- 只收继承 `Illuminate\Database\Eloquent\Model` 的类
- 表名：`$table`；未声明则标「待确认」（不要按类名脑补）
- 字段：`$fillable` / `$guarded` / `$casts` / 属性注释
- 关系：`hasMany` `hasOne` `belongsTo` `belongsToMany` `morphTo` 等
- 禁止写 `tableName()` `rules()` `attributeLabels()`

### F

候选：`*Service*` `*Handler*` `*Action*`，以及实有的 `app/**/Services`、`app/Actions`。  
确认：某个 Controller action / Command `handle` / Job `handle` 能 new/调用到其中一类，再落到 Model 或写库。  
只有 Service 目录、追不到入口 → 本仓库无。

### L

候选：`app/Enums` 或任意处的 `*Status*` / `*State*` 枚举；Model `$casts` 指到该枚举；`transition*` / `to*Status` / Spatie `HasStates`。  
确认：必须打开枚举或迁移方法。  
**禁止**把 `UserModel` / `RoleModel` 等普通 Eloquent 当 L。

### V

候选：同一 interface / contract 的多个实现（看 `AppServiceProvider` 或模块 Provider 的 bind）；`*Manager` `*Factory` `*Driver`；若存在 `strategy/` `adapter/` 也算。  
确认：同入口或同抽象下 ≥2 套。  
**禁止**因没有 `strategy/` 目录就写「本仓库无」。

### X

候选：`*Client*` `*Sdk*` `*Rpc*`；`app/**/interfaces` 下的 RPC 口；`Illuminate\Support\Facades\Http` / `Http::` 封装类。  
确认：能指出调用方类。只列本仓库真实文件。  
禁止填写其它剖面的客户端目录名。

---

## 6. thinkphp（仅官网失败）

仅当剖面是 `thinkphp` **且**第 4 步官网失败。路径必须在当前仓库真实存在；没有的行标「本仓库无」。

工程实际（ThinkPHP 5 / 6 / 8，**有哪个用哪个**）：

- 多应用（常见）：`app/{应用}/controller` `app/{应用}/model` `app/{应用}/service`，路由在 `route/{应用}.php` 或 `app/{应用}/route`
- 单应用：`app/controller` `app/model` `app/service`，路由 `route/app.php`
- 不要假设一定有 `validate/` `middleware/` `job/`；存在再扫
- 禁止套 Laravel 的 `app/Http/Controllers`、Yii 的 `backend/modules`

### 域

1. 多应用：`app/` 下带 `controller` 的子目录名（`common` 有 Model 无入口则不算业务域，或标支撑）
2. 单应用：`app/controller` 下的分组 / `route/*.php` 的 prefix
3. 子应用名以目录为准，禁止套其它仓库的域名

### HTTP

- 路由：`route/*.php`，以及 `app/*/route`
- Controller：`app/controller`、`app/*/controller`（ThinkPHP 目录名是小写 `controller`，不要改成 `Controllers`）
- 以路由登记 + controller 方法为准；推不出的入参标「待确认」

### 异步

- 命令：`app/command`、`app/*/command`，入口常是根目录 `think`
- 队列：`app/job`、`app/*/job`；再看 composer 是否有 `topthink/think-queue`
- 没有则写「本仓库无 CLI/Consumer」

### Model

**不要读 migrations。**

- 目录：`app/model`、`app/*/model`（以实有为准）
- 只收继承 `think\Model` 的类
- 表名：`$table`（全名）或 `$name`（无前缀）；都未声明标「待确认」
- 字段：`$schema` / `$type` / 属性注释
- 关系：`hasOne` `hasMany` `belongsTo` 等
- 禁止写 Yii `tableName()` / `rules()`，禁止写 Eloquent `$fillable`（除非该类真有该属性）

### F

候选：`app/**/service`、`*Service*`。  
确认：某个 controller 方法 / command / job 能调用到 Service，再落到 Model。  
只有 service 目录、追不到入口 → 本仓库无。

### L

候选：`*Status*` / `*State*` 枚举或常量类；Model 上 status 字段 + 迁移方法。  
必须打开枚举或迁移方法。  
**禁止**把普通 `think\Model` 列表当 L。

### V

候选：同一接口多实现；`*Driver` `*Gateway`；若存在 `strategy/` `adapter/` 也算。  
确认：同入口 ≥2 套。  
**禁止**因没有 `strategy/` 就写「本仓库无」。

### X

候选：`*Client*` `*Sdk*` `*Rpc*`；`app/**/interfaces`。  
确认：能指出调用方类。只列本仓库真实文件。  
禁止填写其它剖面的客户端目录名。

---

## 7. yii2（仅官网失败）

仅当剖面是 `yii2` **且**第 4 步官网失败（composer 已含 `yiisoft/yii2`）。  
在**本节内部**区分形态：同时存在 `backend/` + `common/` + `console/`（或根目录有 `yii` 入口脚本）→ advanced；否则 basic。这不是框架判定，只是同一框架的目录形态。

### 域

- advanced：`backend/modules/*` 业务子目录、controller 分组、`common/` 下明显限界目录
- basic：`controllers/` 分组、`modules/*`
- `domain=` 必须在这张名单里

### HTTP

- advanced：`backend/modules/*/controllers`、`backend/config/routes*`
- basic：`controllers/`、`config/web.php` 路由
- 方法以路由配置 + controller action 为准；推不出标「待确认」

### 异步

- `console/controllers`、`commands/`、consumer 类、MQ 配置
- 没有则写「本仓库无 CLI/Consumer」

### Model

**不要读 migrations。**

- 目录：本仓库实有的 `backend/models`、`common/models`、`models/`（不要默认只有一个）
- 表名：`tableName()`
- 字段：`rules()` / `attributeLabels()` / 属性注释
- 关系：AR 关系方法
- 禁止把 Eloquent `$fillable` 写进 Yii 文档

### F

候选：`service/` `handler/` `task/` `slice/`（及 `application/` `usecase/` `processor/`）。  
确认：controller / console / consumer 能追到其中一类。

### L

候选：`enums/` 下 `*Status*` / `*State*`；或 Model 上成组 status + `getNext*` / 迁移方法。  
必须打开枚举或迁移方法。禁止任意 Enum、禁止普通 Model 列表。

### V

候选：`strategy/` `adapter/` `chain/` `provider/` `plugin/`；同一接口多实现。  
确认：同入口 ≥2 套。

### X

候选：`launcher/`、`internal` 路由、`request/` 客户端、`*Client.php` / `*Sdk.php`。  
确认：能指出调用方类。只列本仓库真实路径。

---

## 8. generic

官网失败、或 composer 不在第 2 节表里时走本节。**按本仓库实有路径扫**，不要套第 5～7 节。

- 域：顶层业务包、`src/` / `app/` 下的模块名、路由前缀（以实有为准）
- HTTP：实际 controller 目录 + 路由文件（名称以仓库为准）
- 异步：`command*`、`job*`、consumer、MQ 配置；没有则写无
- Model：本仓库 Model 目录里能看到的表名/字段/关系声明；不要套另外三节的 API 名称
- F/L/V/X：只用第 3 节行为 + 文件名模式（`*Service*` `*Client*` `*Status*`）。找不到就「本仓库无」或「待确认」
- 禁止回退套 laravel / thinkphp / yii2 节的目录

---

## 写 01-domains/INDEX.md（共用）

路径固定：`docs/agents/01-domains/INDEX.md`。只做域短索引，**禁止**把 api-list 粘进来。  
域名单来自本轮**已读剖面节**，不要用其它节的例子。

| 触发 | 策略 |
|---|---|
| `mode=domains` | 重生域表 |
| `mode=surfaces` 单跑 | **先写域索引，再扫 API**；api-list 完成后回填「HTTP 条数」（能数出来才写数字） |
| `mode=full` 子代理 D | 重生域表（子代理 B **不写**此文件） |
| `mode=data` / `mode=flows` | **仅当该文件不存在**时创建；已存在则只给本轮碰到的域补「深潜」链接，不改职责原文 |
| `mode=diagrams` / `entrypoint` | 不写此文件 |

重生时：职责与入口以本轮探测为准；若仓库里已有 `03-deep-dives/flows/{domain}.md`，深潜列保留相对链接。

模板（遵守「文档版式」）：

```markdown
# {system} 业务域索引

- 系统：
- 剖面：
- 探测时间：
- 真源：（controller 目录 / 路由前缀 / 模块名，必须是本仓库路径）
- 域数量：N

HTTP 见 [../02-surfaces/api-list.md](../02-surfaces/api-list.md)。CLI / Consumer 见 [../02-surfaces/cli-and-consumers.md](../02-surfaces/cli-and-consumers.md)。模型见 [../03-deep-dives/data-model.md](../03-deep-dives/data-model.md)。

## 域

| 域 | 职责（一句话） | 入口路径 | HTTP 条数 | 深潜 |
|---|---|---|---:|---|
| {slug} | … | `controllers/…` · `routes/…` | 12 或 — | [flows](../03-deep-dives/flows/{slug}.md) 或 — |

## 缺口

> 无路由的 controller、空模块、命名对不上的目录。没有则写「本轮未见」。
```

职责必须来自本轮打开的 README / 模块注释 / 控制器目录名；推不出写 `待确认`，禁止用其它仓库的外号。
