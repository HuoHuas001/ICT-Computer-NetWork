# 第14章 数据中心网络（CloudFabric / VXLAN / EVPN）

> **口径声明**：本章是**华为 CloudFabric 解决方案**口径，不是 RFC 通用口径。
> 凡题目主语带 `CloudFabric`、`NCE-Fabric`、`CE 系列`、`Multi-POD`、`Multi-Site` 的，**以方案文档为准**，不要用通用网络原理反推（见 `17-exam-traps.md` §17.1）。

## 14.1 CloudFabric 方案总览

**一句话**：CloudFabric = 数据中心网络的 SDN 方案，**Underlay 用 Spine-Leaf 全三层 CLOS，Overlay 用 VXLAN，控制面用 BGP EVPN，管理面用 NCE-Fabric**。

| 平面 | 技术 | 作用 |
|---|---|---|
| 管理面 | **iMaster NCE-Fabric**（原 Agile Controller-DCN） | Fabric 自动化部署、租户/VPC 编排、VXLAN 隧道自动建立、健康巡检 |
| 控制面 | **BGP EVPN**（MP-BGP，AFI 25 / SAFI 70） | 传递 MAC/IP 可达性，替代 VXLAN 泛洪学习 |
| 数据面 | **VXLAN**（UDP 4789） | 大二层 Overlay，VNI 隔离租户 |
| 物理面（Underlay） | **Spine-Leaf + OSPF/IS-IS/BGP** | 只跑三层路由，无 STP、无环路、ECMP |

**方案特征（判断题常考）**

- **全以太（Ethernet Everywhere）**：DC 内不做 FCoE/InfiniBand 专用网，存储、计算、业务统一 IP Fabric。
- **去堆叠 / 双活**：接入侧用 **M-LAG** 替代堆叠，避免堆叠控制面单点。
- **CLOS 无阻塞**：任意两台 Leaf 之间固定 **2 跳**（Leaf→Spine→Leaf），路径等长，天然 ECMP。
- **自动化**：设备**零配置上线（ZTP）**，配置由控制器下发，人工不登录设备。

**方案组件速记**

| 组件 | 定位 | 易混 |
|---|---|---|
| **NCE-Fabric** | 管**数据中心**网络 | ≠ NCE-Campus（园区） |
| **NCE-Campus** | 管**园区**网络（CloudCampus） | ≠ NCE-Fabric（DC） |
| **NCE-WAN** | 管 **SD-WAN** 广域 | — |
| CE 系列交换机 | CloudEngine，DC 专用（CE12800/CE8850/CE6865/CE6881…） | ≠ S 系列（园区接入） |

## 14.2 Spine-Leaf（CLOS）架构

```text
        [Spine1]      [Spine2]        ← 只做转发，不接服务器
         /  |  \      /  |  \
   [Leaf1] [Leaf2] [Leaf3] [Leaf4]    ← 接服务器 / 接 Border
      |       |       |       |
   Server  Server  Server  Server
```

| 规则 | 说明 |
|---|---|
| Spine **不接服务器** | 只与 Leaf 互联；Spine 数量决定东西向带宽 |
| Leaf 接服务器 + 上联 Spine | 每台 Leaf 与**所有 Spine** 互联（full-mesh） |
| 任意 Leaf 间跳数固定 | **2 跳**，因此时延一致、可 ECMP |
| Underlay 只用三层 | **不跑 STP**，靠路由收敛；链路全部参与转发 |
| 收敛比（oversubscription） | 接入带宽 : 上联带宽，常见 **3:1 / 4:1**，越接近 1:1 越无阻塞 |
| 扩展方式 | **横向加 Leaf**（scale-out），不改动已有连线 |

**为什么不用传统三层架构（核心-汇聚-接入）**：汇聚层存在 STP 阻塞链路 + 生成树收敛慢 + 南北向瓶颈；Spine-Leaf 把"汇聚"打平成"全互联 Spine"，所有链路 active。

## 14.3 设备角色

| 角色 | 位置 | 职责 | 关键点 |
|---|---|---|---|
| **Spine** | 骨干 | 高速转发、作为 **BGP EVPN 的 RR** | 通常不启 VXLAN 网关功能，纯 Underlay + RR |
| **Leaf** | 接入 | 接服务器、做 **VTEP**（NVE 接口） | 常兼**分布式网关**（VBDIF） |
| **Border Leaf** | 边界 | 连接出口路由器 / 防火墙 / 外部网络 / 其他 POD / 其他 Site | 做 **VXLAN↔VLAN 转换**、跨域路由、外部路由引入 |
| **Edge Leaf / Service Leaf** | 服务区 | 接防火墙、负载均衡等 L4-7 设备 | 服务链引流 |
| **出口路由器 / DCI 网关** | 最外侧 | 连接 Internet、专线、其他 DC | 不参与 Fabric 内部 EVPN |

- **Border Leaf 是 Fabric 的唯一出口**：所有南北向流量、跨 POD、跨 Site 流量都经过 Border Leaf，因此它是**故障域边界**也是**策略集中点**。
- 服务器双归：一台服务器两条线上联**两台 Leaf**（M-LAG 或 bond），不用堆叠。

## 14.4 VXLAN 数据面

**封装（外层到内层）**：`外层以太头 | 外层 IP 头 | UDP 头 | VXLAN 头(8B) | 原始以太帧`

| 字段 | 取值 | 考点 |
|---|---|---|
| 外层 UDP 目的端口 | **4789**（IANA 标准，华为默认） | 目的端口 4789，源端口由内层流哈希生成（供 ECMP） |
| VXLAN 头长度 | **8 字节** | 标志(8b)+保留(24b)+**VNI(24b)**+保留(8b) |
| VNI | **24 bit**，约 **1600 万**个租户网段 | 与 VLAN 的 12 bit/4094 对比是必考点 |
| 单层封装额外开销 | **50 字节**（14 外层以太 + 20 IP + 8 UDP + 8 VXLAN） | Underlay MTU 建议 ≥ 1550，DC 内常配 jumbo 9000 |

**关键术语**

| 术语 | 含义 |
|---|---|
| **VTEP** | VXLAN 隧道端点，封装/解封装 VXLAN 报文（通常在 Leaf 上） |
| **NVE** | 华为承载 VTEP 的逻辑接口（`interface nve 1`），下挂 `vni` 与 `source` |
| **VNI** | VXLAN 网络标识，一个 VNI = 一个二层广播域（Overlay 的"VLAN"） |
| **BD** | 桥域（Bridge Domain），VNI 与本地 VLAN 的映射载体（`bridge-domain 10` + `vxlan vni 5010`） |
| **VBDIF** | BD 的三层接口（≈ Overlay 的 VLANIF），配**网关 IP** |
| **分布式网关** | 每台 Leaf 都建 VBDIF 且**IP/MAC 全 Fabric 一致**，虚拟机迁移网关不变 |
| **集中式网关** | 只在 Border Leaf/Spine 建网关，流量绕行，扩展性差（早期方案） |
| **头端复制（Head-End Replication）** | 无 EVPN 时用组播/多播列表泛洪 BUM 流量 |

**报文转发路径（同 VNI、跨 Leaf）**

1. 服务器发出普通以太帧 → Leaf1 查 BD/VNI，命中 EVPN 表项（目的 VTEP = Leaf2）。
2. Leaf1 封装：外层 SIP/DIP = 本 Leaf/对端 Leaf 的 VTEP 地址（Underlay 路由可达），VNI = 5010。
3. Underlay 按外层 IP 逐跳转发（Spine 只查 IP，不看 VNI）。
4. Leaf2 解封装 → 按内层目的 MAC 送服务器。

> **易错**：Spine **不需要知道 VNI**，它只做 IP 转发；VXLAN 是"边缘智能、核心简单"。

## 14.5 BGP EVPN 控制面

**为什么用 EVPN 而不是泛洪学习**：VXLAN 本身无控制面，纯泛洪学习会产生大量 BUM 复制、无法做 ARP 抑制与主机路由；EVPN 把 MAC/IP 变成**BGP 路由**发布，实现**按需建隧道**。

| 要点 | 内容 |
|---|---|
| 地址族 | **L2VPN EVPN**，AFI **25**，SAFI **70** |
| 邻居关系 | Leaf ↔ Spine（Spine 作 **RR**，Leaf 为客户端），Leaf 之间**不直连 BGP** |
| 承载网络 | Underlay 用 OSPF/IS-IS 打通 Loopback（VTEP 地址），EVPN 建在 Loopback 之上 |
| 下一跳 | 路由下一跳 = 对端 **VTEP 地址**（Loopback） |

**5 类路由（必背）**

| 类型 | 名称 | 作用 |
|---|---|---|
| **Type 1** | Ethernet Auto-Discovery（A-D） | 快速收敛、多归（M-LAG）水平分割/别名 |
| **Type 2** | **MAC/IP Advertisement** | 发布 MAC 可达性与主机 IP（**ARP 抑制**、主机路由） |
| **Type 3** | **Inclusive Multicast Ethernet Tag（IMET）** | 发布 VTEP，构建**头端复制列表**（BUM 流量） |
| **Type 4** | Ethernet Segment | 多归接入的 ES 标识（DF 选举） |
| **Type 5** | **IP Prefix** | 发布 IP 前缀路由（跨 VNI/三层互通、对接外部） |

- **ARP 抑制**：Type 2 携带 IP 后，Leaf 可代答 ARP，减少广播——DC 里"广播域很大但广播很少"的原因。
- **Type 2 是 MAC 学习的主通道**，Type 3 是"谁能收广播"的名单，Type 5 是"三层怎么走"。

## 14.6 Multi-POD（单 DC 多 POD）

```text
             [Backbone / Border 层]
        /            |            \
   [POD1: Spine+Leaf] [POD2] [POD3]
```

| 项 | 说明 |
|---|---|
| POD | 一个独立的 Spine-Leaf 单元（自带 Spine），是**故障域与扩容单元** |
| POD 间互联 | 经各自 **Border Leaf** 上联到 Backbone/DCI 层，**不做 POD 间 full-mesh** |
| 流量 | POD 内东西向不出 POD；跨 POD 走 Border Leaf（三层/EVPN Type 5） |
| 收益 | 单 POD 故障不影响其他 POD；按 POD 平滑扩容；规划清晰（每 POD 一个地址块） |
| 典型规模 | 小规模单 POD；中大型 2–4 个 POD + Backbone |

## 14.7 Multi-Site（跨 DC 互联）★第54题考点

**结构**：每个数据中心是一个**独立 Fabric**（各自 Spine/Leaf/Border Leaf，各自 NCE-Fabric），Site 之间由 **Border Leaf ↔ Border Leaf** 建立互联。

| 项 | CloudFabric Multi-Site 口径 |
|---|---|
| 跨 DC 承载 | **VXLAN 隧道**（Overlay 直接延伸到对端 Site） |
| 控制面 | **BGP EVPN**（跨 Site 传递 Type 2 / Type 5） |
| 是否用 MPLS 承载 | **不使用**。方案不含 VPLS / MPLS L3VPN 承载 |
| 物理链路 | Border Leaf 上联出口路由器/DCI 专线，只作**三层 IP 传输** |
| 网关 | 各 Site 独立分布式网关；跨 Site 三层互通靠 Type 5 或 Border Leaf 静态/EBGP 引入 |
| 管理 | NCE-Fabric 统一编排（多 Site 协同），Site 内自治 |

> **判断规则（重要）**：题目说"CloudFabric Multi-Site 跨 DC 互联使用 VXLAN 隧道（BGP EVPN 承载），不含 MPLS 承载" → **正确**。
> 不要拿运营商 DCI（VPLS / MPLS L3VPN / SR）去反驳：那是**运营商网络**的方案，不是 CloudFabric 的方案。
> 详见 `17-exam-traps.md` §17.4 第54题复盘。

## 14.8 M-LAG 与堆叠（iStack / CSS）

| 维度 | **M-LAG** | **iStack（盒式堆叠）** | **CSS（框式集群）** |
|---|---|---|---|
| 本质 | 两台设备**控制面独立**、转发面双活 | 多台设备**合并为一台逻辑设备**（单控制面） | 两台框式合并为一台（单控制面） |
| 控制面 | **双主**（各自有主控、各自跑协议） | 主/备/从，仅主设备转发控制报文 | 主/备 |
| 服务器侧 | 服务器双网卡 bond/双归到两台 | 服务器可跨成员双归 | 同左 |
| 关键链路 | **peer-link**（成员间同步与转发） | **堆叠线**（专用堆叠口/业务口） | **集群线**（主控板间） |
| 防环 | peer-link + **单向隔离**（M-LAG 主备口），配 **DFS 组 / DF 选举** | 堆叠内部无环 | 同左 |
| 升级 | **可逐台升级，业务不中断** | 通常**整机重启**（业务中断） | 通常整机重启 |
| 故障影响 | 单台故障，另一台继续转发（收敛秒级） | 主设备故障需**主备倒换**，期间可能中断 | 同左 |
| 适用 | DC 接入、大二层双活（**CloudFabric 首选**） | 园区接入/小规模 | 园区核心/汇聚 |
| 数量 | 2 台 | 视型号（常见最多 9 台） | 常见 2 台（部分型号 4 台） |

- **M-LAG 必背细节**：两台设备使用**相同的虚拟 MAC**、**相同的 IP**、**相同的 STP 优先级/系统 MAC**（对外表现为一台）；`display dfs-group`、`display m-lag` 排障；**双主检测**用心跳链路（`dfs-group ... dual-active detection`）。
- **堆叠必背细节**：堆叠 ID 冲突会导致无法加入；优先级高者为主；`display stack`、`display stack configuration`。
- **判断题陷阱**："M-LAG 是把两台设备合并成一台" → **错**，那是堆叠；M-LAG 是双活双控制面。

## 14.9 CloudFabric 部署流程（自动化视角）

1. **物理连线**（Spine-Leaf full-mesh、服务器双归）。
2. **设备零配置上线**：空配置启动 → DHCP 拿临时管理 IP → 通过 **ZTP**（DHCP Option 148/PNP）注册到 NCE-Fabric。
3. **Underlay 自动化**：控制器下发接口 IP、Loopback、OSPF/IS-IS、BGP。
4. **Fabric 创建**：选 Spine/Leaf/Border 角色，自动生成 EVPN 配置与 VTEP。
5. **租户网络编排**：建 VPC/租户 → 分配 VNI/VLAN → 自动下发 BD/VBDIF/EVPN 实例。
6. **对接外部**：Border Leaf 配 VXLAN↔VLAN 转换、外部路由引入、防火墙服务链。

## 14.10 易错点

- **VXLAN ≠ 必须 EVPN**：通用上 VXLAN 可以无控制面（泛洪学习）或用 OpenFlow；但 **CloudFabric 用 BGP EVPN**。
- **VNI 24 bit / VLAN 12 bit**：别把 4094 套到 VXLAN 上。
- **Spine 不做网关、不看 VNI**：只做 IP 转发 + EVPN RR。
- **Border Leaf 不是"备用 Leaf"**：它是**唯一出口**，承担跨 POD/跨 Site/外部对接。
- **分布式网关要求 VBDIF 的 IP 与 MAC 全 Fabric 一致**，否则虚拟机迁移后网关不可达。
- **去堆叠**：CloudFabric 用 M-LAG，不要答"用 iStack 做 DC 双活"。
- **NCE-Fabric 管 DC，NCE-Campus 管园区**，两个控制器别写反。
- **Multi-Site 用 VXLAN，不用 MPLS 承载**（方案限定，见 §14.7）。
