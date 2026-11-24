# 台账模板

把本目录的 `*.example.json` 复制到你的工作区 `hidden_files/`，去掉 `.example` 后缀、删掉示例条目（或改成 `[]`）即可开始。字段口径如下，登记纪律见 mercari-ledger 技能。

## pool.json / pool_pdd.json（候选池，淘宝线／拼多多线各一份）

| 字段 | 说明 |
|---|---|
| `source` | 货源平台：taobao / pdd / 1688 |
| `source_id` | 货源商品 ID（去重就靠它） |
| `source_url` | 货源链接 |
| `title_hint` | 货源标题（仅备考，正式标题按 copywriting 另写） |
| `source_price` | 货源价（注明币种） |
| `account` | 拟上账号代称（A / B） |
| `stock` | 现货状态：none（未购）/ ordered（在途）/ in_hand（已到手）——只有 ordered、in_hand 能进发布 |
| `status` | pooled / dropped / drafted / listed |
| `screened` | 三道闸是否过完：{dedup, infringement, single_sku} |
| `note` | 备注（dropped 要写明原因，如侵权款式名） |

## drafts.json（草稿台账）

| 字段 | 说明 |
|---|---|
| `draft_key` | 草稿标识（账号内唯一，如出品页 URL 或本地编号） |
| `account` | 账号代称 |
| `title` | 已填标题 |
| `price` | 价格（JPY） |
| `source` / `source_id` | 货源 |
| `created` | 建草稿日期 |
| `attrs_pending` | 还差哪些手选项（如カラー/素材发布后补设） |

## listed.json（在售台账）

| 字段 | 说明 |
|---|---|
| `m_id` | Mercari 商品 ID（m 开头） |
| `account` | 账号代称 |
| `title` | 标题 |
| `price` | 价格（JPY） |
| `source` / `source_id` | 货源 |
| `listed_at` | 上架日期 |
| `status` | listed / sold / delisted |
| `buyer` | 售出后填买家名 |
| `sold_price` / `sold_at` | 售出后填 |
