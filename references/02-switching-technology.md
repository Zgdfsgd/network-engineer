# 交换技术全解

## 一、VLAN原理与配置

参考来源：IEEE 802.1Q标准、华为/H3C/思科官方配置手册

### VLAN核心概念

- **VLAN（虚拟局域网）**：将物理网络逻辑划分为多个广播域，隔离流量、提升安全
- **VLAN标签**：802.1Q在以太网帧中插入4字节Tag，其中12位VLAN ID（范围1-4094）
- **默认VLAN**：所有端口默认属于VLAN 1

### 端口类型

| 端口类型 | 说明 | 适用场景 |
|---------|------|---------|
| Access | 仅属于单个VLAN，收发不带Tag的帧 | 连接终端设备（PC/打印机） |
| Trunk | 承载多个VLAN流量，帧带Tag | 交换机互联 |
| Hybrid | 可灵活配置Tag/Untag | 华为/H3C特有，灵活场景 |

### 华为/H3C VLAN配置

```
# 创建VLAN
vlan batch 10 20 30

# Access端口配置
interface GigabitEthernet 0/0/1
 port link-type access
 port default vlan 10

# Trunk端口配置
interface GigabitEthernet 0/0/24
 port link-type trunk
 port trunk allow-pass vlan 10 20 30

# Hybrid端口配置
interface GigabitEthernet 0/0/2
 port link-type hybrid
 port hybrid tagged vlan 10 20
 port hybrid untagged vlan 30
 port hybrid pvid vlan 30
```

### 思科IOS VLAN配置

```
# 创建VLAN
vlan 10
 name Office_VLAN10

# Access端口
interface FastEthernet0/1
 switchport mode access
 switchport access vlan 10

# Trunk端口
interface FastEthernet0/24
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30
```

### 常见报错

- Trunk两端端口模式不一致→跨交换机同VLAN不通
- Access端口接了Trunk对端→VLAN标签丢失
- VLAN未创建就配置端口→配置无效

## 二、STP生成树协议

参考来源：IEEE 802.1D(STP)、802.1w(RSTP)、802.1s(MSTP)

### STP核心作用

解决二层环路、防止广播风暴、避免MAC地址表震荡。

### STP端口状态

| 状态 | 说明 | 转发数据 | 学习MAC |
|------|------|---------|---------|
| Disabled | 端口关闭 | 否 | 否 |
| Blocking | 阻塞状态，仅接收BPDU | 否 | 否 |
| Listening | 侦听BPDU，选举根桥 | 否 | 否 |
| Learning | 学习MAC地址 | 否 | 是 |
| Forwarding | 正常转发 | 是 | 是 |

### STP选举规则

1. **根桥选举**：Bridge ID最小的交换机（优先级+MAC地址，优先级默认32768）
2. **根端口选举**：非根桥到根桥路径开销最小的端口
3. **指定端口选举**：每个网段到根桥开销最小的端口
4. **阻塞端口**：既非根端口也非指定端口的端口被阻塞

### RSTP改进（802.1w）

- 端口状态简化为3种：Discarding/Learning/Forwarding
- 收敛速度从50秒提升到秒级
- 引入边缘端口（Edge Port）概念，连接终端的端口可立即进入转发

### MSTP改进（802.1s）

- 支持多实例，不同VLAN可走不同路径
- 实现负载均衡

### 华为/H3C STP配置

```
# 启用STP
stp enable

# 配置STP模式
stp mode rstp

# 配置根桥（方法一：直接指定）
stp root primary

# 配置根桥（方法二：调整优先级）
stp priority 0

# 配置边缘端口
interface GigabitEthernet 0/0/1
 stp edged-port enable
```

### 思科STP配置

```
# 启用RSTP
spanning-tree mode rapid-pvst

# 配置根桥
spanning-tree vlan 10 root primary

# 配置边缘端口
interface FastEthernet0/1
 spanning-tree portfast
```

### 实战提示

- 二层环路是企业内网高频故障，优先检查STP状态
- 边缘端口必须配置，否则终端接入会触发TCN BPDU风暴
- 使用`display stp brief`(华为)或`show spanning-tree summary`(思科)快速查看STP状态

## 三、LACP端口聚合

参考来源：IEEE 802.3ad

### 核心概念

- 将多个物理端口捆绑为一个逻辑端口，提升带宽、提供链路冗余
- LACP模式：主动(Active)/被动(Passive)，至少一端为Active才能协商

### 华为/H3C LACP配置

```
# 创建聚合组
interface Eth-Trunk 1
 mode lacp-static

# 添加成员端口
interface GigabitEthernet 0/0/1
 eth-trunk 1
interface GigabitEthernet 0/0/2
 eth-trunk 1

# 配置LACP优先级（指定主动端）
lacp priority 0

# 配置最大活动端口数
interface Eth-Trunk 1
 max active-linknumber 2
```

### 思科LACP配置

```
# 创建Port-Channel
interface range GigabitEthernet0/1-2
 channel-group 1 mode active
 channel-protocol lacp

# 配置LACP优先级
lacp system-priority 1
```

### 注意事项

- 聚合端口速率、双工、VLAN配置必须一致
- 建议先配置Eth-Trunk/Port-Channel，再加入物理端口

## 四、端口安全

### 华为/H3C端口安全配置

```
interface GigabitEthernet 0/0/1
 port-security enable
 port-security max-mac-num 2
 port-security protect-action shutdown
```

### 思科端口安全配置

```
interface FastEthernet0/1
 switchport port-security
 switchport port-security maximum 2
 switchport port-security violation shutdown
 switchport port-security mac-address sticky
```

### 保护动作对比

| 动作 | 华为/H3C | 思科 | 说明 |
|------|---------|------|------|
| 丢弃 | protect | protect | 丢弃违规帧，不告警 |
| 告警 | restrict | restrict | 丢弃违规帧，发送SNMP告警 |
| 关闭 | shutdown | shutdown | 端口关闭，需手动恢复 |

## 五、广播风暴抑制

```
# 华为/H3C
interface GigabitEthernet 0/0/1
 broadcast-suppression 5
 multicast-suppression 5
 unicast-suppression 5

# 思科
interface FastEthernet0/1
 storm-control broadcast level 5.00
 storm-control multicast level 5.00
 storm-control action shutdown
```
