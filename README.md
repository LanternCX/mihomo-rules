# Mihomo 内核分组规则备份

仓库中的节点地址、认证信息、服务域名和传输路径均为脱敏占位符，使用前需填入自己的配置。

公共 DNS、连通性检测、规则下载地址，以及本机监听地址和保留网段予以保留；这些地址不是私有服务器信息。

分组：

- `Default`
- `Auto`
- `Select`
- `Develop`
- `Media/Social`
- `Microsoft`
- `Direct`
- `DNS`

运行模式为 rule。

客户端配置关闭 allow-lan，避免代理端口对局域网开放。

客户端启用 tcp-concurrent、unified-delay 和严格进程匹配。

### Default

`Default` 是默认出海入口。

该分组包含华为云中转、`DIRECT`、`Auto`、`Select` 和三条 DMIT IPv4 节点。

### Auto

`Auto` 包含华为云中转及三条 DMIT IPv4 协议节点：HY2、Reality 和 XHTTP。

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

`Microsoft` 提供华为云中转、`DIRECT`、其他策略组和 DMIT IPv4 节点，支持手动切换 OneDrive 与 Microsoft 服务的线路。华为云选项排在首位。

### Direct

`Direct` 只包含 `DIRECT`。

用于国内、局域网和明确直连流量。

### 华为云备用链路

`☁️ 华为云中转 → DMIT` 节点提供 VLESS + WebSocket + TLS 中转入口，可在直连 DMIT 不稳定时选用。使用前需填写服务器地址、UUID、TLS 服务名、WebSocket 路径与 Host。

该节点可在 `Default`、`Auto`、`Select`、`Develop`、`Media/Social`、`Microsoft` 中选用。华为云选项排在首位，手动选择组在没有已保存选择时使用首项。

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

每个规则集使用独立的相对缓存路径，避免多个规则集覆盖同一文件。

## DNS

### Clash DNS

Clash 配置启用 DNS 接管。

DNS 服务仅监听本机回环地址，避免局域网设备访问解析端口。

DNS 查询、上游域名引导解析和代理服务器域名解析均使用 DoH，失败时不回退到明文 DNS。

代理服务器域名解析使用 `proxy-server-nameserver`，同样配置为 DoH，避免 Clash 为连接代理节点而先发起明文 DNS 查询。

使用 Fake-IP 模式，Fake-IP 地址池为 `198.18.0.1/16`。

TUN 已启用，使用 mixed 栈，开启自动路由和自动接口识别。

TUN 开启 strict route，并劫持常规 DNS 请求与 TCP DNS 请求，减少系统 DNS 绕过代理的概率。

### DNS 分流

国内域名同时配置阿里、腾讯直连 DoH，以及经 `🔎 DNS` 查询的 Cloudflare、Google DoH；国外域名使用经 `🔎 DNS` 查询的 Cloudflare、Google DoH。

`direct-nameserver` 同样包含这四个上游，并关闭 `direct-nameserver-follow-policy`，避免直连解析被单一域名策略限制。多个上游并发查询，某个供应商或直连路径超时后，其他可用上游仍可返回结果。

普通查询额外配置 Cloudflare、Google DoH 作为 `fallback`。`nameserver-policy` 优先于普通查询的 fallback，因此每条域名策略也分别配置多个上游；fallback 结果仍受 `fallback-filter` 筛选。

`🔎 DNS` 是独立的自动回退组，按 Reality、XHTTP、HY2 顺序选用可用的 IPv4 节点，每 60 秒通过 IP 形式的 HTTPS 地址检查连通性。不绑定华为云，也不随 `Default` 的手动选择变化。三个协议共享同一台 VPS，因此不能覆盖整台 VPS 离线的故障。

`default-nameserver` 和 `proxy-server-nameserver` 使用 IP 形式的 DoH，同时配置直连和经 `🔎 DNS` 的上游。DNS 组内节点的服务器地址必须填写 IP，避免解析代理服务器时形成循环依赖。

网站连接仍按原有规则分流。若所有加密上游及代理路径均不可用，解析会失败，不降级为明文查询。

局域网、本机和私有地址保留直连处理。
