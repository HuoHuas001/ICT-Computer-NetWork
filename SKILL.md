---
name: huawei-ict-computer-network
description: 计算机网络应试速查手册（华为 HCIA-Datacom 11 章 PDF + CloudFabric/CloudCampus/USG 方案增补）。当需要做网络题目、查协议参数或数字（端口号/协议号/定时器/优先级/帧长）、查华为 VRP 配置命令、判断 VLAN/STP/路由/ACL/NAT 行为、背 OSPF LSA 与 WLAN/CAPWAP 流程、答 CloudFabric/数据中心 VXLAN-EVPN、WLAN 进阶（Wi-Fi 6/漫游/认证）、防火墙（安全策略/HRP/IPSec/内容安全）类题目，或判断题拿不准需要查陷阱库时使用。覆盖分层模型、以太网与交换、IPv4/IPv6、RIP/OSPF、TCP/UDP、DNS/DHCP/HTTP/FTP、ACL/AAA、PPP/PPPoE/NAT、WLAN、SNMP/SDN，以及 14–17 号方案与答题层分片。
---

# 计算机网络速查手册

分片式知识手册，用于做题时**快速定位答案**，不要线性通读。

## 结构

```text
computer-network/
├── SKILL.md            ← 本文件（路由与使用规则）
└── references/         ← 18 个分片，按需只读命中项
    ├── 00-index.md
    ├── 01-basics-vrp.md … 11-ops.md
    ├── 12-numbers.md
    ├── 13-commands.md
    └── 14-datacenter.md … 17-exam-traps.md   ← 方案增补 + 答题层
```

> 14–17 号分片是**方案口径增补**（11 章 PDF 未覆盖）：数据中心 CloudFabric、WLAN 进阶、防火墙、判断题陷阱库。
> 它们不来自 PDF，而是按华为方案文档整理；**遇到与 PDF 冲突时，看题目主语属哪一边**（见下）。

## 使用流程

0. **先看主语**：题干/选项里出现具体方案或产品名（CloudFabric、CloudCampus、NCE-\*、USG6000、CE/AirEngine、Multi-Site/Multi-POD）→ 以该**方案文档**为准，**不要用 RFC 或通用原理反推**。判断题拿不准先看 `references/17-exam-traps.md`。
1. **先读 `references/00-index.md`**：按主题或"我要查……"表格定位到 1 个（最多 2 个）分片。
2. **只读命中的分片**（路径相对本 skill 目录）：
   - 纯数字问题（端口、定时器、优先级、长度、特殊地址、公式）→ `references/12-numbers.md`
   - 配置命令问题 → `references/13-commands.md`
   - 数据中心/VXLAN/EVPN/M-LAG → `references/14-datacenter.md`
   - Wi-Fi 6/7、转发模式、漫游、无线认证 → `references/15-wlan-deep.md`
   - 防火墙安全策略/NAT/HRP/VPN/内容安全 → `references/16-security-firewall.md`
   - 判断题陷阱、错题复盘、官方口径对照 → `references/17-exam-traps.md`
   - 原理/行为判断 → 对应章节分片的概念表与"易错点"
3. 回答时**引用分片文件名 + 小节号**（如 `03-switching.md §3.1`），便于回查原文。
4. 分片里没有的细节，明确说"手册未收录"，不要凭印象编数字；提示用户查对应章节 PDF。

## 分片清单

| 文件 | 内容 |
|---|---|
| `references/00-index.md` | 主题索引、易混对照表 |
| `references/01-basics-vrp.md` | 网络概念/分类/拓扑、分层与封装、VRP 命令行 |
| `references/02-access-layer.md` | 介质/线序/编码/复用、MAC、CSMA/CD、以太网帧、交换机 |
| `references/03-switching.md` | VLAN/802.1Q/接口类型、STP/RSTP/MSTP、链路聚合 |
| `references/04-network-layer.md` | IPv4 报文、子网划分、路由表与优先级、ARP/ICMP、IPv6 |
| `references/05-routing.md` | 静态路由、RIP、OSPF、OSPFv3、VLAN 间路由 |
| `references/06-transport.md` | TCP/UDP |
| `references/07-application.md` | DNS/DHCP/Telnet/SSH/HTTP/HTTPS/邮件/FTP/TFTP/NTP |
| `references/08-security.md` | ACL、AAA/RADIUS |
| `references/09-wan.md` | PPP/PAP/CHAP、PPPoE、NAT |
| `references/10-wlan.md` | WLAN 术语、802.11、CSMA/CA、CAPWAP、AP 上线、组网配置 |
| `references/11-ops.md` | 网络管理、SNMP、SDN/OpenFlow、iMaster NCE、自动化 |
| `references/12-numbers.md` | 附录A 全部数字速查表（端口/协议号/优先级/定时器/长度/地址/公式） |
| `references/13-commands.md` | 附录B VRP 命令速查（15 类场景） |
| `references/14-datacenter.md` | **增补** CloudFabric、Spine-Leaf/CLOS、VXLAN、BGP EVPN、Border Leaf、Multi-POD/Multi-Site、M-LAG vs 堆叠 |
| `references/15-wlan-deep.md` | **增补** 802.11ax/be、信道频宽、隧道/直接转发、漫游与 802.11k/v/r、WPA2/WPA3、802.1X/Portal、NCE-Campus 云管理 |
| `references/16-security-firewall.md` | **增补** 安全区域/安全策略、状态检测、NAT 顺序、HRP、IPSec/SSL VPN、GRE、IPS/AV/URL、Anti-DDoS |
| `references/17-exam-traps.md` | **增补** 方案限定原则、六类陷阱、错题复盘（含第54题）、45 条判断陷阱库、答题 checklist |

> 项目根目录另有一份合并版 `../../计算机网络.md`，适合打印或整篇 Ctrl+F；内容与分片一致，**以分片为准**。

## 检索技巧

- 分片是 Markdown，可直接 grep：`grep -n "LSA" references/05-routing.md`。
- 常见检索词：`PVID`、`BPDU`、`RPC`、`LSA`、`DR`、`DORA`、`PASV`、`反掩码`、`Easy IP`、`5246`、`161`、`tracert`。
- 方案增补检索词：`VXLAN`、`VNI`、`NVE`、`VTEP`、`EVPN`、`Type 2`、`Multi-Site`、`Border Leaf`、`M-LAG`、`iStack`、`OFDMA`、`BSS Coloring`、`直接转发`、`漫游组`、`SAE`、`EAPOL`、`Option 148`、`安全区域`、`会话表`、`server-map`、`HRP`、`IKE`、`4500`、`方案限定`。

## 回答约定

- 结论先行、表格优先：先给答案/取值，再给 1–2 行依据。
- 数字带单位与适用语境（如"Hello 10 s / Dead 40 s（BMA、P2P）"）。
- 配置类给**完整可粘贴命令块**，并标注视图（`<Huawei>` / `[Huawei]` / `[Huawei-GE0/0/0]`）。
- **判断题先定口径**：主语带方案名按方案文档答；绝对化表述先怀疑；拿不准时明确说明依据是方案文档还是通用标准。
