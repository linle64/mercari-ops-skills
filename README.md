# Mercari Ops Skills — 煤炉双号运营技能库

一套在真实运营中打磨出来的 [Mercari](https://jp.mercari.com/)（煤炉）转卖运营技能（Agent Skills），覆盖从选品到对账的完整链条：**选品筛查 → 取图核款 → 文案 → 上架发布 → 评论巡逻 → 取引消息 → 台账对账**。

这些技能原本是为一套「双账号（淘宝线＋拼多多线）无货源转卖」的日常运营写的，里面的每条规矩几乎都对应一次真实踩坑：图组混款上架被买家一眼看穿、取引消息误判连发 5 份、克隆浏览器登录态被清空、文件选择器选不中行……脱敏后公开，希望对同样在用 AI agent 做电商运营的人有用。

## 完全不懂？直接丢给你的 Muse

你不需要读完这个仓库。把仓库地址发给你的 AI 助手（Muse 等支持 Skills 的 agent），只说一句：

> 按这个仓库的 mercari-setup 技能，帮我把 Mercari 运营从零搭起来。

它会按 [mercari-setup](skills/mercari-setup/SKILL.md) 向导自己干完：问你几个问题（卖什么、几个账号）、建目录和台账、搭好远程通道（SSH＋Tailscale＋桌面自动化桥）、建好克隆浏览器、装好每日定时任务，最后拿一件商品试跑验收。**你本人只需要做三件事**：在它打开的窗口里亲自登录账号（密码不经过它）、批准它起草的对外消息、决定进什么货。其余巡逻、上架、对账都由它接管。

仓库里的 `templates/` 是配套模板：台账 JSON 的字段口径与示例、五个定时任务的正文模板，搭建时由你的 agent 取用。

## 技能一览

| 技能 | 内容 |
|---|---|
| [mercari-setup](skills/mercari-setup/SKILL.md) | **从零搭建向导**：访谈→脚手架→SSH+Tailscale 通道→克隆浏览器→定时任务→试跑验收；附 `references/ssh-tailscale.md`（通道与桥的搭建定版）和 `references/clone-browsers.md`（克隆 profile 迁移与启动） |
| [mercari-ops](skills/mercari-ops/SKILL.md) | **总纲（先读这个）**：全流程地图、双号分工、授权边界、硬规则总表；附 `references/environment.md`（CDP 克隆浏览器、桌面自动化桥、文件选择器、风控拦截判定的环境定版） |
| [mercari-select](skills/mercari-select/SKILL.md) | 选品与筛查：只认 Mercari 已售实证、筛查前置三道闸（去重／侵权目视／单 SKU 图组）、候选池与自动补位、现货优先 |
| [mercari-listing](skills/mercari-listing/SKILL.md) | 上架全流程：取图拼原图直链、逐张核款、双号文案规范、CDP 发布定版流程（类目 rowButton、wizard 覆盖坑、配送最后设、编辑页补特徴）、改价 |
| [mercari-comments](skills/mercari-comments/SKILL.md) | 评论巡逻与回复：数字必须回源页查证、只起草等确认、无礼不回、议价不承诺 |
| [mercari-transaction-messages](skills/mercari-transaction-messages/SKILL.md) | 取引消息发送与核实：只点一次、核实只走 API 层（get_messages 解析计数）、DOM 回读为什么不可靠 |
| [mercari-ledger](skills/mercari-ledger/SKILL.md) | 台账与对账：登记口径、每日轻量对账＋全量对账、7 天／30 天观察规则、删除评审标准 |

## 使用方式

把 `skills/` 下的目录拷进你的 agent 的技能目录（如 `~/.claude/skills/` 或你的运行时对应的 custom-skills 目录），agent 在处理 Mercari 相关任务时会自动按 description 触发对应技能。先读 `mercari-ops` 总纲，再按任务进分技能。

技能里的路径（`hidden_files/`、`tools/`）、账号代称（A 号／B 号）、定时节奏都是示例约定，按你自己的环境替换即可。脚本本身不在本仓库内——技能描述的是流程与判据，脚本索引在总纲文末，按用途自备。

## 注意事项

- 本仓库是**运营流程知识**，不包含任何账号信息、凭据或现成自动化脚本。
- 自动化操作电商平台账号存在平台规则与风控风险（技能里专门有拦截判定与冷却纪律），请自行评估并遵守平台条款。
- 侵权筛查、对外口径等规矩是这门生意的生存线，不是形式主义：仿款一眼识破即下架警告，消息里提海外发货等于自曝无货源。

## License

MIT
