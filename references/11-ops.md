# 第11章 网络运维

## 11.1 网络管理基础

**五大功能（ISO 定义）**

| 功能 | 内容 |
|---|---|
| **配置管理** | 采集/监控/下发配置，配置审计与变更管理 |
| **性能管理** | 监控带宽、时延、丢包、CPU/内存，容量规划 |
| **故障管理** | 故障检测、告警、定位、恢复（目标是网络始终可用） |
| **安全管理** | 防未授权访问与攻击，账号/权限/日志 |
| **计费管理** | 记录资源使用、核算费用 |

**管理方式演进**：CLI/Web 单机管理（"一对一"）→ **SNMP 集中式管理** → **iMaster NCE 智能化（管控析一体）**。

## 11.2 SNMP

| 版本 | 特点 | 适用 |
|---|---|---|
| **SNMPv1** | 团体名认证（明文），5 种操作 | 小型网络、安全要求低 |
| **SNMPv2c** | 增加 **GetBulk**、**Inform**（需确认的告警）、更丰富错误码；仍用团体名 | 大中型网络 |
| **SNMPv3** | **身份验证 + 加密（认证/加密密钥）**，安全性高 | 对安全要求高的网络 |

- 架构：**NMS（网管站）+ Agent（被管理设备）+ MIB**。
- 端口：**Agent 用 UDP 161**（接收请求），**NMS 用 UDP 162**（接收 Trap/Inform）。
- **MIB**：被管理对象的树形数据库，用 **OID**（如 `1.3.6.1.2.1`）定位；分**公有 MIB**（标准）与**私有 MIB**（厂商，如华为 `1.3.6.1.4.1.2011`）。

| 操作 | 方向 | 说明 |
|---|---|---|
| **Get** | NMS → Agent | 读取一个/多个节点 |
| **GetNext** | NMS → Agent | 按字典序取下一个节点（用于遍历表） |
| **GetBulk**（v2c+） | NMS → Agent | 批量读取 |
| **Set** | NMS → Agent | 修改节点值 |
| **Response** | Agent → NMS | 响应 |
| **Trap** | Agent → NMS | **主动告警，不需确认** |
| **Inform**（v2c+） | Agent → NMS | 主动告警，**需 NMS 确认**，可重传 |

```text
[Huawei] snmp-agent
[Huawei] snmp-agent sys-info version v2c
[Huawei] snmp-agent community read cipher Huawei123      # 只读团体名
[Huawei] snmp-agent community write cipher Huawei123     # 可写
[Huawei] snmp-agent mib-view included iso iso
[Huawei] snmp-agent trap enable
[Huawei] snmp-agent target-host trap address udp-domain 192.168.27.10 params securityname Huawei123 v2c
[Huawei] snmp-agent target-host inform address udp-domain 192.168.27.10 params securityname Huawei123 v2c
<Huawei> display snmp-agent community
<Huawei> display snmp-agent sys-info
<Huawei> display snmp-agent target-host
```

> 排错口诀：`snmp-agent` 总开关 → 版本 → 团体名/用户 → MIB 视图 → Trap 目的主机 → 告警开关。

## 11.3 SDN 与 OpenFlow

**传统分布式网络的问题**：设备逐台配置、协议复杂（RFC 数量爆炸）、**基于最短路径选路易拥塞**、故障定位难（平均定位耗时 76 min，超 85% 故障靠投诉发现）。

**SDN 三大特征**：**转控分离**、**集中控制**、**开放可编程接口**。

| 层次 | 内容 |
|---|---|
| **协同应用层** | OSS/BSS、云平台、业务编排等上层应用 |
| **控制器层** | SDN 控制器（核心，掌握全局视图，计算路径并下发流表） |
| **设备层** | 交换机/路由器，**仅保留数据平面**，按流表转发 |
| **NBI 北向接口** | 控制器 ↔ 应用，典型 **RESTful** |
| **SBI 南向接口** | 控制器 ↔ 设备，典型 **OpenFlow、NETCONF、SNMP、PCEP、OVSDB** |

**OpenFlow 三种消息**

| 类型 | 发起方 | 子类型举例 |
|---|---|---|
| **Controller-to-Switch** | 控制器 | Configuration、Read-State、**Packet-out**、Modify-State、Send-Packet |
| **Asynchronous** | 交换机 | **Packet-in**（无匹配上送）、**Flow-Removed**、**Port-Status** |
| **Symmetric** | 双方均可 | Hello、Echo、Error、Vendor |

**流表项 7 要素**：**Match Fields（匹配域）、Priority（优先级，高者先匹配）、Counters（统计报文/字节数）、Instructions（动作指令）、Timeouts（Idle/Hard 超时）、Cookie、Flags**。
- 匹配域 **12 元组**（v1.0）：入端口、源/目的 MAC、以太网类型、VLAN ID、VLAN 优先级、源/目的 IP、IP 协议、IP ToS、TCP/UDP 源/目的端口。
- 多级流表：从 **table0** 开始依次匹配（table0–table255）。
- 常见动作：**Output（指定端口）、IN_PORT（从入端口发回）、Drop、Set-Field（改字段）、Group**。

**工作流程**：主机发包 → 交换机查流表**无匹配 → Packet-in 上报控制器** → 控制器计算路径并下发流表 → 后续报文直接匹配转发。

```text
# Mininet 常用
sudo mn --controller=none --topo=tree,depth=2,fanout=2
sudo mn --topo=linear,3 --switch=ovsk
sudo mn -c                       # 清理
ovs-vsctl show
ovs-ofctl dump-flows s1
ovs-ofctl add-flow s1 priority=10,arp,actions=output:2
```

## 11.4 华为 iMaster NCE 与自动化

- **iMaster NCE**：华为**管控析一体**的自动驾驶网络管理与控制系统；南向实现设备集中管理与配置下发，北向开放给上层平台。
  - **NCE-Fabric（数据中心）**：ZTP 零配置开局、意图理解与业务快速部署、**变更仿真**（预判风险）、**FabricInsight** AI 智能运维。
  - **NCE-Campus（园区）**：设备即插即用（APP 扫码 / DHCP / 注册查询中心）、**业务随行**（基于**安全组**的策略，与 IP/位置解耦）、**一网多用**（VN 虚拟网络）、终端智能识别、有线无线融合、基于 **Telemetry** 的体验可视。
- 配置下发协议：**NETCONF**（XML/YANG 模型）、SNMP、OpenFlow、PCEP。
- 网络自动化工具：**Ansible、SaltStack、Puppet、Chef**（声明式/无代理）；网络工程师常用 **Python** 脚本。
  - **telnetlib**（标准库，Telnet 明文，教学用）
  - **paramiko**（第三方，`pip install paramiko`；**SSHClient** 做 SSH 会话、**SFTPClient** 做文件传输）

```python
import paramiko
ssh = paramiko.SSHClient()
ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())
ssh.connect('192.168.27.10', username='python', password='Huawei@123')
cli = ssh.invoke_shell()
cli.send('system-view\n')
print(cli.recv(65535).decode())
ssh.close()
```

**易错点**
- SNMP **161 是 Agent 端口，162 是 NMS 收告警端口**（别记反）。
- Trap **不需确认**，Inform **需要确认**（v2c 起支持）。
- OpenFlow 是**南向**接口协议；RESTful 是**北向**。
- SDN 的本质是"集中大脑 + 开放接口"，**不等于 OpenFlow**（OpenFlow 只是南向实现之一）。
