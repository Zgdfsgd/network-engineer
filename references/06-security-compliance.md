# 网络安全与合规

## 一、ACL访问控制列表

参考来源：华为/H3C/思科安全配置规范、等保2.0通用要求

### ACL类型

| 类型 | 编号范围 | 说明 |
|------|---------|------|
| 基本ACL | 2000-2999（华为）/1-99（思科） | 仅基于源IP过滤 |
| 高级ACL | 3000-3999（华为）/100-199（思科） | 基于五元组（源IP/目的IP/源端口/目的端口/协议） |
| 二层ACL | 4000-4999 | 基于MAC地址过滤 |

### 华为/H3C ACL配置

```
acl number 3000
 rule 5 permit tcp source 192.168.1.0 0.0.0.255 destination 192.168.2.0 0.0.0.255 destination-port eq 80
 rule 10 deny ip source any destination any

interface GigabitEthernet 0/0/1
 traffic-filter inbound acl 3000
```

### 思科ACL配置

```
ip access-list extended WEB_ACCESS
 permit tcp 192.168.1.0 0.0.0.255 192.168.2.0 0.0.0.255 eq 80
 deny ip any any

interface GigabitEthernet0/1
 ip access-group WEB_ACCESS in
```

### ACL注意事项

- 规则按序号匹配，命中即执行，注意规则顺序
- 默认隐含deny any any
- 华为通配符与思科通配符计算方式相同（0表示匹配，1表示忽略）

## 二、防火墙基础

### 包过滤防火墙

基于五元组（源IP、目的IP、源端口、目的端口、协议）拦截恶意流量。

### 防火墙安全区域（华为/H3C）

| 区域 | 安全级别 | 说明 |
|------|---------|------|
| Trust | 85 | 内网可信区域 |
| Untrust | 5 | 外网不可信区域 |
| DMZ | 50 | 隔离区（服务器） |
| Local | 100 | 防火墙自身 |

### 安全策略配置原则

- 默认拒绝所有流量
- 按需放行，最小权限原则
- 高安全级别→低安全级别默认允许（需策略放行）
- 低安全级别→高安全级别默认拒绝

## 三、NAT地址转换

### NAT类型

| 类型 | 说明 | 适用场景 |
|------|------|---------|
| SNAT/源NAT | 内网→外网，转换源地址 | 内网用户访问互联网 |
| DNAT/目的NAT | 外网→内网，转换目的地址 | 服务器对外发布 |
| PAT/端口NAT | 多对一，复用端口 | 节省公网IP |

### 华为/H3C NAT配置

```
# 定义NAT地址组
nat address-group 1 202.100.1.10 202.100.1.20

# 接口配置NAT
interface GigabitEthernet 0/0/1
 nat outbound 2000 address-group 1

# 服务器映射（DNAT）
interface GigabitEthernet 0/0/1
 nat server protocol tcp global 202.100.1.10 80 inside 192.168.1.10 80
```

### 思科NAT配置

```
# PAT（动态NAT）
access-list 1 permit 192.168.1.0 0.0.0.255
ip nat inside source list 1 interface GigabitEthernet0/1 overload

# 静态NAT（服务器映射）
ip nat inside source static tcp 192.168.1.10 80 202.100.1.10 80

interface GigabitEthernet0/0
 ip nat inside
interface GigabitEthernet0/1
 ip nat outside
```

## 四、VPN技术

### IPsec VPN

参考来源：RFC 4301/IPsec架构、RFC 2409/IKE

核心组件：
- **IKE**：密钥交换协议，负责协商安全参数
- **AH**：认证头，提供数据完整性和来源认证
- **ESP**：封装安全载荷，提供加密+认证

IPsec VPN两种模式：
| 模式 | 说明 | 适用场景 |
|------|------|---------|
| 传输模式 | 仅加密数据载荷 | 端到端加密 |
| 隧道模式 | 加密整个原始IP包 | 站点到站点VPN |

华为/H3C IPsec配置框架：
```
# 定义感兴趣流
acl number 3000
 rule permit ip source 192.168.1.0 0.0.0.255 destination 192.168.2.0 0.0.0.255

# IKE提议
ike proposal 1
 encryption-algorithm aes-256
 dh group14
 authentication-algorithm sha2-256

# IKE对等体
ike peer PEER1
 pre-shared-key cipher XXXXXX
 remote-address 202.100.1.2

# IPsec提议
ipsec proposal PROP1
 encapsulation-mode tunnel
 transform esp
 esp authentication-algorithm sha2-256
 esp encryption-algorithm aes-256

# IPsec策略
ipsec policy POL1 10 isakmp
 ike-peer PEER1
 proposal PROP1
 security acl 3000
```

### SSL VPN

比IPsec更轻量，无需客户端，通过浏览器访问。适配远程办公场景。

## 五、AAA认证

| 协议 | 端口 | 特点 |
|------|------|------|
| RADIUS | 1812/认证、1813/计费 | UDP传输、仅加密密码、适合网络设备 |
| TACACS+ | 49 | TCP传输、加密全部数据、适合管理员认证 |

## 六、等保2.0合规要求

参考来源：GB/T 22239-2019

### 网络设备合规检查点

| 检查项 | 要求 | 等级 |
|--------|------|------|
| 账号分权管理 | 不同管理员不同权限，避免共享账号 | 二级+ |
| 日志审计 | 开启操作日志，留存≥6个月 | 二级+ |
| 远程访问限制 | 限制管理来源IP，使用加密协议(SSH) | 二级+ |
| 密码策略 | 复杂度要求、定期更换、失败锁定 | 二级+ |
| 漏洞管理 | 定期安全扫描、及时修补漏洞 | 三级+ |
| 冗余设计 | 关键设备冗余、链路冗余 | 三级+ |
| 设备性能冗余 | 处理能力预留30% | 三级+ |

### 国密算法应用

参考来源：GB/T 32907(SM4)、GB/T 32918(SM2)、GB/T 32905(SM3)

| 算法 | 类型 | 对标国际算法 | 应用场景 |
|------|------|------------|---------|
| SM2 | 非对称加密/签名 | RSA/ECC | 数字证书、密钥交换 |
| SM3 | 哈希算法 | SHA-256 | 数据完整性校验 |
| SM4 | 对称加密 | AES-128 | 数据加密传输/存储 |

等保三级强制要求：采用国密算法SM2/SM3/SM4。推荐"国密+国际算法"双证书策略。

## 七、法律法规合规要点

### 《网络安全法》（2026年1月1日施行修改版）

- 网络运营者需落实网络安全保护责任
- 禁止利用网络从事危害网络安全、非法入侵、数据窃取等行为
- 网络日志留存时间不少于6个月
- 新增人工智能条款

### 《数据安全法》

- 规范数据处理活动，保障数据安全
- 明确数据出境安全评估制度
- 涉及数据存储、传输的网络设计需满足合规要求

### 《个人信息保护法》

- 保护个人信息权益
- 网络系统涉及用户数据的部分需落实保护措施

### 《网络数据安全管理条例》（2025年1月1日起施行）

- 规范网络数据处理活动
- 日常运维中处理网络数据的行为准则

### 《关键信息基础设施安全保护条例》

- 针对政企、运营商等关键网络设施
- 要求强化安全防护、定期安全测评、应急演练

## 八、合规检查点自动触发规则

当用户咨询以下场景时，自动提醒合规要求：

| 场景 | 自动触发的合规检查点 |
|------|-------------------|
| 网络方案设计 | 是否涉及等保定级、数据分类 |
| 远程访问配置 | 是否限制来源IP、是否使用加密协议 |
| VPN配置 | 是否符合数据出境要求 |
| 日志配置 | 是否满足6个月留存要求 |
| 密码/认证配置 | 是否满足复杂度和定期更换要求 |
| 服务器发布 | 是否需要等保测评、是否配置WAF |
