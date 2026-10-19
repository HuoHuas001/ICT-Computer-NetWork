# 第7章 应用层


### 7.1 应用层概述

- OSI 的**会话层、表示层、应用层**对应 TCP/IP 的**应用层**。
- 会话层：建立/维持/释放会话（有序释放、废弃）。表示层：**数据格式转换、压缩、加密**（解决异构系统兼容）。
- 两种模式：**C/S（客户/服务器）** 与 **P2P（对等体，每台既是客户也是服务器）**。

### 7.2 DNS

| 项 | 值 |
|---|---|
| 端口 | **53**，主要 **UDP**，报文过长（TC 标志置位）或**区域传送**时用 **TCP** |
| 域名结构 | 层次树：根 → 顶级域(TLD) → 二级域 → 主机；**FQDN** 完全限定域名（末尾带 `.`） |
| 服务器类型 | **根**（全球 13 套装置）、**顶级域**、**权威**、**本地（缓存）** |
| 查询方式 | **递归**（客户端→本地 DNS，本地 DNS 负责到底）、**迭代**（本地 DNS→根/顶级/权威，逐级问） |
| 正/反向 | 正向：域名 → IP；反向：IP → 域名（`in-addr.arpa`，PTR） |

**资源记录类型**

| 类型 | 编号 | 作用 |
|---|---|---|
| **A** | 1 | 域名 → IPv4 |
| **NS** | 2 | 指定权威域名服务器 |
| **CNAME** | 5 | 别名 |
| **MX** | 15 | 邮件服务器 |
| **TXT** | 16 | 文本信息 |
| **AAAA** | 28 | 域名 → IPv6 |
| **PTR** | 12 | 反向解析 |

- 解析过程（迭代）：本机缓存/hosts → 本地 DNS → 根（返回顶级域地址）→ 顶级域（返回权威地址）→ 权威（返回结果）→ 本地 DNS 缓存并回客户端。
- 缓存：客户端内存缓存 + `C:\Windows\System32\drivers\etc\hosts` 静态文件；**DDNS** 支持动态 IP 映射固定域名。
- 测试工具：`nslookup`（可交互/非交互）。

```text
[Huawei] dns resolve
[Huawei] dns server 8.8.8.8
<Huawei> nslookup www.example.com
```

### 7.3 DHCP

| 项 | 值 |
|---|---|
| 端口 | **服务器 UDP 67，客户端 UDP 68** |
| 报文 | **Discover → Offer → Request → ACK**（DORA）；另有 NAK、Decline、Release、Inform |
| 广播/单播 | Discover 广播；Offer 可单播；**Request 必须广播**（通知其他 DHCP 服务器已选定的服务器）；ACK 单播（或广播） |
| 租期 | 缺省 1 天；**50%** 租期单播续租（Request），**87.5%** 仍无响应则广播重新申请 |

- 为什么 Request 要广播：让网络中其他 DHCP 服务器知道客户端已选择了某台服务器，释放它们预分配的地址。
- 分配顺序：已静态绑定 → 曾分配过（租期记录）→ 地址池中可用地址；冲突则发 **Decline**。
- **DHCP 中继**：客户端与服务器不在同一广播域时，由中继设备把广播报文转成单播发给服务器（`dhcp relay`）。
- **Option 43**：AC 通过 DHCP 把 AC 的 IP 告知 AP（WLAN 场景必考）。

```text
# 接口地址池
[Huawei] dhcp enable
[Huawei] interface GigabitEthernet0/0/0
[Huawei-GE0/0/0] ip address 10.1.1.1 24
[Huawei-GE0/0/0] dhcp select interface
[Huawei-GE0/0/0] dhcp server dns-list 8.8.8.8
[Huawei-GE0/0/0] dhcp server excluded-ip-address 10.1.1.1
[Huawei-GE0/0/0] dhcp server lease day 1
# 全局地址池
[Huawei] ip pool pool1
[Huawei-ip-pool-pool1] network 10.1.1.0 mask 255.255.255.0
[Huawei-ip-pool-pool1] gateway-list 10.1.1.1
[Huawei-ip-pool-pool1] dns-list 8.8.8.8
[Huawei-ip-pool-pool1] excluded-ip-address 10.1.1.1 10.1.1.10
[Huawei-ip-pool-pool1] lease day 1
[Huawei] interface GigabitEthernet0/0/0
[Huawei-GE0/0/0] dhcp select global
# 中继
[Huawei] interface GigabitEthernet0/0/1
[Huawei-GE0/0/1] dhcp select relay
[Huawei-GE0/0/1] dhcp relay server-ip 10.1.1.100
```

### 7.4 Telnet 与 SSH

| 项 | Telnet | SSH |
|---|---|---|
| 端口 | **23** | **22** |
| 安全性 | **明文**（含密码），不安全 | 加密、认证，安全 |
| 关键概念 | **NVT** 网络虚拟终端；命令用 **IAC（255）** 引导；选项协商 **WILL/DO/WONT/DONT** | 三组件：**传输层协议 / 用户认证协议 / 连接协议** |

- SSH 五阶段：① 版本号协商 ② 算法协商（密钥交换、加密、MAC、压缩）③ 密钥交换（生成会话密钥）④ 用户认证（口令 / 公钥）⑤ 会话交互。
- Telnet 认证方式：`none`、`password`、`aaa`（用户名+密码）。

```text
[R1] telnet server enable
[R1] user-interface vty 0 4
[R1-ui-vty0-4] authentication-mode { password | aaa }
[R1-ui-vty0-4] protocol inbound { telnet | ssh | all }
[R1] stelnet server enable
[R1] rsa local-key-pair create
[R1] aaa
[R1-aaa] local-user python password cipher Huawei@123
[R1-aaa] local-user python service-type ssh
[R1-aaa] local-user python privilege level 3
<Huawei> telnet 10.1.1.1
<Huawei> stelnet 10.1.1.1
```

### 7.5 HTTP 与 HTTPS

- **HTTP：80/TCP**；**HTTPS：443/TCP**（HTTP + TLS/SSL）。WWW 基于 HTML，浏览器/服务器模式。
- URL 组成：`协议://主机[:端口]/路径/文件名`，默认端口 80 可省略。
- 请求报文 = **请求行 + 请求头 + 空行 + 请求体**；响应报文 = **状态行 + 响应头 + 空行 + 响应体**。

| 方法 | 含义 |
|---|---|
| **GET** | 请求资源（参数在 URL，长度受限） |
| **POST** | 提交数据（数据在报文体） |
| **HEAD** | 只要响应头，不要响应体 |
| PUT / DELETE | 上传/删除资源 |
| OPTIONS / TRACE / CONNECT | 查询支持方法 / 回显请求 / 建立隧道（HTTPS 代理） |

| 状态码 | 含义 | 常见值 |
|---|---|---|
| 1xx | 信息（临时响应） | 100 Continue |
| 2xx | 成功 | **200 OK**、204 |
| 3xx | 重定向 | **301 永久**、302 临时、304 未修改 |
| 4xx | 客户端错误 | **400 语法错**、401 未认证、**403 禁止**、**404 找不到** |
| 5xx | 服务器错误 | **500 内部错误**、502 网关错、503 不可用 |

- **HTTPS 全流程（四阶段）**：TCP 三次握手 → **TLS 四次握手**（协商密钥/证书验证）→ 加密通信（Application Data）→ TCP 四次挥手。
- TLS 两层：**记录协议**（封装加密）+ **握手协议**（协商算法、交换密钥、证书认证）。
- TLS 三类算法：**对称加密**（数据加密，如 AES）、**非对称加密**（密钥交换/签名，如 RSA/ECC）、**哈希**（完整性，如 SHA256）。
- HTTPS 解决的问题：防窃听（加密）、防篡改（完整性校验）、防冒充（证书/身份认证）。

### 7.6 电子邮件

| 协议 | 端口 | 作用 |
|---|---|---|
| **SMTP** | **25**（加密 SMTPS **465**） | **发送**邮件、服务器间转发 |
| **POP3** | **110**（加密 POP3S **995**） | **接收**，下载到本地并（默认）删除服务器副本 |
| **IMAP** | **143**（加密 IMAPS **993**） | **接收**，邮件留在服务器、多端同步 |

- 邮件地址格式：`用户名@域名`；组成：信封/首部（From、To、Subject）+ 正文 + 附件。
- 交互均为**请求/响应**文本协议，返回码：`220 服务就绪`、`250 动作完成`、`331 需要密码`、`354 开始发送数据`、`500 语法错`、`530 未登录`、`550 操作未执行`。
- **MIME**：扩展邮件标准，支持图片/音视频/多语言；用 **Base64** 等编码把二进制转成 ASCII，首部增加 `Content-Type`、`Content-Transfer-Encoding` 等。
- Base64：3 字节 → 4 个 ASCII 字符（如 `Man` → `TWFu`）。

### 7.7 FTP 与 TFTP

| 项 | FTP | TFTP |
|---|---|---|
| 传输层 | **TCP** | **UDP** |
| 端口 | **控制连接 21，数据连接 20（主动模式）** | **69** |
| 连接数 | **两个连接**（控制连接全程保持；数据连接每次传输临时建立） | 单连接 |
| 认证 | 用户名 + 密码 | **无认证** |
| 用途 | 大文件、可靠传输 | 小文件（设备升级/配置备份） |

- **主动模式（PORT）**：客户端随机端口 → 服务器 21 建控制连接；客户端用 PORT 命令告知自己的监听端口，**服务器用 20 端口主动连客户端**。
- **被动模式（PASV）**：客户端发 PASV，服务器回一个随机端口，**由客户端主动连服务器**（穿越 NAT/防火墙更友好）。
- 传输模式：**ASCII**（文本，做编码转换）与 **Binary**（二进制，原样传输，**传设备版本文件用 Binary**）。
- 常见响应码：`220 服务就绪`、`331 需要密码`、`230 登录成功`、`150 打开数据连接`、`226 传输完成`、`530 未登录`、`550 操作失败`。

**TFTP 五种报文**：RRQ（读请求）、WRQ（写请求）、DATA、ACK、ERROR。

```text
[R1] ftp server enable
[R1] aaa
[R1-aaa] local-user ftpuser password cipher Huawei@123
[R1-aaa] local-user ftpuser service-type ftp
[R1-aaa] local-user ftpuser ftp-directory flash:
<Huawei> ftp 10.1.1.1
[Huawei-ftp] get vrpcfg.zip
[Huawei-ftp] put ar2220.cc
<Huawei> tftp 10.1.2.2 get ar2220.cc
<Huawei> tftp 10.1.2.2 put vrpcfg.zip
```

### 7.8 NTP

- **UDP 123**；用于全网设备时钟同步（日志、计费、证书、故障定位都依赖时间一致）。
- 4 个时间戳：**T1** 客户端发送、**T2** 服务器收到、**T3** 服务器回复、**T4** 客户端收到。
  - **Offset（时钟偏差）= [(T2 − T1) + (T3 − T4)] / 2**
  - **Delay（往返时延）= (T4 − T1) − (T3 − T2)**
- 工作模式：**单播服务器/客户端**、**对等体（Symmetric）**、**广播**、**组播**。
- 网络结构：主时间服务器（stratum 1，接 GPS/原子钟）→ 二级时间服务器 → 客户端；stratum 越小越权威。

```text
[R1] ntp-service refclock-master 2                  # 本地时钟作主时钟
[R2] ntp-service unicast-server 10.1.2.2            # 指定 NTP 服务器
<Huawei> display ntp-service status
<Huawei> display ntp-service sessions
```

---
