# 服务器与系统配置

## 一、Windows Server网络配置

参考来源：Windows Server官方文档

### IP地址配置

图形界面：控制面板→网络和共享中心→更改适配器设置→IPv4属性

命令行（PowerShell）：
```powershell
Set-NetIPAddress -InterfaceIndex 12 -IPAddress 192.168.1.10 -PrefixLength 24
Set-NetIPInterface -InterfaceIndex 12 -DefaultGateway 192.168.1.1
Set-DnsClientServerAddress -InterfaceIndex 12 -ServerAddresses 8.8.8.8,8.8.4.4
```

### 远程桌面配置

1. 系统属性→远程→允许远程连接
2. 防火墙放行RDP（端口3389）
3. 建议修改默认端口+限制来源IP

### Windows防火墙配置

```powershell
Enable-NetFirewallRule -DisplayGroup "Remote Desktop"
New-NetFirewallRule -DisplayName "Allow_HTTP" -Direction Inbound -Protocol TCP -LocalPort 80 -Action Allow
```

## 二、Linux网络配置

参考来源：CentOS/Ubuntu官方文档

### 网卡配置（CentOS 7+）

nmcli方式：
```bash
nmcli connection add type ethernet con-name eth0 ifname eth0 ip4 192.168.1.10/24 gw4 192.168.1.1
nmcli connection modify eth0 ipv4.dns "8.8.8.8 8.8.4.4"
nmcli connection up eth0
```

配置文件方式（/etc/sysconfig/network-scripts/ifcfg-eth0）：
```
TYPE=Ethernet
BOOTPROTO=static
IPADDR=192.168.1.10
NETMASK=255.255.255.0
GATEWAY=192.168.1.1
DNS1=8.8.8.8
ONBOOT=yes
```

### Ubuntu Netplan配置（/etc/netplan/01-netcfg.yaml）

```yaml
network:
  version: 2
  ethernets:
    eth0:
      addresses:
        - 192.168.1.10/24
      gateway4: 192.168.1.1
      nameservers:
        addresses: [8.8.8.8, 8.8.4.4]
```

### 常用网络诊断命令

```bash
ip addr show          # 查看IP地址
ip route show         # 查看路由表
ping 8.8.8.8          # 测试连通性
traceroute 8.8.8.8    # 跟踪路由路径
ss -tlnp              # 查看监听端口
netstat -tlnp         # 查看监听端口（旧版）
tcpdump -i eth0       # 抓包分析
nslookup domain.com   # DNS查询
```

## 三、DHCP服务器

### Windows Server DHCP配置

1. 安装DHCP角色：服务器管理器→添加角色→DHCP服务器
2. 授权DHCP服务器
3. 创建作用域：IP范围、子网掩码、排除地址、租约时间
4. 配置选项：网关(003)、DNS(006)、域名(015)

### Linux DHCP服务器（dnsmasq）

```bash
apt install dnsmasq
```

配置文件（/etc/dnsmasq.conf）：
```
interface=eth0
dhcp-range=192.168.1.100,192.168.1.200,255.255.255.0,12h
dhcp-option=3,192.168.1.1
dhcp-option=6,8.8.8.8,8.8.4.4
```

### DHCP常见故障

| 故障现象 | 排查方向 |
|---------|---------|
| 终端无法获取IP | DHCP服务状态、地址池耗尽、防火墙拦截UDP67/68 |
| 获取到错误IP | 存在非法DHCP服务器、VLAN配置错误 |
| IP冲突 | 静态IP与DHCP地址池重叠 |

## 四、DNS服务器

### Windows Server DNS配置

1. 安装DNS角色
2. 创建正向查找区域
3. 添加A记录、CNAME记录、MX记录
4. 配置转发器

### Linux DNS服务器（BIND）

```bash
apt install bind9
```

配置文件（/etc/bind/named.conf.options）：
```
options {
    directory "/var/cache/bind";
    forwarders { 8.8.8.8; 8.8.4.4; };
    allow-query { any; };
    recursion yes;
};
```

### DNS记录类型

| 类型 | 说明 | 示例 |
|------|------|------|
| A | 域名→IPv4 | www.example.com → 192.168.1.10 |
| AAAA | 域名→IPv6 | www.example.com → 2001:db8::1 |
| CNAME | 域名别名 | blog.example.com → www.example.com |
| MX | 邮件交换 | example.com → mail.example.com |
| NS | 名称服务器 | example.com → ns1.example.com |
| PTR | IP→域名（反向解析） | 10.1.168.192.in-addr.arpa → www.example.com |
| SRV | 服务定位 | _ldap._tcp.example.com |

### DNS常见故障

| 故障现象 | 排查方向 |
|---------|---------|
| 域名无法解析 | DNS服务状态、转发器配置、网络连通 |
| 解析缓慢 | 递归查询超时、缓存未启用 |
| 解析结果错误 | 记录配置错误、DNS缓存污染 |

## 五、Web服务器

### Nginx基础配置

```nginx
server {
    listen 80;
    server_name www.example.com;
    root /var/www/html;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

### HTTPS配置

```nginx
server {
    listen 443 ssl;
    server_name www.example.com;
    ssl_certificate /etc/ssl/certs/server.crt;
    ssl_certificate_key /etc/ssl/private/server.key;
    ssl_protocols TLSv1.2 TLSv1.3;
}
```

## 六、服务器安全加固

### Linux安全加固清单

| 项目 | 操作 |
|------|------|
| SSH安全 | 禁用root登录、修改默认端口、使用密钥认证 |
| 防火墙 | 仅开放必要端口、限制来源IP |
| 账号管理 | 删除无用账号、密码复杂度策略、定期更换 |
| 日志审计 | 启用syslog、日志留存≥6个月 |
| 系统更新 | 定期安全补丁、内核升级 |
| 服务最小化 | 关闭不必要的服务和端口 |
| 文件权限 | 敏感文件权限控制、/tmp安全挂载 |

### Windows安全加固清单

| 项目 | 操作 |
|------|------|
| 远程访问 | 修改RDP端口、限制来源IP、网络级别身份验证 |
| 防火墙 | 启用高级安全防火墙、仅放行必要端口 |
| 账号策略 | 密码复杂度、锁定策略、定期更换 |
| 补丁管理 | 启用自动更新、定期检查 |
| 服务管理 | 禁用不必要的服务 |
| 审计策略 | 启用登录审计、对象访问审计 |
