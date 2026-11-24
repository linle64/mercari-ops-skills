# 克隆浏览器搭建定版

为什么要克隆：自动化需要带远程调试端口（CDP）的浏览器实例，又不能动用户日常浏览器的窗口和标签。解法是把真实浏览器 profile 克隆一份到独立目录，用调试端口另开——登录态随 profile 迁移，自动化在克隆里跑，用户原浏览器不受打扰。

分工：Chrome 克隆（端口 9223）＝A 号；Brave 克隆（端口 9224）＝B 号。

## 1. 复制 profile（件要带齐）

从真实浏览器用户数据目录（如 Chrome 的 `~/Library/Application Support/Google/Chrome`）复制到克隆目录（如 `~/cdp-chrome`）：

- 根目录的 `Local State`
- `Default` 下：`Cookies`（浏览器运行时用 sqlite 热备复制：`sqlite3 Cookies ".backup <目标>"`，直接 cp 容易拷到写一半的库）、`Preferences`、`Local Storage`、`IndexedDB`、`Session Storage`、`Device Bound Sessions`、`WebStorage`

只复制 Cookies 会显示未登录——存储件必须带齐。

## 2. 启动（必须走 GUI 会话）

```
open -na "Google Chrome" --args --user-data-dir=~/cdp-chrome --remote-debugging-port=9223
open -na "Brave Browser" --args --user-data-dir=~/cdp-brave --remote-debugging-port=9224
```

**铁律**：克隆只能经图形会话用 `open -na` 启动。经 SSH 直拉的进程拿不到 macOS Keychain，浏览器会判定 Cookies 不可解密并**清零**，登录态全失。这一步只能由桥（见 ssh-tailscale.md）或用户本人在终端执行。

有扩展显示器时把克隆窗口放扩展屏，不占用户主屏。

## 3. 登录（用户本人）

- 首开验收：打开 Mercari 的 mypage，能看到已登录账号就是迁移成功。
- 迁移失败或货源平台没登录态时，**请用户本人在克隆窗口里登录**（Mercari、淘宝、拼多多逐个来）。密码、验证码永远用户本人输入，agent 不代输、不经手。
- 遇只有密码框的再认证页：点密码框下方自动填充候选里的邮箱条目即可填入（前提：窗口在前台、页面有焦点）。

## 4. 日常纪律

- 关克隆：`pkill -f 'user-data-dir=.*/cdp-chrome'`（Brave 同理），精确匹配，别误伤用户原浏览器。
- **不要用「整库覆盖式」的克隆刷新脚本**：会把克隆里自己登录的会话冲掉。登录态过期就在克隆里重登，不要重拷 profile。
- 导航用 `open -a "<浏览器>" '<URL>'`；不要用 osascript 经 SSH 操作（会挂死）。
- CDP 驱动方式与表单定版在 mercari-listing 的 publish-flow.md；环境坑（Chrome 克隆读不到桌面文件内容等）在 mercari-ops 的 environment.md。
