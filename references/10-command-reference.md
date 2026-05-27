# 三厂商命令速查对照表

参考来源：华为VRP官方命令参考、H3C Comware官方命令参考、思科IOS命令参考

---

## 一、系统基础命令

| 功能 | 华为VRP | H3C Comware | 思科IOS |
|------|---------|------------|---------|
| 进入系统视图 | system-view | system-view | configure terminal |
| 退出当前视图 | quit | quit | exit |
| 查看当前配置 | display current-configuration | display current-configuration | show running-config |
| 保存配置 | save | save | write memory |
| 查看版本信息 | display version | display version | show version |
| 查看时钟 | display clock | display clock | show clock |
| 设置主机名 | sysname SW1 | sysname SW1 | hostname SW1 |

## 二、接口配置命令

| 功能 | 华为VRP | H3C Comware | 思科IOS |
|------|---------|------------|---------|
| 进入接口 | interface GE0/0/1 | interface GE1/0/1 | interface G0/1 |
| 配置IP地址 | ip address 192.168.1.1 24 | ip address 192.168.1.1 24 | ip address 192.168.1.1 255.255.255.0 |
| 启用接口 | undo shutdown | undo shutdown | no shutdown |
| 关闭接口 | shutdown | shutdown | shutdown |
| 查看接口状态 | display interface brief | display interface brief | show ip interface brief |
| 查看接口详情 | display interface GE0/0/1 | display interface GE1/0/1 | show interface G0/1 |
| 配置接口描述 | description To-Core-SW | description To-Core-SW | description To-Core-SW |

## 三、VLAN配置命令

| 功能 | 华为VRP | H3C Comware | 思科IOS |
|------|---------|------------|---------|
| 创建VLAN | vlan 10 | vlan 10 | vlan 10 |
| 批量创建VLAN | vlan batch 10 20 30 | vlan 10 20 30 | vlan 10,20,30 |
| VLAN命名 | name Office | name Office | name Office |
| 删除VLAN | undo vlan 10 | undo vlan 10 | no vlan 10 |
| 查看VLAN | display vlan | display vlan | show vlan |
| Access端口 | port link-type access + port default vlan 10 | port link-type access + port access vlan 10 | switchport mode access + switchport access vlan 10 |
| Trunk端口 | port link-type trunk + port trunk allow-pass vlan 10 20 | port link-type trunk + port trunk permit vlan 10 20 | switchport mode trunk + switchport trunk allowed vlan 10,20 |
| Trunk允许所有VLAN | port trunk allow-pass vlan all | port trunk permit vlan all | switchport trunk allowed vlan all |

## 四、STP配置命令

| 功能 | 华为VRP | H3C Comware | 思科IOS |
|------|---------|------------|---------|
| 启用STP | stp enable | stp enable | spanning-tree vlan 10 |
| 配置STP模式 | stp mode rstp | stp mode rstp | spanning-tree mode rapid-pvst |
| 配置根桥 | stp root primary | stp root primary | spanning-tree vlan 10 root primary |
| 配置优先级 | stp priority 0 | stp priority 0 | spanning-tree vlan 10 priority 0 |
| 配置边缘端口 | stp edged-port enable | stp edged-port | spanning-tree portfast |
| 查看STP状态 | display stp brief | display stp brief | show spanning-tree summary |
| 查看STP端口 | display stp interface GE0/0/1 | display stp interface GE1/0/1 | show spanning-tree interface G0/1 |

## 五、链路聚合命令

| 功能 | 华为VRP | H3C Comware | 思科IOS |
|------|---------|------------|---------|
| 创建聚合接口 | interface Eth-Trunk 1 | interface Bridge-Aggregation 1 | interface Port-Channel 1 |
| 配置LACP模式 | mode lacp-static | link-aggregation mode dynamic | channel-protocol lacp + channel-group 1 mode active |
| 添加成员端口 | eth-trunk 1 | port link-aggregation group 1 | channel-group 1 mode active |
| 查看聚合状态 | display eth-trunk 1 | display link-aggregation summary | show etherchannel summary |

## 六、路由配置命令

| 功能 | 华为VRP | H3C Comware | 思科IOS |
|------|---------|------------|---------|
| 静态路由 | ip route-static 192.168.2.0 24 192.168.1.2 | ip route-static 192.168.2.0 24 192.168.1.2 | ip route 192.168.2.0 255.255.255.0 192.168.1.2 |
| 默认路由 | ip route-static 0.0.0.0 0.0.0.0 192.168.1.2 | ip route-static 0.0.0.0 0.0.0.0 192.168.1.2 | ip route 0.0.0.0 0.0.0.0 192.168.1.2 |
| 查看路由表 | display ip routing-table | display ip routing-table | show ip route |
| 查看OSPF邻居 | display ospf peer brief | display ospf peer | show ip ospf neighbor |
| 查看BGP邻居 | display bgp peer | display bgp peer | show ip bgp summary |
| 查看BGP路由 | display bgp routing-table | display bgp routing-table | show ip bgp |

## 七、OSPF配置命令

| 功能 | 华为VRP | H3C Comware | 思科IOS |
|------|---------|------------|---------|
| 启用OSPF | ospf 1 router-id 1.1.1.1 | ospf 1 router-id 1.1.1.1 | router ospf 1 |
| 宣告网段 | network 192.168.1.0 0.0.0.255 | network 192.168.1.0 0.0.0.255 | network 192.168.1.0 0.0.0.255 area 0 |
| 配置区域 | area 0 | area 0 | area 0 |
| 配置Stub区域 | stub | stub | area 1 stub |
| 配置NSSA | nssa | nssa | area 1 nssa |
| 配置开销 | ospf cost 10 | ospf cost 10 | ip ospf cost 10 |

## 八、BGP配置命令

| 功能 | 华为VRP | H3C Comware | 思科IOS |
|------|---------|------------|---------|
| 启用BGP | bgp 65001 | bgp 65001 | router bgp 65001 |
| 配置邻居 | peer 10.1.1.2 as-number 65002 | peer 10.1.1.2 as-number 65002 | neighbor 10.1.1.2 remote-as 65002 |
| 宣告网段 | network 192.168.1.0 255.255.255.0 | network 192.168.1.0 24 | network 192.168.1.0 mask 255.255.255.0 |
| 配置更新源 | peer 10.1.1.2 connect-interface LoopBack0 | peer 10.1.1.2 connect-interface LoopBack0 | neighbor 10.1.1.2 update-source Loopback0 |
| EBGP多跳 | peer 10.1.1.2 ebgp-max-hop 2 | peer 10.1.1.2 ebgp-max-hop 2 | neighbor 10.1.1.2 ebgp-multihop 2 |

## 九、安全配置命令

| 功能 | 华为VRP | H3C Comware | 思科IOS |
|------|---------|------------|---------|
| 创建ACL | acl number 3000 | acl number 3000 | ip access-list extended ACL_NAME |
| 允许规则 | rule permit tcp source 192.168.1.0 0.0.0.255 | rule permit tcp source 192.168.1.0 0.0.0.255 | permit tcp 192.168.1.0 0.0.0.255 |
| 拒绝规则 | rule deny ip source any | rule deny ip source any | deny ip any any |
| 应用ACL(入) | traffic-filter inbound acl 3000 | packet-filter 3000 inbound | ip access-group ACL_NAME in |
| 应用ACL(出) | traffic-filter outbound acl 3000 | packet-filter 3000 outbound | ip access-group ACL_NAME out |
| 查看ACL | display acl 3000 | display acl 3000 | show access-lists |

## 十、NAT配置命令

| 功能 | 华为VRP | H3C Comware | 思科IOS |
|------|---------|------------|---------|
| NAT地址组 | nat address-group 1 202.1.1.10 202.1.1.20 | nat address-group 1 202.1.1.10 202.1.1.20 | ip nat pool POOL1 202.1.1.10 202.1.1.20 |
| 动态NAT | nat outbound 2000 address-group 1 | nat outbound 2000 address-group 1 | ip nat inside source list 1 pool POOL1 overload |
| 静态NAT | nat server protocol tcp global 202.1.1.10 80 inside 192.168.1.10 80 | nat server protocol tcp global 202.1.1.10 80 inside 192.168.1.10 80 | ip nat inside source static tcp 192.168.1.10 80 202.1.1.10 80 |
| NAT内部接口 | - | - | ip nat inside |
| NAT外部接口 | - | - | ip nat outside |

## 十一、VRRP配置命令

| 功能 | 华为VRP | H3C Comware | 思科IOS |
|------|---------|------------|---------|
| 配置VRRP | vrrp vrid 10 virtual-ip 192.168.10.1 | vrrp vrid 10 virtual-ip 192.168.10.1 | standby 10 ip 192.168.10.1 |
| 配置优先级 | vrrp vrid 10 priority 120 | vrrp vrid 10 priority 120 | standby 10 priority 120 |
| 配置抢占 | vrrp vrid 10 preempt-mode timer delay 20 | vrrp vrid 10 preempt-mode timer delay 20 | standby 10 preempt |
| 查看VRRP | display vrrp brief | display vrrp brief | show standby brief |

## 十二、诊断命令

| 功能 | 华为VRP | H3C Comware | 思科IOS |
|------|---------|------------|---------|
| Ping | ping 192.168.1.1 | ping 192.168.1.1 | ping 192.168.1.1 |
| Traceroute | tracert 192.168.1.1 | tracert 192.168.1.1 | traceroute 192.168.1.1 |
| 查看ARP表 | display arp | display arp | show arp |
| 查看MAC表 | display mac-address | display mac-address | show mac address-table |
| 查看CPU | display cpu-usage | display cpu-usage | show processes cpu |
| 查看内存 | display memory-usage | display memory-usage | show processes memory |
| 查看日志 | display logbuffer | display logbuffer | show logging |
| 查看告警 | display alarm active | display alarm active | show alarms |
| 抓包 | capture-packet interface GE0/0/1 | capture-packet interface GE1/0/1 | (需配置SPAN) |
| Telnet测试 | - | - | telnet 192.168.1.1 80 |

## 十三、设备管理命令

| 功能 | 华为VRP | H3C Comware | 思科IOS |
|------|---------|------------|---------|
| 配置SSH | stelnet server enable | ssh server enable | ip domain-name + crypto key generate rsa |
| 配置用户 | local-user admin password cipher XXX | local-user admin password cipher XXX | username admin secret XXX |
| 配置用户权限 | local-user admin privilege level 15 | local-user admin authorization-attribute level 3 | username admin privilege 15 |
| 配置VTY | user-interface vty 0 4 | user-interface vty 0 4 | line vty 0 4 |
| 配置认证模式 | authentication-mode aaa | authentication-mode scheme | login local |
