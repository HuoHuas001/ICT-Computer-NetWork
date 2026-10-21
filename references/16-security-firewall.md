# 第16章 防火墙与安全进阶（USG6000 / NAT / HRP / VPN / 内容安全）

> 承接 `08-security.md`（ACL、AAA/RADIUS）。本章是**华为 USG6000 系列**口径。
> 命令为 V5 语法骨架（`security-policy` 为 V5R1 之后的新策略模型），**版本不同命令略有差异**，做题按教材版本。

## 16.1 防火墙 vs 路由器（第一考点）

| 维度 | 路由器（ACL 包过滤） | 防火墙（USG） |
|---|---|---|
| 默认动作 | ACL **末尾隐含 permit**（rule 4294967294） | **默认拒绝（default deny）**，未命中任何策略即丢弃 |
| 状态 | **无状态**，逐包查 ACL | **状态检测**，首包查策略，后续包查**会话表** |
| 匹配依据 | 五元组（ACL 规则） | **区域 + 地址 + 服务/应用 + 用户 + 时间段** |
| NAT | 与 ACL 独立 | **NAT 策略与安全策略分离**，且顺序有讲究（§16.5） |
| 部署位置 | 任意 | **网络边界、区域之间**（串接） |

- **"防火墙默认允许"是错的**；**"ACL 未命中默认允许"在华为路由器上是对的**——两者不要混。
- **状态检测（ASPF/SPF）**：只检查**首包**是否被策略允许，之后同一会话的回程包/后续包直接放行（会话表项），因此**不需要为回程流量写反向策略**。

## 16.2 安全区域

| 区域 | 默认优先级 | 放什么 |
|---|---|---|
| **Local** | **100** | 防火墙**自身**（管理面、去往/来自设备的流量） |
| **Trust** | **85** | 内网/高信任（员工区） |
| **DMZ** | **50** | 对外提供服务的服务器区 |
| **Untrust** | **5** | Internet/低信任 |
| 自定义 | 1–100 | 按需（如 Server、Guest） |

- **接口必须加入区域**才能参与策略匹配；一个接口只能属于**一个**区域。
- 优先级用途：老版本包过滤按"高→低"方向判断；**V5 新策略模型下以 `source-zone/destination-zone` 显式匹配**，优先级不再是放行依据（但仍是必背数字）。
- 同一区域内部流量默认**不经过**安全策略（同区域互通由二层/三层直接转发），需要管控要单独配（`same-zone` 或划不同区域）。

```text
[Huawei] firewall zone trust
[Huawei-zone-trust] add interface GigabitEthernet1/0/1
[Huawei] interface GigabitEthernet1/0/1
[Huawei-GE1/0/1] service-manage ping permit      # 允许该接口被 ping（Local 域）
```

## 16.3 安全策略

**匹配条件**：源/目的**安全区域**、源/目的**地址（地址组）**、**服务/应用**（端口或应用识别）、**用户**（认证后）、**时间段**。
**动作**：`permit` / `deny`（另有 `no-match` 用于策略内继续）。

- **从上到下匹配，命中第一条即生效**（不像 ACL 有隐含 permit，最后有**缺省动作**）。
- **缺省动作默认 deny**；改缺省动作要谨慎（`security-policy default action permit` 极少用）。
- 策略可**移动/插入**（`rule move`），编号不决定顺序，**位置决定顺序**。

```text
[Huawei] security-policy
[Huawei-policy-security] rule name trust_to_untrust
[Huawei-policy-security-rule-trust_to_untrust] source-zone trust
[Huawei-policy-security-rule-trust_to_untrust] destination-zone untrust
[Huawei-policy-security-rule-trust_to_untrust] source-address 10.1.1.0 mask 255.255.255.0
[Huawei-policy-security-rule-trust_to_untrust] service http https dns
[Huawei-policy-security-rule-trust_to_untrust] action permit
[Huawei] display security-policy rule all
```

## 16.4 会话表与状态检测

| 项 | 说明 |
|---|---|
| 会话表五元组 | 源 IP/端口、目的 IP/端口、协议（+ 区域、应用、NAT 后信息） |
| 建表时机 | **首包通过安全策略后建会话**，后续包只查会话（不再查策略） |
| 老化 | TCP/UDP/ICMP 各有老化时间（**TCP 通常最长，UDP/ICMP 短**；具体值按版本，教材口径为准） |
| **Server-map** | NAT Server / ASPF 生成的**临时放行表**，允许"先由内部发起"或"多通道协议"通过 |
| 多通道协议 | FTP（主动模式 20 端口）、SIP/H.323 等需 **ASPF** 识别并动态开孔 |
| 排障命令 | `display firewall session table`、`display firewall server-map`、`reset firewall session table` |

- **状态检测的核心价值**：回程流量自动放行；**非首包不查策略**，性能高。
- **陷阱**："防火墙每个包都要查安全策略" → **错**，只查首包。

## 16.5 NAT（防火墙侧）

| 类型 | 命令 | 说明 |
|---|---|---|
| **NAT No-PAT** | `action source-nat address-group grp` | 一对一，**不转换端口**，可被外部主动访问 |
| **NAPT** | 地址池 + `pat` | 多对一，**转换端口** |
| **Easy IP** | `action source-nat easy-ip` | 用**出接口地址**做 NAPT |
| **NAT Server** | `nat server ... global <公网> <端口> inside <私网> <端口>` | **目的 NAT**，发布内部服务器 |
| **双向 NAT** | 源 NAT + 目的 NAT 同时 | 服务器看到的是转换后的源地址 |
| **NAT ALG** | `detect ftp` 等 | 处理携带 IP/端口的应用层协议 |
| **Hairpin（内网访问公网地址）** | 需 `nat server` + 内部源 NAT + 策略 | 内网用户访问自家公网映射 |

```text
[Huawei] nat address-group grp1
[Huawei-address-group-grp1] mode pat
[Huawei-address-group-grp1] section 0 202.1.1.10 202.1.1.20
[Huawei] nat-policy
[Huawei-policy-nat] rule name snat1
[Huawei-policy-nat-rule-snat1] source-zone trust
[Huawei-policy-nat-rule-snat1] destination-zone untrust
[Huawei-policy-nat-rule-snat1] source-address 10.1.1.0 mask 255.255.255.0
[Huawei-policy-nat-rule-snat1] action source-nat address-group grp1
# NAT Server
[Huawei] nat server web1 protocol tcp global 202.1.1.100 8080 inside 10.1.1.10 80
```

**★ 顺序（最易错）**

```text
入方向：目的 NAT（NAT Server） → 路由查表 → 安全策略 → 源 NAT → 出接口
```

- 因此 **NAT Server 之后，安全策略里的目的地址要写服务器的私网地址**（不是公网地址）。
- 因此 **源 NAT 之后，安全策略里的源地址要写私网原地址**（不是转换后的公网地址）。
- 判断题："配置 NAT Server 后，安全策略中应使用公网地址" → **错**。
- **NAT 与 ACL 的关系**：华为路由器上 NAT 的匹配用 ACL；**防火墙上用 nat-policy**，且 NAT 不做"策略放行"（放行仍靠 security-policy）。

## 16.6 HRP 双机热备

| 模式 | 说明 |
|---|---|
| **主备（active/standby）** | 一台转发，一台待命；故障时切换，**资源利用率 50%** |
| **负载分担（active/active）** | 两台都转发，**两个 VRRP 组互为主备**（各带一半流量） |

**必备条件与机制**

| 项 | 说明 |
|---|---|
| 心跳口（HRP 心跳） | **专用直连链路**（建议独立接口/独立 VLAN），不走业务口 |
| VRRP | 对上下行提供**虚拟网关**，实现流量牵引与切换 |
| **HRP 同步内容** | **命令/配置**、**会话表（状态）**、时间；`hrp auto-sync` |
| 切换 | 主设备故障 → VRRP 切换（秒级）+ 会话表已同步 → **已有连接不中断** |
| 抢占 | `hrp preempt`，需同优先级/延时避免震荡 |
| 一致性要求 | **同型号、同版本、同接口编号、同策略**（否则无法建立 HRP） |

```text
[Huawei] hrp enable
[Huawei] hrp interface GigabitEthernet1/0/3 remote 10.1.2.2
[Huawei] hrp preempt delay 60
[Huawei] hrp auto-sync
[Huawei] hrp mirror session enable       # 会话表镜像同步
<Huawei> display hrp state                # Active / Standby
```

- **双机热备 ≠ 负载均衡**：主备模式另一台**不转发业务**。
- **没有会话同步**时切换会**断开已有连接**（这是"为什么要 HRP"的标准答案）。

## 16.7 IPSec VPN

**两阶段**

| 阶段 | 内容 | 报文 |
|---|---|---|
| **阶段 1（IKE SA / ISAKMP SA）** | 协商加密/认证算法、DH 交换、建立管理隧道 | IKEv1 主模式（6 包）/野蛮模式（3 包）；IKEv2（4 包） |
| **阶段 2（IPSec SA）** | 协商 ESP/AH 参数、生成数据密钥 | 快速模式（IKEv1 3 包） |

| 项 | 要点 |
|---|---|
| 协议 | **AH**（只认证，不加密，**不能穿越 NAT**）、**ESP**（加密+认证，可 NAT-T） |
| 模式 | **传输模式**（只封 IP 载荷，主机间）、**隧道模式**（整包封装，网关间，**VPN 常用**） |
| 端口 | IKE **UDP 500**；**NAT-T：UDP 4500** |
| 算法 | 加密 AES/3DES/DES；认证 SHA-2/SHA-1/MD5；DH group 1/2/5/14… |
| 感兴趣流 | 由 **ACL** 定义（源/目的网段），两侧**互为镜像** |
| 触发 | 有匹配感兴趣流的流量才触发协商（也可 `sa trigger` 手工触发） |
| 抗重放 | 序列号 + 窗口（ESP 提供） |

```text
[Huawei] acl 3000
[Huawei-acl-adv-3000] rule permit ip source 10.1.1.0 0.0.0.255 destination 10.2.2.0 0.0.0.255
[Huawei] ike proposal 10
[Huawei-ike-proposal-10] encryption-algorithm aes-256
[Huawei-ike-proposal-10] authentication-algorithm sha2-256
[Huawei-ike-proposal-10] dh group14
[Huawei] ike peer peer1 v2
[Huawei-ike-peer-peer1] ike-proposal 10
[Huawei-ike-peer-peer1] remote-address 202.1.1.2
[Huawei-ike-peer-peer1] pre-shared-key cipher Huawei@123
[Huawei] ipsec proposal p1
[Huawei-ipsec-proposal-p1] esp encryption-algorithm aes-256
[Huawei-ipsec-proposal-p1] esp authentication-algorithm sha2-256
[Huawei] ipsec policy pol1 10 isakmp
[Huawei-ipsec-policy-isakmp-pol1-10] security acl 3000
[Huawei-ipsec-policy-isakmp-pol1-10] ike-peer peer1
[Huawei-ipsec-policy-isakmp-pol1-10] proposal p1
[Huawei] interface GigabitEthernet1/0/0
[Huawei-GE1/0/0] ipsec policy pol1
<Huawei> display ipsec sa
```

- **NAT 与 IPSec 共存**：IPSec 流量必须**在 NAT 之前被识别并旁路**（NAT 策略里排除，或靠 NAT-T 封装到 4500）。
- **IKEv1 vs IKEv2**：IKEv2 报文更少、支持 MOBIKE（移动/多地址）、更适移动端；华为 `ike peer ... v2`。

## 16.8 SSL VPN

| 项 | 说明 |
|---|---|
| 基础 | **TLS（HTTPS，443）**，无需预配隧道，**天然穿 NAT/代理** |
| 接入方式 | **Web 代理**（改写的网页访问）、**端口转发**（TCP 端口映射）、**网络扩展（Network Extension）**（虚拟网卡，全 IP 接入） |
| 关键概念 | **虚拟网关（virtual-gateway）**、认证域、用户/用户组、角色授权 |
| 对比 IPSec | SSL VPN **客户端零配置、按用户授权、适合出差/移动办公**；IPSec **站点间固定隧道、性能高** |

```text
[Huawei] v-gateway vg1 interface GigabitEthernet1/0/0 private
[Huawei] v-gateway vg1
[Huawei-v-gateway-vg1] security ipsec
```

## 16.9 GRE 与 L2TP

| 协议 | 封装 | 端口/协议号 | 特点 |
|---|---|---|---|
| **GRE** | IP 协议号 **47** 外层封装 | 无端口 | 可承载**组播/广播/非 IP**（路由协议、VXLAN 早期承载）；**不加密**，常与 IPSec 组合（GRE over IPSec） |
| **L2TP** | UDP **1701** | — | 二层隧道，常与 IPSec 组合（L2TP over IPSec）做远程接入 |
| **IPSec** | ESP(50)/AH(51) | UDP 500/4500 | 加密 + 认证，站点间/远程接入 |

## 16.10 内容安全

| 功能 | 检测对象 | 关键点 |
|---|---|---|
| **IPS（入侵防御）** | 攻击特征/异常行为 | **签名检测 + 异常检测**；可阻断（inline）或仅告警；**无法保证发现未知攻击** |
| **AV（防病毒）** | 文件/协议载荷 | **流扫描**（边传边扫）与**代理扫描**（先缓存再扫）；可做文件还原 |
| **URL 过滤** | HTTP/HTTPS 访问的 URL | 依赖**URL 分类库（需 License 与升级）**；HTTPS 需**解密**才能看 URL |
| **文件过滤** | 文件类型/大小/方向 | 按扩展名阻断 |
| **应用识别（SA）** | 应用层协议 | 基于特征识别 P2P/IM/游戏，用于策略与 QoS |
| **反垃圾邮件** | SMTP/POP3/IMAP | 特征 + 黑白名单 |
| **DNS 过滤/Tube** | 域名 | 恶意域名拦截 |

- **IDS 旁路 vs IPS 串接**：IPS 必须**在线串接**才能阻断；旁路只能检测告警。
- **性能**：内容安全是 CPU 密集型，开启后吞吐下降，需选型（`display engine` / 硬件加速）。

## 16.11 Anti-DDoS

| 攻击 | 特征 | 防御手段 |
|---|---|---|
| **SYN Flood** | 大量 SYN 无 ACK，占满半连接表 | **SYN Cookie / 源认证 / 首包丢弃 / TCP 代理** |
| **UDP Flood** | 大流量 UDP 打满带宽 | 限速、指纹过滤、特征清洗 |
| **ICMP Flood** | 大量 ping | 限速/丢弃 |
| **CC 攻击** | 高频合法 HTTP 请求耗尽服务 | 请求频率限制、人机校验、缓存 |
| **畸形报文/分片攻击** | 非法标志位、分片重叠 | 报文合法性检查、分片重组限制 |
| **反射放大**（DNS/NTP/memcached） | 小请求大响应 | 源端口过滤、上游清洗 |

- 华为：**AntiDDoS8000**（专业清洗设备）+ 防火墙本地 Anti-DDoS；云侧 **Anti-DDoS 流量清洗**。
- **清洗中心原理**：流量牵引（BGP/路由）→ 清洗 → 回注。

## 16.12 易错点

- **防火墙默认拒绝，路由器 ACL 默认允许**——最经典的互换陷阱。
- **只查首包**：不是每个包都过策略（状态检测）。
- **NAT Server 后策略写私网地址**；**源 NAT 后策略写私网源地址**。
- **AH 不能穿 NAT**（无端口），ESP + NAT-T 才行。
- **HRP 主备模式下备用设备不转发业务**；负载分担要两个 VRRP 组。
- **IPS 不能 100% 拦住未知攻击**；**URL 过滤对 HTTPS 需解密**。
- **GRE 不加密**（"GRE 是加密隧道"→ 错）。
- **同区域流量默认不受策略管控**（同区域直通）。
