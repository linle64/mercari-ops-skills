# 通道搭建定版：SSH + Tailscale + agent-desktop 桥

适用场景：agent 跑在云端 VM 上，浏览器和 Mercari 登录态在一台 Mac（执行机）上。目标是让 agent 能稳定地：① 在 Mac 上执行命令和传文件（SSH）；② 操作 Mac 的 GUI（agent-desktop，经桥）；③ 用 CDP 驱动克隆浏览器（见 clone-browsers.md）。

占位符：`<MAC_TAILNET_IP>`（Mac 的 100.x 地址）、`<MAC_USER>`（Mac 的 SSH 用户名）、`<AGENT_HOME>`（agent 主机上存放密钥的家目录绝对路径）。全部换成用户自己的值，**这些值只留在用户本地 config，不要写进任何要外发的文件**。

## 1. Tailscale 组网

1. Mac 与 agent 主机都安装 Tailscale，登录**同一个 tailnet**。
2. Mac 侧确认在线，记下它的 tailnet IP（100.x.x.x）或 MagicDNS 名。
3. agent 主机上 `ping <MAC_TAILNET_IP>` 能通，才继续。

## 2. Mac 开 SSH

1. Mac：系统设置 → 通用 → 共享 → 打开「远程登录」，允许 `<MAC_USER>` 访问。
2. agent 主机生成密钥（已有就复用）：`ssh-keygen -t ed25519`，把公钥追加到 Mac 的 `~/.ssh/authorized_keys`（这一步需要用户在 Mac 前配合一次，或用密码登录完成首次写入）。

## 3. SSH config（模板）

agent 主机 `~/.ssh/config`：

```
Host mac-exec
  HostName <MAC_TAILNET_IP>
  User <MAC_USER>
  IdentityFile <AGENT_HOME>/.ssh/id_ed25519
```

固定打法：`ssh -F <AGENT_HOME>/.ssh/config mac-exec '<cmd>'`。

**坑（实证）**：agent 进程的有效用户可能不是家目录属主（如以 root 运行、HOME 指向别处），ssh 默认读的 config 和 `~` 展开都会错位。所以必须 `-F` 显式指定 config 绝对路径，IdentityFile 也写绝对路径，不要用 `~`。

## 4. 出网代理备选（直连不通时才用）

如果 agent 主机的出网流量被强制走 HTTP 代理，SSH 直连 tailnet 地址会被代理劫持、连接重置。两个解法任选：

- **首选**：`ProxyCommand tailscale nc %h %p`（agent 主机装了 Tailscale CLI 并已入网时最干净），加到上面的 Host 段里。
- **备选**：自备一个最小 CONNECT 脚本作 ProxyCommand：读环境变量里的代理地址，向代理发 `CONNECT <MAC_TAILNET_IP>:22`，协商成功后把标准输入输出接到隧道上。脚本放 agent 主机本地 `tools/`，config 里 `ProxyCommand python3 <path> %h %p`。

验收标准不变：`ssh -F ... mac-exec 'echo ok'` 返回 ok。

## 5. agent-desktop 安装（Mac 侧）

桌面自动化工具（Vercel Labs 的 agent-desktop，基于 macOS 无障碍树）：

1. `npm config set allow-scripts=agent-desktop --location=user`（不设会被 npm 拦截 postinstall），再 `npm install -g agent-desktop`。注意：`npm install -g --allow-scripts=...` 的行内写法会被解析成读本地 package.json，别用。
2. 把 `$(npm prefix -g)/bin` 写进 `~/.zshrc`；调用时用全路径最稳。
3. 系统设置 → 隐私与安全性 → 给它开**无障碍**和**录屏**两项权限（用户本人操作）。
4. 经 SSH 调用要带登录 shell 补 PATH：`ssh -F ... mac-exec "zsh -lic 'agent-desktop status'"`，status 显示 accessibility 与 screen_recording 均 granted 才算装好一半——录屏还有下面的归属坑。

## 6. 桥脚本（录屏权限的定版解法）

**坑**：macOS 按「启动进程的应用」归属录屏权限。经 SSH 拉起的 agent-desktop 常驻进程，录屏会被钉死 denied（系统设置里开关开着也没用，SSH 截图只能截到壁纸）。

**解法**：杀掉 SSH 拉起的常驻进程（`pkill -f 'agent[-]desktop'`），改由 Mac 图形会话里的桥脚本拉起：

```zsh
# ~/ad-bridge.sh —— 用户在 Mac 终端里运行：nohup zsh ~/ad-bridge.sh &>/dev/null &
while true; do
  if [[ -f /tmp/ad-job.sh ]]; then
    mv /tmp/ad-job.sh /tmp/ad-job.run.sh
    zsh /tmp/ad-job.run.sh > /tmp/ad-job.out 2>&1
    rm -f /tmp/ad-job.run.sh
  fi
  sleep 1
done
```

agent 侧把要跑的命令写成 job 文件投递（用 scp 暂存再 cp 落位，直接写容易被桥读到半截），结果读 `/tmp/ad-job.out`。**此后截图与物理操作一律经桥执行，不再从 SSH 直接调 agent-desktop。**

## 7. 连通性验收（全过才算通道搭好）

- [ ] `ssh -F <config> mac-exec 'echo ok'` 返回 ok
- [ ] 经桥执行 `agent-desktop status`：无障碍＋录屏均 granted
- [ ] 经桥截图：能截到真实窗口内容（不是只有壁纸）
- [ ] `open -a "Google Chrome" 'https://jp.mercari.com/'` 能在 Mac 上打开页面
- [ ] 文件双向传通：agent → Mac（scp job 文件）、Mac → agent（取回截图／图片）
