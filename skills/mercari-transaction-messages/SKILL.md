---
name: mercari-transaction-messages
description: Mercari 取引消息（成交后订单内私信）的起草、发送与核实流程。凡涉及给买家发取引消息、发货说明、进度通知时，使用本技能。核心教训来自一次真实事故：发送后用页面 DOM 回读判断成败，页面不把新消息渲染进 DOM 文本，误判失败后重复点击，连发了 5 份。核实只走 API 层。
---

# Send and verify Mercari transaction messages

## Draft the message

- Show the full message text to the user and wait for an explicit confirmation before any send.
- Never mention that the item comes from overseas or is in international transit. Say the item is being prepared and the buyer will be notified when it is ready.
- Keep the wording to what the transaction needs and leave out account details the buyer does not need.

## Send it once

- Send with one click in the logged-in Mercari session the user already uses.
- Do not click send again when the result looks uncertain. An uncertain send may already have succeeded.
- Do not change the text and resend to fix a doubt. Draft any correction and wait for the user's confirmation first.

## Verify at the API layer

- 定版发送流程：先记录基线消息数 → 填入文本并校验字数 → **只点一次**发送 → 重载页面、捕获页面自身发出的 get_messages 响应 → **先 JSON.parse 解码再计数**（响应里的日文是 \uXXXX 转义，直接在原始字符串里搜关键词恒为 0，会误判未发送）。
- Never judge a send by reading the page DOM text. DOM 回读只能作第二信号，定案以 get_messages 解析计数为准。
- 裸 fetch 调 get_messages 会 400（缺 App 专用 header），核实只能捕获页面自身发出的那次响应（如经 CDP Network.getResponseBody）。
- When the API check does not show the message, treat it as possibly sent. Re-check the API or server record and do not click send again.
- Count the copies the API or server record shows. Report success only when exactly one copy is there. Mercari 取引消息**无撤回功能**，重复发送不可逆，宁可少发不可多发。

## Report the result

- Report the copy count and how you verified it.
- If more than one copy went out, say so at once. Draft one short apology and wait for the user's confirmation before sending it.
- When a send stays unconfirmed after an API re-check, stop and report what is unresolved instead of retrying.
