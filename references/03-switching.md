# 第3章 交换技术


### 3.1 VLAN 与 Trunk

**为什么要 VLAN**：缩小广播域、抑制广播风暴、隔离业务、提高安全性与管理灵活性。

**802.1Q 标签（4 字节，插在源 MAC 之后）**

| 字段 | 长度 | 取值/含义 |
|---|---|---|
| TPID | 2 B | **0x8100** 表示 802.1Q 帧（各厂商可自定义，邻居不一致则需改 TPID） |
| PRI | 3 bit | 优先级 0–7，越大越优先（QoS） |
| CFI | 1 bit | 以太网中为 **0**（标准格式） |
| VID | 12 bit | VLAN ID，0–4095；**0 和 4095 保留**，可配 **1–4094** |

- 两种帧：**Untagged**（普通以太帧，终端收发都是这种）与 **Tagged**（802.1Q 帧，交换机间链路用）。
- **缺省 VLAN = VLAN 1**，所有接口初始都属于 VLAN 1。
- VLAN 划分方式：**基于接口（最常用）**、基于 MAC、基于 IP 子网、基于协议、基于策略。

**三种接口类型的收发规则（考试核心）**

| 接口 | 收到 Untagged 帧 | 收到 Tagged 帧 | 发出帧 |
|---|---|---|---|
| **Access** | 打上 **PVID**，从该 VLAN 转发 | VID == PVID → 接收；否则丢弃 | **剥离 Tag**，只发 Untagged（只能属于 1 个 VLAN） |
| **Trunk** | 打上 **PVID** | VID 在 allow-pass 列表中 → 接收；否则丢弃 | VID == PVID → **剥离**后发；VID ≠ PVID 且在允许列表 → **带 Tag** 发 |
| **Hybrid** | 打上 **PVID** | 在 tagged/untagged 允许列表中 → 接收 | 在 `untagged vlan` 列表 → 剥离 Tag 发；在 `tagged vlan` 列表 → 带 Tag 发 |

> 口诀：**Access 出必不带 Tag；Trunk 出只有 PVID 那一个 VLAN 不带 Tag；Hybrid 可任意**。
> Hybrid 是 Access 与 Trunk 的超集，**基于 MAC 的 VLAN 必须在 Hybrid 接口上配置**。

**配置命令**

```text
[SW1] vlan batch 10 20 30            # 批量创建
[SW1] vlan 10                        # 创建并进入 VLAN 视图
[SW1] interface GigabitEthernet0/0/1
[SW1-GE0/0/1] port link-type access
[SW1-GE0/0/1] port default vlan 10   # Access 缺省 VLAN
[SW1-GE0/0/2] port link-type trunk
[SW1-GE0/0/2] port trunk allow-pass vlan 10 20      # 或 all
[SW1-GE0/0/2] port trunk pvid vlan 10               # 可选，缺省 1
[SW1-GE0/0/3] port link-type hybrid
[SW1-GE0/0/3] port hybrid pvid vlan 10
[SW1-GE0/0/3] port hybrid untagged vlan 10 20       # 发出时剥离 Tag
[SW1-GE0/0/3] port hybrid tagged vlan 30            # 发出时带 Tag
[SW1] display vlan [vlan-id] [verbose]
```

**基于 MAC 划分 VLAN（配置在 Hybrid 接口）**

```text
[SW1] vlan 10
[SW1-vlan10] mac-vlan mac-address 00e0-fc01-0001    # 可加掩码/掩码长度(1-48)
[SW1] interface GigabitEthernet0/0/1
[SW1-GE0/0/1] port link-type hybrid
[SW1-GE0/0/1] mac-vlan enable
[SW1] display mac-vlan { all | mac-address ... | vlan vlan-id }
```
- 优点：主机移动无需重配 VLAN。注意 MAC 不能设为 `0000-0000-0000`、`FFFF-FFFF-FFFF` 或组播地址。

### 3.2 生成树协议（STP/RSTP/MSTP）

**二层环路三大危害**：广播风暴、MAC 地址表漂移（震荡）、重复帧/多帧复制。

**STP 基本概念**

| 概念 | 说明 |
|---|---|
| BID（桥 ID） | **16 bit 桥优先级 + 48 bit MAC**，优先级默认 **32768**，步长 4096；**BID 最小者为根桥** |
| 根桥 Root | 全网唯一，逻辑中心；**可抢占**；只有根桥周期性（2 s）主动发配置 BPDU |
| Cost | 端口开销，带宽越高越小（缺省用 **IEEE 802.1t** 算法） |
| RPC | 根路径开销 = 到根桥沿途**入方向**端口 Cost 累加 |
| PID（端口 ID） | 高 **4 bit 端口优先级**（默认 128，步长 16）+ 低 12 bit 端口号 |
| BPDU | 配置 BPDU（拓扑计算，目的 MAC `01-80-C2-00-00-01`）+ TCN BPDU（拓扑变更通知） |

**端口 Cost 表（本书推荐值，必背）**

| 速率/模式 | 802.1d-1998 | 802.1t（华为缺省） | 华为算法 |
|---|---|---|---|
| 100M 半双工 | 19 | 200000 | 200 |
| 100M 全双工 | 18 | 199999 | 199 |
| 100M 聚合 2 口 | 15 | 100000 | 180 |
| 1000M 全双工 | 4 | 20000 | 20 |
| 1000M 聚合 2 口 | 3 | 10000 | 18 |
| 10G 全双工 | 2 | 2000 | 2 |
| 10G 聚合 2 口 | 1 | 1000 | 1 |
| 40G 全双工 | 1 | 500 | 1 |
| 40G 聚合 2 口 | 1 | 250 | 1 |
| 100G 全双工 | 1 | 200 | 1 |
| 100G 聚合 2 口 | 1 | 100 | 1 |

**定时器（默认值，必背）**：`Hello Time = 2 s`，`Max Age = 20 s`，`Forward Delay = 15 s`。
**收敛时间**：根桥故障 / 非直连链路故障 ≈ **50 s = 20 + 15×2**；直连链路故障 ≈ **30 s = 15×2**。

**STP 计算四步**

1. 选举**一个根桥**（BID 最小）。
2. 每台非根桥选举**一个根端口**：比较顺序 ① 最小 RPC → ② 最小对端 BID → ③ 最小对端 PID。
3. 每条链路选举**一个指定端口**：比较 ① 最小 RPC（本机该口） → ② 本机 BID → ③ 本机 PID。**根桥上所有端口都是指定端口**。
4. 剩下的端口（非根非指定）**阻塞**，成为**预备端口（Alternate/Blocking）**。

> 配置 BPDU 比较原则（收到 BPDU 与自己 BPDU 比）：**最小根 BID → 最小 RPC → 最小发送者 BID → 最小发送者 PID**；优则更新，劣则丢弃（自己更优就发自己的）。

**STP 五种端口状态**

| 状态 | 收/发 BPDU | 学 MAC | 转发用户数据 | 停留时间 |
|---|---|---|---|---|
| Disabled | 否 | 否 | 否 | — |
| Blocking 阻塞 | 只收 | 否 | 否 | Max Age |
| Listening 侦听 | 收发 | 否 | 否 | Forward Delay 15 s |
| Learning 学习 | 收发 | **是** | 否 | Forward Delay 15 s |
| Forwarding 转发 | 收发 | 是 | 是 | — |

- 只有**根端口或指定端口**才能进入 Forwarding。
- 拓扑变更：下游发 **TCN BPDU** → 上游逐级转发至根桥 → 根桥回 **TCA** 标志并置 **TC 位** → 全网把 MAC 老化时间从 300 s 临时改为 **Forward Delay（15 s）**。

**RSTP（802.1w）对 STP 的改进**

| 项目 | STP | RSTP |
|---|---|---|
| 端口状态 | 5 种 | **3 种：Discarding / Learning / Forwarding** |
| 端口角色 | 根、指定、阻塞 | **根、指定、Alternate（替代）、Backup（备份）** + 边缘端口 |
| 收敛 | 被动等定时器 | 主动协商（Proposal/Agreement），**边缘端口直连终端可直接 Forwarding** |
| BPDU | 只有根桥周期性发 | **所有设备每 2 s 都发**；邻居 3 个 Hello（6 s）收不到即认为失效 |

**MSTP（802.1s）**
- **MST 域**（同域名、同修订级别、同 VLAN 映射表）+ **MSTI 实例**；**Instance 0 = 缺省实例（CIST）**，缺省所有 VLAN 映射到实例 0。
- 端口角色增加 **Master（域边缘连总根的端口）**、**域边缘端口**。
- 术语：**总根 CIST Root / 域根 Regional Root / 主桥 Master Bridge**。
- **VBST**：华为方案，每个 VLAN 一棵树，VLAN 多时开销大 → 所以用 MSTP 把多个 VLAN 映射到一个实例。

**配置命令**

```text
[SW1] stp mode { stp | rstp | mstp }        # 缺省 MSTP
[SW1] stp enable                            # 全局/接口使能
[SW1] stp root primary                      # 主根桥（优先级自动 0）
[SW1] stp root secondary                    # 备份根桥（优先级 4096）
[SW1] stp priority 4096                     # 手动改桥优先级（步长 4096）
[SW1] stp pathcost-standard { dot1d-1998 | dot1t | legacy }
[SW1-GE0/0/1] stp cost 20000                # 接口开销
[SW1-GE0/0/1] stp priority 16               # 接口优先级（步长 16）
[SW1-GE0/0/1] stp edged-port enable         # 边缘端口（接终端）
<SW1> display stp brief                     # 看各口角色/状态
<SW1> display stp [instance x] [interface ...]

# MSTP 域配置（四要素必须一致：域名、映射表、修订级别、（同一域））
[SW1] stp region-configuration
[SW1-mst-region] region-name huawei
[SW1-mst-region] instance 1 vlan 10 20
[SW1-mst-region] instance 2 vlan 30 40
[SW1-mst-region] revision-level 1
[SW1-mst-region] active region-configuration
[SW1] stp instance 1 root primary
[SW1] stp instance 2 root secondary
```

> 负载分担考点：让 SW1 做实例 1 的根、SW2 做实例 2 的根，则 VLAN10/20 走一侧、VLAN30/40 走另一侧。

### 3.3 链路聚合（Eth-Trunk）

- 作用：**提高带宽**（突破 STP 单链路限制）+ **链路冗余备份** + 负载分担。
- 术语：聚合组 LAG、成员接口/成员链路、活动接口、**非活动（Unselected）接口**、LACP。

| 模式 | 特点 | 适用 |
|---|---|---|
| **手工模式** | 不协商，全部链路都转发，平均分担；一端故障只能靠物理状态感知 | 对端不支持 LACP |
| **LACP 模式** | 通过 **LACPDU** 协商；系统 LACP 优先级小者成为**主动端**，由主动端按接口优先级选活动接口；支持最大/最小活动接口数、备份链路 | 推荐，可检测链路错连 |

- 同一聚合组成员接口要求：**速率、双工、VLAN 配置等参数一致**（缺省不支持不同速率，需 `mixed-rate link enable`）。
- 缺省系统 LACP 优先级 **32768**；接口 LACP 优先级缺省 **32768**；负载分担缺省 **src-dst-ip**。

```text
[SW1] interface eth-trunk 1
[SW1-Eth-Trunk1] mode { lacp | manual load-balance }
[SW1-Eth-Trunk1] trunkport gigabitethernet 0/0/1 to 0/0/3
[SW1-Eth-Trunk1] max active-linknumber 2
[SW1-Eth-Trunk1] least active-linknumber 2
[SW1-Eth-Trunk1] load-balance { dst-ip | dst-mac | src-ip | src-mac | src-dst-ip | src-dst-mac }
[SW1-Eth-Trunk1] port link-type trunk
[SW1-Eth-Trunk1] port trunk allow-pass vlan 10 20
[SW1] lacp priority 30000                    # 系统优先级，小者主动
[SW1-GE0/0/1] eth-trunk 1                    # 另一种加入方式
[SW1-GE0/0/1] lacp priority 100              # 接口优先级
<SW1> display eth-trunk 1
```
- 负载分担模式选择原则：**流量中哪个参数变化频繁，就按哪个参数分担**（IP 变化频繁→按 IP）。

---
