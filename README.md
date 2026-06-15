# 🌐 MultiPath TCP (MPTCP) 多路径传输完全教程

> **利用多条网络路径同时传输数据，提升带宽与可靠性**
> **Leverage multiple network paths simultaneously for higher bandwidth and reliability**

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![MPTCP Version](https://img.shields.io/badge/MPTCP-v0.96-blue)](https://www.mptcp.dev)

---

## 📑 目录 / Table of Contents

- [1. 什么是 MPTCP？ / What is MPTCP?](#1-什么是-mptcp--what-is-mptcp)
- [2. 工作原理 / How It Works](#2-工作原理--how-it-works)
- [3. 核心优势 / Key Benefits](#3-核心优势--key-benefits)
- [4. 使用场景 / Use Cases](#4-使用场景--use-cases)
- [5. 环境要求 / Requirements](#5-环境要求--requirements)
- [6. 部署方式一：Docker 部署 / Docker Deployment](#6-部署方式一docker-部署--docker-deployment)
- [7. 部署方式二：原生安装 / Native Installation](#7-部署方式二原生安装--native-installation)
- [8. 配置示例 / Configuration Examples](#8-配置示例--configuration-examples)
- [9. 常用命令 / Common Commands](#9-常用命令--common-commands)
- [10. 监控与调试 / Monitoring & Debugging](#10-监控与调试--monitoring--debugging)
- [11. 常见问题 / FAQ](#11-常见问题--faq)
- [12. 参考资源 / References](#12-参考资源--references)
- [☕ 支持 / Support](#-支持--support)

---

## 1. 什么是 MPTCP？ / What is MPTCP?

### 中文

**MultiPath TCP（MPTCP）** 是标准 TCP 协议的扩展，由 IETF 在 RFC 6824（后由 RFC 8684 更新）中定义。它允许在单个 TCP 连接中同时使用多条网络路径（例如同时使用 Wi-Fi 和 4G/5G），从而：

- **提升吞吐量**：聚合多个接口的带宽
- **提高可靠性**：一条路径故障，流量自动切换到其他路径
- **无缝切换**：连接不会中断，应用层无感知

MPTCP 在 Linux 内核 5.6+ 中已原生支持（需编译内核选项 `CONFIG_MPTCP=y`），是下一代网络传输的重要技术。

### English

**MultiPath TCP (MPTCP)** is an extension to the standard TCP protocol, defined by the IETF in RFC 6824 (updated by RFC 8684). It enables the use of multiple network paths simultaneously within a single TCP connection (e.g., using Wi-Fi and 4G/5G at the same time), providing:

- **Increased throughput**: Aggregate bandwidth across multiple interfaces
- **Enhanced reliability**: Automatic failover when one path fails
- **Seamless handover**: Connection remains intact, transparent to applications

MPTCP is natively supported in Linux kernel 5.6+ (with `CONFIG_MPTCP=y`), making it a key technology for next-generation network transport.

---

## 2. 工作原理 / How It Works

### 中文

MPTCP 在传输层工作，位于应用层和网络层之间。它的核心机制包括：

| 组件 | 作用 | 说明 |
|------|------|------|
| **MPTCP 路径管理器** | 管理多条子流 | 决定何时创建/删除子流 |
| **MPTCP 调度器** | 数据分发策略 | 决定数据在子流间的分配方式 |
| **MPTCP 拥塞控制** | 拥塞控制算法 | 跨子流的拥塞管理（如 LIA, OLIA, Balia） |

**数据流过程：**

1. 客户端发起标准 TCP 三次握手，附加 MP_CAPABLE 选项
2. 服务器响应 MP_CAPABLE，协商 MPTCP 能力
3. 双方通过 ADD_ADDR 和 MP_JOIN 选项添加额外子流
4. 数据通过调度器分发到各子流
5. 接收端通过 DSS（Data Sequence Signal）选项重组数据

### English

MPTCP operates at the transport layer, sitting between the application layer and the network layer. Its core components include:

| Component | Purpose | Description |
|-----------|---------|-------------|
| **Path Manager** | Manage subflows | Decides when to create/remove subflows |
| **Scheduler** | Data distribution strategy | Determines how data is allocated across subflows |
| **Congestion Control** | Congestion management | Cross-subflow congestion algorithms (LIA, OLIA, Balia) |

**Data Flow Process:**

1. Client initiates standard TCP three-way handshake with MP_CAPABLE option
2. Server responds with MP_CAPABLE, negotiating MPTCP capability
3. Both sides add additional subflows via ADD_ADDR and MP_JOIN options
4. Data is distributed across subflows by the scheduler
5. Receiver reassembles data using DSS (Data Sequence Signal) options

---

## 3. 核心优势 / Key Benefits

| 优势 / Benefit | 中文说明 | English Description |
|---------------|----------|-------------------|
| **带宽聚合** | 同时使用 Wi-Fi + 蜂窝网络 | Aggregate Wi-Fi + cellular bandwidth simultaneously |
| **故障切换** | 任意路径中断，连接不中断 | Any path fails, connection continues |
| **负载均衡** | 智能分发流量到最优路径 | Intelligent traffic distribution across optimal paths |
| **无缝切换** | 移动场景下网络切换无感 | Seamless handover between networks in mobile scenarios |
| **应用透明** | 无需修改应用代码 | No application code changes needed |

---

## 4. 使用场景 / Use Cases

### 中文

1. **移动设备**：在 Wi-Fi 和移动数据之间无缝切换，视频通话不中断
2. **数据中心**：多网卡带宽聚合，提升服务器吞吐能力
3. **边缘计算**：多条网络链路冗余，提高可靠性
4. **卫星通信**：结合多条卫星链路，提升整体带宽
5. **远程办公**：VPN 连接通过多条网络路径加速

### English

1. **Mobile Devices**: Seamless handover between Wi-Fi and cellular during video calls
2. **Data Centers**: Multi-NIC bandwidth aggregation for improved server throughput
3. **Edge Computing**: Multiple network link redundancy for higher reliability
4. **Satellite Communications**: Combining multiple satellite links for increased bandwidth
5. **Remote Work**: Accelerating VPN connections via multiple network paths

---

## 5. 环境要求 / Requirements

### 中文

| 组件 | 最低要求 | 推荐配置 |
|------|---------|---------|
| Linux 内核 | 5.6+（MPTCP 支持） | 6.0+（完整功能） |
| Docker | 20.10+ | 24.0+ |
| 网络接口 | 至少 2 个 | 无限制 |
| 内存 | 256 MB | 1 GB+ |
| 磁盘 | 1 GB | 10 GB+ |

### English

| Component | Minimum | Recommended |
|-----------|---------|-------------|
| Linux Kernel | 5.6+ (MPTCP support) | 6.0+ (full features) |
| Docker | 20.10+ | 24.0+ |
| Network Interfaces | At least 2 | Unlimited |
| Memory | 256 MB | 1 GB+ |
| Disk | 1 GB | 10 GB+ |

---

## 6. 部署方式一：Docker 部署 / Docker Deployment

### 中文

使用 Docker 快速搭建 MPTCP 测试环境是最便捷的方式。

#### 6.1 拉取镜像

```bash
# 拉取支持 MPTCP 的 Docker 镜像（需宿主内核支持 MPTCP）
docker pull alpine:latest

# 或者使用 multiconn 项目（基于 MPTCP 的多路径代理）
docker pull nicolaka/netshoot:latest
```

#### 6.2 启动 MPTCP 测试容器

```bash
# 创建多个网络接口的网络
docker network create --driver bridge net1
docker network create --driver bridge net2

# 启动 MPTCP 测试客户端
docker run -it --rm \
  --cap-add=NET_ADMIN \
  --network net1 \
  --name mptcp-client \
  alpine:latest /bin/sh
```

#### 6.3 完整 Docker Compose 部署

创建 `docker-compose.yml`：

```yaml
version: '3.8'

services:
  mptcp-server:
    image: alpine:latest
    container_name: mptcp-server
    cap_add:
      - NET_ADMIN
      - SYS_MODULE
    volumes:
      - ./scripts:/scripts
    networks:
      - net1
      - net2
    command: >
      sh -c "
      apk add --no-cache iperf3 iproute2
      && modprobe mptcp_connect 2>/dev/null || true
      && iperf3 -s
      "

  mptcp-client:
    image: alpine:latest
    container_name: mptcp-client
    cap_add:
      - NET_ADMIN
      - SYS_MODULE
    volumes:
      - ./scripts:/scripts
    networks:
      - net1
      - net2
    depends_on:
      - mptcp-server
    command: >
      sh -c "
      apk add --no-cache iperf3 iproute2
      && sleep 3
      && iperf3 -c mptcp-server -t 30 -P 4
      "

networks:
  net1:
    driver: bridge
  net2:
    driver: bridge
```

#### 6.4 运行测试

```bash
# 启动
docker compose up -d

# 查看日志
docker compose logs -f mptcp-client

# 清理
docker compose down
```

### English

Using Docker to quickly set up an MPTCP test environment is the easiest approach.

#### 6.1 Pull Image

```bash
# Pull an MPTCP-capable Docker image (requires host kernel MPTCP support)
docker pull alpine:latest

# Or use netshoot for network debugging
docker pull nicolaka/netshoot:latest
```

#### 6.2 Start MPTCP Test Container

```bash
# Create multiple bridge networks
docker network create --driver bridge net1
docker network create --driver bridge net2

# Start MPTCP test client
docker run -it --rm \
  --cap-add=NET_ADMIN \
  --network net1 \
  --name mptcp-client \
  alpine:latest /bin/sh
```

#### 6.3 Complete Docker Compose Setup

Create `docker-compose.yml`:

```yaml
version: '3.8'

services:
  mptcp-server:
    image: alpine:latest
    container_name: mptcp-server
    cap_add:
      - NET_ADMIN
      - SYS_MODULE
    volumes:
      - ./scripts:/scripts
    networks:
      - net1
      - net2
    command: >
      sh -c "
      apk add --no-cache iperf3 iproute2
      && modprobe mptcp_connect 2>/dev/null || true
      && iperf3 -s
      "

  mptcp-client:
    image: alpine:latest
    container_name: mptcp-client
    cap_add:
      - NET_ADMIN
      - SYS_MODULE
    volumes:
      - ./scripts:/scripts
    networks:
      - net1
      - net2
    depends_on:
      - mptcp-server
    command: >
      sh -c "
      apk add --no-cache iperf3 iproute2
      && sleep 3
      && iperf3 -c mptcp-server -t 30 -P 4
      "

networks:
  net1:
    driver: bridge
  net2:
    driver: bridge
```

#### 6.4 Run the Test

```bash
# Start
docker compose up -d

# Check logs
docker compose logs -f mptcp-client

# Clean up
docker compose down
```

---

## 7. 部署方式二：原生安装 / Native Installation

### 中文

在 Linux 系统上原生部署 MPTCP 需要支持 MPTCP 的内核。

#### 7.1 检查内核支持

```bash
# 检查内核是否支持 MPTCP
uname -r
# 需要 5.6+

# 检查 MPTCP 内核模块
lsmod | grep mptcp
# 或
modinfo mptcp_pm

# 检查内核配置
cat /boot/config-$(uname -r) | grep MPTCP
# 应该看到 CONFIG_MPTCP=y 或 CONFIG_MPTCP=m
```

#### 7.2 Ubuntu/Debian 安装 MPTCP 内核

```bash
# Ubuntu 20.04+ 默认包含 MPTCP
# 确认内核版本
apt update
apt install -y linux-generic

# 对于旧版本，可以使用 MPTCP 官方仓库
wget -qO - https://www.mptcp.dev/mptcp.gpg.key | gpg --dearmor -o /usr/share/keyrings/mptcp.gpg
echo "deb [signed-by=/usr/share/keyrings/mptcp.gpg] https://packages.mptcp.dev/ubuntu $(lsb_release -cs) main" | tee /etc/apt/sources.list.d/mptcp.list
apt update
apt install -y linux-image-mptcp
```

#### 7.3 启用 MPTCP

```bash
# 加载 MPTCP 内核模块
modprobe mptcp_pm
modprobe mptcp_connect
modprobe mptcp_crypto

# 永久加载
echo "mptcp_pm" >> /etc/modules
echo "mptcp_connect" >> /etc/modules
echo "mptcp_crypto" >> /etc/modules

# 设置 MPTCP 路径管理器
ip mptcp limits set subflows 4
ip mptcp limits set add_addr_accepted 4

# 设置默认路径管理器
sysctl -w net.mptcp.enabled=1
echo "net.mptcp.enabled=1" >> /etc/sysctl.conf
```

#### 7.4 CentOS/RHEL 安装

```bash
# RHEL 9 和 CentOS Stream 9 包含 MPTCP
yum install -y kernel-mptcp

# 或者编译安装
git clone https://github.com/multipath-tcp/mptcp_net-next.git
cd mptcp_net-next
make defconfig
make -j$(nproc)
make modules_install
make install
```

#### 7.5 安装测试工具

```bash
# 安装 iperf3（带宽测试）
apt install -y iperf3
# 或 yum install -y iperf3

# 安装网络工具
apt install -y iproute2 net-tools tcpdump

# 安装 MPTCP 专用工具
git clone https://github.com/multipath-tcp/mptcp-tools.git
cd mptcp-tools
make
make install
```

### English

Native deployment of MPTCP on Linux requires a kernel with MPTCP support.

#### 7.1 Check Kernel Support

```bash
# Check if kernel supports MPTCP
uname -r
# Requires 5.6+

# Check MPTCP kernel modules
lsmod | grep mptcp
# or
modinfo mptcp_pm

# Check kernel config
cat /boot/config-$(uname -r) | grep MPTCP
# Should show CONFIG_MPTCP=y or CONFIG_MPTCP=m
```

#### 7.2 Install MPTCP Kernel on Ubuntu/Debian

```bash
# Ubuntu 20.04+ includes MPTCP by default
# Verify kernel version
apt update
apt install -y linux-generic

# For older versions, use the MPTCP official repo
wget -qO - https://www.mptcp.dev/mptcp.gpg.key | gpg --dearmor -o /usr/share/keyrings/mptcp.gpg
echo "deb [signed-by=/usr/share/keyrings/mptcp.gpg] https://packages.mptcp.dev/ubuntu $(lsb_release -cs) main" | tee /etc/apt/sources.list.d/mptcp.list
apt update
apt install -y linux-image-mptcp
```

#### 7.3 Enable MPTCP

```bash
# Load MPTCP kernel modules
modprobe mptcp_pm
modprobe mptcp_connect
modprobe mptcp_crypto

# Persistent loading
echo "mptcp_pm" >> /etc/modules
echo "mptcp_connect" >> /etc/modules
echo "mptcp_crypto" >> /etc/modules

# Set MPTCP path manager limits
ip mptcp limits set subflows 4
ip mptcp limits set add_addr_accepted 4

# Enable MPTCP globally
sysctl -w net.mptcp.enabled=1
echo "net.mptcp.enabled=1" >> /etc/sysctl.conf
```

#### 7.4 Install on CentOS/RHEL

```bash
# RHEL 9 and CentOS Stream 9 include MPTCP
yum install -y kernel-mptcp

# Or compile from source
git clone https://github.com/multipath-tcp/mptcp_net-next.git
cd mptcp_net-next
make defconfig
make -j$(nproc)
make modules_install
make install
```

#### 7.5 Install Testing Tools

```bash
# Install iperf3 (bandwidth testing)
apt install -y iperf3
# or yum install -y iperf3

# Install network tools
apt install -y iproute2 net-tools tcpdump

# Install MPTCP-specific tools
git clone https://github.com/multipath-tcp/mptcp-tools.git
cd mptcp-tools
make
make install
```

---

## 8. 配置示例 / Configuration Examples

### 8.1 路径管理器配置 / Path Manager Configuration

#### 中文

MPTCP 支持多种路径管理器，通过 `sysctl` 切换：

| 路径管理器 | 模式 | 说明 |
|-----------|------|------|
| `default` | 默认 | 仅使用初始子流 |
| `fullmesh` | 全连接 | 每个地址对都建立子流 |
| `ndiffports` | 多端口 | 通过不同端口创建子流 |
| `binder` | 绑定模式 | 绑定到特定接口 |

```bash
# 查看当前路径管理器
sysctl net.mptcp.pm_type

# 设置为 fullmesh 模式
sysctl -w net.mptcp.pm_type=1

# 设置为 ndiffports 模式（使用 4 个不同端口）
sysctl -w net.mptcp.pm_type=2
sysctl -w net.mptcp.ndiffports=4
```

#### English

MPTCP supports multiple path managers, switchable via `sysctl`:

| Path Manager | Mode | Description |
|-------------|------|-------------|
| `default` | Default | Only uses the initial subflow |
| `fullmesh` | Full Mesh | Creates subflows for every address pair |
| `ndiffports` | Multi-Port | Creates subflows on different ports |
| `binder` | Bind Mode | Binds to specific interfaces |

```bash
# View current path manager
sysctl net.mptcp.pm_type

# Set to fullmesh mode
sysctl -w net.mptcp.pm_type=1

# Set to ndiffports mode (4 different ports)
sysctl -w net.mptcp.pm_type=2
sysctl -w net.mptcp.ndiffports=4
```

### 8.2 调度器配置 / Scheduler Configuration

#### 中文

调度器决定数据如何分发到各子流：

| 调度器 | 策略 | 适用场景 |
|--------|------|---------|
| `default` | 最低 RTT | 选择延迟最低的路径 |
| `roundrobin` | 轮询 | 带宽聚合 |
| `redundant` | 冗余 | 最高可靠性 |
| `blest` | 阻塞估计 | 弱网环境优化 |

```bash
# 查看当前调度器
cat /sys/module/mptcp_connect/parameters/scheduler

# 设置为 roundrobin（轮询调度）
sysctl -w net.mptcp.scheduler=roundrobin

# 设置为 redundant（冗余发送）
sysctl -w net.mptcp.scheduler=redundant
```

#### English

The scheduler determines how data is distributed across subflows:

| Scheduler | Strategy | Use Case |
|-----------|----------|----------|
| `default` | Lowest RTT | Pick the lowest-latency path |
| `roundrobin` | Round Robin | Bandwidth aggregation |
| `redundant` | Redundant | Maximum reliability |
| `blest` | Blocking Estimation | Weak network optimization |

```bash
# View current scheduler
cat /sys/module/mptcp_connect/parameters/scheduler

# Set to roundrobin
sysctl -w net.mptcp.scheduler=roundrobin

# Set to redundant
sysctl -w net.mptcp.scheduler=redundant
```

### 8.3 拥塞控制配置 / Congestion Control Configuration

#### 中文

MPTCP 拥有专用的拥塞控制算法：

| 算法 | 特点 | 推荐场景 |
|------|------|---------|
| LIA | 基础算法，公平友好 | 通用场景 |
| OLIA | 优化版，更好性能 | 高带宽场景 |
| Balia | 平衡算法 | 混合场景 |
| wVegas | 基于延迟 | 低延迟场景 |

```bash
# 查看可用拥塞控制算法
sysctl net.ipv4.tcp_available_congestion_control

# 设置 MPTCP 拥塞控制
sysctl -w net.ipv4.tcp_congestion_control=olia
```

#### English

MPTCP has dedicated congestion control algorithms:

| Algorithm | Features | Recommended For |
|-----------|----------|-----------------|
| LIA | Basic algorithm, TCP-friendly | General purpose |
| OLIA | Optimized, better performance | High bandwidth |
| Balia | Balanced algorithm | Mixed scenarios |
| wVegas | Delay-based | Low latency scenarios |

```bash
# View available congestion control algorithms
sysctl net.ipv4.tcp_available_congestion_control

# Set MPTCP congestion control
sysctl -w net.ipv4.tcp_congestion_control=olia
```

### 8.4 应用级 MPTCP 配置 / Application-Level MPTCP Configuration

#### 中文

通过 socket 选项控制应用是否使用 MPTCP：

```c
#include <netinet/tcp.h>
#include <linux/mptcp.h>

int enable = 1;
setsockopt(sockfd, SOL_TCP, TCP_ULP, "mptcp", strlen("mptcp"));
```

通过环境变量控制：

```bash
# 让 curl 使用 MPTCP
curl --tcp-nodelay --interface 0.0.0.0 https://example.com

# Python 中使用（需支持 MPTCP 的 socket）
export MPTCP_ENABLE=1
python3 -c "
import socket
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.setsockopt(socket.SOL_TCP, 42, b'mptcp')  # 42 = TCP_ULP
"
```

#### English

Control MPTCP usage at the application level via socket options:

```c
#include <netinet/tcp.h>
#include <linux/mptcp.h>

int enable = 1;
setsockopt(sockfd, SOL_TCP, TCP_ULP, "mptcp", strlen("mptcp"));
```

Via environment variables:

```bash
# Make curl use MPTCP
curl --tcp-nodelay --interface 0.0.0.0 https://example.com

# Python usage (requires MPTCP-capable socket)
export MPTCP_ENABLE=1
python3 -c "
import socket
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.setsockopt(socket.SOL_TCP, 42, b'mptcp')  # 42 = TCP_ULP
"
```

---

## 9. 常用命令 / Common Commands

### 中文

| 命令 / Command | 作用 / Purpose | 示例 / Example |
|----------------|---------------|----------------|
| `ip mptcp` | MPTCP 管理命令 | `ip mptcp show` |
| `ss -M` | 显示 MPTCP 连接 | `ss -Mti` |
| `sysctl net.mptcp` | 查看 MPTCP 参数 | `sysctl -a \| grep mptcp` |
| `iperf3 -c host -P 4` | 多流带宽测试 | `iperf3 -c server -P 4 -t 30` |
| `tcpdump -i any 'tcp[tcpflags] & tcp-syn != 0'` | 抓 MPTCP SYN 包 | `tcpdump -nn -i any 'tcp port 5201'` |

```bash
# 查看所有 MPTCP 连接
ip mptcp show

# 查看 MPTCP 详细信息
ss -Mti

# 显示 MPTCP 统计信息
cat /proc/net/mptcp

# 查看 MPTCP 内核参数
sysctl -a | grep mptcp

# 测试多路径带宽
iperf3 -c 192.168.1.100 -P 4 -t 30

# 抓包分析 MPTCP 选项
tcpdump -nn -i any 'tcp port 5201' -X
```

### English

| Command | Purpose | Example |
|---------|---------|---------|
| `ip mptcp` | MPTCP management | `ip mptcp show` |
| `ss -M` | Show MPTCP connections | `ss -Mti` |
| `sysctl net.mptcp` | View MPTCP parameters | `sysctl -a \| grep mptcp` |
| `iperf3 -c host -P 4` | Multi-stream bandwidth test | `iperf3 -c server -P 4 -t 30` |
| `tcpdump -i any 'tcp[tcpflags] & tcp-syn != 0'` | Capture MPTCP SYN packets | `tcpdump -nn -i any 'tcp port 5201'` |

```bash
# View all MPTCP connections
ip mptcp show

# View MPTCP details
ss -Mti

# Show MPTCP statistics
cat /proc/net/mptcp

# View MPTCP kernel parameters
sysctl -a | grep mptcp

# Test multi-path bandwidth
iperf3 -c 192.168.1.100 -P 4 -t 30

# Capture and analyze MPTCP options
tcpdump -nn -i any 'tcp port 5201' -X
```

---

## 10. 监控与调试 / Monitoring & Debugging

### 10.1 查看 MPTCP 连接状态 / Check MPTCP Connection Status

#### 中文

```bash
# 查看详细 MPTCP 连接信息
cat /proc/net/mptcp

# 示例输出：
# token  rem_id  loc_id  loc_port rem_port  inode
# 12345  0       0       12345   5201      54321
#    subflows: 2
#    0: 192.168.1.2:12345 -> 192.168.1.100:5201
#    1: 192.168.2.2:12346 -> 192.168.2.100:5201

# 使用 ss 命令
ss -Mti | head -20
```

#### English

```bash
# View detailed MPTCP connection info
cat /proc/net/mptcp

# Example output:
# token  rem_id  loc_id  loc_port rem_port  inode
# 12345  0       0       12345   5201      54321
#    subflows: 2
#    0: 192.168.1.2:12345 -> 192.168.1.100:5201
#    1: 192.168.2.2:12346 -> 192.168.2.100:5201

# Using ss command
ss -Mti | head -20
```

### 10.2 数据包捕获与分析 / Packet Capture & Analysis

#### 中文

```bash
# 抓取 MPTCP 数据包
tcpdump -nn -i any 'tcp port 5201' -w mptcp.pcap

# 使用 tshark 分析 MPTCP 选项
tshark -r mptcp.pcap -Y "mptcp" -T fields -e mptcp.mp_capable

# 检查 MPTCP 握手
tcpdump -nn -i any 'tcp[tcpflags] & (tcp-syn) != 0 and tcp port 5201'
```

#### English

```bash
# Capture MPTCP packets
tcpdump -nn -i any 'tcp port 5201' -w mptcp.pcap

# Analyze MPTCP options with tshark
tshark -r mptcp.pcap -Y "mptcp" -T fields -e mptcp.mp_capable

# Check MPTCP handshake
tcpdump -nn -i any 'tcp[tcpflags] & (tcp-syn) != 0 and tcp port 5201'
```

### 10.3 性能测试 / Performance Testing

#### 中文

```bash
# 单路径基线测试
iperf3 -c server -t 30

# 多路径 MPTCP 测试
iperf3 -c server -t 30 -P 4

# 双向测试
iperf3 -c server -t 30 --bidir

# 查看每个子流的吞吐量
iperf3 -c server -t 30 -P 4 --json | jq '.end.sum_sent.bits_per_second / 1e6'
```

#### English

```bash
# Single-path baseline test
iperf3 -c server -t 30

# Multi-path MPTCP test
iperf3 -c server -t 30 -P 4

# Bidirectional test
iperf3 -c server -t 30 --bidir

# View per-subflow throughput
iperf3 -c server -t 30 -P 4 --json | jq '.end.sum_sent.bits_per_second / 1e6'
```

### 10.4 日志与调试 / Logs & Debugging

#### 中文

```bash
# 启用 MPTCP 调试日志
echo 'file net/mptcp/* +p' > /sys/kernel/debug/dynamic_debug/control

# 查看 MPTCP 内核日志
dmesg | grep MPTCP
dmesg | grep mptcp

# 跟踪 MPTCP 事件
perf trace -e 'mptcp:*'
```

#### English

```bash
# Enable MPTCP debug logging
echo 'file net/mptcp/* +p' > /sys/kernel/debug/dynamic_debug/control

# View MPTCP kernel logs
dmesg | grep MPTCP
dmesg | grep mptcp

# Trace MPTCP events
perf trace -e 'mptcp:*'
```

---

## 11. 常见问题 / FAQ

### Q1: 如何确认 MPTCP 是否正在工作？
**A1**: 运行 `ip mptcp show` 或 `ss -Mti`，查看是否有 MPTCP 连接。也可以用 `cat /proc/net/mptcp` 查看具体信息。

**EN**: Run `ip mptcp show` or `ss -Mti` to check for MPTCP connections. Use `cat /proc/net/mptcp` for detailed info.

### Q2: MPTCP 和普通 TCP 有什么兼容性问题？
**A2**: MPTCP 兼容普通 TCP。如果对端不支持 MPTCP，连接会回退到标准 TCP。服务器需要配置 MPTCP 才能使用多路径。

**EN**: MPTCP is backward-compatible with regular TCP. If the peer doesn't support MPTCP, the connection falls back to standard TCP. The server needs MPTCP configuration for multi-path usage.

### Q3: 为什么我的 MPTCP 只有一条子流？
**A3**: 可能原因：
- 只有一张网卡有 IP 地址
- 路径管理器配置为默认（仅用初始路径）
- 防火墙阻止了 ADD_ADDR 或 MP_JOIN 选项
- 对端未配置 MPTCP

尝试设置 `sysctl -w net.mptcp.pm_type=1`（fullmesh 模式）

**EN**: Possible reasons:
- Only one NIC has an IP address
- Path manager is set to default (initial path only)
- Firewall blocks ADD_ADDR or MP_JOIN options
- Peer is not configured for MPTCP

Try setting `sysctl -w net.mptcp.pm_type=1` (fullmesh mode)

### Q4: MPTCP 在移动网络中的表现如何？
**A4**: MPTCP 在移动网络（4G/5G）中表现优异，可以实现在 Wi-Fi 和蜂窝网络之间的无缝切换，视频通话和实时音视频应用不会中断。

**EN**: MPTCP performs excellently in mobile networks (4G/5G), enabling seamless handover between Wi-Fi and cellular networks without interrupting video calls or real-time audio/video applications.

### Q5: 如何为特定应用启用或禁用 MPTCP？
**A5**: 使用 cgroup 或 socket 选项控制。通过 `cgroup v2` 可以按进程组控制 MPTCP：

```bash
# 创建 cgroup
mkdir -p /sys/fs/cgroup/net/mptcp-enabled
echo 1 > /sys/fs/cgroup/net/mptcp-enabled/mptcp.enabled
echo $PID > /sys/fs/cgroup/net/mptcp-enabled/cgroup.procs
```

**EN**: Use cgroups or socket options. Control MPTCP per process group via `cgroup v2`:

```bash
# Create cgroup
mkdir -p /sys/fs/cgroup/net/mptcp-enabled
echo 1 > /sys/fs/cgroup/net/mptcp-enabled/mptcp.enabled
echo $PID > /sys/fs/cgroup/net/mptcp-enabled/cgroup.procs
```

### Q6: MPTCP 对 NAT 环境是否友好？
**A6**: MPTCP 在 NAT 环境中可能会遇到问题，因为 ADD_ADDR 包含内部 IP 地址。建议使用 TURN/STUN 或 SOCKS 代理配合。MPTCP 的 NAT 遍历仍在开发中。

**EN**: MPTCP may have issues in NAT environments because ADD_ADDR contains internal IP addresses. Using TURN/STUN or SOCKS proxies is recommended. MPTCP NAT traversal is still under development.

### Q7: 如何在 Docker 中使用 MPTCP？
**A7**: Docker 容器默认使用宿主网络命名空间。要让容器使用 MPTCP，需要：
1. 宿主内核支持 MPTCP
2. 使用 `--network host` 或创建多个网络接口
3. 容器内需要适当的 `NET_ADMIN` 权限

**EN**: Docker containers use the host's network namespace by default. To use MPTCP in containers:
1. Host kernel must support MPTCP
2. Use `--network host` or create multiple network interfaces
3. Container needs `NET_ADMIN` capability

### Q8: MPTCP 的带宽聚合效果如何？
**A8**: MPTCP 的带宽聚合效果取决于调度器和网络条件。轮询调度（roundrobin）通常能接近理论带宽之和（~90%），但延迟差异较大的路径可能效果打折扣。

**EN**: MPTCP bandwidth aggregation depends on the scheduler and network conditions. Round-robin scheduling typically achieves ~90% of the theoretical combined bandwidth, but paths with large latency differences may see reduced gains.

### Q9: MPTCP 的延迟是多少？
**A9**: MPTCP 的单路径延迟与标准 TCP 基本一致。多路径模式下，延迟取决于所选调度器：default 选择最低延迟路径，redundant 模式会增加延迟。

**EN**: Single-path MPTCP latency is nearly identical to standard TCP. In multi-path mode, latency depends on the scheduler: default picks the lowest-latency path, while redundant mode increases latency.

### Q10: MPTCP 是否支持 IPv6？
**A10**: 是的，MPTCP 完全支持 IPv6，支持 IPv4 子流和 IPv6 子流的混合使用。需要在 `/etc/sysctl.conf` 中启用 IPv6 支持。

```bash
sysctl -w net.ipv6.conf.all.disable_ipv6=0
```

**EN**: Yes, MPTCP fully supports IPv6, including mixed IPv4 and IPv6 subflows. Enable IPv6 support in `/etc/sysctl.conf`:

```bash
sysctl -w net.ipv6.conf.all.disable_ipv6=0
```

---

## 12. 参考资源 / References

### 中文

| 资源 | 链接 |
|------|------|
| IETF RFC 8684 (MPTCP v1) | https://datatracker.ietf.org/doc/rfc8684/ |
| MPTCP 官方网站 | https://www.mptcp.dev |
| Linux MPTCP 上游 | https://www.mptcp.dev/linux-mptcp.html |
| MPTCP Linux 源码 | https://github.com/multipath-tcp/mptcp_net-next |
| MPTCP 工具集 | https://github.com/multipath-tcp/mptcp-tools |
| Multipath TCP Linux 内核文档 | https://docs.kernel.org/networking/mptcp-sysctl.html |

### English

| Resource | Link |
|----------|------|
| IETF RFC 8684 (MPTCP v1) | https://datatracker.ietf.org/doc/rfc8684/ |
| MPTCP Official Site | https://www.mptcp.dev |
| Linux MPTCP Upstream | https://www.mptcp.dev/linux-mptcp.html |
| MPTCP Linux Source | https://github.com/multipath-tcp/mptcp_net-next |
| MPTCP Tools | https://github.com/multipath-tcp/mptcp-tools |
| Multipath TCP Linux Kernel Docs | https://docs.kernel.org/networking/mptcp-sysctl.html |

---

## ☕ 支持 / Support

如果这个教程对你有帮助，欢迎请我喝杯咖啡：

**USDT (TRC20)**

```
TVbQerV1SF4MXB1JCcAzQxarewHwEPYTKm
```
