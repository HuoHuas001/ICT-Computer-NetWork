# 第9章 广域网技术


### 9.1 PPP

- 广域网设备角色：**CE（用户边缘）、PE（运营商边缘）、P（运营商骨干）**。
- PPP 是**点到点**链路层协议，用于串行/租用线路、拨号。相对于 HDLC 的优势：**认证（PAP/CHAP）、错误检测、多链路捆绑（MP）、压缩、可扩展（PPPoE）**。

**PPP 帧格式**

| 字段 | 值 |
|---|---|
| Flag | **0x7E**（01111110），帧首尾 |
| Address | **0xFF**（标准广播地址） |
| Control | **0x03**（无序号帧） |
| Protocol | 2 B：**0xC021 = LCP，0x8021 = IPCP，0x0021 = IPv4，0xC223 = CHAP，0xC023 = PAP** |
| Data | 上层数据 |
| FCS | 2 B 校验 |

> 透明传输：出现 0x7E / 0x7D 时转义为 **0x7D + (字节 XOR 0x20)**。

**三个建立阶段**
1. **链路层协商（LCP）**：协商 MRU、认证方式、魔术字等。
2. **认证协商（可选）**：PAP 或 CHAP。
3. **网络层协商（NCP/IPCP）**：协商 IP 地址、DNS 等（静态或动态分配）。

**LCP 报文类型**：Configure-Request / Configure-Ack / Configure-Nak（参数不匹配，建议新值）/ Configure-Reject（参数不识别）、Terminate-Request/Ack（关闭链路）、Code-Reject、Protocol-Reject、Echo-Request/Reply（保活）。
**接口状态机**：Initial → Starting → (Req-Sent / Ack-Rcvd / Ack-Sent) → **Opened** → Closing/Stopping。

**PAP vs CHAP（必考）**

| 项 | PAP | CHAP |
|---|---|---|
| 握手次数 | **2 次** | **3 次** |
| 传输 | **明文** | **密文（MD5 摘要）** |
| 发起方 | 被验证方发送用户名/密码 | **验证方主动发 Challenge（ID + 随机数）** |
| 安全性 | 低 | 高，可防重放 |
| 重试 | 由被验证方反复发送 | 由验证方控制次数 |

```text
[R1] interface Serial1/0/0
[R1-Serial1/0/0] link-protocol ppp                 # 串口缺省即 PPP
[R1-Serial1/0/0] ppp authentication-mode pap       # 验证方
[R1-Serial1/0/0] ppp pap local-user r2 password cipher Huawei@123
[R1] aaa
[R1-aaa] local-user r2 password cipher Huawei@123
[R1-aaa] local-user r2 service-type ppp
[R2-Serial1/0/0] ppp pap local-user r2 password cipher Huawei@123   # 被验证方
# CHAP
[R1-Serial1/0/0] ppp authentication-mode chap
[R1-Serial1/0/0] ppp chap user r1
[R2-Serial1/0/0] ppp chap user r2
[R1-aaa] local-user r2 password cipher Huawei@123
[R1-aaa] local-user r2 service-type ppp
<Huawei> display interface Serial1/0/0
<Huawei> debugging ppp all
```

### 9.2 PPPoE

- 场景：家庭/企业宽带拨号上网（DSL/以太网承载 PPP）。
- 以太网类型：**发现阶段 0x8863，会话阶段 0x8864**。
- 三个阶段：
  1. **Discovery**：**PADI**（广播找服务器）→ **PADO**（服务器应答）→ **PADR**（选择）→ **PADS**（确认会话 ID）。
  2. **Session**：PPP 协商（LCP → 认证 → IPCP）+ 数据传输。
  3. **Terminate**：PADT 结束会话。
- Client/Server 模型；PPPoE Client 通常由路由器（拨号接口）担任。

```text
[R1] dialer-rule 1 ip permit
[R1] interface Dialer1
[R1-Dialer1] link-protocol ppp
[R1-Dialer1] ppp chap user user1
[R1-Dialer1] ppp chap password cipher Huawei@123
[R1-Dialer1] ip address ppp-negotiate
[R1-Dialer1] dialer user user1
[R1-Dialer1] dialer bundle 1
[R1-Dialer1] dialer-group 1
[R1-Dialer1] nat outbound 2000
[R1] interface GigabitEthernet0/0/1
[R1-GE0/0/1] pppoe-client dial-bundle-number 1
<Huawei> display dialer interface Dialer1
<Huawei> display pppoe-client session summary
```

### 9.3 NAT

**背景**：IPv4 仅 32 bit（约 43 亿），公网地址枯竭且分配不均；NAT 是过渡技术，同时提供**隐藏内网地址的安全性**；终极方案是 **IPv6**。

| 类型 | 映射关系 | 是否转换端口 | 特点 / 场景 |
|---|---|---|---|
| **静态 NAT** | 私有 ↔ 公有 **一对一固定** | 否 | 支持**双向互访**（外网可访问内网服务器）；不节省地址 |
| **动态 NAT（No-PAT）** | 从地址池**临时**分配，一对一 | **否** | 先到先得，会话结束释放；仍不节省地址 |
| **NAPT（地址端口转换）** | 多对一（**公有 IP + 端口**） | **是** | 大量内网主机共享少量公网 IP，**最常用** |
| **Easy IP** | 多对一，直接用**出接口地址** | 是 | 适用**动态获取公网地址**（DHCP/PPPoE 拨号） |
| **NAT Server** | 公网 `IP:端口` ↔ 私网 `IP:端口` | 端口映射 | 让外网主动访问内网服务器（端口映射） |

- NAT 转换表项有时效性，长时间不用会删除。
- 转换方向：出方向改**源 IP**（私→公），回包改**目的 IP**（公→私）。

```text
# 静态 NAT（两种写法）
[R1] nat static global 122.1.2.3 inside 192.168.1.3
[R1-GE0/0/1] nat static enable
# 动态 NAT（No-PAT，需地址池）
[R1] nat address-group 1 122.1.2.1 122.1.2.2
[R1] acl 2000
[R1-acl-basic-2000] rule permit source 192.168.1.0 0.0.0.255
[R1-GE0/0/1] nat outbound 2000 address-group 1 no-pat
# NAPT（缺省就转换端口）
[R1-GE0/0/1] nat outbound 2000 address-group 1
# Easy IP
[R1-GE0/0/1] nat outbound 2000
# NAT Server
[R1-GE0/0/1] nat server protocol tcp global 122.1.2.253 80 inside 192.168.1.10 80
[R1-GE0/0/1] nat server global 122.1.2.253 inside 192.168.1.10
<Huawei> display nat static
<Huawei> display nat address-group 1
<Huawei> display nat outbound
<Huawei> display nat server
<Huawei> display nat session all
```

**易错点**
- **NAPT 与 Easy IP 的差别只在"用不用地址池"**（Easy IP 用接口地址）。
- **动态 NAT 不转换端口**，所以地址池里有几个地址，就最多几台主机同时上网。
- 静态 NAT 与 NAT Server 都支持外网主动访问，但 NAT Server 可只映射某个端口。

---
