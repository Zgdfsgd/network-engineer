# 路由技术全解

## 一、静态路由与默认路由

参考来源：华为/H3C/思科官方配置手册

### 静态路由

适用于小型网络、网络结构固定、不需要动态学习的场景。

华为/H3C：
```
ip route-static 192.168.2.0 255.255.255.0 192.168.1.2
```

思科：
```
ip route 192.168.2.0 255.255.255.0 192.168.1.2
```

### 默认路由

所有未知流量转发至下一跳，适用于出口路由。

华为/H3C：
```
ip route-static 0.0.0.0 0.0.0.0 192.168.1.2
```

思科：
```
ip route 0.0.0.0 0.0.0.0 192.168.1.2
```

### 浮动静态路由（备份路由）

通过调整优先级实现主备切换：

华为/H3C：
```
ip route-static 0.0.0.0 0.0.0.0 192.168.1.2 preference 60
ip route-static 0.0.0.0 0.0.0.0 192.168.1.6 preference 100
```

思科：
```
ip route 0.0.0.0 0.0.0.0 192.168.1.2
ip route 0.0.0.0 0.0.0.0 192.168.1.6 10
```

## 二、RIP路由协议

参考来源：RFC 1058(RIPv1)、RFC 2453(RIPv2)

### 协议特征

| 特性 | RIPv1 | RIPv2 |
|------|-------|-------|
| 更新方式 | 广播 | 组播(224.0.0.9) |
| 子网掩码 | 不携带 | 携带 |
| VLSM支持 | 不支持 | 支持 |
| 认证 | 无 | 明文/MD5 |
| 最大跳数 | 15（16不可达） | 15（16不可达） |

### 华为/H3C RIP配置

```
rip 1
 version 2
 network 192.168.1.0
 network 10.0.0.0
 undo summary
```

### 思科RIP配置

```
router rip
 version 2
 network 192.168.1.0
 network 10.0.0.0
 no auto-summary
```

### RIP防环机制

- 最大跳数限制（16跳不可达）
- 水平分割（Split Horizon）
- 毒性反转（Poison Reverse）
- 触发更新（Triggered Update）

## 三、OSPF路由协议（主流企业协议）

参考来源：RFC 2328(OSPFv2)、RFC 5340(OSPFv3)

### 协议特征

- 链路状态协议，无跳数限制
- 适用于中大型网络
- 划分区域，骨干区域必须为Area 0
- 收敛速度快，支持VLSM/CIDR

### 核心概念

| 概念 | 说明 |
|------|------|
| Router ID | OSPF路由器唯一标识，建议手动指定 |
| Area | 区域划分，减少LSA泛洪范围 |
| DR/BDR | 广播网络中选举指定路由器，减少邻接关系 |
| ABR | 区域边界路由器，连接不同区域 |
| ASBR | 自治系统边界路由器，引入外部路由 |
| LSA类型 | Router LSA(1)/Network LSA(2)/Summary LSA(3/4)/AS External LSA(5)/NSSA LSA(7) |

### OSPF区域类型

| 类型 | 说明 | 允许的LSA |
|------|------|----------|
| 骨干区域(Area 0) | 必须存在，所有区域必须与Area 0直连或虚连接 | 所有类型 |
| 普通区域 | 标准区域 | 所有类型 |
| Stub区域 | 不接收外部路由(LSA 5)，用默认路由替代 | 1/2/3 |
| Totally Stub | 仅保留默认路由，不接收区域间和外部路由 | 1/2 |
| NSSA | 允许引入少量外部路由(LSA 7) | 1/2/3/7 |

### 华为/H3C OSPF配置

```
ospf 1 router-id 1.1.1.1
 area 0
  network 192.168.1.0 0.0.0.255
 area 1
  network 10.0.0.0 0.0.0.255
```

### 思科OSPF配置

```
router ospf 1
 router-id 1.1.1.1
 network 192.168.1.0 0.0.0.255 area 0
 network 10.0.0.0 0.0.0.255 area 1
```

### OSPF邻居状态机

| 状态 | 说明 |
|------|------|
| Down | 未收到Hello包 |
| Init | 收到对方Hello包但未看到自己的Router ID |
| 2-Way | 双向通信建立，广播网络选举DR/BDR |
| ExStart | 协商主从关系，确定DD序列号 |
| Exchange | 交换DD报文（LSA摘要） |
| Loading | 请求和更新LSA |
| Full | 邻接关系完全建立 |

### OSPF常见故障

- 邻居无法建立：检查Hello/Dead时间、区域ID、MTU、认证、子网掩码
- 路由不收敛：检查LSA泛洪、区域规划、路由过滤
- DR/BDR选举异常：检查优先级和Router ID

## 四、BGP边界网关协议

参考来源：RFC 4271

### 协议特征

- 路径矢量协议，互联网骨干网核心协议
- 适用于大型城域网、跨运营商路由交换
- 基于TCP（端口179）建立邻居
- 丰富的路由策略和属性

### BGP类型

| 类型 | 说明 | 适用场景 |
|------|------|---------|
| IBGP | 同一AS内的BGP邻居 | AS内部路由传递 |
| EBGP | 不同AS间的BGP邻居 | 跨AS路由交换 |

### BGP选路原则（13步，前8步高频考点）

1. 优选Weight值最高的路由（思科特有）
2. 优选Local Preference值最高的路由
3. 优选本地始发的路由
4. 优选AS Path最短的路由
5. 优选Origin属性最优的路由（IGP>EGP>Incomplete）
6. 优选MED值最小的路由
7. 优选EBGP路由优于IBGP路由
8. 优选到下一跳IGP度量值最小的路由

### 华为/H3C BGP配置

```
bgp 65001
 router-id 1.1.1.1
 peer 10.1.1.2 as-number 65002
 peer 10.1.1.2 connect-interface GigabitEthernet 0/0/1
 ipv4-family unicast
  peer 10.1.1.2 enable
  network 192.168.1.0 255.255.255.0
```

### 思科BGP配置

```
router bgp 65001
 bgp router-id 1.1.1.1
 neighbor 10.1.1.2 remote-as 65002
 neighbor 10.1.1.2 update-source GigabitEthernet0/1
 network 192.168.1.0 mask 255.255.255.0
```

### BGP常见故障

- 邻居状态卡在Idle/Active：检查TCP连接、路由可达性、EBGP多跳
- 路由不通：检查next-hop可达性、路由策略、IGP是否通告BGP下一跳

## 五、IS-IS路由协议

参考来源：ISO 10589、RFC 1195

### 协议特征

- 链路状态协议，类似OSPF
- 基于CLNS架构，直接运行在数据链路层
- 适用于ISP和大型企业骨干网
- 扩展性好，适合IPv6和MPLS场景

### 与OSPF对比

| 特性 | IS-IS | OSPF |
|------|-------|------|
| 运行层级 | 数据链路层 | 网络层(IP) |
| 区域划分 | 基于路由器(Level-1/Level-2) | 基于接口(Area) |
| 扩展性 | TLV结构，易扩展 | LSA类型固定 |
| 适用场景 | ISP骨干网 | 企业网 |

## 六、路由策略与路由重分布

### 路由策略

华为/H3C：
```
ip ip-prefix PREFIX_10 permit 10.0.0.0 8
route-policy RP_DENY_10 deny node 10
 if-match ip-prefix PREFIX_10
route-policy RP_DENY_10 permit node 20
```

思科：
```
ip prefix-list PREFIX_10 seq 5 permit 10.0.0.0/8
route-map RP_DENY_10 deny 10
 match ip address prefix-list PREFIX_10
route-map RP_DENY_10 permit 20
```

### 路由重分布注意事项

- 不同协议的度量值体系不同，重分布时必须设置种子度量值
- 双向重分布可能导致路由环路和次优路径
- 建议使用路由策略控制重分布范围

## 七、协议选型决策树

```
需要动态路由？
├── 否 → 静态路由（小型网络/固定拓扑）
└── 是 → 网络规模？
    ├── 小型（<15跳）→ RIP v2
    ├── 中大型 → 单一AS？
    │   ├── 是 → OSPF（企业网首选）
    │   │       或 IS-IS（ISP骨干网）
    └── 否 → BGP（跨AS/运营商互联）
```
