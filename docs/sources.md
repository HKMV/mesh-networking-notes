# 出处与未确认事项

> 抓取时间：**2026-11**。产品迭代快，链接和结论都可能过期，引用前请自行复核。

## 许可证据（仓库 LICENSE 原文）

| 产品 | 仓库 | LICENSE 原文要点 |
|---|---|---|
| Tailscale | `tailscale/tailscale` | `LICENSE` 首行 `BSD 3-Clause License`。官方博客说明：Linux / Android 客户端完全开源，Windows / macOS / iOS 的 GUI 包装闭源，**控制面闭源** |
| ZeroTier | `zerotier/ZeroTierOne` | 根 `LICENSE.txt` 指向三份：`node/ osdep/ service/` 等 → `LICENSE-MPL.txt`（MPL-2.0）；`nonfree/` → 仅限非商业的 source-available 许可；`ext/` 保留各自原许可 |
| NetBird | `netbirdio/netbird` | 根 `LICENSE`：「This BSD-3-Clause license applies to all parts of the repository except for the directories `management/`, `signal/`, `relay/` and `combined/`. Those directories are licensed under the GNU Affero General Public License version 3.0 (AGPLv3).」 |
| EasyTier | `EasyTier/EasyTier` | `LICENSE` 首行 `GNU LESSER GENERAL PUBLIC LICENSE / Version 3, 29 June 2007` |
| vnt | `vnt-dev/vnt` | `LICENSE` 首行 `Apache License / Version 2.0, January 2004`；`Cargo.toml` 写 `license = "Apache-2.0"`。**服务端 `vnt-dev/vnts` 仓库未见 LICENSE 文件，未确认** |

## 官方文档链接

### Tailscale
- 是什么 / WireGuard：https://tailscale.com/docs/concepts/what-is-tailscale
- 控制面挂掉时的行为：https://tailscale.com/docs/reference/coordination-server-down
- 控制数据面说明：https://tailscale.com/docs/concepts/control-data-planes
- 自定义 coordination server（指名 Headscale）：https://tailscale.com/docs/how-to/set-up-custom-coordination-server
- 为何控制面闭源 / 开源范围：https://tailscale.com/blog/opensource
- Custom DERP（标注 alpha）：https://tailscale.com/docs/reference/derp-servers/custom-derp-servers
- DERP 与 peer relay：https://tailscale.com/docs/reference/derp-servers 、https://tailscale.com/docs/features/peer-relay
- Subnet routers：https://tailscale.com/docs/features/subnet-routers
- OSI / 层级说明：https://tailscale.com/docs/concepts/tailscale-osi
- IPv6：https://tailscale.com/docs/concepts/ipv6

### ZeroTier
- 协议（VL1 / VL2、以太网仿真、加密套件）：https://docs.zerotier.com/protocol/
- 关于 controller：https://docs.zerotier.com/what-is-a-controller/ 、https://docs.zerotier.com/controller/
- root 宕机行为：https://docs.zerotier.com/faq/rootdowntime/
- 中继（含自建 TCP relay）：https://docs.zerotier.com/relay/ 、https://docs.zerotier.com/faq/relaying/
- 兼容性分级（Tier1/2/3）：https://docs.zerotier.com/compatibility/
- Release notes（1.16.0：moons 标 deprecated、network-specific relays、默认构建不含 controller、`ZT_NONFREE` 的许可变化）：https://github.com/zerotier/ZeroTierOne/blob/master/RELEASE-NOTES.md
- 二层桥接：https://docs.zerotier.com/bridging/
- 物理网段路由：https://docs.zerotier.com/route-between-phys-and-virt/

### NetBird
- 介绍 / 定位：https://docs.netbird.io/introduction
- 架构与组件：https://docs.netbird.io/about-netbird/how-netbird-works
- NAT 与连接优先级：https://docs.netbird.io/about-netbird/understanding-nat-and-connectivity
- 端口与防火墙：https://docs.netbird.io/about-netbird/ports-and-firewalls
- 自建 quickstart：https://docs.netbird.io/selfhosted/selfhosted-quickstart
- 自建 advanced：https://docs.netbird.io/selfhosted/selfhosted-guide
- 配置文件（SQLite / Postgres、relay 配置）：https://docs.netbird.io/selfhosted/maintenance/configuration-files
- Networks（现用术语）：https://docs.netbird.io/manage/networks
- IPv6 overlay：https://docs.netbird.io/manage/settings/ipv6
- 中继排查：https://docs.netbird.io/help/troubleshooting-relayed-connections

### EasyTier
- 简介：https://easytier.cn/guide/introduction.html
- FAQ（明确说了没有也不需要中心控制面板）：https://easytier.cn/guide/faq.html
- 自建公共共享节点：https://easytier.cn/guide/network/host-public-server.html
- 配置项（`--relay-network-whitelist`、`--private-mode`、`--ipv6` 等）：https://easytier.cn/guide/network/configurations.html
- 快速组网：https://easytier.cn/guide/network/quick-networking.html
- 子网代理（点对网）：https://easytier.cn/guide/network/point-to-networking.html
- P2P 说明：https://easytier.cn/guide/aboutp2p.html
- 许可说明页：https://easytier.cn/guide/license.html
- README（中文）：https://github.com/EasyTier/EasyTier/blob/main/README_CN.md

### vnt
- 官方文档站（**注意：内容停留在 1.x**）：https://rustvnt.com/guide/introduction.html
- 自建服务端：https://rustvnt.com/guide/server.html
- WireGuard 接入说明：https://rustvnt.com/guide/wg.html
- README（**2.0，参数以此为准**）：https://github.com/vnt-dev/vnt/blob/main/README.md
- 服务端仓库：https://github.com/vnt-dev/vnts
- Releases（平台产物）：https://github.com/vnt-dev/vnt/releases

## 已查但官方未说清的事项（不在正文里编，如实列出）

**Tailscale**
- 官方没有一句话写「不支持二层」，只有 OSI 页的间接表述（工作在 network layer、广播/组播行为可能不同）。正文按「无 L2 桥接功能」表述，属归纳。
- 官方从未提供自托管控制面的发行版或部署文档；「官方不提供自建」是针对这一点的结论，口径可变。
- Headscale 的许可证未核实。

**ZeroTier**
- 自建 root server 只有「属 enterprise engagement」的表述，无 DIY 步骤。
- moons（自建 root）的去向：现文档已无 moon 页面（`docs.zerotier.com/moon/` 等路径实测 404），但官方在 `RELEASE-NOTES.md` 1.16.0（2025-08-21）里写明「"Moons" are now considered even more extra *deprecated* and should not be used in new deployments.」，替代方案 network-specific relays 当时仍是 preview / beta。**结论只在 release notes 里，不在文档正文** —— 照文档站查是查不到的。
- 托管面板 ZeroTier Central 能否自建：文档有 "Install Central" 入口，但对应页面抓不到内容，未确认。
- `docs.zerotier.com/rfc4193/`、`/6plane/` 返回 404，IPv6 细节只能引自协议页与术语页。

**NetBird**
- Signal 是否可省、纯直连场景能否完全不部署 relay：官方未明确。
- 自建 external relay 是否属付费/Enterprise 能力：未确认。
- 三处平台清单（安装页 / README / 支持矩阵）互不完全一致，且官网自称支持矩阵 "work in progress"。

**EasyTier**
- 二层能力：官方文档全站未见 TAP / 二层 / 以太网 / ARP 的任何说明，正文写「官方未提及」，不推断有无。
- 文档每页页脚写「基于 Apache License 2.0 发布」，与仓库 `LICENSE`（LGPL-3.0）字面冲突 —— 页脚大概率指文档站本身，但官方未明说，正文只写仓库 LICENSE。
- 默认虚拟网段与默认监听端口在多页之间不一致（`10.126.126.0/24` vs `10.0.0.1` 起；11011/11012/11013 等），正文不给单一默认值。
- DHCP 是否分配 IPv6、子网代理是否支持 IPv6 网段：未确认。

**vnt**
- 「完全无服务器组网」只有 README 特性列表一句，无文档页；`peer_address` 写法、收敛行为均未确认。
- 数据面协议官方从未技术性说明，任何「类 WireGuard」的说法都是推测。
- 文档站与 README 参数断层（1.x vs 2.0），且文档站「自建服务器」页仍写「vnts 不支持 wss」，与 vnts README 直接冲突。
- 点对网参数 `-o / -i` 只见于 1.x 文档站，v2 命令行写法未确认。
- 服务端 vnts 的许可证：仓库无 LICENSE 文件，未确认。
