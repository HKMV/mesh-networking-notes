# 五款 Mesh VPN 方案对比

> **取材原则**：本页每一格都能在官方文档或官方仓库里找到出处，出处在 [sources.md](sources.md)。
> 官方没写清楚的，这里写 `未确认`，不拿印象补。抓取时间 2026-11。
> 五款各自最容易撞上的一条限制，单独记在 [pitfalls.md](pitfalls.md)（现象 / 原因 / 怎么绕的）。

## 速查表

| 维度 | EasyTier | NetBird | Tailscale | ZeroTier | vnt |
|---|---|---|---|---|---|
| 协议基础 | 自有隧道，加密可选 AES-GCM / WireGuard；官方**未**称「基于 WireGuard」 | 官方明确「WireGuard overlay」 | 官方明确基于 WireGuard | **自有协议**（官方：「original」，分 VL1 传输层 / VL2 以太网仿真层） | 官方未说明数据面协议，仅「多通道 UDP」+ 传输层 tcp-tls / quic / wss |
| 是否需要中心组件 | **不需要**（官方：「没有也不需要类似 ZeroTier、Tailscale 的中心控制面板」） | 需要：Management / Signal / Relay 四组件 | 需要：coordination server（单点中心） | 需要：root servers（planet）+ network controller 两级 | 默认需要服务端 vnts；官方另称支持「完全无服务器组网」 |
| 能否自建中心 | 无中心可自建；可选 `--config-server` 下发配置 | **能**，官方 self-hosted quickstart / advanced | **官方不提供**，控制面闭源；官方承认第三方 Headscale | **能**，但 controller 在仓库 `nonfree/`，**仅限非商业** | vnts 可自建（服务端许可证未确认） |
| 中继/转发自建 | 能，「共享节点」，可 `--relay-network-whitelist` 限制或 `--private-mode` 关闭 | 能，relay 可自建（relay 目录 AGPLv3） | 能，custom DERP，官方标注 **alpha** | 部分：TCP relay 可自建；root 兼作中继，自建 root 属 enterprise | 能，vnts 即中继 |
| 二层能力 | 官方未提及（只有 TUN / 三层） | 未见二层表述 | 官方无 L2 桥接；文档说明工作在 network layer，广播/组播行为可能不同 | **原生支持**，Layer 2 Bridge，以太网仿真 | 支持 `tap` 模式，完整透传 Ethernet 帧（仅 README 一句） |
| 子网/整网段 | 支持，「子网代理」（点对网 / 网对网） | 支持，「Networks」（旧 Routes 已废弃） | 支持，「subnet routers」 | 支持，「Managed Routes」 | 支持，「点对网 / 网对网」 |
| IPv6 | 支持 `--ipv6` 双栈（虚拟地址分配细节未确认） | 支持双栈 overlay，每账号独立 IPv6 前缀 | 支持，每设备 `fd7a:115c:a1e0::/48` ULA | 支持，RFC4193 / 6PLANE | 有虚拟 IPv6（`fd76:6e74::/96`），但不进 NAT / 端口映射 / 子网出口 |
| 许可 | 仓库 **LGPL-3.0** | 客户端 BSD-3；`management/ signal/ relay/ combined/` 为 **AGPLv3** | 客户端 BSD-3；**控制面闭源** | `node/ osdep/ service/` 等 **MPL-2.0**；`nonfree/`（含 controller）为 source-available **仅非商业** | **Apache-2.0**（服务端 vnts 未见 LICENSE，未确认） |
| 官方平台 | Win / macOS / Linux / FreeBSD / Android，含 x86/ARM/MIPS | Linux / macOS / Windows / Android / iOS + NAS、路由固件、容器 | macOS / iOS / Windows / Linux / Android + 容器、NAS、OPNsense | Tier1：Win / macOS / Linux / iOS / Android / Docker；Tier2：FreeBSD / OpenWRT / OPNsense 等 | 官方只列 Win / macOS / Linux / Android（iOS 走 WireGuard 接入） |

## 逐款要点

### EasyTier

- 官方定位：**Rust + Tokio** 驱动的「简单、安全、去中心化的异地组网方案」。
- 架构：官方明说**没有也不需要中心控制面板**；任一节点都能承担转发，公网节点可自建为「共享节点」供无公网 IP 的节点发现与中转。
- 组网标识是「网络名 + 网络密钥」，两者都相同才能组网；虚拟 IP 可静态指定或 DHCP 分配。
- 与选型相关的关键点：**控制面不是必需品**，所以不存在「控制面挂了网络起不来」这一类故障；代价是没有统一的设备/ACL 管理面板，权限与可达性要靠网络密钥和路由配置来控制。
- 注意：`--relay-network-whitelist` 不限定时，自建共享节点默认会给所有网络做转发，公网节点必须先收紧这点。

### NetBird

- 官方架构：Client / **Management**（控制面）/ **Signal**（信令）/ **Relay**（中继）四组件。
- 直连优先、失败回退中继，官方原文：「NetBird always prefers a direct peer-to-peer (P2P) connection and falls back to a relay server when a direct path can't be established.」中继路径仍是端到端加密，中继看不到内容。
- Signal 不可达时官方原话是连接会「silently lands on the relay」—— 这是排查「为什么在走中继」时最容易被忽略的一条。
- 自建路径有两级：quickstart（单容器 + 内置 IdP）和 advanced（多容器 + 自有 IdP），数据库默认 SQLite，可换 Postgres。
- 术语变动要注意：官方现在用 **Networks**，旧的 **Routes** 已标 deprecated（exit node 除外）。查资料时别拿旧页面的说法。

### Tailscale

- 官方明确基于 WireGuard，并在此之上加 NAT 穿透、TCP 传输、访问控制。
- 中心组件官方叫 **coordination server**，官方自述是「the single, centralized component」；它挂掉时「网络大部分仍可继续工作」，但无法新增设备/用户、无法轮换密钥、无法更新 ACL。
- **控制面闭源**：官方有专节解释原因，且**不提供自建控制面**；只提供把客户端指向自定义 coordination server 的能力，并在官方文档里指名第三方 **Headscale**。
- 中继叫 **DERP**，支持自建 custom DERP，但官方标注 **alpha**，并列出多项限制（不能放在 NAT / 负载均衡后、不支持 device sharing 等）。另有官方的 **peer relays**（用 tailnet 内设备当中继）。
- 没有 L2 桥接；官方 OSI 页面提示广播/组播行为可能与传统 LAN 不同。

### ZeroTier

- 协议是自有实现，不是 WireGuard；VL2 层直接做以太网仿真，所以**二层桥接是原生能力**，这是它跟另外几款最本质的区别。
- 两级中心：root servers（planet，官方运营）与 network controller。没有 controller 时只有 ad-hoc 网络（公开、无访问控制）。
- **自建 controller 有个大坑**：官方提供自建方案，但代码位于仓库 `nonfree/`，许可是 source-available **仅限非商业使用**，商用要单独谈商业许可；自建根节点在官方口径里属于 enterprise engagement。
- 中继可自建的是 TCP relay（HTTPS 回退，官方慢），root 兼作中继那条路没有 DIY 指引。

### vnt

- Rust 实现；官方**没有**说基于 WireGuard —— WireGuard 在它语境里是「接入方式」（iOS 用 WireGuard 接入 vnt 网络），不是底层协议。
- 默认形态是「服务端 vnts + 客户端」，组网编号 `-k` + 服务端地址 `-s`；官方 README 另称支持「完全无服务器组网」（固定虚拟 IP + `peer_address` 种子 + Gossip），但**该模式没有官方文档页**。
- **版本断层必须注意**：官方文档站 `rustvnt.com` 内容停留在 1.x（`vnt-cli`、`-w`），仓库 README 是 2.0 且明确「和 1.0 不兼容」（`vnt2_cli`、`-k`、`-s`）。**参数以 README 为准**，混用会浪费大量时间 —— 这条与其它四款的同类问题一起记在 [pitfalls.md](pitfalls.md)。
- 二层：README 写 `tap` 模式完整透传 Ethernet 帧（文档站无 TAP 内容）。
- 加密取决于是否设密码：官方 README 明确「未设置密码时，节点间数据不加密」。

## 我的结论

**主用 EasyTier**：中继（共享节点）自己搭，且官方设计上就不依赖中心控制面 —— 少一个会挂的东西，也少一处需要长期维护的服务。

**NetBird 做备用**：备用的价值是**故障域不同**。它自带一整套 Management / Signal / Relay，跟 EasyTier 那头没有任何共用组件；代价是要多维护一份，所以只当备用，不参与日常流量。

**另外三款为什么没进主备**：不是能力不行 —— ZeroTier 的二层、Tailscale 的生态与 ACL、vnt 的轻量都各有优势。卡在三点：一是这一轮主用要求「中继完全自控 + 不引入额外中心」，Tailscale 干脆不提供自建控制面，ZeroTier 的 controller 商用受限；二是多平台里 FreeBSD / 路由器这一档的覆盖，EasyTier 与 vnt 更完整；三是维护成本，多一套控制面就多一份长期负担。折下来选了 EasyTier。
