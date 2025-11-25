# Wireshark 使用手册与过滤器完全指南

## 📚 目录
- [1. Wireshark 基础介绍](#1-wireshark-基础介绍)
- [2. Display Filter（显示过滤器）详解](#2-display-filter显示过滤器详解)
- [3. Capture Filter（捕获过滤器）详解](#3-capture-filter捕获过滤器详解)
- [4. 常用协议过滤器](#4-常用协议过滤器)
- [5. CTF/安全分析常用过滤器](#5-ctf安全分析常用过滤器)
- [6. 高级技巧与实战案例](#6-高级技巧与实战案例)

---

## 1. Wireshark 基础介绍

### 1.1 什么是 Wireshark
Wireshark 是世界上最流行的网络协议分析工具，用于捕获和交互式浏览网络流量。

### 1.2 两种过滤器类型
- **Capture Filter（捕获过滤器）**：在捕获前过滤，减少捕获的数据量
- **Display Filter（显示过滤器）**：在捕获后过滤，用于分析已捕获的数据

### 1.3 界面布局
```
+------------------------------------------+
|  菜单栏 | 工具栏 | Display Filter 输入框   |
+------------------------------------------+
|  Packet List（数据包列表）                 |
+------------------------------------------+
|  Packet Details（数据包详细信息）          |
+------------------------------------------+
|  Packet Bytes（数据包字节流）             |
+------------------------------------------+
```

---

## 2. Display Filter（显示过滤器）详解

### 2.1 基本语法

#### 2.1.1 比较运算符
```
==  (eq)    等于
!=  (ne)    不等于
>   (gt)    大于
<   (lt)    小于
>=  (ge)    大于等于
<=  (le)    小于等于
```

#### 2.1.2 逻辑运算符
```
and  (&&)   逻辑与
or   (||)   逻辑或
not  (!)    逻辑非
xor  (^^)   逻辑异或
```

#### 2.1.3 搜索运算符
```
contains    包含
matches     正则表达式匹配
in          在范围内
```

### 2.2 基础过滤器

#### 2.2.1 按协议过滤
```wireshark
# 显示特定协议
http
tcp
udp
dns
icmp
arp
tls
ssh
ftp
smtp

# 排除特定协议
!http
not tcp
```

#### 2.2.2 按 IP 地址过滤
```wireshark
# 源 IP 地址
ip.src == 192.168.1.1
ip.src == 10.0.0.0/8

# 目标 IP 地址
ip.dst == 192.168.1.100

# 源或目标 IP
ip.addr == 192.168.1.1

# IP 地址段
ip.addr == 192.168.1.0/24

# 多个 IP 地址
ip.addr == 192.168.1.1 or ip.addr == 192.168.1.2

# 排除 IP
!(ip.addr == 192.168.1.1)
```

#### 2.2.3 按端口过滤
```wireshark
# TCP 端口
tcp.port == 80
tcp.srcport == 443
tcp.dstport == 8080

# UDP 端口
udp.port == 53
udp.srcport == 67

# 端口范围
tcp.port >= 1000 and tcp.port <= 2000

# 多个端口
tcp.port == 80 or tcp.port == 443 or tcp.port == 8080
```

#### 2.2.4 按 MAC 地址过滤
```wireshark
# 源 MAC 地址
eth.src == 00:11:22:33:44:55

# 目标 MAC 地址
eth.dst == AA:BB:CC:DD:EE:FF

# 任意 MAC 地址
eth.addr == 00:11:22:33:44:55
```

### 2.3 HTTP 过滤器（重点）

#### 2.3.1 HTTP 请求方法
```wireshark
# GET 请求
http.request.method == "GET"

# POST 请求
http.request.method == "POST"

# 其他方法
http.request.method == "PUT"
http.request.method == "DELETE"
http.request.method == "HEAD"
http.request.method == "OPTIONS"
```

#### 2.3.2 HTTP 状态码
```wireshark
# 所有响应
http.response

# 特定状态码
http.response.code == 200
http.response.code == 404
http.response.code == 500

# 状态码范围
http.response.code >= 400
http.response.code >= 400 and http.response.code < 500  # 4xx 错误
http.response.code >= 500  # 5xx 错误
```

#### 2.3.3 HTTP 主机和 URI
```wireshark
# 主机名
http.host == "www.example.com"
http.host contains "google"

# 完整 URI
http.request.uri == "/api/v1/users"
http.request.uri contains "/admin"
http.request.uri matches "\\.(php|asp|jsp)$"

# 请求的完整 URL
http.request.full_uri contains "password"
```

#### 2.3.4 HTTP 头部
```wireshark
# User-Agent
http.user_agent contains "Mozilla"
http.user_agent contains "curl"
http.user_agent contains "python"

# Cookie
http.cookie contains "session"
http.cookie contains "PHPSESSID"

# Referer
http.referer contains "google.com"

# Authorization
http.authorization

# Content-Type
http.content_type contains "application/json"
http.content_type == "text/html"
```

#### 2.3.5 HTTP 内容过滤
```wireshark
# 请求/响应包含特定字符串
http contains "password"
http contains "admin"
http contains "flag{"

# 请求体包含
http.request.body contains "username"

# 响应体包含
http.response.body contains "error"
```

### 2.4 TCP 过滤器（重点）

#### 2.4.1 TCP 标志位
```wireshark
# SYN 包（连接建立）
tcp.flags.syn == 1

# SYN-ACK 包
tcp.flags.syn == 1 and tcp.flags.ack == 1

# ACK 包
tcp.flags.ack == 1

# FIN 包（连接终止）
tcp.flags.fin == 1

# RST 包（连接重置）
tcp.flags.reset == 1

# PSH 包（立即推送数据）
tcp.flags.push == 1

# URG 包（紧急数据）
tcp.flags.urg == 1

# 只有 SYN 标志（三次握手第一步）
tcp.flags == 0x002
```

#### 2.4.2 TCP 连接分析
```wireshark
# TCP 三次握手
tcp.flags.syn == 1 and tcp.flags.ack == 0  # 第一步
tcp.flags.syn == 1 and tcp.flags.ack == 1  # 第二步
tcp.flags.syn == 0 and tcp.flags.ack == 1  # 第三步

# TCP 重传
tcp.analysis.retransmission

# TCP 乱序
tcp.analysis.out_of_order

# TCP 快速重传
tcp.analysis.fast_retransmission

# TCP 重复 ACK
tcp.analysis.duplicate_ack

# TCP 窗口满
tcp.analysis.window_full

# TCP 零窗口
tcp.analysis.zero_window
```

#### 2.4.3 TCP 流追踪
```wireshark
# 追踪特定 TCP 流
tcp.stream eq 0
tcp.stream eq 5

# 结合其他条件
tcp.stream eq 10 and http
```

#### 2.4.4 TCP 序列号和窗口
```wireshark
# 序列号
tcp.seq == 1000

# 确认号
tcp.ack == 2000

# 窗口大小
tcp.window_size > 0
tcp.window_size == 0  # 零窗口
```

### 2.5 DNS 过滤器

```wireshark
# DNS 查询
dns.flags.response == 0

# DNS 响应
dns.flags.response == 1

# 查询特定域名
dns.qry.name == "www.example.com"
dns.qry.name contains "google"

# DNS 查询类型
dns.qry.type == 1   # A 记录
dns.qry.type == 28  # AAAA 记录
dns.qry.type == 5   # CNAME
dns.qry.type == 15  # MX 记录
dns.qry.type == 16  # TXT 记录

# DNS 响应码
dns.flags.rcode == 0   # 无错误
dns.flags.rcode == 3   # NXDOMAIN（域名不存在）
```

### 2.6 TLS/SSL 过滤器

```wireshark
# TLS 握手
ssl.handshake.type == 1  # Client Hello
ssl.handshake.type == 2  # Server Hello

# TLS 版本
ssl.handshake.version == 0x0303  # TLS 1.2
ssl.handshake.version == 0x0304  # TLS 1.3

# 服务器名称指示（SNI）
tls.handshake.extensions_server_name contains "example.com"

# TLS 应用数据
ssl.app_data

# TLS 证书
ssl.handshake.certificate
```

### 2.7 FTP 过滤器

```wireshark
# FTP 命令
ftp.request.command == "USER"
ftp.request.command == "PASS"
ftp.request.command == "LIST"
ftp.request.command == "RETR"
ftp.request.command == "STOR"

# FTP 响应
ftp.response.code == 230  # 登录成功
ftp.response.code == 530  # 登录失败

# FTP 数据传输
ftp-data
```

### 2.8 ICMP 过滤器

```wireshark
# ICMP 类型
icmp.type == 8   # Echo Request (ping)
icmp.type == 0   # Echo Reply
icmp.type == 3   # Destination Unreachable
icmp.type == 11  # Time Exceeded

# Ping 请求和响应
icmp.type == 8 or icmp.type == 0
```

### 2.9 ARP 过滤器

```wireshark
# ARP 请求
arp.opcode == 1

# ARP 响应
arp.opcode == 2

# 查找特定 IP 的 ARP
arp.src.proto_ipv4 == 192.168.1.1
arp.dst.proto_ipv4 == 192.168.1.1
```

### 2.10 数据包长度过滤

```wireshark
# 按长度过滤
frame.len > 1000
frame.len < 100
frame.len >= 64 and frame.len <= 128

# IP 层长度
ip.len > 500

# TCP 载荷长度
tcp.len > 0  # 有数据的 TCP 包
tcp.len == 0  # 无数据的 TCP 包（如握手包）
```

### 2.11 时间过滤

```wireshark
# 时间间隔
frame.time_delta > 1  # 与上一个包间隔超过 1 秒

# 相对时间
frame.time_relative > 10  # 相对于第一个包超过 10 秒
```

### 2.12 字符串搜索（重点）

```wireshark
# 包含特定字符串（任何层）
frame contains "password"
frame contains "flag{"
frame contains "admin"

# 特定协议包含字符串
http contains "login"
tcp contains "secret"
udp contains "key"

# 大小写不敏感（使用 matches）
http.request.uri matches "(?i)admin"

# 正则表达式
http.host matches ".*\\.example\\.com"
http.request.uri matches "/api/v[0-9]+/"
```

### 2.13 组合过滤器（高级）

```wireshark
# HTTP POST 请求到特定主机
http.request.method == "POST" and http.host == "api.example.com"

# 来自特定 IP 的 HTTP 流量
http and ip.src == 192.168.1.100

# 非标准端口的 HTTP
http and tcp.port != 80 and tcp.port != 443

# 大数据包
ip.len > 1000 and tcp

# 可疑的 DNS 查询
dns and frame contains "exe"

# 失败的 HTTP 请求
http.response.code >= 400

# TLS 握手失败
ssl.alert_message

# 扫描行为检测
tcp.flags.syn == 1 and tcp.flags.ack == 0 and tcp.window_size <= 1024

# SQL 注入尝试
http.request.uri contains "select" or http.request.uri contains "union"

# XSS 尝试
http contains "<script>" or http contains "javascript:"

# 敏感信息泄露
http contains "password" or http contains "token" or http contains "api_key"
```

---

## 3. Capture Filter（捕获过滤器）详解

### 3.1 基本语法

捕获过滤器使用 **BPF（Berkeley Packet Filter）** 语法。

#### 3.1.1 基本格式
```
[协议] [方向] [类型] [值]
```

### 3.2 常用捕获过滤器

#### 3.2.1 按主机过滤
```
# 特定主机
host 192.168.1.1

# 源主机
src host 192.168.1.1

# 目标主机
dst host 192.168.1.1

# 排除主机
not host 192.168.1.1
```

#### 3.2.2 按网络过滤
```
# 整个网段
net 192.168.1.0/24
net 192.168.0.0 mask 255.255.0.0

# 源网络
src net 10.0.0.0/8

# 目标网络
dst net 172.16.0.0/16
```

#### 3.2.3 按端口过滤
```
# 特定端口
port 80
port 443

# 源端口
src port 12345

# 目标端口
dst port 22

# 端口范围
portrange 1000-2000

# 多个端口
port 80 or port 443 or port 8080
```

#### 3.2.4 按协议过滤
```
# TCP
tcp

# UDP
udp

# ICMP
icmp

# ARP
arp

# 组合协议和端口
tcp port 80
udp port 53
```

#### 3.2.5 逻辑组合
```
# AND
host 192.168.1.1 and port 80

# OR
host 192.168.1.1 or host 192.168.1.2

# NOT
not port 22

# 复杂组合
(host 192.168.1.1 or host 192.168.1.2) and (port 80 or port 443)

# 排除特定流量
not broadcast and not multicast and not arp
```

#### 3.2.6 MAC 地址过滤
```
# 特定 MAC
ether host 00:11:22:33:44:55

# 源 MAC
ether src 00:11:22:33:44:55

# 目标 MAC
ether dst AA:BB:CC:DD:EE:FF

# 广播
ether broadcast

# 多播
ether multicast
```

#### 3.2.7 高级捕获过滤器
```
# 只捕获 TCP SYN 包
tcp[tcpflags] & tcp-syn != 0

# 只捕获 TCP SYN-ACK 包
tcp[tcpflags] & (tcp-syn|tcp-ack) == (tcp-syn|tcp-ack)

# 只捕获 TCP RST 包
tcp[tcpflags] & tcp-rst != 0

# HTTP GET 请求（简化）
tcp dst port 80 and tcp[((tcp[12:1] & 0xf0) >> 2):4] = 0x47455420

# DNS 查询
udp port 53

# 大于 1000 字节的包
greater 1000

# 小于 100 字节的包
less 100

# VLAN 流量
vlan
vlan 100
```

### 3.3 Capture vs Display Filter 对比

| 功能 | Capture Filter | Display Filter |
|------|----------------|----------------|
| 语法 | BPF | Wireshark 特有 |
| 时机 | 捕获前 | 捕获后 |
| 性能 | 减少磁盘占用 | 不影响捕获 |
| 灵活性 | 较低 | 非常灵活 |
| 撤销 | 不可撤销 | 可随时修改 |

---

## 4. 常用协议过滤器

### 4.1 Web 协议

```wireshark
# HTTP/HTTPS
http or tls

# WebSocket
websocket

# HTTP/2
http2
```

### 4.2 邮件协议

```wireshark
# SMTP（发送邮件）
smtp

# POP3（接收邮件）
pop

# IMAP（接收邮件）
imap
```

### 4.3 文件传输

```wireshark
# FTP
ftp or ftp-data

# SFTP（SSH 文件传输）
ssh

# SMB/CIFS（Windows 文件共享）
smb or smb2
```

### 4.4 远程访问

```wireshark
# SSH
ssh

# Telnet
telnet

# RDP（远程桌面）
rdp

# VNC
vnc
```

### 4.5 数据库

```wireshark
# MySQL
mysql

# PostgreSQL
pgsql

# MSSQL
tds

# MongoDB
mongo
```

### 4.6 认证协议

```wireshark
# Kerberos
kerberos

# LDAP
ldap

# RADIUS
radius
```

### 4.7 VPN 协议

```wireshark
# IPSec
esp or ah or isakmp

# OpenVPN
openvpn

# PPTP
pptp
```

---

## 5. CTF/安全分析常用过滤器

### 5.1 数据提取

```wireshark
# 导出 HTTP 对象
文件 → 导出对象 → HTTP

# 提取文件
frame contains "flag{" or frame contains "FLAG{"

# Base64 编码数据
frame matches "[A-Za-z0-9+/]{20,}={0,2}"

# 查找图片
http.content_type contains "image"
frame contains "PNG" or frame contains "JFIF" or frame contains "GIF89"
```

### 5.2 凭证捕获

```wireshark
# HTTP 认证
http.authorization

# FTP 登录
ftp.request.command == "USER" or ftp.request.command == "PASS"

# Telnet 明文密码
telnet contains "password"

# HTTP POST 登录
http.request.method == "POST" and (http contains "password" or http contains "username")
```

### 5.3 恶意流量检测

```wireshark
# 可疑 User-Agent
http.user_agent contains "sqlmap" or http.user_agent contains "nmap"

# SQL 注入尝试
http.request.uri contains "union select" or http.request.uri contains "' or 1=1"

# XSS 尝试
http.request.uri contains "<script>" or http.request.uri contains "alert("

# 命令注入
http contains "| cat " or http contains "; ls " or http contains "& dir "

# 文件包含漏洞
http.request.uri contains "../" or http.request.uri contains "..%2f"

# Webshell 特征
http contains "eval(" or http contains "base64_decode" or http contains "system("
```

### 5.4 端口扫描检测

```wireshark
# SYN 扫描
tcp.flags.syn == 1 and tcp.flags.ack == 0

# 同一源 IP 的多个 SYN
tcp.flags.syn == 1 and tcp.flags.ack == 0
# 然后按 Statistics → Conversations
```

### 5.5 数据外泄检测

```wireshark
# 大量数据传输
frame.len > 10000

# DNS 隧道
dns and frame.len > 512

# ICMP 隧道
icmp and frame.len > 100
```

---

## 6. 高级技巧与实战案例

### 6.1 Follow Stream（追踪流）

```
右键数据包 → Follow → TCP Stream / UDP Stream / HTTP Stream / TLS Stream
```

快捷键：`Ctrl+Alt+Shift+T`（TCP Stream）

### 6.2 统计功能

```wireshark
# 协议层次统计
Statistics → Protocol Hierarchy

# 对话统计
Statistics → Conversations

# 端点统计
Statistics → Endpoints

# HTTP 请求统计
Statistics → HTTP → Requests

# IO 图表
Statistics → IO Graph
```

### 6.3 专家信息

```
Analyze → Expert Information
```

显示警告、错误和注意事项。

### 6.4 导出功能

```
# 导出特定数据包
File → Export Specified Packets

# 导出对象（HTTP）
File → Export Objects → HTTP

# 导出为 CSV
File → Export Packet Dissections → As CSV
```

### 6.5 着色规则

```
View → Coloring Rules
```

自定义数据包着色以快速识别。

### 6.6 过滤器宏

创建可重用的过滤器表达式：

```
Analyze → Display Filter Macros
```

### 6.7 实战案例

#### 案例 1：分析 HTTP 登录流程
```wireshark
http.request.method == "POST" and http.request.uri contains "login"
```

#### 案例 2：查找 Flag
```wireshark
frame contains "flag{" or frame contains "FLAG{" or frame contains "CTF{"
```

#### 案例 3：检测 ARP 欺骗
```wireshark
arp.duplicate-address-detected or arp.duplicate-address-frame
```

#### 案例 4：分析 DNS 异常
```wireshark
dns.flags.rcode != 0  # 非正常响应
dns.qry.name matches ".*\\.[a-z]{10,}$"  # 异常长的域名
```

#### 案例 5：提取传输的文件
```wireshark
http.content_type contains "application/octet-stream"
```
然后：File → Export Objects → HTTP

#### 案例 6：查找隐藏数据
```wireshark
# ICMP 负载异常
icmp and frame.len > 100

# DNS TXT 记录
dns.txt

# HTTP 响应中的 base64
http.response and frame matches "[A-Za-z0-9+/]{50,}={0,2}"
```

---

## 7. 快捷键参考

| 快捷键 | 功能 |
|--------|------|
| `Ctrl+E` | 开始/停止捕获 |
| `Ctrl+K` | 捕获选项 |
| `Ctrl+F` | 查找数据包 |
| `Ctrl+N` | 查找下一个 |
| `Ctrl+G` | 跳转到数据包 |
| `Ctrl+M` | 标记数据包 |
| `Ctrl+Alt+Shift+T` | Follow TCP Stream |
| `Ctrl+R` | 应用显示过滤器 |
| `Ctrl+/` | 清除显示过滤器 |
| `Ctrl+→` | 展开所有 |
| `Ctrl+←` | 折叠所有 |

---

## 8. 过滤器速查表

### 8.1 按协议

| 过滤器 | 说明 |
|--------|------|
| `http` | HTTP 协议 |
| `dns` | DNS 协议 |
| `tcp` | TCP 协议 |
| `udp` | UDP 协议 |
| `icmp` | ICMP 协议 |
| `arp` | ARP 协议 |
| `tls/ssl` | TLS/SSL 协议 |

### 8.2 按 IP

| 过滤器 | 说明 |
|--------|------|
| `ip.src == x.x.x.x` | 源 IP |
| `ip.dst == x.x.x.x` | 目标 IP |
| `ip.addr == x.x.x.x` | 任意方向 IP |

### 8.3 按端口

| 过滤器 | 说明 |
|--------|------|
| `tcp.port == 80` | TCP 端口 |
| `udp.port == 53` | UDP 端口 |
| `tcp.srcport == 443` | TCP 源端口 |
| `tcp.dstport == 22` | TCP 目标端口 |

### 8.4 按内容

| 过滤器 | 说明 |
|--------|------|
| `frame contains "text"` | 包含字符串 |
| `http.request.uri contains "admin"` | URI 包含 |
| `http.host == "example.com"` | HTTP 主机 |

---

## 9. 命令行工具

### 9.1 tshark（命令行 Wireshark）

```bash
# 捕获并显示
tshark -i eth0

# 捕获到文件
tshark -i eth0 -w output.pcap

# 读取文件
tshark -r input.pcap

# 应用显示过滤器
tshark -r input.pcap -Y "http"

# 捕获过滤器
tshark -i eth0 -f "port 80"

# 提取特定字段
tshark -r input.pcap -Y "http" -T fields -e http.host -e http.request.uri

# 统计
tshark -r input.pcap -q -z http,tree
```

### 9.2 editcap（编辑 pcap 文件）

```bash
# 分割文件（按数量）
editcap -c 1000 input.pcap output.pcap

# 分割文件（按时间，秒）
editcap -i 60 input.pcap output.pcap

# 提取特定时间范围
editcap -A "2024-01-01 00:00:00" -B "2024-01-02 00:00:00" input.pcap output.pcap
```

### 9.3 mergecap（合并 pcap 文件）

```bash
mergecap -w output.pcap file1.pcap file2.pcap file3.pcap
```

---

## 10. CTF 实战技巧

### 10.1 快速定位 Flag

```wireshark
# 直接搜索 Flag 格式
frame contains "flag{" or frame contains "CTF{" or frame contains "FLAG{"

# 正则搜索
frame matches "flag\\{[a-zA-Z0-9_]+\\}"

# 搜索 base64 可能的 Flag
frame matches "[A-Za-z0-9+/]{20,}={0,2}"
```

### 10.2 数据提取流程

1. **查看协议层次**：`Statistics → Protocol Hierarchy`
2. **查看 HTTP 对象**：`File → Export Objects → HTTP`
3. **追踪流**：右键 → `Follow → TCP/HTTP Stream`
4. **搜索关键字**：`Ctrl+F` → String/Regex
5. **导出数据包**：`File → Export Specified Packets`

### 10.3 隐写分析

```wireshark
# 查找图片文件
http.content_type contains "image" or frame contains "\x89PNG" or frame contains "JFIF"

# 查找压缩文件
frame contains "PK\x03\x04"  # ZIP
frame contains "Rar!"        # RAR

# 查找可执行文件
frame contains "MZ"  # Windows PE

# ICMP 数据隐写
icmp and frame.len > 100
```

### 10.4 协议分析

```wireshark
# 异常 DNS
dns.qry.name.len > 50

# 异常 HTTP 头
http.request.line.len > 1000

# 非标准端口
http and !(tcp.port == 80 or tcp.port == 443 or tcp.port == 8080)
```

** 如何查看完整的三次握手**

  1. 找到对应的 TCP Stream：
    tcp.stream eq <数字>
  2. 或者在主数据包列表中按 Packet 号排序，在 1745 之后应该能看到一个 [ACK]
    包
  3. 使用过滤器查看握手：
    tcp.port == 55104 and tcp.flags.syn == 1

​       tcp.stream eq <对应的stream号> and tcp.seq <= 10

---

## 11. 性能优化建议

### 11.1 捕获优化

1. **使用捕获过滤器**减少数据量
2. **限制捕获接口**只捕获需要的接口
3. **设置快照长度**：`-s 96`（只捕获前 96 字节）
4. **使用环形缓冲区**：避免磁盘写满

### 11.2 分析优化

1. **关闭不需要的协议解析**：`Analyze → Enabled Protocols`
2. **使用配置文件**：不同场景使用不同配置
3. **分割大文件**：使用 editcap 分割
4. **禁用名称解析**：`View → Name Resolution`

---

## 12. 常见问题

### Q1: 为什么看不到 HTTPS 内容？
**A**: HTTPS 流量是加密的，需要导入服务器私钥或配置浏览器导出 SSL 密钥日志文件。

```
设置方法：
Edit → Preferences → Protocols → TLS → (Pre)-Master-Secret log filename
```

### Q2: 如何捕获 localhost 流量？
**A**:
- Windows: 安装 npcap 并选择 Adapter for loopback traffic capture
- Linux: 使用 `lo` 接口
- macOS: 创建回环接口 `sudo ifconfig lo0 alias 127.0.0.2`

### Q3: 过滤器显示红色？
**A**: 语法错误，检查：
- 拼写错误
- 运算符使用错误
- 括号不匹配

### Q4: 如何保存过滤器？
**A**:
```
显示过滤器工具栏右侧 → + 按钮 → 保存
或：Analyze → Display Filter Expression → Favorites
```

---

## 13. 学习资源

### 官方文档
- Wireshark 官方网站：https://www.wireshark.org
- Display Filter 参考：https://wiki.wireshark.org/DisplayFilters
- Capture Filter 参考：https://wiki.wireshark.org/CaptureFilters

### 推荐练习
- Wireshark Sample Captures：https://wiki.wireshark.org/SampleCaptures
- CTF 题目练习：
  - picoCTF
  - CTFtime
  - HackTheBox

### 书籍推荐
- 《Wireshark网络分析就这么简单》
- 《Wireshark网络分析的艺术》
- 《Practical Packet Analysis》

---

## 附录：常用端口号参考

| 端口 | 协议 | 说明 |
|------|------|------|
| 20/21 | FTP | 文件传输 |
| 22 | SSH | 安全远程登录 |
| 23 | Telnet | 远程登录 |
| 25 | SMTP | 邮件发送 |
| 53 | DNS | 域名解析 |
| 80 | HTTP | Web 服务 |
| 110 | POP3 | 邮件接收 |
| 143 | IMAP | 邮件接收 |
| 443 | HTTPS | 安全 Web 服务 |
| 445 | SMB | Windows 文件共享 |
| 3306 | MySQL | 数据库 |
| 3389 | RDP | 远程桌面 |
| 5432 | PostgreSQL | 数据库 |
| 6379 | Redis | 缓存数据库 |
| 8080 | HTTP Alt | 备用 Web 端口 |
| 27017 | MongoDB | 数据库 |

---

**最后更新**: 2025-10-16
**作者**: CTF/Security Analysis Documentation
**版本**: v1.0

**提示**: 建议将本文档与实际 pcap 文件结合练习，才能熟练掌握 Wireshark！
