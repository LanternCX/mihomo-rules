# Mihomo 内核分组规则备份

分组：

- `Default`
- `Auto`
- `Select`
- `Develop`
- `Media/Social`
- `Microsoft`
- `Direct`

运行模式为 rule。

客户端配置关闭 allow-lan，避免代理端口对局域网开放。

客户端启用 tcp-concurrent、unified-delay 和严格进程匹配。

### Default

`Default` 是默认出海入口。

该分组包含 `DIRECT`、`Auto`、`Select` 和三条 IPv4 节点。

### Auto

`Auto` 包含三条 IPv4 协议节点：HY2、Reality 和 XHTTP。

IPv6 节点没有放入 `Auto`，避免无 IPv6 路由时断网。

### Select

`Select` 包含 `DIRECT`、全部 IPv4 节点和 IPv6 灰度节点。

IPv6 节点只在这里手动选择。

### Develop

`Develop` 覆盖 GitHub 和 AI 类规则。

该分组可选择默认出海、自动选择、手动选择或具体 IPv4 节点。

### Media/Social

`Media/Social` 覆盖流媒体和社交类规则。

范围包括 YouTube、X、Telegram、Discord、Reddit、Facebook、Instagram 等类别。

### Microsoft

`Microsoft` 默认把 `DIRECT` 放在前面。

该配置保留 OneDrive、Microsoft 服务在可直连时优先直连的使用习惯，同时允许手动切换出海。

### Direct

`Direct` 只包含 `DIRECT`。

用于国内、局域网和明确直连流量。

## 规则覆盖

- LAN
- Hijacking
- GitHub
- AI：OpenAI、Claude、Gemini、Copilot
- Telegram
- X / Twitter
- YouTube
- Discord
- Reddit
- Facebook
- Instagram
- Netflix、Disney、Spotify、TikTok、GlobalMedia
- Microsoft 与 OneDrive
- China / ChinaMax
- Proxy / Global

## DNS

### Clash DNS

Clash 配置启用 DNS 接管。

当前 DNS 上游使用 DoH，不使用明文 53 端口作为主要查询路径。

代理服务器域名解析使用 `proxy-server-nameserver`，同样配置为 DoH，避免 Clash 为连接代理节点而先发起明文 DNS 查询。

使用 Fake-IP 模式，Fake-IP 地址池为 `198.18.0.1/16`。

TUN 已启用，使用 mixed 栈，开启自动路由和自动接口识别。

TUN 开启 strict route，并劫持常规 DNS 请求与 TCP DNS 请求，减少系统 DNS 绕过代理的概率。

### DNS 分流

国内域名走国内 DoH 解析策略。

国外域名走海外 DoH 解析策略。

fallback、direct-nameserver 和 proxy-server-nameserver 都使用 DoH。

局域网、本机和私有地址保留直连处理。