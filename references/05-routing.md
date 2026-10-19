# 第5章 路由技术


### 5.1 静态路由

```text
# 三种写法
[Huawei] ip route-static 20.1.1.0 24 10.0.0.2                 # 只写下一跳（推荐）
[Huawei] ip route-static 20.1.1.0 24 Serial1/0/0              # 只写出接口
[Huawei] ip route-static 20.1.1.0 24 Serial1/0/0 10.0.0.2     # 出接口 + 下一跳
[Huawei] ip route-static 0.0.0.0 0 10.0.0.2                   # 缺省路由
[Huawei] ip route-static 20.1.1.0 24 10.0.0.2 preference 100   # 浮动路由（备份）
<Huawei> display ip routing-table protocol static
<Huawei> display ip routing-table 20.1.1.1
```

| 概念 | 要点 |
|---|---|
| **出接口 vs 下一跳** | **广播型接口（以太网）不能只写出接口**（需 ARP 找目的 MAC）；**点对点接口（串口）可互换**但效率不同；建议**同时指定**，避免路由递归 |
| **路由递归/迭代** | 下一跳不直连时，需再查表找可达下一跳；会降低效率 |
| **缺省路由** | `0.0.0.0/0`，所有具体路由都不匹配时使用；常用于出口设备 |
| **浮动路由** | 同一目的、**优先级不同**，主用失效时备份接管 |
| **等价路由** | 优先级与开销**都相同** → 同时生效、负载分担 |

- 静态路由优缺点：配置简单、对设备要求低、**不占带宽**；但**不能自动适应拓扑变化**、需人工维护，适合小型/稳定/末梢网络。
- 报文**逐跳转发**，所以**路径上每一台设备都要有往返双向的路由**。

### 5.2 RIP

| 项 | 值 |
|---|---|
| 类型 | **距离矢量（Bellman-Ford）**，IGP |
| 度量 | **跳数**，最大 **15**，**16 = 不可达** |
| 封装 | **UDP 520** |
| 更新 | 周期 **30 s** 广播/组播全表；**180 s** 未收到 → 标记不可达；**120 s** 后删除 |
| 版本 | RIP-1（有类、广播、无掩码、不支持 VLSM/认证）；**RIP-2（无类、组播 224.0.0.9、携带掩码、支持认证/VLSM/汇总）** |
| 报文 | 头部 4 B + 路由项（每项 20 B），**一个报文最多 25 条路由** |

- 三个特点：**只和相邻路由器交换**、交换**自己的全部路由表**、**固定周期**交换。
- 缺陷：**好消息传得快、坏消息传得慢（慢收敛）**，可能产生**环路**。
- 防环机制：
  - **水平分割 Split Horizon**：从某接口学到的路由不再从该接口发回。
  - **毒性反转 Poison Reverse**：从某接口学到的路由，从原接口发回时**跳数置 16**（明确告知不可达）。
  - **触发更新**：拓扑变化立即发送，不等 30 s。
  - 最大跳数 15 限制、抑制时间。

```text
[Huawei] rip 1
[Huawei-rip-1] version 2
[Huawei-rip-1] network 10.0.0.0            # 只配主类网段，不配掩码
[Huawei-rip-1] summary                     # 开启自动汇总（RIP-2）
[Huawei-rip-1] undo summary                # 关闭自动汇总
[Huawei-rip-1] silent-interface GigabitEthernet0/0/1
[Huawei-GE0/0/1] rip summary-address 10.1.0.0 255.255.0.0   # 手动汇总
[Huawei-GE0/0/1] rip split-horizon                          # 水平分割
[Huawei-GE0/0/1] rip poison-reverse                         # 毒性反转
<Huawei> display rip 1 route
<Huawei> display rip 1 neighbor
```

### 5.3 OSPF

**四步工作流程**：建立邻居 → 泛洪 LSA 组建 **LSDB** → 运行 **SPF（Dijkstra）** 计算最短路径 → 生成路由表。

**基础术语**

| 术语 | 要点 |
|---|---|
| Area | 逻辑分组，用 Area ID 标识；**Area 0 为骨干区域**，其他区域必须与它相连 |
| Router-ID | 唯一标识路由器：**手工配置 > 最大 Loopback 地址 > 最大物理接口地址** |
| Cost | 接口开销 = **参考带宽 / 接口带宽**（参考值缺省 10⁸ = 100 Mbps，`bandwidth-reference` 可改）；GE 口缺省 Cost = 1；一条路由 Cost = 沿途**入接口** Cost 累加 |
| 邻居 vs 邻接 | 邻居=收到 Hello；**邻接（Full）= 交换了 LSDB** |

**5 种报文（协议号 89，直接封装在 IP 中）**

| 报文 | 作用 |
|---|---|
| **Hello** | 发现/维持邻居、选举 DR/BDR |
| **DD (Database Description)** | 交互 LSDB 摘要（含主从协商） |
| **LSR (Link State Request)** | 请求缺失 LSA 详情 |
| **LSU (Link State Update)** | 携带完整 LSA 泛洪 |
| **LSAck** | 确认收到 LSU |

**报文头部关键字段**：Version、Type、Packet Length、**Router ID**、**Area ID**、Checksum、Authentication、Auth Type。
**组播地址**：`224.0.0.5`（AllSPFRouters）、`224.0.0.6`（AllDRouters）。
**定时器（Broadcast/P2P）**：Hello **10 s**，Router Dead **40 s**（NBMA：30 s / 120 s）。

**邻居状态机（7/8 种）**

`Down → Init → 2-Way → ExStart → Exchange → Loading → Full`
（NBMA 还有 Attempt）

- **2-Way**：双向通信建立（DRother 之间停留在此状态）。
- **ExStart**：协商主从（Router-ID 大者为主）。
- **Loading**：发送 LSR 请求完整 LSA。
- **Full**：LSDB 同步完成（**只有 Full 才是邻接**）。

**DR / BDR 选举**
- 只在 **MA 网络（Broadcast/NBMA）** 选举；**P2P、P2MP 不选举**。
- 比较：**DR 优先级（默认 1，大者优）** → **Router-ID（大者优）**；优先级 **0 表示不参选**。
- **DR 不可抢占**（已有 DR 时不因新来更高优先级而更换），只有 DR 故障 BDR 才接替。
- 目的：减少 MA 网络中邻接关系数量（n 台设备由 n(n−1)/2 降为 2n−3 左右），减少 LSA 泛洪。

**LSA 类型（必背）**

| 类型 | 名称 | 产生者 | 泛洪范围 / 作用 |
|---|---|---|---|
| **1** | Router LSA | 每台 OSPF 路由器 | 本区域内，描述路由器链路/接口状态与开销 |
| **2** | Network LSA | **DR** | 本区域内，描述 MA 网络及所连路由器 |
| **3** | Network Summary LSA | **ABR** | 区域间，描述**其他区域的网段路由** |
| **4** | ASBR Summary LSA | **ABR** | 描述到 **ASBR** 的路由 |
| **5** | AS External LSA | **ASBR** | 全 AS（除特殊区域），描述**外部路由** |
| **7** | NSSA External LSA | NSSA 内 ASBR | 仅在本 NSSA 内，到 ABR 转为 Type 5 |

**路由器角色**：内部路由器（Internal）、**ABR**（连接 ≥2 个区域，且有一个是 Area 0）、**骨干路由器**（接口在 Area 0）、**ASBR**（引入外部路由）。

**区域类型**

| 区域 | 允许的 LSA | 特点 |
|---|---|---|
| 标准区域 | 1/2/3/4/5 | 全功能 |
| **Stub 末节** | 1/2/3（+ 默认路由 3 类） | ABR 阻挡 **4/5 类**，下发**一条 3 类缺省路由** |
| **Totally Stub 完全末节** | 1/2 + 默认路由 | ABR 阻挡 **3/4/5 类**（`stub no-summary`），只有一条默认路由 |
| **NSSA 次末节** | 1/2/3/7 | 允许自己引入外部路由（7 类），ABR 转 5 类；不允许 4/5 类进入 |

- 多区域原则：**非骨干区域之间不能直接相连**（需经 Area 0）；否则要用**虚连接（Virtual Link）**。

```text
[Huawei] ospf 1 router-id 1.1.1.1
[Huawei-ospf-1] area 0
[Huawei-ospf-1-area-0.0.0.0] network 10.0.0.0 0.0.0.255     # 反掩码（通配符）
[Huawei-ospf-1-area-0.0.0.0] network 1.1.1.1 0.0.0.0
[Huawei-ospf-1] bandwidth-reference 10000                   # 修改参考带宽
[Huawei-GE0/0/0] ospf cost 10                               # 接口开销
[Huawei-GE0/0/0] ospf dr-priority 100                       # DR 优先级（0 不参选）
[Huawei-ospf-1-area-0.0.0.1] stub                           # 末节
[Huawei-ospf-1-area-0.0.0.1] stub no-summary                # 完全末节
[Huawei-ospf-1-area-0.0.0.1] nssa
[Huawei-ospf-1] import-route direct                         # 引入外部路由（成为 ASBR）
[Huawei-ospf-1-area-0.0.0.1] abr-summary 10.1.0.0 255.255.0.0
[Huawei-ospf-1] default-route-advertise always               # 下发默认路由
<Huawei> display ospf peer [brief]      # 邻居
<Huawei> display ospf lsdb              # 链路状态数据库
<Huawei> display ospf routing           # OSPF 路由表
<Huawei> display ospf interface         # 接口开销/网络类型
<Huawei> display ospf brief
```

**OSPF 网络类型**

| 类型 | 典型链路 | 是否选 DR |
|---|---|---|
| Broadcast（BMA） | 以太网 | 是 |
| NBMA | 帧中继 | 是（需手动指定邻居） |
| Point-to-Point | PPP/串口 | 否 |
| Point-to-Multipoint | 改造后的 NBMA | 否 |

> 链路两端网络类型必须一致，否则无法建立邻居。

### 5.4 OSPFv3（IPv6）

| 相同点 | 不同点 |
|---|---|
| 同样是链路状态、5 种报文、**协议号 89**、SPF 计算、DR/BDR 选举、区域与 LSA 思想 | **基于链路（Link）运行**而非网段；**用链路本地地址 FE80::/10 作报文源和下一跳**；支持**一个链路上多实例（Instance ID）**；报文/LSA 中去掉 IP 地址语义；**新增 Type 8 Link-LSA、Type 9 Intra-Area-Prefix-LSA** |

- 组播：**FF02::5**（AllSPFRouters）、**FF02::6**（AllDRouters）。
- LSA 对应关系：Router-LSA(1)、Network-LSA(2)、Inter-Area-Prefix-LSA(3)、Inter-Area-Router-LSA(4)、AS-External-LSA(5)、NSSA(7) + 新增 **Link-LSA(8)**、**Intra-Area-Prefix-LSA(9)**。

```text
[Huawei] ospfv3 1
[Huawei-ospfv3-1] router-id 1.1.1.1
[Huawei-GE0/0/0] ospfv3 1 area 0
<Huawei> display ospfv3 peer
<Huawei> display ospfv3 lsdb
<Huawei> display ospfv3 routing
```

### 5.5 VLAN 间路由

| 方案 | 做法 | 优缺点 |
|---|---|---|
| **单臂路由**（路由器子接口） | 交换机上联口配 **Trunk**；路由器一个物理口划多个子接口，每个子接口 `dot1q termination vid X` + IP 作为该 VLAN 网关 | 省物理口，但**流量都经一条链路**，易成瓶颈 |
| **VLANIF（三层交换机）** | `interface vlanif 10` + `ip address`，VLANIF 编号 = VLAN ID | **主流方案**，一次路由多次交换，性能高 |
| 物理接口直连 | 每个 VLAN 一个路由器物理口 | 浪费接口，扩展性差 |

```text
# 单臂路由
[R1] interface GigabitEthernet0/0/1.10
[R1-GE0/0/1.10] dot1q termination vid 10
[R1-GE0/0/1.10] ip address 192.168.10.254 24
[R1-GE0/0/1.10] arp broadcast enable          # 子接口必须开启 ARP 广播
# 三层交换机
[SW1] interface Vlanif 10
[SW1-Vlanif10] ip address 192.168.10.254 24
```

> 三层转发时：**源/目的 MAC 逐跳改变，源/目的 IP 不变**；交换机查路由后由**路由模块**重新封装二层。

---
