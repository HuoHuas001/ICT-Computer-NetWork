# 第10章 WLAN

## 10.1 基本概念与组网方式

- **WLAN**：以无线电波（2.4 GHz / 5 GHz 等）为传输介质的局域网。无线网络按范围分 **WPAN / WLAN / WMAN / WWAN**。
- 优点：**移动性、布线少、成本低、易扩展**。
- **IEEE 802.11** 是标准（聚焦 TCP/IP 模型**下两层**）；**Wi-Fi** 是 Wi-Fi 联盟的**兼容性认证商标**。

| 标准 | Wi-Fi 世代 | 频段 | 理论最高速率 | 关键技术 |
|---|---|---|---|---|
| 802.11（原始） | — | 2.4 GHz | **2 Mbit/s** | FHSS / DSSS |
| **802.11b** | Wi-Fi 1 | 2.4 GHz | **11 Mbit/s**（1/2/5.5/11） | HR/DSSS，11 个 22 MHz 重叠信道 |
| **802.11a** | Wi-Fi 2 | **5 GHz** | **54 Mbit/s** | OFDM，52 个子载波 |
| **802.11g** | Wi-Fi 3 | 2.4 GHz | **54 Mbit/s**（净约 24.7） | ERP-OFDM，14 个频段 |
| **802.11n** | Wi-Fi 4 | 2.4 + 5 GHz | **600 Mbit/s**（常见 300） | **MIMO + OFDM**，20/40 MHz |
| **802.11ac** | Wi-Fi 5 | **5 GHz** | **6.9 Gbit/s** | MU-MIMO、OFDM，20/40/80/160 MHz |
| **802.11ax** | Wi-Fi 6 | 2.4 + 5 GHz | **9.6 Gbit/s** | **OFDMA + MU-MIMO**、1024-QAM |

**组网方式**

| 方式 | 说明 | 场景 |
|---|---|---|
| **Ad-Hoc（IBSS）** | 站点直连、无中心、多跳自组织；需彼此能"看到" | 临时组网 |
| **胖 AP（FAT AP）** | 独立工作，自带 DHCP/NAT/安全/网管，需单独配置 | 家庭 / SOHO |
| **瘦 AP（FIT AP）+ AC** | AP 只做射频与加密，**由 AC 集中管理**（配置、漫游、安全、射频调优） | 中大型企业/园区 |
| 云管理 AP | 由云管理平台统一下发配置 | 中小企业 |

- 胖瘦对比：胖 AP 功能全但**零配置能力差、大规模运维难**；瘦 AP **本身零配置、易大规模组网**，但依赖 AC。

**关键术语（必考）**

| 术语 | 含义 |
|---|---|
| **STA** | 无线终端（支持 802.11 的设备） |
| **AP** | 接入点，连接无线与有线，半双工，覆盖半径可达百米 |
| **BSS** | 基本服务集 = 一个 AP 的覆盖范围；分 IBSS（自组）与 Infrastructure BSS |
| **BSSID** | **AP 的 MAC 地址**，标识一个 BSS（同一 AP 内 BSSID 与 SSID 可不同） |
| **SSID** | 服务集标识符，**无线网络名称**，最长 32 字符 |
| **ESS** | 扩展服务集 = 多个 BSS 经 **DS** 串联，**同一 ESS 内所有 AP 使用相同 SSID**，可漫游 |
| **DS** | 分布式系统，AP 间转发帧的骨干网，负责追踪 STA 位置 |
| ESSID | ESS 的名称（= SSID） |

- 相邻 AP 使用**互不重叠信道**，覆盖重叠区约 **10%–15%**（保证漫游不掉线）。

## 10.2 802.11 物理层与 MAC 层

**物理层技术**：FHSS（跳频）、DSSS（直接序列扩频）、HR/DSSS、**OFDM**（正交频分复用，把信道划分为子信道并行传输）、ERP、**MIMO-OFDM**、**OFDMA**（频域划分子载波给多用户）。
**调制**：BPSK → QPSK → 16-QAM → 64-QAM → 256-QAM → 1024-QAM（阶数越高，速率越高、抗干扰越差）。

**CSMA/CA（无线必考）**

- **CS**：载波侦听（物理载波侦听 + **虚拟载波侦听**，靠帧中的 Duration 字段/NAV）。
- **MA**：多址访问。
- **CA**：冲突**避免**（不是检测）。
- 无线**不能用 CSMA/CD** 的原因：① 信号衰减大，难以检测冲突；② 发送以帧为单位，检测到冲突也无法中途停止（隐藏终端问题）。
- 机制：**先听后发 → 随机退避 → 发完整帧 → 等 ACK 确认**；可选 **RTS/CTS** 解决隐藏终端；并使用**停止等待协议**。
- 帧间间隔：**SIFS < PIFS < DIFS**（优先级相反）；ACK、CTS 用 SIFS。

**帧分类**

| 类别 | 典型帧 | 作用 |
|---|---|---|
| **管理帧** | **Beacon**（信标，AP 周期发送，默认约 100 ms）、**Probe Request/Response**（探测）、**Authentication / Deauthentication**、**Association Request/Response**、**Reassociation**（漫游重关联）、Disassociation | 加入/退出网络、建立链路 |
| **控制帧** | **RTS / CTS**、**ACK**、PS-Poll（省电轮询）、Block Ack | 信道取得与确认 |
| **数据帧** | Data | 承载上层数据 |

**帧格式**：Frame Control（含 Type/Subtype、To DS/From DS、More Flag、Protected Frame）→ Duration/ID → **Address 1/2/3/4** → Sequence Control（分片重组、丢弃重复帧）→ Frame Body → **FCS（32 位 CRC）**。

| 场景 | Address1（RA） | Address2（TA） | Address3 |
|---|---|---|---|
| STA → AP | BSSID（AP） | STA 源 | 目的 MAC |
| AP → STA | STA 目的 | BSSID（AP） | 源 MAC |
| AP ↔ AP（WDS） | — | — | 用 Address4 |

## 10.3 CAPWAP 与 AP 上线

- **CAPWAP**（无线接入点控制和配置协议）：基于 **UDP** 的应用层协议，用于 **AP 自动发现 AC、状态维护、配置下发、固件升级、数据隧道封装**。

| 通道 | 端口 | 内容 |
|---|---|---|
| **控制隧道** | **UDP 5246** | 管理报文（AP 与 AC 之间的控制/管理消息） |
| **数据隧道** | **UDP 5247** | 业务数据（802.11 帧封装后转发） |

**隧道建立 8 个阶段**

`DHCP（AP 获取 IP）→ Discovery（发现 AC）→ DTLS（可选加密握手）→ Join（建立控制通道）→ Image Data（版本升级）→ Configuration（配置下发）→ Data Check（Change State Event）→ Run（正常上线）`

- **AP 获取 IP**：静态配置（少用）或 **DHCP**（常用，可顺带下发 **Option 43** 告知 AC 地址）。
- **AC 发现方式**：静态指定、**广播/组播**发现、**DHCP Option 43**、DNS、注册查询中心。
- **AP 接入控制**：判断黑名单 → 预配置列表（MAC/SN）→ 是否需要认证 → 白名单 → **手工确认**（输入 MAC/SN）。
- **心跳**：AP 定期发送 **Echo/Keepalive（默认 25 s）**，AC 回应后 AP 进入 **normal** 状态。
- 上线判断：`display ap all` 中 State = **nor** 表示正常。

## 10.4 STA 接入过程（三阶段）

| 阶段 | 内容 |
|---|---|
| **① 扫描** | **主动扫描**：STA 发 **Probe Request**（带 SSID = 定向；不带 = 全信道广播），AP 回 Probe Response<br>**被动扫描**：STA 只监听 **Beacon** 帧（省电、发现慢）<br>实际中两者结合，STA 不断切换信道监听 |
| **② 认证（链路认证）** | **开放系统认证**（默认，两次握手，实际不校验）<br>**共享密钥认证**（WEP，四次握手，需预共享密钥） |
| **③ 关联** | 协商速率、信道等链路服务；AP 回 Association Response；**漫游时用 Reassociation** |

- 之后才是真正的用户接入认证（**802.1X / Portal / MAC 认证**）与 DHCP 获取地址。

## 10.5 组网与转发模式

| 维度 | 类型 | 特点 |
|---|---|---|
| AC 与 AP 位置 | **二层组网** | AC 与 AP 同广播域，AP 可广播直接发现 AC，配置简单 |
| | **三层组网** | 跨网段，需 **Option 43 / 静态指定 AC**，中间网络要保证可达 |
| AC 连接位置 | **直连式** | AC 串在业务路径上（AP 上联经 AC） |
| | **旁挂式** | AC 挂在旁路，只做管理，业务由汇聚/核心转发 |
| 数据转发 | **直接转发（本地转发）** | 业务数据在 AP 本地转成 802.3 直接入网，**不经过 AC**；AC 负担小，万兆园区推荐 |
| | **隧道转发（集中转发）** | 业务数据封装进 CAPWAP 数据隧道到 AC，**经 AC 转发**；安全可控、便于集中管理 |

- **管理 VLAN**：承载 AC-AP 的 CAPWAP 管理报文；**业务 VLAN**：承载 STA 上网数据。
- 地址规划：AC 源地址（管理 VLAN，静态配置）、AP 地址（管理 VLAN，DHCP）、STA 地址（业务 VLAN，DHCP）。

## 10.6 配置命令

```text
# 1. 基础与 AC 源地址
[AC] vlan batch 100 101
[AC] interface Vlanif 100
[AC-Vlanif100] ip address 10.100.100.1 24
[AC] capwap source interface Vlanif100        # 或 capwap source ip-address x.x.x.x
[AC] wlan

# 2. 域管理模板（国家码）
[AC-wlan-view] regulatory-domain-profile name domain
[AC-wlan-regulate-domain-profile-domain] country-code cn
[AC-wlan-regulate-domain-profile-domain] quit
[AC-wlan-view] ap-group name ap-group1
[AC-wlan-ap-group-ap-group1] regulatory-domain-profile domain

# 3. 离线导入 AP（或用 ap auth-mode 自动认证）
[AC-wlan-view] ap-id 0 ap-mac 00e0-fc0a-1234
[AC-wlan-ap-0] ap-name ap1
[AC-wlan-ap-0] ap-group ap-group1

# 4. SSID / 安全 / VAP 模板
[AC-wlan-view] ssid-profile name employee
[AC-wlan-ssid-prof-employee] ssid employee
[AC-wlan-view] security-profile name employee
[AC-wlan-sec-prof-employee] security wpa-wpa2 psk pass-phrase a1234567 aes
[AC-wlan-view] vap-profile name employee
[AC-wlan-vap-prof-employee] forward-mode { tunnel | direct-forward }
[AC-wlan-vap-prof-employee] service-vlan vlan-id 101
[AC-wlan-vap-prof-employee] ssid-profile employee
[AC-wlan-vap-prof-employee] security-profile employee
[AC-wlan-ap-group-ap-group1] vap-profile employee wlan 1 radio all

# 5. 验证
[AC] display ap all
[AC] display vap all
[AC] display vap ssid employee
[AC] display station all
[AC] display ap-group name ap-group1
```

**易错点**
- SSID 最长 **32 字符**，不能含 `?`、`/`、空格，不能出现双引号。
- 直连式二层组网常用**隧道转发**；旁挂式常用**直接转发**。
- 802.11 的 ACK 是**必需**的（数据帧必须逐帧确认），这是"停止等待"的体现。
- CAPWAP 两个端口易混：**5246 控制、5247 数据**。
