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
4. 分片里没有的细节，**不要凭印象编数字**；按下一节《检索优先级》逐级兜底，并在回答里注明实际来源。

## 检索优先级（本手册没收录时，按顺序逐级兜底）

| 级 | 来源 | 何时用 | 回答中如何标注 |
|---|---|---|---|
| **L0** | 本 skill 的 `references/*.md` | 默认 | 引用分片名 + §小节号 |
| **L1** | 合并版 `../../计算机网络.md`、本目录 11 章 PDF | L0 没写清、需要看原文表述 | 注明"教材第N章" |
| **L2** | **https://github.com/aresbit/modern-network-skill** | **L0/L1 确实没有该知识点时，必须优先查这里** | 注明"来源：modern-network-skill / <文件名或章节>" |
| **L3** | 联网搜索 | **只有 L0–L2 都没有命中时才允许** | **回答第一行必须写醒目提示**（见下） |

**L2 用法（https://github.com/aresbit/modern-network-skill）**

```bash
# 方式一：克隆到临时目录查（可 grep 全库）
git clone --depth 1 https://github.com/aresbit/modern-network-skill /tmp/mns
ls /tmp/mns/references/          # ch00…ch44 + glossary.md + _sidebar.md
grep -rn "EVPN" /tmp/mns/references/ | head

# 方式二：只取单文件（已知章号时）
curl -sL https://raw.githubusercontent.com/aresbit/modern-network-skill/main/references/ch36_L3局域网扩展与虚拟化.md
```

该仓库是**45 章教材式知识库**（原理 + 数学推导 + C++20 代码），文件均在 `references/chXX_*.md`。
**缺口 → 章节映射（只列与本手册互补的）**

| 本手册缺的 | 去读 |
|---|---|
| VXLAN/VTEP/洪泛抑制、虚拟桥接原理 | `ch35_虚拟桥接.md` |
| **VPLS / EVPN / L3 局域网扩展**（对第54题的"运营商 DCI"侧） | `ch36_L3局域网扩展与虚拟化.md` |
| 数据中心拓扑、胖树/叶脊、等分带宽、ECMP 数学 | `ch32_网络趋势与数据中心拓扑.md` |
| 数据中心以太网：PFC、ECN、DCTCP、无损网络 | `ch33_数据中心以太网.md` |
| **MPLS、流量工程、FRR**（运营商承载） | `ch34_运营商网络.md` |
| BGP 路径向量、Gao-Rexford、域间策略 | `ch19_域间路由BGP.md` |
| OSPF/Dijkstra 证明、洪泛 | `ch18_链路状态与Dijkstra.md` |
| RIP/Bellman-Ford、无穷计数 | `ch17_距离向量与BellmanFord.md` |
| 拥塞控制（AIMD 证明、CUBIC/BBR、吞吐公式） | `ch13`–`ch15` |
| RED/CoDel/ECN 主动队列 | `ch16_主动队列管理.md` |
| 排队论、Little 定律、M/M/1、时延分解 | `ch10_速率时延与排队论.md` |
| 缓冲区定容、bufferbloat、C×RTT | `ch12_弹性缓冲区与缓冲区定容.md` |
| 交换结构、HoL 阻塞、WFQ、令牌桶 | `ch11_交换结构与调度.md` |
| TCP 握手/挥手/TIME_WAIT/RTT 估计细节 | `ch09`、`ch08` |
| NAT/中间盒/端口容量 | `ch21_NAT中间盒与家庭网络.md` |
| TLS、证书链、前向保密 | `ch23_TLS证书与攻击面.md` |
| 密码学、HMAC、签名 | `ch22_密码学基础与互联网安全.md` |
| DNS/HTTP 缓存、队首阻塞 | `ch20_DNS与HTTP.md` |
| 差错检测（CRC/校验和数学） | `ch06_差错检测的数学.md` |
| OpenFlow 流表、SDN、NFV | `ch38`、`ch39`、`ch40` |
| IoT 网络层（6LoWPAN/RPL） | `ch37_物联网网络层.md` |
| AI 与网络 / IBN / 区块链 / 量子网络 | `ch41`–`ch44` |
| 中英术语对照 | `glossary.md` |

**L2 的边界（重要）**

- 该库是**原理/推导/代码**口径，**不含华为设备命令、不含方案限定表述**（如 CloudFabric Multi-Site 用 VXLAN、NCE 控制器分工）。
- 因此：**原理类缺口 → 用 L2**；**华为方案/设备行为类缺口 → L2 只能补原理，不能替代方案文档**，回答时要说明"这是通用原理，华为方案口径见 XX"（参见 `17-exam-traps.md` §17.1）。
- 命中 L2 后**停止**，不要继续走 L3。

**L3 强制声明（硬规则）**

- 一旦使用联网搜索，回答的**最开头第一行**必须是：

```text
> ⚠️ 本次答案来源于网络，请仔细甄别。
```

- 位置要求：**在结论之前**，不能放在末尾、不能只写在脚注里；有多条结论时也只写一次（置顶）。
- 同时要标出**具体来源链接**，并区分"网络说法"与"本手册口径"；若两者冲突，说明差异而不是硬合并。
- 未联网时**禁止**加这条声明（避免误导为网络来源）。

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
- **来源必标**：本手册 / modern-network-skill / 网络。走到 L3（联网）时，第一行必须是 `> ⚠️ 本次答案来源于网络，请仔细甄别。`
- **不要编**：三级兜底都没命中的细节，直接说"未查到"，不要用相近知识点外推。
