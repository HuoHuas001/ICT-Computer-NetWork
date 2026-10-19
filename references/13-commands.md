# 附录B VRP 命令速查

> 约定：`<Huawei>` = 用户视图，`[Huawei]` = 系统视图，`[Huawei-GE0/0/0]` = 接口视图。
> 所有 `display`/`save`/`reboot` 等在日常排障与做题中最常出现，务必分清**用户视图 vs 系统视图**。

## B.1 基础运维

```text
<Huawei> system-view                       # 进入系统视图
[Huawei] sysname AR1
[Huawei] interface GigabitEthernet0/0/0
[Huawei-GE0/0/0] ip address 10.1.1.1 24
[Huawei-GE0/0/0] shutdown / undo shutdown
<Huawei> display current-configuration     # 当前运行配置
<Huawei> display saved-configuration
<Huawei> save
<Huawei> reset saved-configuration
<Huawei> display startup
<Huawei> reboot
[Huawei] header login information "Welcome"
[Huawei] user-interface vty 0 4
[Huawei-ui-vty0-4] authentication-mode { none | password | aaa }
[Huawei-ui-vty0-4] set authentication password cipher Huawei@123
[Huawei-ui-vty0-4] user privilege level { 0 - 15 }
[Huawei-ui-vty0-4] protocol inbound { all | telnet | ssh }
[Huawei] idle-timeout 10
[Huawei] clock timezone BJ add 08:00:00
<Huawei> clock datetime 09:00:00 2024-01-01
```

## B.2 VLAN / Trunk / Hybrid

```text
[SW1] vlan batch 10 20 30
[SW1] vlan 10
[SW1] interface GigabitEthernet0/0/1
[SW1-GE0/0/1] port link-type { access | trunk | hybrid }
[SW1-GE0/0/1] port default vlan 10                     # Access
[SW1-GE0/0/2] port trunk allow-pass vlan { 10 20 | all }
[SW1-GE0/0/2] port trunk pvid vlan 10
[SW1-GE0/0/3] port hybrid pvid vlan 10
[SW1-GE0/0/3] port hybrid untagged vlan 10 20
[SW1-GE0/0/3] port hybrid tagged vlan 30
[SW1] display vlan [verbose]
[SW1] display port vlan
# 基于 MAC 的 VLAN
[SW1-vlan10] mac-vlan mac-address 00e0-fc01-0001
[SW1-GE0/0/1] mac-vlan enable
[SW1] display mac-vlan all
```

## B.3 STP / RSTP / MSTP

```text
[SW1] stp mode { stp | rstp | mstp }
[SW1] stp enable
[SW1] stp root { primary | secondary }
[SW1] stp priority 4096
[SW1-GE0/0/1] stp cost 20000
[SW1-GE0/0/1] stp priority 16
[SW1-GE0/0/1] stp edged-port enable
[SW1] stp pathcost-standard { dot1d-1998 | dot1t | legacy }
[SW1] stp region-configuration
[SW1-mst-region] region-name huawei
[SW1-mst-region] instance 1 vlan 10 20
[SW1-mst-region] revision-level 1
[SW1-mst-region] active region-configuration
[SW1] stp instance 1 root primary
<SW1> display stp [instance 1] [brief]
<SW1> display stp region-configuration
```

## B.4 链路聚合

```text
[SW1] interface eth-trunk 1
[SW1-Eth-Trunk1] mode { lacp | manual load-balance }
[SW1-Eth-Trunk1] trunkport gigabitethernet 0/0/1 to 0/0/3
[SW1-GE0/0/1] eth-trunk 1
[SW1-Eth-Trunk1] max active-linknumber 2
[SW1-Eth-Trunk1] least active-linknumber 2
[SW1-Eth-Trunk1] load-balance { src-ip | dst-ip | src-dst-ip | src-mac | dst-mac | src-dst-mac }
[SW1-Eth-Trunk1] mixed-rate link enable
[SW1] lacp priority 30000
[SW1-GE0/0/1] lacp priority 100
[SW1] lacp preempt enable
<SW1> display eth-trunk 1
<SW1> display lacp statistics eth-trunk 1
```

## B.5 静态路由 / 默认路由 / 浮动路由

```text
[Huawei] ip route-static 20.1.1.0 24 10.0.0.2
[Huawei] ip route-static 20.1.1.0 24 Serial1/0/0 10.0.0.2
[Huawei] ip route-static 0.0.0.0 0 10.0.0.2
[Huawei] ip route-static 20.1.1.0 24 10.0.0.2 preference 100
[Huawei] ip route-static 20.1.1.0 24 NULL0
<Huawei> display ip routing-table [protocol static | verbose]
<Huawei> display ip routing-table 20.1.1.1
```

## B.6 RIP

```text
[Huawei] rip 1
[Huawei-rip-1] version 2
[Huawei-rip-1] network 10.0.0.0
[Huawei-rip-1] summary / undo summary
[Huawei-rip-1] silent-interface GigabitEthernet0/0/1
[Huawei-GE0/0/1] rip summary-address 10.1.0.0 255.255.0.0
[Huawei-GE0/0/1] rip split-horizon
[Huawei-GE0/0/1] rip poison-reverse
[Huawei-GE0/0/1] rip authentication-mode md5 usual cipher Huawei@123
<Huawei> display rip 1 route / neighbor / database
```

## B.7 OSPF / OSPFv3

```text
[Huawei] ospf 1 router-id 1.1.1.1
[Huawei-ospf-1] area 0
[Huawei-ospf-1-area-0.0.0.0] network 10.0.0.0 0.0.0.255
[Huawei-ospf-1] bandwidth-reference 10000
[Huawei-GE0/0/0] ospf cost 10
[Huawei-GE0/0/0] ospf dr-priority 100
[Huawei-GE0/0/0] ospf network-type { broadcast | p2p | p2mp | nbma }
[Huawei-ospf-1-area-0.0.0.1] stub [ no-summary ]
[Huawei-ospf-1-area-0.0.0.1] nssa
[Huawei-ospf-1] import-route { direct | static | rip 1 } [ cost x ]
[Huawei-ospf-1] default-route-advertise [ always ]
[Huawei-ospf-1-area-0.0.0.1] abr-summary 10.1.0.0 255.255.0.0
[Huawei-ospf-1] vlink-peer 3.3.3.3
<Huawei> display ospf peer [brief]
<Huawei> display ospf lsdb [router|network|summary|asbr|ase]
<Huawei> display ospf routing
<Huawei> display ospf interface
<Huawei> display ospf error
# OSPFv3
[Huawei] ospfv3 1
[Huawei-ospfv3-1] router-id 1.1.1.1
[Huawei-GE0/0/0] ospfv3 1 area 0
<Huawei> display ospfv3 peer / lsdb / routing / interface
```

## B.8 VLAN 间路由

```text
# 单臂路由（子接口）
[R1] interface GigabitEthernet0/0/1.10
[R1-GE0/0/1.10] dot1q termination vid 10
[R1-GE0/0/1.10] ip address 192.168.10.254 24
[R1-GE0/0/1.10] arp broadcast enable
# 三层交换机 VLANIF
[SW1] interface Vlanif 10
[SW1-Vlanif10] ip address 192.168.10.254 24
```

## B.9 DHCP

```text
[Huawei] dhcp enable
# 接口地址池
[Huawei-GE0/0/0] dhcp select interface
[Huawei-GE0/0/0] dhcp server dns-list 8.8.8.8
[Huawei-GE0/0/0] dhcp server excluded-ip-address 10.1.1.1
[Huawei-GE0/0/0] dhcp server lease day 1
# 全局地址池
[Huawei] ip pool pool1
[Huawei-ip-pool-pool1] network 10.1.1.0 mask 255.255.255.0
[Huawei-ip-pool-pool1] gateway-list 10.1.1.1
[Huawei-ip-pool-pool1] dns-list 8.8.8.8
[Huawei-ip-pool-pool1] lease day 1
[Huawei-GE0/0/0] dhcp select global
# 中继
[Huawei-GE0/0/1] dhcp select relay
[Huawei-GE0/0/1] dhcp relay server-ip 10.1.1.100
# Option 43（WLAN 场景，通知 AP 的 AC 地址）
[Huawei-ip-pool-pool1] option 43 sub-option 2 ip-address 10.100.100.1
<Huawei> display ip pool
<Huawei> display dhcp server statistics
```

## B.10 NAT

```text
[R1] nat static global 122.1.2.3 inside 192.168.1.3
[R1-GE0/0/1] nat static enable
[R1] nat address-group 1 122.1.2.1 122.1.2.2
[R1] acl 2000
[R1-acl-basic-2000] rule permit source 192.168.1.0 0.0.0.255
[R1-GE0/0/1] nat outbound 2000 address-group 1 no-pat    # 动态 NAT
[R1-GE0/0/1] nat outbound 2000 address-group 1           # NAPT
[R1-GE0/0/1] nat outbound 2000                           # Easy IP
[R1-GE0/0/1] nat server protocol tcp global 122.1.2.253 80 inside 192.168.1.10 80
<Huawei> display nat static / address-group / outbound / server / session all
```

## B.11 ACL / AAA / RADIUS

```text
[Huawei] acl number { 2000-2999 | 3000-3999 }
[Huawei-acl-basic-2000] rule 5 permit source 10.1.1.0 0.0.0.255
[Huawei-acl-adv-3001] rule 5 deny tcp source 10.1.2.0 0.0.0.255 destination 10.1.3.0 0.0.0.255 destination-port eq 21
[Huawei-acl-adv-3001] rule 10 permit ip
[Huawei] time-range work 08:30 to 21:00 daily
[Huawei] interface GigabitEthernet0/0/1
[Huawei-GE0/0/1] traffic-filter { inbound | outbound } acl 3001
<Huawei> display acl all
# AAA
[Huawei] radius-server template rd1
[Huawei-radius-rd1] radius-server authentication 10.1.1.100 1812
[Huawei-radius-rd1] radius-server accounting 10.1.1.100 1813
[Huawei-radius-rd1] radius-server shared-key cipher Huawei@123
[Huawei] aaa
[Huawei-aaa] authentication-scheme sch1
[Huawei-aaa-authen-sch1] authentication-mode { local | radius | hwtacacs } [ local ]
[Huawei-aaa] accounting-scheme acc1
[Huawei-aaa-accounting-acc1] accounting-mode radius
[Huawei-aaa] domain huawei.com
[Huawei-aaa-domain-huawei.com] authentication-scheme sch1
[Huawei-aaa-domain-huawei.com] radius-server rd1
[Huawei-aaa] local-user user1 password cipher Huawei@123
[Huawei-aaa] local-user user1 service-type { telnet | ssh | ftp | ppp }
[Huawei-aaa] local-user user1 privilege level 3
<Huawei> display domain name huawei.com
```

## B.12 PPP / PPPoE

```text
[R1-Serial1/0/0] link-protocol ppp
[R1-Serial1/0/0] ppp authentication-mode { pap | chap }
[R1-Serial1/0/0] ppp pap local-user r2 password cipher Huawei@123
[R1-Serial1/0/0] ppp chap user r1
[R1-Serial1/0/0] ppp chap password cipher Huawei@123
[R1-Serial1/0/0] ip address ppp-negotiate
[R1] aaa
[R1-aaa] local-user r2 password cipher Huawei@123
[R1-aaa] local-user r2 service-type ppp
<Huawei> display interface Serial1/0/0
<Huawei> debugging ppp all
# PPPoE 客户端
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
[R1-GE0/0/1] pppoe-client dial-bundle-number 1
<Huawei> display dialer interface Dialer1
```

## B.13 WLAN（AC 侧）

```text
[AC] vlan batch 100 101
[AC] interface Vlanif 100
[AC-Vlanif100] ip address 10.100.100.1 24
[AC] capwap source interface Vlanif100
[AC] wlan
[AC-wlan-view] regulatory-domain-profile name domain
[AC-wlan-regulate-domain-profile-domain] country-code cn
[AC-wlan-view] ap-group name ap-group1
[AC-wlan-ap-group-ap-group1] regulatory-domain-profile domain
[AC-wlan-view] ap-id 0 ap-mac 00e0-fc0a-1234
[AC-wlan-ap-0] ap-name ap1
[AC-wlan-ap-0] ap-group ap-group1
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
[AC] display ap all
[AC] display vap all / display vap ssid employee
[AC] display station all
[AC] display ap-group name ap-group1
```

## B.14 应用层服务

```text
[Huawei] ftp server enable
[Huawei] aaa
[Huawei-aaa] local-user ftpuser password cipher Huawei@123
[Huawei-aaa] local-user ftpuser service-type ftp
[Huawei-aaa] local-user ftpuser ftp-directory flash:
[Huawei] telnet server enable
[Huawei] stelnet server enable
[Huawei] rsa local-key-pair create
[Huawei] dns resolve
[Huawei] dns server 8.8.8.8
[Huawei] ntp-service refclock-master 2
[Huawei] ntp-service unicast-server 10.1.2.2
[Huawei] snmp-agent
[Huawei] snmp-agent sys-info version v2c
[Huawei] snmp-agent community read cipher Huawei123
[Huawei] snmp-agent trap enable
[Huawei] snmp-agent target-host trap address udp-domain 192.168.27.10 params securityname Huawei123 v2c
<Huawei> display ntp-service status
<Huawei> display snmp-agent community
```

## B.15 排障命令（高频）

```text
<Huawei> ping [-a 源IP] [-c 次数] [-s 包长] 目的IP
<Huawei> tracert 目的IP
<Huawei> display ip interface brief
<Huawei> display interface GigabitEthernet0/0/0
<Huawei> display arp all
<Huawei> display mac-address
<Huawei> display ip routing-table
<Huawei> display ip routing-table protocol ospf
<Huawei> display ospf peer brief
<Huawei> display tcp status
<Huawei> display cpu-usage
<Huawei> display memory-usage
<Huawei> display logbuffer
<Huawei> terminal monitor / terminal debugging
<Huawei> debugging ip packet
<Huawei> reset counters interface
```
