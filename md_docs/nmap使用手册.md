# Nmap 使用手册

## 简介

Nmap (Network Mapper) 是一个开源的网络探测和安全审计工具。它被设计用于快速扫描大型网络,也可以用于单个主机。Nmap使用原始IP数据包来确定网络上可用的主机、这些主机提供的服务、运行的操作系统、使用的防火墙类型等信息。

### 主要功能
- 🔍 主机发现 (Host Discovery)
- 🚪 端口扫描 (Port Scanning)
- 🔬 服务版本检测 (Version Detection)
- 💻 操作系统检测 (OS Detection)
- 🔥 防火墙/IDS规避 (Firewall Evasion)
- 📝 脚本引擎 (NSE - Nmap Scripting Engine)

### 安装

#### macOS
```bash
# Homebrew安装
brew install nmap

# 验证安装
nmap --version
```

#### Linux (Debian/Ubuntu)
```bash
sudo apt update
sudo apt install nmap

# 验证安装
nmap --version
```

#### Linux (RHEL/CentOS/Fedora)
```bash
sudo yum install nmap
# 或
sudo dnf install nmap
```

#### Windows
从官网下载安装包: https://nmap.org/download.html

## 基本用法

### 基本语法
```bash
nmap [扫描类型] [选项] {目标}
```

### 目标指定方式

```bash
# 单个IP
nmap 192.168.1.1

# 主机名
nmap scanme.nmap.org

# 多个目标
nmap 192.168.1.1 192.168.1.2 192.168.1.3

# IP范围
nmap 192.168.1.1-254
nmap 192.168.1.0/24

# CIDR表示法
nmap 192.168.1.0/24

# 从文件读取目标
nmap -iL targets.txt

# 随机选择目标
nmap -iR 100  # 随机扫描100个主机

# 排除目标
nmap 192.168.1.0/24 --exclude 192.168.1.1
nmap 192.168.1.0/24 --excludefile exclude.txt
```

## 主机发现

主机发现用于确定网络中哪些主机是活动的。

### 主机发现选项

| 选项 | 说明 |
|------|------|
| `-sL` | 列表扫描,仅列出目标 |
| `-sn` | Ping扫描,禁用端口扫描 |
| `-Pn` | 跳过主机发现,假定所有主机在线 |
| `-PS [端口列表]` | TCP SYN Ping |
| `-PA [端口列表]` | TCP ACK Ping |
| `-PU [端口列表]` | UDP Ping |
| `-PE/PP/PM` | ICMP Ping (Echo/Timestamp/Netmask) |
| `-PR` | ARP Ping (局域网) |
| `-n` | 不进行DNS解析 |
| `-R` | 始终进行DNS解析 |
| `--dns-servers` | 指定DNS服务器 |
| `--system-dns` | 使用系统DNS解析器 |

### 主机发现示例

```bash
# 列表扫描(仅列出目标,不扫描)
nmap -sL 192.168.1.0/24

# Ping扫描(仅发现主机,不扫描端口)
nmap -sn 192.168.1.0/24

# 跳过主机发现(假定所有主机在线)
nmap -Pn 192.168.1.1

# TCP SYN Ping(向80端口发送SYN包)
nmap -PS80 192.168.1.1

# TCP ACK Ping(向443端口发送ACK包)
nmap -PA443 192.168.1.1

# UDP Ping
nmap -PU53 192.168.1.1

# ICMP Echo Ping
nmap -PE 192.168.1.1

# ARP Ping(局域网最有效)
nmap -PR 192.168.1.0/24

# 不进行DNS解析(加快扫描速度)
nmap -n 192.168.1.1

# 组合使用
nmap -sn -PS22,80,443 -PA80,443 192.168.1.0/24
```

## 端口扫描技术

### 端口扫描类型

| 选项 | 扫描类型 | 说明 |
|------|----------|------|
| `-sS` | TCP SYN扫描 | 半开放扫描,默认扫描方式,需要root权限 |
| `-sT` | TCP Connect扫描 | 完整TCP连接,不需要特权 |
| `-sU` | UDP扫描 | UDP端口扫描,速度较慢 |
| `-sA` | TCP ACK扫描 | 用于检测防火墙规则 |
| `-sW` | TCP窗口扫描 | 类似ACK扫描,可区分开放/关闭端口 |
| `-sM` | TCP Maimon扫描 | 发送FIN/ACK包 |
| `-sN/-sF/-sX` | TCP NULL/FIN/Xmas扫描 | 隐蔽扫描技术 |
| `-sI` | 空闲扫描 | 极其隐蔽的扫描 |
| `-sY` | SCTP INIT扫描 | SCTP协议扫描 |
| `-sZ` | SCTP COOKIE-ECHO扫描 | SCTP协议扫描 |
| `-sO` | IP协议扫描 | 确定支持的IP协议 |
| `-b` | FTP反弹扫描 | 通过FTP服务器扫描 |

### 端口扫描示例

```bash
# TCP SYN扫描(默认,隐蔽且快速)
sudo nmap -sS 192.168.1.1

# TCP Connect扫描(无需root)
nmap -sT 192.168.1.1

# UDP扫描(查找DNS、SNMP等服务)
sudo nmap -sU 192.168.1.1

# 同时扫描TCP和UDP
sudo nmap -sS -sU 192.168.1.1

# TCP ACK扫描(探测防火墙规则)
sudo nmap -sA 192.168.1.1

# TCP NULL扫描(隐蔽扫描)
sudo nmap -sN 192.168.1.1

# TCP FIN扫描
sudo nmap -sF 192.168.1.1

# TCP Xmas扫描
sudo nmap -sX 192.168.1.1

# 空闲扫描(使用僵尸主机)
sudo nmap -sI zombie_host target_host

# IP协议扫描
sudo nmap -sO 192.168.1.1
```

### 端口指定

```bash
# 扫描特定端口
nmap -p 22 192.168.1.1
nmap -p 22,80,443 192.168.1.1

# 端口范围
nmap -p 1-100 192.168.1.1
nmap -p 1-65535 192.168.1.1

# 常用端口(默认扫描1000个常用端口)
nmap 192.168.1.1

# 快速扫描(100个最常用端口)
nmap -F 192.168.1.1

# 扫描所有65535个端口
nmap -p- 192.168.1.1
nmap -p 1-65535 192.168.1.1

# 按服务名指定端口
nmap -p http,https,ssh 192.168.1.1

# 按协议指定端口
nmap -p U:53,T:80,443 192.168.1.1  # U=UDP, T=TCP

# 扫描前N个常用端口
nmap --top-ports 10 192.168.1.1
nmap --top-ports 100 192.168.1.1

# 按端口使用频率扫描
nmap --port-ratio 0.1 192.168.1.1  # 扫描出现频率>10%的端口
```

## 服务和版本检测

### 版本检测选项

| 选项 | 说明 |
|------|------|
| `-sV` | 探测开放端口的服务版本 |
| `--version-intensity <0-9>` | 设置版本检测强度(默认7) |
| `--version-light` | 轻量级版本检测(强度2) |
| `--version-all` | 尝试所有探测(强度9) |
| `--version-trace` | 显示版本检测详细信息 |

### 服务检测示例

```bash
# 基本版本检测
nmap -sV 192.168.1.1

# 轻量级版本检测(更快但可能不准确)
nmap -sV --version-light 192.168.1.1

# 高强度版本检测(更准确但更慢)
nmap -sV --version-all 192.168.1.1

# 指定检测强度(0-9)
nmap -sV --version-intensity 5 192.168.1.1

# 显示版本检测详细过程
nmap -sV --version-trace 192.168.1.1

# 组合使用
nmap -sS -sV -p- 192.168.1.1
```

## 操作系统检测

### OS检测选项

| 选项 | 说明 |
|------|------|
| `-O` | 启用OS检测 |
| `--osscan-limit` | 限制OS检测只针对有希望的主机 |
| `--osscan-guess` | 猜测OS检测结果 |
| `--fuzzy` | 模糊OS检测 |

### OS检测示例

```bash
# 基本OS检测
sudo nmap -O 192.168.1.1

# OS检测+版本检测
sudo nmap -O -sV 192.168.1.1

# 激进的OS检测和猜测
sudo nmap -O --osscan-guess 192.168.1.1

# 限制OS检测范围(提高准确性)
sudo nmap -O --osscan-limit 192.168.1.1

# 完整的信息收集扫描
sudo nmap -A 192.168.1.1  # 包含OS、版本、脚本、路由追踪
```

## NSE脚本引擎

Nmap脚本引擎(NSE)允许用户编写和共享简单的脚本来自动化各种网络任务。

### 脚本类别

| 类别 | 说明 |
|------|------|
| `auth` | 认证相关脚本 |
| `broadcast` | 网络广播发现 |
| `brute` | 暴力破解 |
| `default` | 默认脚本(-sC使用) |
| `discovery` | 网络发现 |
| `dos` | 拒绝服务测试 |
| `exploit` | 漏洞利用 |
| `external` | 使用外部资源 |
| `fuzzer` | 模糊测试 |
| `intrusive` | 可能影响目标的脚本 |
| `malware` | 恶意软件检测 |
| `safe` | 安全的脚本 |
| `version` | 版本检测增强 |
| `vuln` | 漏洞检测 |

### NSE脚本选项

| 选项 | 说明 |
|------|------|
| `-sC` | 使用默认脚本集 |
| `--script=<脚本>` | 运行指定脚本 |
| `--script-args=<参数>` | 传递脚本参数 |
| `--script-args-file=<文件>` | 从文件读取脚本参数 |
| `--script-trace` | 显示脚本执行详情 |
| `--script-updatedb` | 更新脚本数据库 |
| `--script-help=<脚本>` | 显示脚本帮助 |

### NSE脚本示例

```bash
# 使用默认脚本
nmap -sC 192.168.1.1

# 使用特定脚本
nmap --script=http-title 192.168.1.1

# 使用多个脚本
nmap --script=http-title,http-headers 192.168.1.1

# 使用脚本类别
nmap --script=vuln 192.168.1.1
nmap --script=default,safe 192.168.1.1

# 使用通配符
nmap --script="http-*" 192.168.1.1
nmap --script="ssh-*" 192.168.1.1

# 排除特定脚本
nmap --script="default and not http-slowloris" 192.168.1.1

# 传递脚本参数
nmap --script=http-put --script-args http-put.url='/uploads/test.txt',http-put.file='test.txt' 192.168.1.1

# 显示脚本帮助
nmap --script-help=http-title

# 更新脚本数据库
nmap --script-updatedb

# 脚本调试
nmap --script=http-title --script-trace 192.168.1.1
```

### 常用NSE脚本

#### 漏洞扫描
```bash
# 通用漏洞扫描
nmap --script=vuln 192.168.1.1

# SMB漏洞(如永恒之蓝)
nmap --script=smb-vuln-* 192.168.1.1

# HTTP漏洞
nmap --script=http-vuln-* 192.168.1.1

# SSL/TLS漏洞
nmap --script=ssl-heartbleed,ssl-poodle 192.168.1.1
```

#### 暴力破解
```bash
# HTTP基本认证暴力破解
nmap --script=http-brute --script-args userdb=users.txt,passdb=pass.txt 192.168.1.1

# SSH暴力破解
nmap -p 22 --script=ssh-brute --script-args userdb=users.txt 192.168.1.1

# FTP暴力破解
nmap -p 21 --script=ftp-brute 192.168.1.1

# MySQL暴力破解
nmap -p 3306 --script=mysql-brute 192.168.1.1
```

#### 信息收集
```bash
# HTTP信息收集
nmap --script=http-title,http-headers,http-methods 192.168.1.1

# SMB信息收集
nmap --script=smb-os-discovery,smb-enum-shares 192.168.1.1

# DNS信息收集
nmap --script=dns-brute domain.com

# WHOIS查询
nmap --script=whois-domain,whois-ip 192.168.1.1

# 地理位置
nmap --script=ip-geolocation-* 192.168.1.1
```

#### Web应用扫描
```bash
# 检测WAF
nmap --script=http-waf-detect 192.168.1.1

# 目录枚举
nmap --script=http-enum 192.168.1.1

# 备份文件检测
nmap --script=http-backup-finder 192.168.1.1

# SQL注入检测
nmap --script=http-sql-injection 192.168.1.1

# XSS检测
nmap --script=http-unsafe-output-escaping 192.168.1.1

# 检测HTTP方法
nmap --script=http-methods 192.168.1.1
```

#### 数据库扫描
```bash
# MySQL枚举
nmap -p 3306 --script=mysql-enum 192.168.1.1

# MySQL数据库列表
nmap -p 3306 --script=mysql-databases --script-args mysqluser=root,mysqlpass=pass 192.168.1.1

# MongoDB信息
nmap -p 27017 --script=mongodb-info 192.168.1.1

# PostgreSQL信息
nmap -p 5432 --script=pgsql-brute 192.168.1.1
```

#### 邮件服务
```bash
# SMTP枚举用户
nmap -p 25 --script=smtp-enum-users 192.168.1.1

# SMTP命令
nmap -p 25 --script=smtp-commands 192.168.1.1

# 开放中继检测
nmap -p 25 --script=smtp-open-relay 192.168.1.1
```

## 时序和性能

### 时序模板

| 选项 | 模板 | 说明 |
|------|------|------|
| `-T0` | Paranoid | 极慢,用于IDS规避 |
| `-T1` | Sneaky | 慢,用于IDS规避 |
| `-T2` | Polite | 降低扫描速度,减少带宽和目标资源占用 |
| `-T3` | Normal | 默认速度 |
| `-T4` | Aggressive | 快速扫描,假定处于快速可靠的网络 |
| `-T5` | Insane | 非常快,可能丢失信息 |

### 性能选项

| 选项 | 说明 |
|------|------|
| `--min-hostgroup <数量>` | 并行扫描主机组的最小大小 |
| `--max-hostgroup <数量>` | 并行扫描主机组的最大大小 |
| `--min-parallelism <数量>` | 最小并行探测数 |
| `--max-parallelism <数量>` | 最大并行探测数 |
| `--min-rtt-timeout <时间>` | 最小RTT超时 |
| `--max-rtt-timeout <时间>` | 最大RTT超时 |
| `--initial-rtt-timeout <时间>` | 初始RTT超时 |
| `--max-retries <次数>` | 最大重试次数 |
| `--host-timeout <时间>` | 主机超时 |
| `--scan-delay <时间>` | 探测间延迟 |
| `--max-scan-delay <时间>` | 最大探测间延迟 |
| `--min-rate <数量>` | 最小发包速率 |
| `--max-rate <数量>` | 最大发包速率 |

### 性能示例

```bash
# 快速扫描
nmap -T4 192.168.1.1

# 慢速隐蔽扫描
nmap -T0 192.168.1.1

# 自定义性能参数
nmap --min-rate=1000 --max-rate=5000 192.168.1.1

# 设置超时
nmap --host-timeout 30m 192.168.1.0/24

# 控制并行度
nmap --max-parallelism 100 192.168.1.0/24

# 添加延迟(避免触发IDS)
nmap --scan-delay 1s 192.168.1.1
```

## 防火墙/IDS规避

### 规避技术

| 选项 | 说明 |
|------|------|
| `-f` | 分片数据包 |
| `--mtu <值>` | 使用指定MTU |
| `-D <诱饵1,诱饵2>` | 使用诱饵扫描 |
| `-S <IP>` | 源地址欺骗 |
| `-e <接口>` | 使用指定网络接口 |
| `-g/--source-port <端口>` | 使用指定源端口 |
| `--data <hex>` | 添加自定义有效载荷 |
| `--data-string <字符串>` | 添加自定义ASCII字符串 |
| `--data-length <数量>` | 添加随机数据 |
| `--ip-options <选项>` | 使用IP选项 |
| `--ttl <值>` | 设置IP TTL |
| `--spoof-mac <MAC>` | MAC地址欺骗 |
| `--badsum` | 使用错误的TCP/UDP校验和 |

### 规避示例

```bash
# 分片数据包
sudo nmap -f 192.168.1.1

# 指定MTU
sudo nmap --mtu 24 192.168.1.1

# 诱饵扫描(隐藏真实IP)
sudo nmap -D RND:10 192.168.1.1
sudo nmap -D decoy1,decoy2,ME,decoy3 192.168.1.1

# 源地址欺骗
sudo nmap -S 192.168.1.100 -e eth0 -Pn 192.168.1.1

# 指定源端口(如53 DNS端口可能不被过滤)
sudo nmap --source-port 53 192.168.1.1
sudo nmap -g 53 192.168.1.1

# 添加随机数据
nmap --data-length 25 192.168.1.1

# MAC地址欺骗
sudo nmap --spoof-mac 0 192.168.1.1  # 随机MAC
sudo nmap --spoof-mac Apple 192.168.1.1  # Apple厂商
sudo nmap --spoof-mac 00:11:22:33:44:55 192.168.1.1  # 指定MAC

# 组合规避技术
sudo nmap -f -T2 -D RND:10 --source-port 53 --data-length 20 192.168.1.1
```

## 输出选项

### 输出格式

| 选项 | 格式 | 说明 |
|------|------|------|
| `-oN <文件>` | Normal | 标准输出 |
| `-oX <文件>` | XML | XML格式 |
| `-oS <文件>` | Script Kiddie | 脚本小子格式 |
| `-oG <文件>` | Grepable | 易于grep的格式 |
| `-oA <基础名>` | All | 同时输出以上所有格式 |

### 输出选项

| 选项 | 说明 |
|------|------|
| `-v` | 增加详细级别 |
| `-vv` | 更详细 |
| `-d` | 调试输出 |
| `-dd` | 更详细的调试 |
| `--reason` | 显示端口状态原因 |
| `--open` | 只显示开放端口 |
| `--packet-trace` | 显示所有发送和接收的数据包 |
| `--iflist` | 显示接口和路由 |
| `--append-output` | 追加到输出文件 |
| `--resume <文件>` | 恢复中断的扫描 |
| `--stylesheet <路径>` | 设置XSL样式表 |
| `--webxml` | 使用Nmap.org的样式表 |
| `--no-stylesheet` | 不使用XSL样式表 |

### 输出示例

```bash
# 标准输出到文件
nmap -oN scan_result.txt 192.168.1.1

# XML输出
nmap -oX scan_result.xml 192.168.1.1

# Grepable输出
nmap -oG scan_result.gnmap 192.168.1.1

# 所有格式输出
nmap -oA scan_result 192.168.1.1

# 详细输出
nmap -v 192.168.1.1
nmap -vv 192.168.1.1

# 只显示开放端口
nmap --open 192.168.1.1

# 显示端口状态原因
nmap --reason 192.168.1.1

# 数据包追踪
nmap --packet-trace 192.168.1.1

# 调试输出
nmap -d 192.168.1.1

# 恢复中断的扫描
nmap --resume scan_result.xml

# 追加输出
nmap --append-output -oN scan.txt 192.168.1.1
```

## 实战案例

### 1. 快速网络扫描
```bash
# 发现局域网活动主机
nmap -sn 192.168.1.0/24

# 快速扫描常用端口
nmap -F 192.168.1.1-254

# 扫描前100个常用端口
nmap --top-ports 100 192.168.1.0/24
```

### 2. 全面扫描单个主机
```bash
# 综合扫描(OS、版本、脚本、路由)
sudo nmap -A -T4 192.168.1.1

# 详细扫描所有端口
sudo nmap -sS -sV -O -p- -T4 -v 192.168.1.1

# 保存结果到文件
sudo nmap -A -T4 -oA full_scan 192.168.1.1
```

### 3. Web服务器扫描
```bash
# 扫描Web服务
nmap -p 80,443 --script=http-enum,http-headers,http-methods,http-title 192.168.1.1

# 检测Web漏洞
nmap -p 80,443 --script=http-vuln-* 192.168.1.1

# 检测WAF
nmap -p 80,443 --script=http-waf-detect,http-waf-fingerprint 192.168.1.1
```

### 4. 数据库服务扫描
```bash
# MySQL扫描
nmap -p 3306 --script=mysql-info,mysql-databases,mysql-variables 192.168.1.1

# MongoDB扫描
nmap -p 27017 --script=mongodb-info,mongodb-databases 192.168.1.1

# Redis扫描
nmap -p 6379 --script=redis-info 192.168.1.1
```

### 5. 漏洞扫描
```bash
# 通用漏洞扫描
nmap --script=vuln 192.168.1.1

# SMB漏洞扫描(永恒之蓝等)
nmap --script=smb-vuln-ms17-010,smb-vuln-ms08-067 192.168.1.1

# SSL/TLS漏洞
nmap --script=ssl-heartbleed,ssl-poodle,ssl-ccs-injection 192.168.1.1
```

### 6. 隐蔽扫描
```bash
# 慢速SYN扫描,使用诱饵
sudo nmap -sS -T2 -f -D RND:10 --source-port 53 192.168.1.1

# 分片+随机诱饵+源端口欺骗
sudo nmap -sS -f --mtu 24 -D RND:5 -g 53 --data-length 20 192.168.1.1
```

### 7. 内网渗透
```bash
# 发现内网活动主机
nmap -sn -T4 192.168.1.0/24 -oG alive_hosts.txt

# 扫描常见服务端口
nmap -p 21,22,23,25,80,110,139,443,445,1433,3306,3389,8080 192.168.1.0/24

# 识别操作系统
sudo nmap -O -sV --osscan-guess 192.168.1.1-254
```

### 8. IPv6扫描
```bash
# IPv6主机发现
nmap -6 fe80::1

# IPv6端口扫描
nmap -6 -p- 2001:db8::1

# IPv6服务检测
nmap -6 -sV 2001:db8::1
```

### 9. 大规模扫描
```bash
# 扫描整个C类网段
nmap -sS -T4 --min-rate=1000 192.168.1.0/24

# 扫描多个网段
nmap -iL network_list.txt -oA large_scan

# 使用并行化加速
nmap --min-hostgroup 256 --max-parallelism 100 192.168.0.0/16
```

### 10. CTF/渗透测试场景
```bash
# 快速信息收集
nmap -sS -sV -T4 --script=default,vuln -oA ctf_recon target.com

# 查找特定服务
nmap -p- --open -sV | grep -i "ssh\|ftp\|smtp"

# 检测防火墙规则
sudo nmap -sA -p 1-1000 192.168.1.1

# 综合扫描并保存结果
sudo nmap -A -T4 -p- --script=default,vuln -oA comprehensive_scan target.com
```

## 常用组合命令

### 基础扫描
```bash
# 快速扫描Top 100端口
nmap --top-ports 100 192.168.1.1

# 扫描所有TCP端口
nmap -p- 192.168.1.1

# 扫描特定端口范围
nmap -p 1-1000 192.168.1.1
```

### 服务识别
```bash
# 基本服务版本检测
nmap -sV 192.168.1.1

# 服务版本+OS检测
sudo nmap -sV -O 192.168.1.1

# 使用脚本增强检测
nmap -sV -sC 192.168.1.1
```

### 全面扫描
```bash
# 激进扫描(OS、版本、脚本、路由)
sudo nmap -A 192.168.1.1

# 完整扫描所有端口+详细输出
sudo nmap -sS -sV -O -p- -v -oA full_scan 192.168.1.1

# 最全面扫描
sudo nmap -sS -sU -T4 -A -v -p- -oA ultimate_scan 192.168.1.1
```

### 隐蔽扫描
```bash
# 基本隐蔽扫描
nmap -sS -T2 192.168.1.1

# 高级隐蔽扫描
sudo nmap -sS -T1 -f -D RND:10 -g 53 192.168.1.1
```

## 端口状态说明

Nmap扫描后会显示端口的状态:

| 状态 | 说明 |
|------|------|
| `open` | 应用程序正在监听该端口 |
| `closed` | 端口可访问,但没有应用程序监听 |
| `filtered` | 无法确定端口状态(被防火墙过滤) |
| `unfiltered` | 端口可访问,但无法确定开放或关闭 |
| `open|filtered` | 无法确定端口是开放还是被过滤 |
| `closed|filtered` | 无法确定端口是关闭还是被过滤 |

## 脚本位置和管理

### 脚本位置
```bash
# macOS (Homebrew)
/opt/homebrew/share/nmap/scripts/

# Linux
/usr/share/nmap/scripts/

# 查看所有可用脚本
ls /usr/share/nmap/scripts/ | grep -i http

# 搜索特定功能脚本
nmap --script-help "*vuln*"
```

### 更新和管理
```bash
# 更新脚本数据库
nmap --script-updatedb

# 查看脚本帮助
nmap --script-help=http-title

# 查看脚本源代码
cat /usr/share/nmap/scripts/http-title.nse
```

## 常见问题和技巧

### 1. 提高扫描速度
```bash
# 使用快速时序模板
nmap -T4 target

# 限制端口扫描范围
nmap --top-ports 1000 target

# 跳过主机发现(如果确定主机在线)
nmap -Pn target

# 不进行DNS解析
nmap -n target

# 增加并行度
nmap --min-parallelism 100 target
```

### 2. 避免检测
```bash
# 慢速扫描
nmap -T1 target

# 添加延迟
nmap --scan-delay 1s target

# 随机目标顺序
nmap --randomize-hosts target1 target2 target3

# 使用诱饵
nmap -D RND:10 target
```

### 3. 解决权限问题
```bash
# SYN扫描需要root权限
sudo nmap -sS target

# 无需root的Connect扫描
nmap -sT target

# macOS可能需要禁用防火墙
sudo pfctl -d
```

### 4. 处理大规模扫描
```bash
# 使用目标文件
nmap -iL targets.txt

# 分段扫描
nmap 192.168.1.1-100
nmap 192.168.1.101-200

# 使用Grepable输出便于后续处理
nmap -oG results.gnmap target

# 提取开放端口
grep "open" results.gnmap
```

### 5. 调试扫描问题
```bash
# 显示详细输出
nmap -v target
nmap -vv target

# 显示调试信息
nmap -d target
nmap -dd target

# 数据包追踪
nmap --packet-trace target

# 显示端口状态原因
nmap --reason target
```

## Nmap输出解析

### XML输出处理
```bash
# 扫描并输出XML
nmap -oX scan.xml target

# 使用xsltproc转换为HTML
xsltproc scan.xml -o scan.html

# 使用Python解析XML
python3 -c "import xml.etree.ElementTree as ET; tree = ET.parse('scan.xml'); root = tree.getroot(); [print(host.find('address').get('addr')) for host in root.findall('.//host')]"
```

### Grepable输出处理
```bash
# 提取所有开放端口
grep "open" scan.gnmap | cut -d " " -f 2

# 提取特定端口
grep "80/open" scan.gnmap

# 统计开放端口数量
grep -o "open" scan.gnmap | wc -l
```

## 安全和法律注意事项

⚠️ **重要提示**:
1. **授权**: 只扫描你有明确授权的系统
2. **合法性**: 未经授权的端口扫描在某些司法管辖区可能是非法的
3. **网络影响**: 大规模扫描可能影响网络性能
4. **IDS/IPS**: 扫描活动可能触发入侵检测系统警报
5. **服务中断**: 某些扫描技术可能导致服务中断

### 合法使用场景
- ✅ 自己的系统和网络
- ✅ 有书面授权的渗透测试
- ✅ 漏洞赏金计划(遵循规则)
- ✅ 安全研究(在隔离环境)
- ✅ CTF比赛和练习环境

### 禁止行为
- ❌ 扫描未经授权的系统
- ❌ 使用扫描结果进行攻击
- ❌ 干扰正常业务运营
- ❌ 未经许可的漏洞测试

## 参考资源

### 官方资源
- **官方网站**: https://nmap.org/
- **官方文档**: https://nmap.org/book/
- **NSE脚本库**: https://nmap.org/nsedoc/
- **邮件列表**: https://nmap.org/mailman/listinfo/

### 学习资源
- **Nmap Network Scanning**: Gordon Lyon的官方书籍
- **Nmap Cookbook**: 实用技巧和方案
- **SecWiki Nmap**: 中文Nmap资源
- **HackTricks Nmap**: 渗透测试技巧

### 在线工具
- **Nmap Online**: 在线Nmap扫描器(谨慎使用)
- **Shodan**: 互联网设备搜索引擎
- **Censys**: 互联网扫描数据库

## 快速参考卡

### 常用命令速查
```bash
# 基础扫描
nmap 192.168.1.1                          # 基本扫描
nmap -v 192.168.1.1                       # 详细扫描
nmap -A 192.168.1.1                       # 激进扫描
nmap -p- 192.168.1.1                      # 全端口扫描

# 主机发现
nmap -sn 192.168.1.0/24                   # Ping扫描
nmap -Pn 192.168.1.1                      # 跳过Ping

# 端口扫描
nmap -sS 192.168.1.1                      # SYN扫描
nmap -sT 192.168.1.1                      # Connect扫描
nmap -sU 192.168.1.1                      # UDP扫描

# 服务检测
nmap -sV 192.168.1.1                      # 版本检测
nmap -O 192.168.1.1                       # OS检测
nmap -sC 192.168.1.1                      # 默认脚本

# 输出
nmap -oN file.txt 192.168.1.1             # 标准输出
nmap -oX file.xml 192.168.1.1             # XML输出
nmap -oA basename 192.168.1.1             # 所有格式

# 性能
nmap -T4 192.168.1.1                      # 快速扫描
nmap -T0 192.168.1.1                      # 慢速扫描

# 规避
nmap -f 192.168.1.1                       # 分片
nmap -D RND:10 192.168.1.1                # 诱饵扫描
```

---

**版本**: Nmap 7.98
**文档更新**: 2025-10
**适用平台**: Linux, macOS, Windows
**许可证**: GPL-2.0
