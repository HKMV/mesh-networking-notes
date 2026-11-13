# 踩坑记录

> **这五条都是我实际部署时撞上的。**「现象」按实机看到的写；「原因」和「怎么绕的」能逐句对到官方文档 —— 不凭印象下结论，出处附在每节末尾，汇总见 [sources.md](sources.md)。
> 官方没写清的地方不编，标 `未确认`。抓取时间 2026-11，产品迭代快，具体参数按自己环境复核。

## Tailscale：自建 DERP 起得来，客户端却不用它

| 环&nbsp;节 | 说&nbsp;明 |
|---|---|
| 现&nbsp;象 | `derper` 进程在跑、日志也正常，`tailscale netcheck` 里就是看不到自建 region；节点间连接仍标 relayed，走的还是官方 DERP |
| 原&nbsp;因 | 自建 DERP 官方标注 **alpha**，并明确「must not be behind NAT devices (such as firewalls) or load balancers」。要求是**公网直连**：放通 HTTP 80 / HTTPS 443 / STUN 3478，且允许**出入向 ICMP**（它靠 ICMP 与源 IP 识别设备）。缺任一项，该 region 探测不到，客户端就继续用官方 DERP |
| 怎&nbsp;么&nbsp;绕&nbsp;的 | 1) 别把 DERP 放在端口映射/反代/负载均衡后面；2) 80、443、3478 全放通，ICMP 双向放通（HTTP 端口不支持自定义，只有 HTTPS/STUN 可通过 `DERPNode.DERPPort` / `DERPNode.STUNPort` 改）；3) 启动用 `sudo derper --hostname=example.com --verify-clients`，只放自己 tailnet 的流量；4) 用 `cmd/derpprobe` 配 `--derp-map=file://` 验证 region 真的可用 —— **别用「进程在跑」当成功的判据** |

**另外三个官方明说的限制**，选型时容易漏：custom DERP 不吃 regional routing、不与 Mullvad exit node 共存、拿不到一部分控制面优化。

出处：[Custom DERP servers（alpha、限制清单、端口表、`--verify-clients`、`derpprobe`）](https://tailscale.com/docs/reference/derp-servers/custom-derp-servers)

## ZeroTier：按旧教程配 moon，新版里它已经没了

| 环&nbsp;节 | 说&nbsp;明 |
|---|---|
| 现&nbsp;象 | 照旧教程自建 root server（moon）来加速/当中继，新版客户端上找不到对应入口，现文档里也搜不到 moon 的任何说明；想改用自建 controller 顶上，发现默认发行包里根本没有 controller |
| 原&nbsp;因 | 1.16.0（2025-08-21）把这块整组换掉了。官方 release notes 原文：「**"Moons" are now considered even more extra *deprecated* and should not be used in new deployments.**」替代品是 **network-specific relays**（当时标 preview / beta）。同一版还改了构建：「**Default binary builds no longer contain the controller.**」—— 自建 controller 要 `make ZT_NONFREE=1`，而带 `ZT_NONFREE` 构建出来的产物许可从 MPL 变成商业专有 |
| 怎&nbsp;么&nbsp;绕&nbsp;的 | 别再按 moon 教程折腾，那条路官方已经不给了。中继改用 **network-specific relays**（还是 preview，上生产前自己压测）或走官方托管；确实要自建控制面，就得接受 `ZT_NONFREE=1` + 商业许可，那就不是「自建省掉授权」而是「换一种授权」—— 方案选型时这条要提前摆到桌面上 |

**顺带**：moons 之外，root servers（planet）仍由官方运营，自建 root 在官方口径里属 enterprise engagement，没有 DIY 步骤。

出处：[RELEASE-NOTES.md（1.16.0：moons deprecated、network-specific relays、默认不含 controller、ZT_NONFREE 的许可变化）](https://github.com/zerotier/ZeroTierOne/blob/master/RELEASE-NOTES.md)、[What is a controller](https://docs.zerotier.com/what-is-a-controller/)

## NetBird：显示 connected，其实在走中继

| 环&nbsp;节 | 说&nbsp;明 |
|---|---|
| 现&nbsp;象 | `netbird status` 里对端标着 **Relay**，吞吐只有直连的零头，其余一切显示正常（都 online，没有报错） |
| 原&nbsp;因 | Signal 不通时会**静默**落到中继。官方原文：「If Signal is unreachable, the peers can't even compare notes, and the connection silently lands on the relay.」—— 对端连路径信息都无法交换，直接回退，**不报错、不告警**，所以人只会看到「慢」 |
| 怎&nbsp;么&nbsp;绕&nbsp;的 | 排查顺序改成先 Signal 后穿透：1) 确认 Signal 端点可达（官方托管为 `signal.netbird.io` TCP/443；自建按自己部署的端口）；2) 再看 STUN 3478 与直连是否被防火墙掐掉；3) 只有这两步都通过，才去钻 NAT 类型。中继路径仍是端到端加密，中继看不到内容，所以这通常是性能问题而非安全问题 |

出处：[Understanding NAT and connectivity](https://docs.netbird.io/about-netbird/understanding-nat-and-connectivity)、[Troubleshooting relayed connections](https://docs.netbird.io/help/troubleshooting-relayed-connections)、[Ports and firewalls](https://docs.netbird.io/about-netbird/ports-and-firewalls)

## EasyTier：自建共享节点默认给别人转发

| 环&nbsp;节 | 说&nbsp;明 |
|---|---|
| 现&nbsp;象 | 公网共享节点跑起来后**出网流量异常**，跟自己设备的实际用量对不上；节点列表里还出现网络名/密钥跟自己无关的节点在借道 |
| 原&nbsp;因 | 官方默认把共享节点对**所有**网络开放转发。`--relay-network-whitelist` 不设就是不限制，`--private-mode` 默认关闭 —— 等于一台公网机器给别人做免费中转 |
| 怎&nbsp;么&nbsp;绕&nbsp;的 | 二选一或都上：`--relay-network-whitelist <网络名> [...]`（只转发白名单网络，支持通配符，多个网络名用空格分隔）；或 `--private-mode true`（网络名/密码与本网络不同的节点，不允许经本节点握手或中转）。再用 `--foreign-relay-bps-limit` 给外部转发限速兜一层。**端口别照抄网上的**：默认网段与默认监听端口在官方多页之间就不一致，以自己实际监听的端口放防火墙 |

出处：[自建公共共享节点](https://easytier.cn/guide/network/host-public-server.html)、[配置项（`--relay-network-whitelist` / `--private-mode` / `--foreign-relay-bps-limit`）](https://easytier.cn/guide/network/configurations.html)

## vnt：照文档站抄参数，2.0 直接不认

| 环&nbsp;节 | 说&nbsp;明 |
|---|---|
| 现&nbsp;象 | 照文档站 `rustvnt.com` 抄 `vnt-cli` 那一套参数启动，客户端起不来（二进制/参数对不上）；换成 2.0 之后才跑通 |
| 原&nbsp;因 | 文档站内容停在 **1.x**（`vnt-cli`、`-w`），仓库 README 已经是 **2.0**，官方原话「vnt2.0 整体重构了一遍，和 1.0 不兼容」。2.0 的二进制叫 `vnt2_cli`（后台看状态用 `vnt2_ctrl`），组网是 `-k <组网编号> -s <服务端地址:端口>`，例：`./vnt2_cli -k 123456 -s vnt2.rustvnt.com:29872` |
| 怎&nbsp;么&nbsp;绕&nbsp;的 | 就是换成 2.0 走通的：按 README 拿 `vnt2_cli`，`-k` 组网编号 + `-s` 服务端地址起来，参数**一律照仓库 README 对应版本**抄，文档站只当概念参考。另两条同源坑：**未设置密码时节点间数据不加密**（必须设）；虚拟 IPv6 段 `fd76:6e74::/96` 不进 NAT / 端口映射 / 子网出口，别指望它当出口地址 |

出处：[vnt README（2.0，参数以此为准）](https://github.com/vnt-dev/vnt/blob/main/README.md)、[文档站（1.x，注意断层）](https://rustvnt.com/guide/introduction.html)

## 这五条放在一起看

- **三条是「默认值 / 默认信任范围」问题**：Tailscale 的网络位置（默认不可能在 NAT 后）、EasyTier 的转发范围（默认不限）、vnt 的加密（默认靠密码决定开关）。共同点是**默认能跑通**，所以不会在部署时暴露，而是运行一段时间后由流量、性能或评审把它们翻出来。
- **两条是「文档落后于代码」**：vnt 的文档站落后一个大版本、ZeroTier 的 moon 已从文档与默认构建里退场却只在 release notes 里说。这两条的共同点是**照文档做会失败**，而且失败方式不像配置错误，像是「这功能不存在了」，很容易被当成自己环境的问题查半天。
- 因此选型时的检查动作很具体：**中继放哪、谁能借道、加密开没开、文档版本与二进制版本是否对得上**。这四条比功能列表更能决定后期维不维护得下去。
