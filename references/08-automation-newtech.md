# 自动化运维与新技术趋势

## 一、Python网络自动化

参考来源：Netmiko官方文档、Ansible官方文档

### Netmiko基础模板

```python
from netmiko import ConnectHandler

device = {
    'device_type': 'huawei',
    'ip': '192.168.1.1',
    'username': 'admin',
    'password': 'password',
    'port': 22,
}

connection = ConnectHandler(**device)
output = connection.send_command("display ip routing-table")
print(output)
connection.disconnect()
```

### Netmiko支持的device_type

| 厂商 | device_type |
|------|------------|
| 华为 | huawei |
| H3C | hp_comware |
| 思科IOS | cisco_ios |
| 思科NX-OS | cisco_nxos |
| 锐捷 | ruijie_os |

### 批量配置下发模板

```python
from netmiko import ConnectHandler
import csv

devices = []
with open('devices.csv', 'r') as f:
    reader = csv.DictReader(f)
    for row in reader:
        devices.append(row)

commands = [
    'interface GigabitEthernet 0/0/1',
    'description Configured_by_Ansible',
    'port link-type access',
    'port default vlan 10',
]

for dev in devices:
    device = {
        'device_type': dev['device_type'],
        'ip': dev['ip'],
        'username': dev['username'],
        'password': dev['password'],
    }
    try:
        conn = ConnectHandler(**device)
        output = conn.send_config_set(commands)
        print(f"{dev['ip']}: {output}")
        conn.save_config()
        conn.disconnect()
    except Exception as e:
        print(f"{dev['ip']}: Error - {e}")
```

### 自动化巡检模板

```python
from netmiko import ConnectHandler
import datetime

device = {
    'device_type': 'huawei',
    'ip': '192.168.1.1',
    'username': 'admin',
    'password': 'password',
}

check_commands = [
    'display cpu-usage',
    'display memory-usage',
    'display interface brief',
    'display ip routing-table',
    'display ospf peer brief',
    'display alarm active',
]

conn = ConnectHandler(**device)
timestamp = datetime.datetime.now().strftime('%Y%m%d_%H%M%S')
filename = f"check_{device['ip']}_{timestamp}.txt"

with open(filename, 'w') as f:
    for cmd in check_commands:
        f.write(f"{'='*50}\n")
        f.write(f"Command: {cmd}\n")
        f.write(f"{'='*50}\n")
        output = conn.send_command(cmd)
        f.write(output + '\n\n')

conn.disconnect()
print(f"Check report saved: {filename}")
```

## 二、Ansible网络自动化

### Ansible Inventory示例

```ini
[switches]
sw1 ansible_host=192.168.1.1
sw2 ansible_host=192.168.1.2

[switches:vars]
ansible_network_os=huawei
ansible_user=admin
ansible_password=password
ansible_connection=network_cli
```

### Ansible Playbook示例

```yaml
---
- name: Configure VLAN on switches
  hosts: switches
  gather_facts: no
  tasks:
    - name: Create VLAN 10
      ce_vlan:
        vlan_id: 10
        name: Office_VLAN10
    - name: Configure access port
      ce_interface:
        interface: GigabitEthernet0/0/1
        mode: access
        access_vlan: 10
```

## 三、SDN与NFV

### SDN（软件定义网络）

核心架构：
| 层级 | 功能 | 代表产品 |
|------|------|---------|
| 应用层 | 网络应用（负载均衡/防火墙等） | OpenStack/ODL应用 |
| 控制层 | 集中控制、下发流表 | ODL/ONOS/RYU |
| 基础设施层 | 数据转发 | OpenFlow交换机 |

### NFV（网络功能虚拟化）

将传统网络设备功能（防火墙/负载均衡/VPN网关）软件化，运行在通用服务器上。

## 四、AIOps智能运维

参考来源：IDC 2026调研、三大运营商自智网络白皮书

### AIOps核心能力

| 能力 | 说明 |
|------|------|
| 异常检测 | 基于ML自动发现网络异常 |
| 根因分析 | 自动定位故障根因 |
| 预测分析 | 预测网络拥塞/设备故障 |
| 自动修复 | 自动执行修复脚本 |

### 自智网络等级

| 等级 | 说明 | 当前水平 |
|------|------|---------|
| L1 | 辅助执行 | 已普及 |
| L2 | 部分自治 | 已普及 |
| L3 | 条件自治 | 部分场景 |
| L4 | 高阶自治 | 运营商部分场景已达 |
| L5 | 完全自治 | 远期目标 |

### AI-Native Network

"AI for Network" + "Network for AI"双向赋能：
- AI for Network：AI驱动网络自主进化、智能运维
- Network for AI：网络为AI算力提供高性能传输（RDMA/智能网卡）

## 五、SRv6/Segment Routing

参考来源：RFC 8402(SR架构)、RFC 8754(SRv6)

### SRv6核心概念

- 基于IPv6数据面的源路由技术
- 用Segment ID(SID)指示转发路径
- 替代传统MPLS，简化网络协议栈

### SRv6优势

| 特性 | MPLS | SRv6 |
|------|------|------|
| 控制平面 | LDP/RSVP-TE | IGP扩展/SDN |
| 数据平面 | 标签 | IPv6扩展头 |
| 扩展性 | 标签空间有限 | IPv6地址空间巨大 |
| 可编程性 | 弱 | 强（SID可编程） |

## 六、零信任网络架构

### 核心原则

- 不再信任任何内外网边界
- 持续验证：每次访问都需认证和授权
- 最小权限：仅授予必要权限
- 微隔离：细粒度访问控制

### 零信任实现要素

| 要素 | 说明 |
|------|------|
| 身份认证 | 多因素认证(MFA)、单点登录(SSO) |
| 设备信任 | 设备合规检查、终端安全评估 |
| 网络隔离 | 微隔离、软件定义边界(SDP) |
| 持续监控 | 行为分析、异常检测 |

## 七、信创网络设备

### 信创生态

| 组件 | 国产方案 |
|------|---------|
| CPU | 华为鲲鹏/海光/龙芯/飞腾 |
| 操作系统 | 麒麟/统信UOS/EulerOS |
| 网络设备 | 华为/新华三/锐捷信创系列 |
| 密码模块 | 国密SM2/SM3/SM4 |

### 信创适配要点

- 设备驱动兼容性验证
- 国密算法替代国际算法
- 管理系统适配国产浏览器
- 供应链安全评估

## 八、5G网络切片

### 核心概念

在单一物理网络上构建多个逻辑隔离的网络实例，满足不同业务需求。

| 切片类型 | 场景 | 特点 |
|---------|------|------|
| eMBB | 大带宽（视频/VR） | 高速率 |
| URLLC | 低延迟（工业控制/自动驾驶） | 超低时延、高可靠 |
| mMTC | 大连接（物联网） | 海量连接 |

## 九、Wi-Fi 7部署

参考来源：IEEE 802.11be

### Wi-Fi 7关键特性

| 特性 | 说明 |
|------|------|
| MLO | 多链路操作，同时使用多个频段 |
| 320MHz信道 | 比Wi-Fi 6的160MHz翻倍 |
| 4096-QAM | 更高调制效率 |
| 多RU | 单用户可分配多个资源单元 |

## 十、免费实验工具推荐

| 工具 | 厂商 | 适用场景 | 获取方式 |
|------|------|---------|---------|
| eNSP | 华为 | 华为设备模拟 | 华为官网免费下载 |
| Packet Tracer | 思科 | 思科设备入门 | 思科网院免费注册 |
| GNS3 | 开源 | 多厂商模拟 | gns3.com免费下载 |
| EVE-NG | 社区 | 企业级网络模拟 | 社区版免费 |
| HCL | 新华三 | H3C设备模拟 | H3C官网免费下载 |
