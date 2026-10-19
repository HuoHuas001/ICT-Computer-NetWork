# 第1章 初识网络


### 1.1 网络分类与拓扑

| 类型 | 覆盖范围 | 典型技术 |
|---|---|---|
| PAN 个人区域网 | ~10 m | 蓝牙、WPAN |
| LAN 局域网 | 几公里内 | 以太网、Wi-Fi |
| MAN 城域网 | 一个城市 | 以太网(10G/100G)、WiMAX |
| WAN 广域网 | 城市/国家 | HDLC、PPP |

- 计算机网络 = 若干**节点(node)** + 连接节点的**链路(link)**；节点可为计算机、集线器、交换机、路由器。
- Internet 两个基本特点：**连通性**、**资源共享**。
- 互联网三阶段：① 1969–1990 单个 ARPANET 走向互连（**1983 年 TCP/IP 成为标准 → 互联网诞生年**，1990 ARPANET 关闭）；② 1985–1993 NSFNET **三级结构**（主干网/地区网/校园网）；③ 1993–现在 **多层次 ISP**（主干 ISP / 地区 ISP / 本地 ISP）+ 内容提供者。
- 我国：1994-04-20 用 64 kbit/s 专线正式接入 Internet；CERNET 是我国第一个 IPv4 主干网；CERNET2 是 CNGI 主干网。

**拓扑对比（常考优缺点）**

| 拓扑 | 优点 | 缺点 |
|---|---|---|
| 星型 | 便于管理、故障隔离容易 | 中心节点故障全网瘫 |
| 总线型 | 省线缆 | 总线故障全网瘫、冲突多 |
| 环型 | 省线缆、数据单向传输 | 单点断环影响全网 |
| 树型 | 层次化、易扩充 | 层级越高故障影响越大 |
| 全网状 | 完全冗余、可靠性最高 | 端口/线缆成本高 |
| 部分网状 | 成本低于全网状，保留冗余 | 非关键节点仍依赖中心 |

### 1.2 协议与标准化组织

- 协议三要素：**语法、语义、时序（同步）**。
- 组织：**IETF**（Internet 标准，RFC）、**IEEE**（802 系列，以太网/WLAN）、**ISO**（OSI）、ITU-T、IANA（IP 分配）。

### 1.3 参考模型与封装

| OSI 七层 | TCP/IP 四层 | PDU | 典型协议/设备 |
|---|---|---|---|
| 应用层 / 表示层 / 会话层 | 应用层 | 数据 Data | HTTP、DNS、FTP、SMTP |
| 传输层 | 传输层 | 段 Segment | TCP、UDP |
| 网络层 | 网络层（Internet 层） | 包 Packet | IP、ICMP、ARP*、OSPF |
| 数据链路层 | 网络接口层 | 帧 Frame | Ethernet、PPP、PPPoE |
| 物理层 | 网络接口层 | 比特 Bit | 双绞线、光纤、电磁波 |

\* ARP 在 TCP/IP 教材中常归网络层，功能上属二三层之间。

- 封装：自上而下加头（数据→段→包→帧→比特）；解封装：自下而上剥头。
- 各层职责速记：应用层给应用软件提供网络服务；传输层**进程到进程**（靠端口号）；网络层**主机到主机**（靠 IP 地址）；链路层**段内（同一链路）通信**；物理层传比特流。

### 1.4 华为 VRP 系统与命令行

**设备与文件系统**

| 存储 | 掉电 | 用途 |
|---|---|---|
| SDRAM | 丢失 | 运行内存（当前配置运行在此） |
| Flash | 不丢 | 系统软件、配置文件、补丁 |
| NVRAM | 不丢 | 日志缓存、定时器超时/断电时保存 |
| SD Card/USB | 不丢 | 大容量备份 |

- 启动流程：上电 → **BootROM**（初始化硬件）→ 加载 VRP 系统软件 → 读取配置。
- 文件类型：`.zip`（配置文件）、`.bin`（系统软件）、`.pat`（补丁）、`.paf`（产品特性包）。

**视图与提示符**

| 视图 | 提示符 | 进入方式 |
|---|---|---|
| 用户视图 | `<Huawei>` | 登录后默认 |
| 系统视图 | `[Huawei]` | `system-view` |
| 接口视图 | `[Huawei-GigabitEthernet0/0/1]` | `interface ge0/0/1` |
| 协议视图 | `[Huawei-ospf-1]` / `[Huawei-rip-1]` | `ospf 1` / `rip 1` |
| 返回上一级 / 用户视图 | — | `quit` / `return`（`Ctrl+Z`） |

**用户级别（权限）**

| 级别 | 名称 | 说明 |
|---|---|---|
| 0 | 参观级 | 网络诊断类（ping、tracert、telnet） |
| 1 | 监控级 | display 类 |
| 2 | 配置级 | 业务配置命令 |
| 3–15 | 管理级 | 文件系统、FTP、命令级别设置等 |

**编辑与帮助技巧**

- `?` 完全帮助/部分帮助；`Tab` 补全关键字；不完整关键字（`dis cu` = `display current-configuration`）。
- `Ctrl+E` 光标到行尾、`Ctrl+X` 删除左侧全部、`Ctrl+Y` 删除光标及其右侧、`Ctrl+Z` 回用户视图、`Ctrl+]` 终止连接。
- `undo xxx` 恢复缺省/删除配置。

**常用命令（必背）**

```text
<Huawei> system-view                       # 进系统视图
[Huawei] sysname AR1                       # 改设备名
<Huawei> clock timezone BJ add 08:00:00
<Huawei> clock datetime 09:00:00 2024-01-01
[Huawei] header login information "..."    # 登录标语
[Huawei] interface GigabitEthernet0/0/0
[Huawei-GE0/0/0] ip address 192.168.1.1 24
[Huawei] user-interface vty 0 4
[Huawei-ui-vty0-4] authentication-mode password
[Huawei-ui-vty0-4] set authentication password cipher Huawei@123
[Huawei-ui-vty0-4] user privilege level 3
[Huawei] idle-timeout 10                   # 超时（分钟）
<Huawei> display current-configuration     # 当前（运行）配置
<Huawei> save                              # 保存到 Flash
<Huawei> display saved-configuration       # 下次启动配置
<Huawei> reset saved-configuration         # 清除已保存配置
<Huawei> display startup                   # 启动配置参数
<Huawei> startup saved-configuration vrpcfg.zip
<Huawei> reboot                            # 重启
```

**文件系统命令**

| 功能 | 命令 |
|---|---|
| 查当前目录 / 列文件 | `pwd` / `dir` |
| 看文本内容 | `more 文件名` |
| 切目录 / 建目录 / 删目录 | `cd` / `mkdir` / `rmdir` |
| 复制 / 移动 / 重命名 | `copy` / `move` / `rename` |
| 删除 / 恢复 / 清空回收站 | `delete` / `undelete` / `reset recycle-bin` |

> **VRP5 vs VRP8**：VRP5 配置命令**直接生效**；VRP8 配置后需 **commit** 提交才生效。

**易错点**
- 华为设备**缺省所有接口属于 VLAN 1**。
- Console 登录串口速率固定 **9600**；Telnet 23 / SSH 22。
- 保存配置用 `save`，只看不保存用 `display current-configuration`。

---
