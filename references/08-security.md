# 第8章 安全技术


### 8.1 ACL

**作用**：匹配并区分流量，用于**包过滤（traffic-filter）**、NAT、路由策略、防火墙策略、QoS 等。

**编号范围（必背）**

| 类别 | 编号范围 | 可匹配内容 |
|---|---|---|
| **基本 ACL** | **2000–2999** | 仅 **源 IP**（及分片、时间范围） |
| **高级 ACL** | **3000–3999** | 源/目的 IP、**协议号**、TCP/UDP 源/目的端口、ICMP 类型等 |
| 二层 ACL | 4000–4999 | 源/目的 MAC、以太网类型 |
| 用户自定义 ACL | 5000–5999 | 自定义规则 |
| 用户 ACL（命名） | 6000–9999 | 命名方式 |

**规则编号与步长**
- 缺省**步长 5**（`step` 可改），首条自动编号 5，之后 10、15…
- 编号**小的先匹配**；**一旦命中即停止匹配**（不再看后续规则）。
- ACL 末尾有**系统隐含规则 `rule 4294967294 permit`**（华为：未匹配任何规则则**允许**通过）。
- **匹配顺序**：
  - `config`（缺省）：按**配置顺序/编号**匹配。
  - `auto`：按**深度优先（精确度）**自动排序，越精确越先匹配（如 10.1.1.0/24 优先于 10.1.0.0/16）。

**通配符（反掩码）**
- 32 bit，**0 = 必须严格匹配，1 = 任意（不关心）**。
- 例：`192.168.1.0 0.0.0.255` → 匹配 192.168.1.0/24。
- 特殊：`0.0.0.0` = 精确匹配一个主机（等价 `host`）；`255.255.255.255` = 任意地址（等价 `any`）。
- 只匹配**奇数 IP** 的写法：`192.168.1.1 0.0.0.254`（最后一位第 0 位为 1 必须匹配）。

```text
# 基本 ACL
[Huawei] acl number 2000
[Huawei-acl-basic-2000] rule 5 permit source 10.1.1.0 0.0.0.255
[Huawei-acl-basic-2000] rule 10 deny source 10.1.0.0 0.0.255.255
[Huawei-acl-basic-2000] rule 15 permit                       # 允许其余
[Huawei] interface GigabitEthernet0/0/1
[Huawei-GE0/0/1] traffic-filter inbound acl 2000
# 高级 ACL
[Huawei] acl number 3001
[Huawei-acl-adv-3001] rule 5 permit ip source 10.1.1.0 0.0.0.255 destination 10.1.3.0 0.0.0.255
[Huawei-acl-adv-3001] rule 10 deny tcp source 10.1.2.0 0.0.0.255 destination 10.1.3.0 0.0.0.255 destination-port eq 21
[Huawei-acl-adv-3001] rule 20 permit icmp source any destination any icmp-type echo
[Huawei-GE0/0/2] traffic-filter outbound acl 3001
# 生效时段
[Huawei] time-range work 08:30 to 21:00 daily
[Huawei-acl-basic-2002] rule 5 permit time-range work
<Huawei> display acl all
<Huawei> display acl 2000
```

> 要点：ACL **只做匹配不做动作**，动作由调用它的业务决定（permit/deny 在不同场景含义不同，如 NAT 中 permit 表示"要转换"）。
> 绑定方向：**inbound = 进入本设备的流量**，**outbound = 从本设备发出的流量**。

### 8.2 AAA 与 RADIUS

- **AAA = Authentication 认证 + Authorization 授权 + Accounting 计费**。
- 架构：**用户（User）— NAS（网络接入服务器，如路由器/交换机）— AAA 服务器（RADIUS/HWTACACS）**。
- **域（Domain）**：`用户名@域名`；无 `@` 则属于**缺省域**（`default_admin`、`default` 等）。每个域可绑定不同认证/授权/计费方案。

| 功能 | 支持方式 |
|---|---|
| 认证 | 不认证、**本地认证**、远端认证（RADIUS / HWTACACS） |
| 授权 | 不授权、本地授权、远端授权（RADIUS 认证与授权绑定，不能单独用 RADIUS 授权；HWTACACS 可分离） |
| 计费 | 不计费、远端计费（RADIUS/HWTACACS） |

- **RADIUS：UDP，认证端口 1812，计费端口 1813**（早期 1645/1646）。NAS 作客户端，把用户信息送服务器，按结果放行/拒绝。
- 授权信息可包括：用户组、VLAN、ACL 编号等。

```text
[Huawei] radius-server template rd1
[Huawei-radius-rd1] radius-server authentication 10.1.1.100 1812
[Huawei-radius-rd1] radius-server shared-key cipher Huawei@123
[Huawei] aaa
[Huawei-aaa] authentication-scheme sch1
[Huawei-aaa-authen-sch1] authentication-mode radius local       # 先远端后本地
[Huawei-aaa] accounting-scheme acc1
[Huawei-aaa-accounting-acc1] accounting-mode radius
[Huawei-aaa] domain huawei.com
[Huawei-aaa-domain-huawei.com] authentication-scheme sch1
[Huawei-aaa-domain-huawei.com] radius-server rd1
[Huawei-aaa] local-user user1 password cipher Huawei@123
[Huawei-aaa] local-user user1 service-type telnet
[Huawei-aaa] local-user user1 privilege level 3
<Huawei> display domain name huawei.com
<Huawei> display aaa offline-record
```

---
