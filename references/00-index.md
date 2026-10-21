# 分片索引（主题 → 文件）

> **口径原则（先读）**：题目主语带**具体方案/产品名**（CloudFabric、CloudCampus、NCE-\*、USG6000、CE/AirEngine…）的，**以该方案文档为准**，不能用 RFC/通用原理反驳。详见 `17-exam-traps.md` §17.1。
>
> **缺口兜底顺序（硬规则）**：本目录分片 → 合并版/11 章 PDF → **https://github.com/aresbit/modern-network-skill** → 联网搜索（**仅前三层全无命中**，且回答第一行必须写 `> ⚠️ 本次答案来源于网络，请仔细甄别。`）。完整映射见 `SKILL.md` §检索优先级。

## 按章

| 分片文件 | 内容 | 关键词 |
|---|---|---|
| `01-basics-vrp.md` | 第1章 网络概念、分类、拓扑、协议三要素、OSI/TCP-IP 分层与封装、华为 VRP 命令行与文件系统 | 分层, PDU, 封装, 拓扑, VRP, 视图, 用户级别, save |
| `02-access-layer.md` | 第2章 传输介质、线序、编码调制、复用、MAC 地址、CSMA/CD、以太网帧、交换机工作原理 | T568B, 光纤, 曼彻斯特, FDM, MAC, 以太网帧, 泛洪 |
| `03-switching.md` | 第3章 VLAN/802.1Q、Access/Trunk/Hybrid、STP/RSTP/MSTP、链路聚合 | VID, PVID, BPDU, 根桥, Cost, 边缘端口, Eth-Trunk, LACP |
| `04-network-layer.md` | 第4章 IPv4 报文、地址与子网划分、VLSM/CIDR、路由表与优先级、ARP/ICMP、IPv6 | TTL, 协议号, 子网, 掩码, 最长匹配, ARP, tracert, GUA, NDP |
| `05-routing.md` | 第5章 静态路由、RIP、OSPF、OSPFv3、VLAN 间路由 | 缺省路由, 浮动路由, 跳数, 水平分割, LSA, DR, 区域, VLANIF |
| `06-transport.md` | 第6章 TCP 首部/握手/挥手/窗口、UDP | seq, ack, SYN, FIN, TIME_WAIT, 滑动窗口, MSS, 端口分类 |
| `07-application.md` | 第7章 DNS、DHCP、Telnet/SSH、HTTP/HTTPS、邮件、FTP/TFTP、NTP | 递归, 迭代, DORA, 租期, 状态码, TLS, SMTP, PASV, Offset |
| `08-security.md` | 第8章 ACL（编号/通配符/匹配顺序）、AAA/RADIUS | permit, deny, 反掩码, rule, 域, 1812 |
| `09-wan.md` | 第9章 PPP（PAP/CHAP）、PPPoE、NAT 五种形态 | LCP, IPCP, 三次握手, PADI, 静态NAT, NAPT, Easy IP, NAT Server |
| `10-wlan.md` | 第10章 WLAN 术语、802.11 速率表、CSMA/CA、帧类型、CAPWAP、AP 上线、STA 接入、组网与配置 | SSID, BSSID, ESS, Beacon, 5246, DTLS, 隧道转发, VAP |
| `11-ops.md` | 第11章 网络管理五功能、SNMP、SDN/OpenFlow、iMaster NCE、Python 自动化 | MIB, OID, Trap, Inform, 转控分离, 流表, NBI, ZTP |
| `14-datacenter.md` | **增补** 数据中心：CloudFabric 方案、Spine-Leaf/CLOS、VXLAN 封装、BGP EVPN 五类路由、Border Leaf、Multi-POD/Multi-Site、M-LAG 与堆叠 | NVE, VTEP, VNI, BD, VBDIF, 分布式网关, Type 2/3/5, 去堆叠, DCI |
| `15-wlan-deep.md` | **增补** WLAN 进阶：802.11ax/be、信道与频宽、隧道/直接转发、二层三层漫游、802.11k/v/r、WPA2/WPA3、802.1X/Portal、NCE-Campus 云管理 | OFDMA, BSS Coloring, 直接转发, 漫游组, SAE, EAPOL, 1812, Option 43/148 |
| `16-security-firewall.md` | **增补** 防火墙：安全区域与安全策略、状态检测/会话表、NAT 与策略顺序、HRP 双机热备、IPSec/SSL VPN、GRE、IPS/AV/URL、Anti-DDoS | USG6000, Trust/Untrust/DMZ/Local, server-map, Easy IP, NAT Server, 心跳, IKE, 4500, 清洗 |
| `17-exam-traps.md` | **增补** 答题层：方案限定 vs 通用原理、六类陷阱、错题复盘（含第54题）、45 条判断题陷阱库、checklist | 方案限定, 绝对化词, 张冠李戴, 数字错位, 顺序颠倒 |

## 按用途（做题时最常用）

| 我要…… | 看 |
|---|---|
| 查端口号、协议号、定时器、默认优先级、长度、特殊地址、公式 | **`12-numbers.md`** |
| 查华为 VRP 配置命令（VLAN/STP/路由/DHCP/NAT/ACL/PPP/WLAN/SNMP） | **`13-commands.md`** |
| 判断 Access/Trunk/Hybrid 收发 Tag 行为 | `03-switching.md` §3.1 |
| 算子网/掩码/可用主机数/汇总 | `04-network-layer.md` §4.2 + `12-numbers.md` A.8 |
| 判断路由优选（最长匹配→优先级→Cost） | `04-network-layer.md` §4.3 |
| 背 LSA 类型、区域类型、OSPF 邻居状态 | `05-routing.md` §5.3 |
| 判断 ACL 命中哪条规则 | `08-security.md` §8.1 |
| 区分 NAT 四种形态与命令差异 | `09-wan.md` §9.3 |
| 背 802.11 a/b/g/n/ac/ax 速率与频段 | `10-wlan.md` §10.1 |
| 说明 AP 上线 / CAPWAP 隧道建立流程 | `10-wlan.md` §10.3 |
| 区分 SNMP Trap 与 Inform、161/162 | `11-ops.md` §11.2 |
| 区分 OpenFlow 三类消息、流表七要素 | `11-ops.md` §11.3 |
| 答"原理/为什么"类简答题 | 各分片开头的概念表与"易错点" |
| 查 CloudFabric / VXLAN / EVPN / Multi-Site / M-LAG | **`14-datacenter.md`** |
| 查 Wi-Fi 6/7 参数、直接转发、漫游、WPA3、Portal | **`15-wlan-deep.md`** |
| 查防火墙安全策略、NAT 顺序、HRP、IPSec/SSL VPN | **`16-security-firewall.md`** |
| 判断题拿不准 / 题里有方案名 / 复盘错题 | **`17-exam-traps.md`** |

## 高频易混对照（速查）

| 易混点 | 正确答案 | 出处 |
|---|---|---|
| Trunk 发出的帧是否带 Tag | 只有 **VID = PVID** 的那个 VLAN 剥离 Tag，其余带 Tag | 03 §3.1 |
| STP 根桥故障恢复时间 | **≈50 s**（Max Age 20 + 2×Forward Delay 15） | 03 §3.2 |
| RSTP 端口状态数量 | **3 种**（Discarding/Learning/Forwarding） | 03 §3.2 |
| OSPF 协议号 / 封装 | **89**，直接封装在 IP | 05 §5.3 |
| RIP 传输层 | **UDP 520** | 05 §5.2 |
| BGP 传输层 | **TCP 179** | 12 A.1 |
| 路由优先级谁最优 | **直连 0 < OSPF 10 < 静态 60 < RIP 100 < OSPF ASE 150 < BGP 255** | 12 A.4 |
| ACL 未命中任何规则 | 华为末尾隐含 **permit**（rule 4294967294） | 08 §8.1 |
| 动态 NAT vs NAPT | 动态 NAT **不转换端口**；NAPT 转换端口 | 09 §9.3 |
| Easy IP 与 NAPT 差别 | 只在于**使用接口地址而非地址池** | 09 §9.3 |
| SNMP 161/162 | 161 = Agent 收请求；162 = NMS 收 Trap | 11 §11.2 |
| CAPWAP 5246/5247 | 5246 控制，5247 数据 | 10 §10.3 |
| 无线为何不用 CSMA/CD | 难检测冲突 + 帧发送不可中断（隐藏终端） | 10 §10.2 |
| TCP 三次握手第三次能否带数据 | 可以；前两次不能 | 06 §6.1 |
| CloudFabric Multi-Site 跨 DC 承载 | **VXLAN 隧道（BGP EVPN）**，**不含 MPLS 承载**（方案限定） | 14 §14.7 |
| VXLAN VNI 位数 | **24 bit**（约 1600 万），不是 VLAN 的 12 bit/4094 | 14 §14.4 |
| EVPN 五类路由 | Type 1 A-D / **2 MAC-IP** / **3 IMET** / 4 ES / **5 IP Prefix** | 14 §14.5 |
| M-LAG vs 堆叠 | M-LAG **双控制面双活**；iStack/CSS **合并为单控制面** | 14 §14.8 |
| NCE 三个控制器 | Fabric=DC，Campus=园区，WAN=SD-WAN | 14 §14.1 |
| 隧道转发 vs 直接转发 | 直接转发业务**不经 AC**（AC 只做管理） | 15 §15.3 |
| 漫游谁决定切换 | **终端**；AP 只能 802.11k/v 引导 | 15 §15.4 |
| WPA3 握手 | **SAE**，不是 4 次握手 | 15 §15.6 |
| 802.1X vs Portal 层次 | 802.1X **二层**（先认证后拿 IP）；Portal **三层**（先拿 IP） | 15 §15.7 |
| 防火墙默认动作 | **deny**（与路由器 ACL 末尾 permit 相反） | 16 §16.1 |
| 防火墙查策略次数 | 只查**首包**，后续查会话表 | 16 §16.4 |
| NAT Server 后安全策略写哪个地址 | **服务器私网地址**（目的 NAT 在策略前） | 16 §16.5 |
| HRP 主备 | 备用**不转发**业务；会话表同步→切换不断连 | 16 §16.6 |
| AH 能否穿 NAT | **不能**（无端口）；用 ESP + NAT-T(4500) | 16 §16.7 |
