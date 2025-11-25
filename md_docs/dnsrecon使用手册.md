# DNSRecon 使用手册

## 目录
- [简介](#简介)
- [安装](#安装)
- [基本概念](#基本概念)
- [基本用法](#基本用法)
- [枚举类型详解](#枚举类型详解)
- [常用参数](#常用参数)
- [实战场景](#实战场景)
  - [标准DNS枚举](#标准dns枚举)
  - [子域名发现](#子域名发现)
  - [区域传送测试](#区域传送测试)
  - [反向DNS查询](#反向dns查询)
  - [SRV记录枚举](#srv记录枚举)
  - [DNSSEC区域遍历](#dnssec区域遍历)
  - [缓存侦察](#缓存侦察)
- [高级技巧](#高级技巧)
- [输出格式](#输出格式)
- [字典文件](#字典文件)
- [实战案例](#实战案例)
- [与其他工具组合](#与其他工具组合)
- [常见问题](#常见问题)
- [最佳实践](#最佳实践)

---

## 简介

**DNSRecon** 是一个用 Python 编写的 DNS 枚举和侦察工具，由 Carlos Perez (darkoperator) 开发。它是渗透测试信息收集阶段的核心工具之一。

### 主要功能

- 🔍 **DNS记录枚举** - 查询各种DNS记录类型
- 🌐 **子域名发现** - 通过字典暴力破解或智能枚举
- 🔄 **区域传送测试** - 检测AXFR漏洞
- 📊 **反向DNS查询** - IP到域名映射
- 🎯 **SRV记录枚举** - 服务发现
- 🗺️ **DNS基础设施拓扑** - 绘制DNS架构
- 🔐 **DNSSEC遍历** - 利用DNSSEC进行枚举
- 💾 **缓存侦察** - DNS缓存分析

### 工作原理

DNSRecon 通过以下方式收集DNS信息：
1. 标准DNS查询（A、AAAA、MX、NS、SOA等）
2. 区域传送请求（AXFR/IXFR）
3. 字典暴力破解子域名
4. DNSSEC NSEC/NSEC3遍历
5. 反向PTR记录查询
6. SRV记录枚举

---

## 安装

### Kali Linux / ParrotOS
```bash
# 系统预装，直接使用
dnsrecon -h

# 更新到最新版本
sudo apt update
sudo apt install dnsrecon

# 或使用pip更新
sudo pip3 install dnsrecon --upgrade
```

### Ubuntu / Debian
```bash
# 从apt安装
sudo apt update
sudo apt install dnsrecon

# 或使用pip
sudo pip3 install dnsrecon
```

### macOS
```bash
# 使用pip安装
pip3 install dnsrecon

# 或使用Homebrew（如果有tap）
brew install dnsrecon
```

### 通用安装（使用pip）
```bash
# Python 3.6+
pip3 install dnsrecon

# 使用虚拟环境（推荐）
python3 -m venv dnsrecon-env
source dnsrecon-env/bin/activate
pip install dnsrecon
```

### 从源码安装
```bash
# 克隆仓库
git clone https://github.com/darkoperator/dnsrecon.git
cd dnsrecon

# 安装依赖
pip3 install -r requirements.txt

# 运行
python3 dnsrecon.py -h

# 可选：添加到PATH
sudo cp dnsrecon.py /usr/local/bin/dnsrecon
sudo chmod +x /usr/local/bin/dnsrecon
```

### 验证安装
```bash
dnsrecon -h
# 或
python3 dnsrecon.py -h

# 检查版本
dnsrecon --version
```

### 依赖项
```bash
# 核心依赖
- Python 3.6+
- dnspython
- netaddr
- lxml (可选，用于XML输出)
```

---

## 基本概念

### DNS记录类型

| 记录类型 | 说明 | 示例 |
|---------|------|------|
| **A** | IPv4地址 | example.com → 93.184.216.34 |
| **AAAA** | IPv6地址 | example.com → 2606:2800:220:1:... |
| **MX** | 邮件服务器 | example.com → mail.example.com |
| **NS** | 域名服务器 | example.com → ns1.example.com |
| **SOA** | 授权起始 | 域名的主要信息 |
| **TXT** | 文本记录 | SPF、DKIM、验证信息 |
| **CNAME** | 别名 | www.example.com → example.com |
| **PTR** | 反向解析 | 93.184.216.34 → example.com |
| **SRV** | 服务记录 | _http._tcp.example.com |
| **CAA** | 证书颁发机构授权 | 限制谁可以颁发SSL证书 |

### DNS枚举方法

1. **标准查询** - 直接查询DNS记录
2. **区域传送 (AXFR)** - 从DNS服务器复制整个区域文件
3. **暴力破解** - 使用字典测试子域名
4. **反向查询** - 从IP查找域名
5. **DNSSEC遍历** - 利用NSEC/NSEC3记录
6. **缓存嗅探** - 检查DNS缓存

---

## 基本用法

### 命令格式
```bash
dnsrecon [选项]
```

### 最简单的使用
```bash
# 标准DNS枚举
dnsrecon -d example.com
```

### 快速示例
```bash
# 标准枚举
dnsrecon -d example.com

# 区域传送
dnsrecon -d example.com -t axfr

# 子域名暴力破解
dnsrecon -d example.com -t brt -D /usr/share/wordlists/subdomains.txt

# 反向查询
dnsrecon -r 192.168.1.0/24

# 指定DNS服务器
dnsrecon -d example.com -n 8.8.8.8

# 保存结果
dnsrecon -d example.com -j results.json
```

---

## 枚举类型详解

### 1. 标准枚举 (std)
**默认枚举类型**，查询常见DNS记录。

```bash
dnsrecon -d example.com -t std
```

**查询的记录类型：**
- SOA (授权起始)
- NS (域名服务器)
- A (IPv4地址)
- AAAA (IPv6地址)
- MX (邮件服务器)
- TXT (文本记录)

**输出示例：**
```
[*] Performing General Enumeration of Domain: example.com
[*] SOA ns1.example.com 93.184.216.34
[*] NS ns1.example.com 93.184.216.34
[*] NS ns2.example.com 93.184.216.35
[*] MX mail.example.com 10
[*] A example.com 93.184.216.34
[*] TXT example.com "v=spf1 include:_spf.example.com ~all"
```

### 2. 区域传送 (axfr)
**测试DNS服务器是否允许区域传送**，这是一个严重的安全配置错误。

```bash
dnsrecon -d example.com -t axfr
```

**工作原理：**
1. 查询目标域的NS记录
2. 对每个NS服务器尝试AXFR请求
3. 如果成功，获取整个区域文件

**成功示例：**
```
[*] Testing NS Servers for Zone Transfer
[*] Checking for Zone Transfer for example.com on NS server ns1.example.com
[+] Zone Transfer successful!
[*] A admin.example.com 192.168.1.10
[*] A api.example.com 192.168.1.20
[*] A dev.example.com 192.168.1.30
[*] A staging.example.com 192.168.1.40
```

**失败示例：**
```
[*] Zone Transfer failed for example.com on ns1.example.com
```

### 3. 子域名暴力破解 (brt)
**使用字典文件暴力枚举子域名**。

```bash
dnsrecon -d example.com -t brt -D subdomains.txt
```

**参数：**
- `-D` 或 `--dictionary` - 字典文件路径

**示例：**
```bash
# 使用自定义字典
dnsrecon -d example.com -t brt -D /opt/SecLists/Discovery/DNS/subdomains-top1million-5000.txt

# 使用内置小字典
dnsrecon -d example.com -t brt

# 指定线程数
dnsrecon -d example.com -t brt -D wordlist.txt --threads 10
```

**输出示例：**
```
[*] Performing host and subdomain brute force.
[+] A www.example.com 93.184.216.34
[+] A api.example.com 93.184.216.35
[+] A admin.example.com 93.184.216.36
[+] CNAME blog.example.com → wordpress.com
```

### 4. 反向DNS查询 (rvl)
**从IP地址范围查找PTR记录**。

```bash
dnsrecon -r 192.168.1.0/24
```

**参数：**
- `-r` 或 `--range` - IP范围（CIDR格式）

**示例：**
```bash
# 单个C段
dnsrecon -r 192.168.1.0/24

# 更大范围
dnsrecon -r 10.0.0.0/16

# 指定DNS服务器
dnsrecon -r 192.168.1.0/24 -n 8.8.8.8
```

**输出示例：**
```
[*] Performing Reverse Lookup from 192.168.1.0 to 192.168.1.255
[+] PTR 192.168.1.1 gateway.local
[+] PTR 192.168.1.10 server1.local
[+] PTR 192.168.1.20 server2.local
```

### 5. SRV记录枚举 (srv)
**枚举SRV服务记录**，用于发现各种服务。

```bash
dnsrecon -d example.com -t srv
```

**常见SRV记录：**
- `_http._tcp` - HTTP服务
- `_https._tcp` - HTTPS服务
- `_ftp._tcp` - FTP服务
- `_ldap._tcp` - LDAP服务
- `_xmpp-client._tcp` - XMPP客户端
- `_xmpp-server._tcp` - XMPP服务器
- `_sip._tcp` - SIP服务
- `_sipfederationtls._tcp` - SIP联邦TLS

**示例：**
```bash
# 标准SRV枚举
dnsrecon -d example.com -t srv

# 指定服务类型
dnsrecon -d example.com -t srv --service _http._tcp
```

**输出示例：**
```
[*] Enumerating SRV Records
[+] SRV _http._tcp.example.com web.example.com 80 10 0
[+] SRV _xmpp-client._tcp.example.com xmpp.example.com 5222 0 0
```

### 6. TLD扩展 (tld)
**测试不同顶级域名扩展**。

```bash
dnsrecon -d example -t tld
```

**测试扩展：**
.com, .net, .org, .info, .biz 等

**输出示例：**
```
[*] Performing TLD Brute force Enumeration
[+] example.com resolves to 93.184.216.34
[+] example.net resolves to 93.184.216.35
[+] example.org resolves to 93.184.216.36
```

### 7. DNSSEC区域遍历 (zonewalk)
**利用DNSSEC的NSEC/NSEC3记录枚举子域名**。

```bash
dnsrecon -d example.com -t zonewalk
```

**工作原理：**
- DNSSEC使用NSEC/NSEC3记录来证明不存在的域名
- 这些记录可能泄露域名列表
- DNSRecon会遍历这些记录发现子域名

**示例：**
```bash
# DNSSEC遍历
dnsrecon -d example.com -t zonewalk

# 指定DNS服务器
dnsrecon -d example.com -t zonewalk -n 8.8.8.8
```

**输出示例：**
```
[*] Performing NSEC Zone Walk
[+] Found via NSEC walking: admin.example.com
[+] Found via NSEC walking: api.example.com
[+] Found via NSEC walking: dev.example.com
```

### 8. 缓存侦察 (snoop)
**检查DNS缓存中的域名**。

```bash
dnsrecon -d example.com -t snoop -D domains.txt -n 192.168.1.1
```

**用途：**
- 检测DNS缓存投毒
- 发现用户访问的域名
- 内网侦察

---

## 常用参数

### 核心参数

| 参数 | 长格式 | 说明 | 示例 |
|------|--------|------|------|
| `-d` | `--domain` | 目标域名（必需） | `-d example.com` |
| `-t` | `--type` | 枚举类型 | `-t axfr` |
| `-n` | `--name_server` | DNS服务器 | `-n 8.8.8.8` |
| `-r` | `--range` | IP范围（反向查询） | `-r 192.168.1.0/24` |
| `-D` | `--dictionary` | 字典文件 | `-D wordlist.txt` |
| `-f` | `--filter` | 过滤结果 | `-f SOA,NS` |
| `-a` | `--axfr` | 执行区域传送 | `-a` |
| `-s` | `--db_server` | 数据库服务器 | `-s localhost` |
| `-g` | `--google` | Google搜索枚举 | `-g` |
| `-b` | `--bing` | Bing搜索枚举 | `-b` |
| `-k` | `--crt` | crt.sh证书透明度查询 | `-k` |
| `-w` | `--whois` | WHOIS查询 | `-w` |
| `-z` | `--threads` | 线程数 | `-z 10` |
| `--tcp` | | 使用TCP查询 | `--tcp` |
| `--lifetime` | | DNS查询超时（秒） | `--lifetime 5` |
| `--db` | | 数据库文件 | `--db results.db` |
| `--iw` | | 忽略通配符解析 | `--iw` |

### 输出参数

| 参数 | 说明 | 示例 |
|------|------|------|
| `-j` | JSON格式输出 | `-j results.json` |
| `-c` | CSV格式输出 | `-c results.csv` |
| `-x` | XML格式输出 | `-x results.xml` |
| `--db` | SQLite数据库 | `--db results.db` |
| `-v` | 详细输出 | `-v` |

### 高级参数

| 参数 | 说明 |
|------|------|
| `--threads N` | 线程数量（暴力破解） |
| `--lifetime N` | DNS查询超时时间 |
| `--tcp` | 使用TCP而非UDP |
| `--disable_check_recursion` | 禁用递归检查 |
| `--disable_check_bindversion` | 禁用BIND版本检查 |

---

## 实战场景

### 标准DNS枚举

#### 场景1：快速信息收集
```bash
# 基本枚举
dnsrecon -d target.com

# 详细输出
dnsrecon -d target.com -v

# 指定可靠的DNS服务器
dnsrecon -d target.com -n 8.8.8.8,1.1.1.1
```

#### 场景2：完整DNS记录收集
```bash
# 收集所有记录类型
dnsrecon -d target.com -t std

# 保存为JSON
dnsrecon -d target.com -t std -j dns_records.json

# 保存为CSV
dnsrecon -d target.com -t std -c dns_records.csv
```

### 子域名发现

#### 场景1：使用字典暴力破解
```bash
# 小字典快速扫描
dnsrecon -d target.com -t brt -D /usr/share/wordlists/dnsmap.txt

# 大字典深度扫描
dnsrecon -d target.com -t brt -D /opt/SecLists/Discovery/DNS/subdomains-top1million-110000.txt

# 多线程加速
dnsrecon -d target.com -t brt -D wordlist.txt --threads 20
```

#### 场景2：组合多种方法
```bash
# 先进行标准枚举和区域传送
dnsrecon -d target.com -t std,axfr

# 然后暴力破解
dnsrecon -d target.com -t brt -D subdomains.txt

# 最后尝试DNSSEC遍历
dnsrecon -d target.com -t zonewalk
```

#### 场景3：创建自定义字典
```bash
# 常见子域名
cat > common_subs.txt << EOF
www
mail
ftp
admin
api
dev
staging
test
beta
demo
portal
vpn
remote
secure
login
dashboard
app
mobile
EOF

# 执行扫描
dnsrecon -d target.com -t brt -D common_subs.txt
```

### 区域传送测试

#### 场景1：基本区域传送
```bash
# 测试所有NS服务器
dnsrecon -d target.com -t axfr

# 测试特定NS服务器
dnsrecon -d target.com -a -n ns1.target.com
```

#### 场景2：批量测试
```bash
# 创建域名列表
cat > domains.txt << EOF
example.com
test.com
demo.com
EOF

# 批量测试
for domain in $(cat domains.txt); do
    echo "[*] Testing $domain"
    dnsrecon -d $domain -t axfr
done
```

### 反向DNS查询

#### 场景1：单个网段
```bash
# C类网段
dnsrecon -r 192.168.1.0/24

# B类网段（较慢）
dnsrecon -r 172.16.0.0/16

# 指定DNS服务器
dnsrecon -r 10.0.0.0/24 -n 8.8.8.8
```

#### 场景2：多个网段
```bash
# IP范围列表
cat > ip_ranges.txt << EOF
192.168.1.0/24
192.168.2.0/24
10.0.0.0/24
EOF

# 批量反向查询
for range in $(cat ip_ranges.txt); do
    echo "[*] Scanning $range"
    dnsrecon -r $range
done
```

### SRV记录枚举

#### 场景1：发现服务
```bash
# 标准SRV枚举
dnsrecon -d target.com -t srv

# 自定义服务列表
cat > services.txt << EOF
_http._tcp
_https._tcp
_ftp._tcp
_ssh._tcp
_ldap._tcp
_kerberos._tcp
_xmpp-client._tcp
_sip._tcp
EOF

# 逐个测试
for service in $(cat services.txt); do
    dig $service.target.com SRV
done
```

### DNSSEC区域遍历

#### 场景1：利用DNSSEC
```bash
# 检查是否启用DNSSEC
dig target.com DNSKEY

# 如果启用，进行遍历
dnsrecon -d target.com -t zonewalk

# 详细输出
dnsrecon -d target.com -t zonewalk -v
```

### 缓存侦察

#### 场景1：内网DNS侦察
```bash
# 检查内网DNS缓存
dnsrecon -t snoop -n 192.168.1.1 -D common_domains.txt

# 常见域名列表
cat > common_domains.txt << EOF
google.com
facebook.com
twitter.com
github.com
microsoft.com
EOF
```

---

## 高级技巧

### 1. 组合枚举类型

```bash
# 多种类型组合
dnsrecon -d target.com -t std,axfr,brt -D wordlist.txt

# 完整枚举流程
dnsrecon -d target.com -t std,axfr,zonewalk,brt -D /opt/SecLists/Discovery/DNS/subdomains-top1million-5000.txt
```

### 2. 使用多个DNS服务器

```bash
# 指定多个DNS服务器
dnsrecon -d target.com -n 8.8.8.8,1.1.1.1,9.9.9.9

# 测试不同DNS服务器的响应差异
for dns in 8.8.8.8 1.1.1.1 9.9.9.9; do
    echo "[*] Testing with DNS: $dns"
    dnsrecon -d target.com -n $dns
done
```

### 3. 过滤和筛选结果

```bash
# 只显示A记录
dnsrecon -d target.com -f A

# 只显示A和AAAA记录
dnsrecon -d target.com -f A,AAAA

# 过滤MX和TXT记录
dnsrecon -d target.com -f MX,TXT
```

### 4. 自动化脚本

```bash
#!/bin/bash
# 完整DNS侦察脚本

DOMAIN=$1
OUTPUT_DIR="dns_recon_${DOMAIN}"

mkdir -p $OUTPUT_DIR

echo "[*] Starting DNS reconnaissance for $DOMAIN"

# 1. 标准枚举
echo "[*] Standard enumeration..."
dnsrecon -d $DOMAIN -t std -j $OUTPUT_DIR/std.json

# 2. 区域传送测试
echo "[*] Zone transfer test..."
dnsrecon -d $DOMAIN -t axfr -j $OUTPUT_DIR/axfr.json

# 3. 子域名暴力破解
echo "[*] Subdomain brute force..."
dnsrecon -d $DOMAIN -t brt -D /opt/SecLists/Discovery/DNS/subdomains-top1million-5000.txt -j $OUTPUT_DIR/subdomains.json

# 4. SRV记录枚举
echo "[*] SRV record enumeration..."
dnsrecon -d $DOMAIN -t srv -j $OUTPUT_DIR/srv.json

# 5. DNSSEC遍历
echo "[*] DNSSEC zone walk..."
dnsrecon -d $DOMAIN -t zonewalk -j $OUTPUT_DIR/zonewalk.json

echo "[+] Reconnaissance complete. Results saved to $OUTPUT_DIR/"
```

### 5. 结果后处理

```bash
# 提取所有子域名
cat std.json | jq -r '.[] | select(.type=="A") | .name' | sort -u > all_subdomains.txt

# 提取所有IP地址
cat std.json | jq -r '.[] | select(.type=="A") | .address' | sort -u > all_ips.txt

# 统计记录类型
cat std.json | jq -r '.[].type' | sort | uniq -c
```

### 6. 性能优化

```bash
# 增加线程数（暴力破解）
dnsrecon -d target.com -t brt -D wordlist.txt --threads 50

# 减少超时时间
dnsrecon -d target.com --lifetime 2

# 使用TCP（更可靠但较慢）
dnsrecon -d target.com --tcp
```

### 7. 绕过防护

```bash
# 使用不同的DNS服务器避免被检测
dnsrecon -d target.com -n 8.8.8.8

# 降低请求速率
dnsrecon -d target.com -t brt -D wordlist.txt --threads 1

# 添加随机延迟（需要脚本实现）
```

---

## 输出格式

### 1. 标准输出（终端）

```bash
dnsrecon -d example.com
```

输出示例：
```
[*] Performing General Enumeration of Domain: example.com
[*] SOA ns1.example.com 93.184.216.34
[*] NS ns1.example.com 93.184.216.34
[*] MX mail.example.com 10
[*] A example.com 93.184.216.34
```

### 2. JSON格式

```bash
dnsrecon -d example.com -j results.json
```

JSON结构：
```json
[
  {
    "type": "SOA",
    "name": "example.com",
    "mname": "ns1.example.com",
    "rname": "admin.example.com",
    "serial": "2024010101"
  },
  {
    "type": "A",
    "name": "example.com",
    "address": "93.184.216.34"
  }
]
```

### 3. CSV格式

```bash
dnsrecon -d example.com -c results.csv
```

CSV结构：
```csv
Type,Name,Address,Target,Port,Priority
A,example.com,93.184.216.34,,,
MX,example.com,,mail.example.com,,10
NS,example.com,93.184.216.34,ns1.example.com,,
```

### 4. XML格式

```bash
dnsrecon -d example.com -x results.xml
```

需要安装 `lxml` 库：
```bash
pip3 install lxml
```

### 5. SQLite数据库

```bash
dnsrecon -d example.com --db dns.db
```

查询数据库：
```bash
sqlite3 dns.db "SELECT * FROM records;"
```

---

## 字典文件

### 推荐字典

#### SecLists（强烈推荐）
```bash
# 安装SecLists
sudo apt install seclists

# 或手动下载
git clone https://github.com/danielmiessler/SecLists.git /opt/SecLists

# 常用字典路径
/opt/SecLists/Discovery/DNS/subdomains-top1million-5000.txt
/opt/SecLists/Discovery/DNS/subdomains-top1million-20000.txt
/opt/SecLists/Discovery/DNS/subdomains-top1million-110000.txt
/opt/SecLists/Discovery/DNS/dns-Jhaddix.txt
/opt/SecLists/Discovery/DNS/deepmagic.com-prefixes-top50000.txt
```

#### 系统内置字典
```bash
# Kali Linux
/usr/share/wordlists/dnsmap.txt
/usr/share/wordlists/fierce-hostlist.txt

# DNSRecon自带（如果从源码安装）
dnsrecon/namelist.txt
```

### 创建自定义字典

#### 常见子域名
```bash
cat > common_subdomains.txt << EOF
www
mail
ftp
webmail
smtp
pop
imap
blog
forum
shop
store
admin
administrator
login
portal
dashboard
panel
cpanel
whm
api
api-dev
api-staging
dev
development
staging
stage
test
testing
qa
uat
demo
beta
alpha
prod
production
app
mobile
m
cdn
static
assets
media
images
img
upload
downloads
files
docs
help
support
vpn
remote
ssh
git
svn
jenkins
gitlab
bitbucket
jira
confluence
wiki
intranet
extranet
partner
vendor
client
customer
crm
erp
hr
finance
billing
payment
secure
ssl
tls
backup
archive
old
new
v1
v2
api-v1
api-v2
EOF
```

#### 环境和功能相关
```bash
cat > environments.txt << EOF
dev
development
develop
staging
stage
test
testing
qa
uat
prod
production
live
preview
demo
beta
alpha
sandbox
lab
training
EOF
```

#### 服务相关
```bash
cat > services.txt << EOF
mail
smtp
pop
pop3
imap
webmail
email
mx
mx1
mx2
ns
ns1
ns2
dns
ftp
sftp
ssh
vpn
rdp
proxy
gateway
firewall
router
switch
EOF
```

---

## 实战案例

### 案例1：目标侦察（Bug Bounty）

```bash
#!/bin/bash
# Bug Bounty DNS侦察脚本

TARGET=$1

echo "[*] Starting reconnaissance on $TARGET"

# 1. 基本信息收集
echo "[+] Step 1: Basic enumeration"
dnsrecon -d $TARGET -t std -j ${TARGET}_std.json

# 2. 区域传送测试
echo "[+] Step 2: Zone transfer test"
dnsrecon -d $TARGET -t axfr -j ${TARGET}_axfr.json

# 3. 子域名枚举 - 快速扫描
echo "[+] Step 3: Quick subdomain scan"
dnsrecon -d $TARGET -t brt -D /opt/SecLists/Discovery/DNS/subdomains-top1million-5000.txt -j ${TARGET}_subdomains_quick.json

# 4. 子域名枚举 - 深度扫描
echo "[+] Step 4: Deep subdomain scan"
dnsrecon -d $TARGET -t brt -D /opt/SecLists/Discovery/DNS/subdomains-top1million-110000.txt --threads 20 -j ${TARGET}_subdomains_deep.json

# 5. DNSSEC遍历
echo "[+] Step 5: DNSSEC zone walk"
dnsrecon -d $TARGET -t zonewalk -j ${TARGET}_zonewalk.json

# 6. SRV记录
echo "[+] Step 6: SRV records"
dnsrecon -d $TARGET -t srv -j ${TARGET}_srv.json

# 7. 提取所有发现的子域名
echo "[+] Step 7: Extracting all subdomains"
cat ${TARGET}_*.json | jq -r '.[] | select(.type=="A" or .type=="CNAME") | .name' | sort -u > ${TARGET}_all_subdomains.txt

echo "[+] Found $(wc -l < ${TARGET}_all_subdomains.txt) unique subdomains"
echo "[+] Results saved to ${TARGET}_all_subdomains.txt"
```

### 案例2：内网渗透

```bash
#!/bin/bash
# 内网DNS侦察

# 1. 反向查询发现主机
echo "[*] Reverse DNS lookup"
dnsrecon -r 192.168.1.0/24 -j reverse_192.168.1.json
dnsrecon -r 10.0.0.0/16 -j reverse_10.0.json

# 2. 测试内网DNS服务器
echo "[*] Testing internal DNS server"
dnsrecon -d internal.local -n 192.168.1.1 -t std,axfr

# 3. 缓存侦察
echo "[*] DNS cache snooping"
dnsrecon -t snoop -n 192.168.1.1 -D common_domains.txt

# 4. 枚举Active Directory
echo "[*] AD enumeration"
dnsrecon -d corp.local -n 192.168.1.1 -t std,srv,brt -D ad_services.txt
```

### 案例3：CTF比赛

```bash
#!/bin/bash
# CTF DNS挑战脚本

DOMAIN=$1

# 1. 快速标准枚举
dnsrecon -d $DOMAIN -t std

# 2. 立即测试区域传送（CTF常见）
dnsrecon -d $DOMAIN -t axfr

# 3. TXT记录查找（可能包含flag）
dig $DOMAIN TXT +short

# 4. 所有子域名快速扫描
dnsrecon -d $DOMAIN -t brt -D /usr/share/wordlists/dirb/common.txt

# 5. 检查特殊记录
dig $DOMAIN ANY

# 6. 查找隐藏信息
dig @ns1.$DOMAIN $DOMAIN AXFR
```

### 案例4：完整域名审计

```bash
#!/bin/bash
# 完整DNS安全审计

DOMAIN=$1
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
OUTPUT_DIR="dns_audit_${DOMAIN}_${TIMESTAMP}"

mkdir -p $OUTPUT_DIR

log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" | tee -a $OUTPUT_DIR/audit.log
}

log "Starting DNS audit for $DOMAIN"

# 1. 基本信息
log "Collecting basic DNS information"
dnsrecon -d $DOMAIN -t std -j $OUTPUT_DIR/01_standard.json

# 2. 安全测试
log "Testing zone transfer vulnerability"
dnsrecon -d $DOMAIN -t axfr -j $OUTPUT_DIR/02_zone_transfer.json

# 3. 全面子域名枚举
log "Comprehensive subdomain enumeration"
dnsrecon -d $DOMAIN -t brt -D /opt/SecLists/Discovery/DNS/dns-Jhaddix.txt --threads 30 -j $OUTPUT_DIR/03_subdomains.json

# 4. DNSSEC分析
log "DNSSEC analysis"
dnsrecon -d $DOMAIN -t zonewalk -j $OUTPUT_DIR/04_dnssec.json

# 5. 服务发现
log "Service discovery"
dnsrecon -d $DOMAIN -t srv -j $OUTPUT_DIR/05_services.json

# 6. 整理结果
log "Consolidating results"
cat $OUTPUT_DIR/*.json | jq -r '.[] | select(.name != null) | .name' | sort -u > $OUTPUT_DIR/all_names.txt
cat $OUTPUT_DIR/*.json | jq -r '.[] | select(.address != null) | .address' | sort -u > $OUTPUT_DIR/all_ips.txt

# 7. 生成报告
log "Generating report"
cat > $OUTPUT_DIR/report.md << EOF
# DNS Audit Report for $DOMAIN

**Date:** $(date)

## Summary
- Total unique names found: $(wc -l < $OUTPUT_DIR/all_names.txt)
- Total unique IPs found: $(wc -l < $OUTPUT_DIR/all_ips.txt)

## Files Generated
- 01_standard.json - Standard DNS records
- 02_zone_transfer.json - Zone transfer test results
- 03_subdomains.json - Subdomain enumeration results
- 04_dnssec.json - DNSSEC analysis
- 05_services.json - Service discovery
- all_names.txt - All discovered names
- all_ips.txt - All discovered IPs

## Findings
$(cat $OUTPUT_DIR/*.json | jq)
EOF

log "Audit complete. Report saved to $OUTPUT_DIR/report.md"
```

---

## 与其他工具组合

### 1. 与 Amass 配合

```bash
# DNSRecon快速扫描
dnsrecon -d target.com -t std,axfr -j dnsrecon.json

# Amass深度枚举
amass enum -d target.com -o amass.txt

# 合并结果
cat dnsrecon.json | jq -r '.[] | select(.type=="A") | .name' > dnsrecon_subs.txt
cat dnsrecon_subs.txt amass.txt | sort -u > all_subdomains.txt
```

### 2. 与 Subfinder 配合

```bash
# Subfinder（使用API）
subfinder -d target.com -o subfinder.txt

# DNSRecon（暴力破解）
dnsrecon -d target.com -t brt -D wordlist.txt -j dnsrecon.json

# 合并去重
cat subfinder.txt <(cat dnsrecon.json | jq -r '.[] | select(.type=="A") | .name') | sort -u > final_subs.txt
```

### 3. 与 Nmap 配合

```bash
# DNSRecon发现子域名
dnsrecon -d target.com -t brt -D wordlist.txt -j subs.json

# 提取IP
cat subs.json | jq -r '.[] | select(.address != null) | .address' | sort -u > ips.txt

# Nmap扫描
nmap -iL ips.txt -sV -oA nmap_scan
```

### 4. 与 MassDNS 配合

```bash
# DNSRecon生成子域名列表
dnsrecon -d target.com -t brt -D wordlist.txt -j dnsrecon.json
cat dnsrecon.json | jq -r '.[] | .name' | sort -u > potential_subs.txt

# MassDNS验证和扩展
massdns -r resolvers.txt -t A -o S potential_subs.txt > massdns_results.txt
```

### 5. 管道使用

```bash
# 发现子域名 -> 提取IP -> 端口扫描
dnsrecon -d target.com -t brt -D wordlist.txt -j /dev/stdout | \
  jq -r '.[] | select(.address != null) | .address' | \
  sort -u | \
  nmap -iL - -p 80,443 --open

# 区域传送 -> 提取主机 -> HTTP探测
dnsrecon -d target.com -t axfr -j /dev/stdout | \
  jq -r '.[] | select(.type=="A") | .name' | \
  httpx -silent
```

---

## 常见问题

### 1. 权限问题

**问题：** `Permission denied` 错误
```bash
# 解决方案：使用sudo或检查文件权限
sudo dnsrecon -d target.com

# 或修改权限
chmod +x dnsrecon.py
```

### 2. DNS查询超时

**问题：** 查询超时
```bash
# 增加超时时间
dnsrecon -d target.com --lifetime 10

# 使用更可靠的DNS服务器
dnsrecon -d target.com -n 8.8.8.8

# 使用TCP
dnsrecon -d target.com --tcp
```

### 3. 依赖问题

**问题：** `ModuleNotFoundError: No module named 'dnspython'`
```bash
# 安装依赖
pip3 install dnspython netaddr

# 或重新安装DNSRecon
pip3 install dnsrecon --force-reinstall
```

### 4. 字典文件问题

**问题：** 找不到字典文件
```bash
# 检查字典路径
ls -la /opt/SecLists/Discovery/DNS/

# 下载SecLists
git clone https://github.com/danielmiessler/SecLists.git /opt/SecLists

# 使用绝对路径
dnsrecon -d target.com -t brt -D /opt/SecLists/Discovery/DNS/subdomains-top1million-5000.txt
```

### 5. 输出格式问题

**问题：** JSON输出格式错误
```bash
# 确保jq已安装
sudo apt install jq

# 检查JSON有效性
cat results.json | jq empty

# 格式化JSON
cat results.json | jq '.' > formatted.json
```

### 6. 区域传送失败

**问题：** 区域传送总是失败
```bash
# 这是正常的，大多数服务器禁用了AXFR

# 验证NS记录
dig target.com NS

# 手动测试特定NS
dig @ns1.target.com target.com AXFR

# 测试所有NS服务器
for ns in $(dig target.com NS +short); do
    echo "Testing $ns"
    dig @$ns target.com AXFR
done
```

### 7. 通配符DNS问题

**问题：** 通配符DNS导致误报
```bash
# 使用 --iw 忽略通配符
dnsrecon -d target.com -t brt -D wordlist.txt --iw

# 手动检测通配符
dig random-$(date +%s).target.com
```

### 8. 速率限制

**问题：** 被DNS服务器限速
```bash
# 降低线程数
dnsrecon -d target.com -t brt -D wordlist.txt --threads 5

# 使用不同DNS服务器
dnsrecon -d target.com -n 8.8.8.8

# 分批处理字典
split -l 1000 wordlist.txt chunk_
for chunk in chunk_*; do
    dnsrecon -d target.com -t brt -D $chunk
    sleep 60
done
```

---

## 最佳实践

### ✅ 推荐做法

1. **始终获得授权**
   - 只在授权范围内进行DNS枚举
   - 记录授权文档

2. **多种方法组合**
   ```bash
   # 不要只依赖一种方法
   dnsrecon -d target.com -t std,axfr,brt,zonewalk
   ```

3. **保存所有结果**
   ```bash
   # 使用JSON格式便于后续处理
   dnsrecon -d target.com -j results_$(date +%Y%m%d).json
   ```

4. **使用可靠的DNS服务器**
   ```bash
   # Google、Cloudflare等公共DNS
   dnsrecon -d target.com -n 8.8.8.8,1.1.1.1
   ```

5. **递增式枚举**
   ```bash
   # 先快速扫描
   dnsrecon -d target.com -t brt -D small_wordlist.txt

   # 再深度扫描
   dnsrecon -d target.com -t brt -D large_wordlist.txt
   ```

### ❌ 避免错误

1. **不要过度扫描**
   - 避免无限大的字典
   - 控制线程数量

2. **不要忽略错误**
   - 注意超时和失败
   - 检查DNS响应

3. **不要只依赖自动化**
   - 手动验证重要发现
   - 理解每个记录的含义

4. **不要忽略隐私**
   - 不要枚举个人域名
   - 遵守法律法规

### 🎯 效率技巧

1. **并行执行**
   ```bash
   # 同时进行多个任务
   dnsrecon -d target.com -t std &
   dnsrecon -d target.com -t axfr &
   dnsrecon -d target.com -t brt -D wordlist.txt &
   wait
   ```

2. **智能字典**
   ```bash
   # 根据目标特点定制字典
   # 如：开发公司用dev, staging, test
   # 电商用shop, store, cart
   ```

3. **结果复用**
   ```bash
   # 使用之前的发现
   cat previous_scan.json | jq -r '.[].name' >> custom_wordlist.txt
   ```

4. **自动化报告**
   ```bash
   # 生成可读报告
   dnsrecon -d target.com -j raw.json
   cat raw.json | jq -r '.[] | "\(.type) \(.name) \(.address // .target)"' > report.txt
   ```

---

## 参考资源

### 官方资源
- **GitHub仓库**: https://github.com/darkoperator/dnsrecon
- **作者博客**: https://www.darkoperator.com

### 字典资源
- **SecLists**: https://github.com/danielmiessler/SecLists
- **DNSDumpster**: https://dnsdumpster.com
- **Sublist3r字典**: https://github.com/aboul3la/Sublist3r

### 学习资源
- **HackTricks DNS**: https://book.hacktricks.xyz/pentesting/pentesting-dns
- **OWASP测试指南**: https://owasp.org/www-project-web-security-testing-guide/
- **DNS RFC文档**: https://www.ietf.org/rfc/

### 相关工具
- **Amass**: https://github.com/OWASP/Amass
- **Subfinder**: https://github.com/projectdiscovery/subfinder
- **MassDNS**: https://github.com/blechschmidt/massdns
- **Fierce**: https://github.com/mschwager/fierce

---

## 速查表

### 快速命令

| 任务 | 命令 |
|------|------|
| 标准枚举 | `dnsrecon -d target.com` |
| 区域传送 | `dnsrecon -d target.com -t axfr` |
| 子域名爆破 | `dnsrecon -d target.com -t brt -D wordlist.txt` |
| 反向查询 | `dnsrecon -r 192.168.1.0/24` |
| SRV枚举 | `dnsrecon -d target.com -t srv` |
| DNSSEC遍历 | `dnsrecon -d target.com -t zonewalk` |
| 保存JSON | `dnsrecon -d target.com -j results.json` |
| 多线程 | `dnsrecon -d target.com -t brt -D wordlist.txt --threads 20` |
| 指定DNS | `dnsrecon -d target.com -n 8.8.8.8` |

### 组合命令

```bash
# 完整侦察
dnsrecon -d target.com -t std,axfr,brt,zonewalk,srv -D /opt/SecLists/Discovery/DNS/subdomains-top1million-5000.txt -j complete.json

# 内网扫描
dnsrecon -d internal.local -n 192.168.1.1 -t std,axfr,brt -D internal_wordlist.txt

# Bug Bounty
dnsrecon -d target.com -t std,axfr,zonewalk,brt -D /opt/SecLists/Discovery/DNS/dns-Jhaddix.txt --threads 30 -j bb_results.json
```

---

**最后更新:** 2025-10-04
**DNSRecon版本:** 1.2.0+
**适用场景:** 渗透测试、Bug Bounty、安全审计、CTF比赛
**作者:** Carlos Perez (@darkoperator)
