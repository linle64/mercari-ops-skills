# 候选池水位守卫（模板，每日傍晚、上架任务之前）

按 mercari-select 技能检查候选池：

1. 以 drafts.json＋listed.json 交叉核对 pool.json／pool_pdd.json 的真实状态（pool 的 status 会滞后，不可直接信）。
2. 统计：池内合格候选数、其中现货（ordered／in_hand）件数，按账号分列。
3. 现货为 0 或合格候选低于水位线（默认 3 件）时，向用户报告缺口；有筛查积压（未过三道闸的候选）时顺手补筛。
4. 日志写 `<工作区>/hidden_files/pool-guard-<日期>.md`。水位正常且无积压时静默。
