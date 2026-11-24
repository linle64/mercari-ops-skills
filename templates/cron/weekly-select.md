# 每周选品（模板）

按 mercari-select 技能执行一轮全量选品：

1. 读 `<工作区>/config.md` 与 mercari-ops 总纲对齐品类和账号分工。
2. 在 Mercari 侧查同品类已售实证，更新走量商家观察名单；只留无品牌款。
3. 到货源平台（淘宝／拼多多）找对应货源，逐件过筛查前置三道闸（去重对 drafts.json＋listed.json、侵权目视、单 SKU 图组）。
4. 过闸的写入 pool.json／pool_pdd.json（改前备份），dropped 的记名写原因。
5. 产出日志 `<工作区>/hidden_files/select-<日期>.md`，向用户报告：新增几件、dropped 几件及原因、池水位。数字回源页原文，不凭记忆。
