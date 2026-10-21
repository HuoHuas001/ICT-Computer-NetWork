# 第15章 WLAN 进阶（802.11ax/be、转发模式、漫游、认证、云管理）

> 承接 `10-wlan.md`（术语/802.11 基础/CSMA/CA/CAPWAP/AP 上线）。本章只写**第10章没覆盖**的部分。
> 口径：华为 **CloudCampus / AirEngine / NCE-Campus** 方案 + IEEE 802.11 标准，两者冲突时看题目主语（见 `17-exam-traps.md` §17.1）。

## 15.1 802.11ax（Wi-Fi 6）与 802.11be（Wi-Fi 7）

| 项 | 802.11ac（Wi-Fi 5） | **802.11ax（Wi-Fi 6）** | **802.11be（Wi-Fi 7）** |
|---|---|---|---|
| 频段 | 5 GHz | **2.4 + 5 GHz** | 2.4 + 5 + **6 GHz** |
| 最大频宽 | 160 MHz | 160 MHz | **320 MHz** |
| 最高调制 | 256-QAM | **1024-QAM** | **4096-QAM** |
| 多址 | OFDM（频分） | **OFDMA（子载波分给多用户）** | OFDMA + **Multi-RU** |
| MIMO | DL MU-MIMO | **DL + UL MU-MIMO** | 增强 MU-MIMO（16 流） |
| 理论峰值 | 6.9 Gbit/s | **9.6 Gbit/s** | **46 Gbit/s** |
| 特色技术 | — | **BSS Coloring、TWT、空间复用（SR）** | **MLO 多链路聚合** |

- **OFDMA vs OFDM**：OFDM 一次把整个信道给**一个**用户；OFDMA 把信道切成 **RU（资源单元）**，一次调度**多个**用户 → 小包/多终端场景效率高（这是 Wi-Fi 6 的核心收益，不是峰值速率）。
- **BSS Coloring**：给同频 BSS 打不同"颜色"，同色才退避 → 缓解**同频干扰/OBSS**，提高密集部署吞吐。
- **TWT（Target Wake Time）**：AP 与终端约定唤醒时间 → **省电**（IoT 关键）。
- **UL MU-MIMO**：上行多用户；802.11ac 只有下行。
- **Wi-Fi 6E** = 802.11ax 扩展到 **6 GHz**（不是新 PHY）；**MLO** 是 Wi-Fi 7 的标志性技术（同一终端同时用两条链路）。

**速率速算（考点）**：`速率 = 子载波数 × 每子载波比特数(调制) × 码率 × 空间流数`。
例：Wi-Fi 6 80 MHz / 1024-QAM / 5/6 码率 / 2 流 ≈ **1201 Mbit/s**；160 MHz 单流 1024-QAM ≈ **600.4 Mbit/s**。

## 15.2 信道与频宽

| 频段 | 可用信道（中国） | 20 MHz 非重叠信道 | 说明 |
|---|---|---|---|
| **2.4 GHz** | **1–13** | **1 / 6 / 11**（仅 3 个） | 中心频率 `2412 + 5×(n−1)` MHz；每信道实际占 20 MHz，信道号间隔 5 MHz → 严重重叠 |
| **5 GHz** | 36–64、149–165（5.8 G） | 多（UNII-1/3 非 DFS） | 信道号间隔 **4**；52–64、100–144 属 **DFS**（雷达避让，需 CAC） |
| **6 GHz** | Wi-Fi 6E/7 新增 | 极多（约 59 个 20 MHz） | 中国尚未开放 6 GHz 给 Wi-Fi（截至现有资料） |

- **2.4 GHz 只能"1/6/11"三信道复用**，这是 2.4 G 容量低、干扰重的根因；5 GHz 才能开 40/80/160 MHz。
- **频宽越大，速率越高、抗干扰越差、可用信道越少**：80 MHz 在 5 G 上通常只剩 2–3 个不重叠组合。
- **2.4 G 建议只留 IoT/兼容**，业务走 5 G（**频段引导 Band Steering**）。
- **DFS 信道**：AP 启动需静默检测雷达（CAC，约 60 s），检测到雷达要**主动避让**；这也是"AP 刚上线 5G 不可用"的常见原因。

## 15.3 隧道转发 vs 直接转发（重点对比）

| 维度 | **隧道转发（tunnel）** | **直接转发（direct-forward，本地转发）** |
|---|---|---|
| 业务数据路径 | STA → AP → **CAPWAP 数据隧道（UDP 5247）→ AC** → 上层网络 | STA → AP → **本地接入交换机** → 上层网络 |
| 是否经 AC | **经过**（AC 是数据面瓶颈） | **不经过**（AC 只管管理/控制） |
| 封装 | 原始以太帧封装在 CAPWAP 数据报文里 | AP 给帧打上**业务 VLAN Tag** 直接上送 |
| AC 压力 | 大（需处理全部业务流量） | 小（只处理管理流量） |
| 对交换机要求 | 低（只需 AP 管理 VLAN） | 高（**必须放通业务 VLAN、业务 DHCP 在接入侧**） |
| 安全策略 | 集中在 AC，易统一 | 分散在接入交换机，需逐台配 ACL |
| 漫游 | AC 内天然统一 | 需保证业务 VLAN 在各 AP 接入端口一致 |
| 适用 | 小规模、强管控 | **中大型园区（推荐）** |

**配置要点（华为 AC）**

```text
[Huawei] vap-profile name vap1
[Huawei-vap-prof-vap1] forward-mode direct-forward    # 或 tunnel
[Huawei-vap-prof-vap1] service-vlan vlan-id 101
```

- 直接转发时 AP 上行口常配 **Hybrid**：`management-vlan`（AP 管理）+ `service-vlan`（业务）都打 Tag 上送，接入交换机 Trunk 放通。
- **判断题陷阱**："CAPWAP 数据隧道一定承载业务数据" → 直接转发下**数据隧道可不存在/不承载业务**，只有管理隧道（5246）必在。

## 15.4 漫游

| 类型 | 条件 | STA 地址 | 认证 | 说明 |
|---|---|---|---|---|
| **二层漫游** | 同一 VLAN / 同一子网，AP 间二层可达 | **不变** | 密钥可缓存，**不重新认证** | 最简单，切换快 |
| **三层漫游** | 跨 VLAN / 跨子网 | **不变**（靠家乡代理） | 需 AC 间协同 | 流量经**家乡 AC/家乡代理**绕行（三角路由） |
| **AC 内漫游** | 同一 AC 下不同 AP | 不变 | 不重新认证 | 二层/三层都可能 |
| **AC 间漫游** | 不同 AC | 不变 | 需**漫游组（mobility group）** | 华为需配 `mobility-group` 并把 AC 加入同组 |

**802.11k / v / r（快速漫游三件套）**

| 标准 | 名称 | 作用 |
|---|---|---|
| **802.11k** | Radio Resource Measurement | AP 向终端提供**邻居报告**（Neighbor Report），终端不用全信道扫描 |
| **802.11v** | BSS Transition Management | AP 用 **BTM 请求**主动引导终端切到更优 AP（负载均衡/踢弱信号） |
| **802.11r** | Fast BSS Transition（FT） | 漫游**提前完成密钥协商**，切换时延降到 **< 50 ms**，适合语音 |

- **漫游本质**：终端**自主决定**切换（AP 无法强制，只能引导）；决定因素 = 信号强度/信噪比 + 负载 + 终端自身算法（各厂商差异大）。
- **粘滞终端（Sticky Client）**：终端死守弱信号 AP。对策：**降低 AP 功率**、开启 **802.11v/k**、设置**漫游门限（RSSI 阈值）**、弱信号剔除。
- **漫游不断线的前提**：**同一 SSID + 同一安全模板 + 同一业务 VLAN（二层漫游）**；三层漫游还需漫游组与家乡代理。

```text
# AC 间三层漫游（漫游组）
[Huawei] mobility-group name mg1
[Huawei-mobility-group-mg1] member ip 10.1.1.2      # 对端 AC
[Huawei] wlan
[Huawei-wlan-view] mobility-group name mg1
```

## 15.5 射频调优与容量规划

| 手段 | 作用 | 华为实现 |
|---|---|---|
| **自动信道选择（ACS）** | 避开同频/邻频干扰 | 射频调优（RRM），`calibrate` / 定时调优 |
| **自动功率调整（TPC）** | 覆盖与干扰平衡 | 功率档位/自动功率，避免"高功率大覆盖互相打架" |
| **频段引导（Band Steering）** | 把双频终端推向 5 G | 优先 5G，2.4G 仅兜底 |
| **负载均衡** | 多 AP 间分摊终端 | 基于用户数/流量，配合 802.11v |
| **频谱导航 / 空口扫描** | 识别干扰源 | 频谱分析、干扰检测 |

**规划经验值**

| 指标 | 经验值 |
|---|---|
| 覆盖重叠区 | **10%–15%**（保证漫游，又不过度干扰） |
| 单 AP 并发终端 | 办公 ~**30–50**；高密场景靠 AP 密度 + 频宽调小 |
| 高密场景频宽 | 2.4G 用 **20 MHz**，5G 用 **40 MHz**（别开 80） |
| 语音场景 | RSSI ≥ −65 dBm、SNR ≥ 25 dB，开 802.11r |
| 信道利用率告警 | > 60% 需扩容/调优 |

## 15.6 无线安全

| 协议 | 加密 | 认证/密钥 | 结论 |
|---|---|---|---|
| **WEP** | RC4（静态密钥） | 共享密钥 | **已破解，禁用** |
| WPA | TKIP/RC4 | PSK / 802.1X | 过渡方案，禁用 |
| **WPA2** | **AES-CCMP** | **PSK（4 次握手）** / **802.1X（Enterprise）** | 主流，仍可被离线字典攻击（抓 4 次握手） |
| **WPA3-Personal** | AES-CCMP/GCMP | **SAE（Dragonfly 对等握手）** | **抗离线字典**，前向保密；强制 **PMF** |
| **WPA3-Enterprise** | GCMP-256 | 802.1X + **192-bit 套件** | 高安全行业 |
| **OWE（Enhanced Open）** | 无 PSK，加密开放网络 | 无 | 替代"开放无加密"公共网络 |

- **WPA2-PSK 四次握手**：`PMK = PBKDF2(PSK, SSID, 4096, 256)` → 双方用 ANonce/SNonce/MAC 派生 **PTK** → 第 3 帧下发 **GTK**（组播密钥）→ 第 4 帧确认。**握手报文可被抓取离线爆破**。
- **WPA3-SAE**：用 **Commit/Confirm** 两轮交换代替 PSK，**不传可离线验证的材料**，且每次连接生成新密钥（前向保密）。
- **PMF（802.11w，管理帧保护）**：保护 deauth/disassoc 等管理帧，防**去认证攻击**（伪造踢下线）。WPA3 强制开启。

## 15.7 认证方式（802.1X / Portal / MAC）

**802.1X（二层认证）**

| 角色 | 说明 |
|---|---|
| **Supplicant** | 终端（自带客户端或系统原生） |
| **Authenticator（认证点）** | AP 或 AC（华为可配认证点在 AC 或 AP） |
| **Authentication Server** | **RADIUS**（UDP **1812** 认证 / **1813** 计费） |

- 报文：**EAPOL**（终端↔认证点，二层，EtherType 0x888E）封装 EAP；认证点↔RADIUS 用 **RADIUS**（EAP-Message 属性承载）。
- **受控端口 / 非受控端口**：未认证时只有**非受控端口**（只过 EAPOL）通，认证通过后**受控端口**开放放行业务。
- **EAP 中继 vs EAP 终结**：中继 = AC 只转发 EAP（透传到 RADIUS）；终结 = AC 自己解 EAP，再用 PAP/CHAP 与 RADIUS 交互（华为默认终结方式可配）。
- 触发方式：**EAP 起始（AP 主动）** 或终端发起；**MAC 旁路（MAB）** 用于打印机等无客户端设备。

**Portal（三层认证）**

1. 终端先拿到 IP（DHCP），访问任意 HTTP → 被**重定向**到 Portal 页面。
2. 用户在页面输入账号密码 → Portal 服务器与 **RADIUS/BAS** 校验 → 通过后下发授权。
3. 认证前通常放行 **DNS/DHCP/Portal 服务器地址**（免认证规则）。

- **802.1X 在接入前认证（二层），Portal 在拿到地址后认证（三层）**——这是最常考的对比。
- **混合认证**：同一 SSID 同时支持 802.1X 与 Portal（终端支持 802.1X 优先）。

**MAC 认证**：以 MAC 为账号，适合哑终端；安全性低，仅作准入兜底。

```text
# 华为 AC 认证模板骨架
[Huawei] radius-server template rd1
[Huawei-radius-rd1] radius-server shared-key cipher Huawei@123
[Huawei-radius-rd1] radius-server authentication 10.1.1.10 1812
[Huawei] aaa
[Huawei-aaa] authentication-scheme sch1
[Huawei-aaa-authen-sch1] authentication-mode radius
[Huawei] authentication-profile name p1
[Huawei-authen-profile-p1] authentication-scheme sch1
[Huawei-authen-profile-p1] radius-server rd1
[Huawei-authen-profile-p1] dot1x-access-profile d1
[Huawei-authen-profile-p1] access-domain dft domain dft force
```

## 15.8 NCE-Campus 云管理组网（CloudCampus）

**架构**：云 AP / 云交换机（**无 AC 或 AC 仅本地转发**）+ **iMaster NCE-Campus**（云管理平台）+ 认证组件（RADIUS/Portal，可云化）。

| 部署模式 | 说明 | 场景 |
|---|---|---|
| **公有云（MSP）** | 华为云托管，多租户 | 中小企业、连锁 |
| **私有云** | 客户机房部署 NCE-Campus 集群 | 大型企业/园区 |
| **本地部署（单机/主备）** | 小型本地化 | 单园区 |

**AP 注册上线（发现云平台）的 4 种方式（考点）**

| 方式 | 说明 |
|---|---|
| **DHCP Option 43 / 148** | DHCP 服务器下发云平台地址（148 常用于携带 `agilecontroller=` 形式信息） |
| **DNS** | 解析约定域名得到平台地址 |
| **注册查询中心（RQC）** | AP 出厂预置，向华为注册查询中心查询自己归属的平台 |
| **扫码 / 手动** | 扫设备条码绑定，或本地命令行指定 |

- 云 AP 与平台之间走 **HTTPS/安全通道（NETCONF/HTTP2 over TLS）**，NAT 环境下可反向注册（**免公网固定 IP**）。
- **CloudCampus 的价值**：免 AC 部署、即插即用（ZTP）、多租户、云端统一运维与大数据（体验分析）。
- **易错**：云管理模式下**业务数据仍由 AP 本地转发**，不绕云平台；云平台只做管理与策略下发。

## 15.9 常见考点速答

| 问题 | 答案 |
|---|---|
| Wi-Fi 6 的核心技术收益来自？ | **OFDMA + BSS Coloring + UL MU-MIMO**（提升密集场景效率，不是峰值速率） |
| 2.4 G 为什么不建议开 40 MHz？ | 2.4 G 总共约 83.5 MHz，40 MHz 几乎占满，**只剩 1 个不重叠信道** |
| 漫游时终端 IP 会变吗？ | **不变**（二层天然不变；三层靠家乡代理） |
| 谁能决定终端切换 AP？ | **终端**；AP 只能通过 802.11k/v 引导 |
| 802.11r 解决什么？ | 漫游**重认证时延**（语音 <50 ms） |
| WPA3 与 WPA2 最大差别？ | **SAE 替代 PSK 4 次握手**，抗离线字典；强制 PMF |
| 直接转发需要 AC 吗？ | 业务不需要，**管理仍需要**（CAPWAP 控制隧道 5246） |
| 认证点在哪？ | 华为可配 **AC 或 AP**；RADIUS 在服务器 |
| Portal 与 802.1X 谁先拿到 IP？ | **Portal 先拿 IP**（三层认证）；802.1X 拿 IP 前就认证 |

## 15.10 易错点

- **隧道转发 ≠ 唯一方式**：中大型园区推荐**直接转发**，别答"必须隧道转发"。
- **WPA3 不是"WPA2 + 更长密码"**：是 **SAE** 新握手，机制不同。
- **802.11ax 不是"5 GHz 专属"**：2.4/5 双频；6 GHz 是 6E 的扩展。
- **DFS 信道要 CAC**：AP 上线 5G 慢 60 s 是正常现象，不是故障。
- **漫游不掉线的根因是"同 SSID + 同安全 + 同 VLAN"**，不是"信号强"。
- **云管理不改变数据转发路径**（云 AP 业务本地转发）。
