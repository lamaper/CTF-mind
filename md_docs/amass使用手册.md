# OWASP Amass 使用手册

## 目录
- [简介](#简介)
- [安装](#安装)
- [核心概念](#核心概念)
- [子命令详解](#子命令详解)
- [基本用法](#基本用法)
- [高级功能](#高级功能)
- [数据源配置](#数据源配置)
- [实战场景](#实战场景)
- [API集成](#api集成)
- [数据库管理](#数据库管理)
- [可视化分析](#可视化分析)
- [性能优化](#性能优化)
- [输出格式](#输出格式)
- [实战案例](#实战案例)
- [与其他工具组合](#与其他工具组合)
- [常见问题](#常见问题)
- [最佳实践](#最佳实践)

---

## 简介

**OWASP Amass** 是目前最强大的子域名枚举和网络映射工具，由 OWASP（开放Web应用安全项目）维护。

### 核心特性

- 🔍 **多源情报收集** - 整合100+数据源（DNS、证书、搜索引擎、API等）
- 🌐 **主动+被动枚举** - 结合主动探测和被动信息收集
- 📊 **深度分析** - ASN、IP、CIDR、关系映射
- 🗺️ **网络拓扑** - 自动绘制目标网络基础设施
- 💾 **数据持久化** - 内置图数据库存储
- 🎯 **精准控制** - 黑白名单、速率限制、递归深度
- 📈 **可视化** - 支持导出为Maltego、GraphViz等格式

### 与其他工具对比

| 工具 | 速度 | 数据源 | 主动探测 | 可视化 | 推荐场景 |
|------|------|--------|----------|--------|----------|
| **Amass** | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ✅ | ✅ | 深度侦察、Bug Bounty |
| Subfinder | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ❌ | ❌ | 快速枚举 |
| DNSRecon | ⭐⭐⭐⭐ | ⭐⭐ | ✅ | ❌ | DNS审计 |
| Gobuster | ⭐⭐⭐⭐⭐ | ⭐ | ✅ | ❌ | 暴力破解 |

### 工作原理

Amass 通过以下方式收集信息：

1. **被动信息收集**
   - 证书透明度日志 (crt.sh)
   - 搜索引擎 (Google, Bing, Baidu)
   - 威胁情报平台 (VirusTotal, AlienVault)
   - DNS数据库 (SecurityTrails, DNSDumpster)
   - Web档案 (Wayback Machine, CommonCrawl)

2. **主动DNS枚举**
   - 暴力破解子域名
   - 区域传送测试
   - DNS记录查询
   - 反向DNS查询

3. **网络映射**
   - ASN枚举
   - IP段识别
   - CIDR范围确定
   - 关系图构建

---

## 安装

### 预编译二进制（推荐）

```bash
# 下载最新版本
# 访问 https://github.com/owasp-amass/amass/releases

# Linux AMD64
wget https://github.com/owasp-amass/amass/releases/download/v4.2.0/amass_Linux_amd64.zip
unzip amass_Linux_amd64.zip
sudo mv amass /usr/local/bin/

# macOS ARM64
wget https://github.com/owasp-amass/amass/releases/download/v4.2.0/amass_Darwin_arm64.zip
unzip amass_Darwin_arm64.zip
sudo mv amass /usr/local/bin/
```

### Kali Linux / Debian

```bash
# 从官方仓库安装
sudo apt update
sudo apt install amass

# 或使用snap
sudo snap install amass
```

### macOS

```bash
# 使用Homebrew
brew install amass
```

### Docker

```bash
# 拉取镜像
docker pull caffix/amass

# 运行
docker run -v ~/.config/amass:/root/.config/amass caffix/amass enum -d example.com
```

### 使用Go安装

```bash
# 需要Go 1.20+
go install -v github.com/owasp-amass/amass/v4/...@master
```

### 从源码编译

```bash
# 克隆仓库
git clone https://github.com/owasp-amass/amass.git
cd amass

# 编译
go install ./...

# 验证
amass -version
```

### 验证安装

```bash
amass -version
# 输出: amass version v4.x.x

amass -h
```

---

## 核心概念

### 子命令架构

Amass 使用子命令架构，每个子命令专注特定功能：

| 子命令 | 功能 | 用途 |
|--------|------|------|
| `intel` | 情报收集 | 发现根域名和ASN |
| `enum` | 子域名枚举 | 主要枚举功能 |
| `viz` | 可视化 | 生成网络拓扑图 |
| `track` | 追踪变化 | 监控域名变化 |
| `db` | 数据库管理 | 查询和管理数据 |

### 数据源类型

1. **证书透明度** - crt.sh, Google CT, Censys
2. **DNS服务** - DNS查询、区域传送
3. **搜索引擎** - Google, Bing, Baidu, Yahoo
4. **Web档案** - Archive.org, CommonCrawl
5. **威胁情报** - VirusTotal, AlienVault OTX
6. **API服务** - SecurityTrails, Shodan, Censys
7. **暴力破解** - 字典、排列组合、变换

### 枚举技术

- **被动枚举** (`-passive`) - 仅使用公开数据源
- **主动枚举** (默认) - DNS查询 + 数据源
- **暴力破解** (`-brute`) - 字典攻击
- **递归枚举** (`-r`) - 递归查找子域名
- **反向DNS** - IP到域名映射

---

## 子命令详解

### 1. intel - 情报收集

**发现目标组织的根域名和ASN**

```bash
# 基本语法
amass intel [选项]
```

#### 常用选项

| 参数 | 说明 | 示例 |
|------|------|------|
| `-org` | 组织名称 | `-org "Example Inc"` |
| `-asn` | ASN编号 | `-asn 15169` |
| `-cidr` | CIDR范围 | `-cidr 192.168.0.0/16` |
| `-ip` | IP地址 | `-ip 8.8.8.8` |
| `-d` | 已知域名 | `-d example.com` |
| `-whois` | WHOIS查询 | `-whois` |
| `-addr` | 显示IP地址 | `-addr` |
| `-p` | 指定端口 | `-p 80,443` |

#### 使用示例

```bash
# 根据组织名称查找域名
amass intel -org "Google LLC"

# 查找ASN相关域名
amass intel -asn 15169

# 根据CIDR查找
amass intel -cidr 8.8.8.0/24

# 反向WHOIS查找
amass intel -d google.com -whois

# 显示IP地址
amass intel -org "Example Inc" -addr

# 组合查询
amass intel -org "Example Inc" -whois -addr -o intel_results.txt
```

### 2. enum - 子域名枚举

**核心枚举功能，最常用的子命令**

```bash
# 基本语法
amass enum [选项]
```

#### 核心参数

| 参数 | 说明 | 示例 |
|------|------|------|
| `-d` | 目标域名（必需） | `-d example.com` |
| `-df` | 域名列表文件 | `-df domains.txt` |
| `-brute` | 启用暴力破解 | `-brute` |
| `-w` | 暴力破解字典 | `-w wordlist.txt` |
| `-wm` | 字典模式 | `-wm default,all` |
| `-active` | 主动扫描 | `-active` |
| `-passive` | 被动模式 | `-passive` |
| `-bl` | 黑名单 | `-bl cloudflare.com` |
| `-blf` | 黑名单文件 | `-blf blacklist.txt` |
| `-include` | 包含域名 | `-include *.example.com` |
| `-exclude` | 排除域名 | `-exclude test.example.com` |

#### 输出参数

| 参数 | 说明 | 示例 |
|------|------|------|
| `-o` | 输出文件 | `-o results.txt` |
| `-dir` | 输出目录 | `-dir ./output` |
| `-json` | JSON输出 | `-json results.json` |
| `-ip` | 显示IP地址 | `-ip` |
| `-src` | 显示数据源 | `-src` |
| `-v` | 详细输出 | `-v` |
| `-silent` | 静默模式 | `-silent` |

#### 高级参数

| 参数 | 说明 | 示例 |
|------|------|------|
| `-r` | DNS解析器 | `-r 8.8.8.8,1.1.1.1` |
| `-rf` | 解析器文件 | `-rf resolvers.txt` |
| `-p` | 端口扫描 | `-p 80,443,8080` |
| `-nf` | 网络过滤 | `-nf 192.168.0.0/16` |
| `-timeout` | 超时时间 | `-timeout 5` |
| `-max-dns-queries` | DNS查询限制 | `-max-dns-queries 10000` |
| `-max-depth` | 递归深度 | `-max-depth 3` |
| `-alts` | 子域名变体 | `-alts` |
| `-min-for-recursive` | 递归触发阈值 | `-min-for-recursive 3` |

#### 数据源控制

| 参数 | 说明 | 示例 |
|------|------|------|
| `-config` | 配置文件 | `-config config.yaml` |
| `-scripts` | 脚本目录 | `-scripts ./scripts` |
| `-norecursive` | 禁用递归 | `-norecursive` |
| `-noalts` | 禁用变体 | `-noalts` |

#### 使用示例

```bash
# 基本枚举
amass enum -d example.com

# 被动模式（快速）
amass enum -passive -d example.com -o passive.txt

# 主动模式（深度）
amass enum -active -d example.com -o active.txt

# 暴力破解模式
amass enum -brute -d example.com -w wordlist.txt

# 多域名枚举
amass enum -df domains.txt -o results.txt

# 显示IP和来源
amass enum -d example.com -ip -src -o detailed.txt

# 排除子域名
amass enum -d example.com -exclude test.example.com,dev.example.com

# 使用配置文件
amass enum -d example.com -config ~/.config/amass/config.yaml

# 限制递归深度
amass enum -d example.com -max-depth 2

# 完整参数示例
amass enum -active -brute -d example.com \
  -w wordlist.txt \
  -ip -src \
  -o results.txt \
  -json results.json \
  -dir ./amass_output \
  -v
```

### 3. viz - 可视化

**生成网络拓扑图和关系图**

```bash
# 基本语法
amass viz [选项]
```

#### 参数说明

| 参数 | 说明 | 示例 |
|------|------|------|
| `-d` | 域名 | `-d example.com` |
| `-d3` | D3.js HTML文件 | `-d3` |
| `-dot` | Graphviz DOT文件 | `-dot` |
| `-gexf` | GEXF格式（Gephi） | `-gexf` |
| `-graphistry` | Graphistry JSON | `-graphistry` |
| `-maltego` | Maltego CSV | `-maltego` |
| `-o` | 输出文件 | `-o graph.html` |
| `-dir` | 数据库目录 | `-dir ./amass_db` |

#### 使用示例

```bash
# 生成D3.js可视化
amass viz -d example.com -d3 -o graph.html

# 生成Graphviz DOT文件
amass viz -d example.com -dot | dot -Tpng -o graph.png

# 导出Maltego格式
amass viz -d example.com -maltego -o maltego.csv

# 导出GEXF（用于Gephi）
amass viz -d example.com -gexf -o network.gexf

# 指定数据库目录
amass viz -d example.com -d3 -dir ./amass_data -o viz.html
```

### 4. track - 变化追踪

**监控域名变化，对比历史数据**

```bash
# 基本语法
amass track [选项]
```

#### 参数说明

| 参数 | 说明 | 示例 |
|------|------|------|
| `-d` | 域名 | `-d example.com` |
| `-since` | 起始时间 | `-since "2024-01-01"` |
| `-o` | 输出文件 | `-o changes.txt` |
| `-dir` | 数据库目录 | `-dir ./amass_db` |

#### 使用示例

```bash
# 跟踪变化
amass track -d example.com

# 对比历史数据
amass track -d example.com -since "2024-01-01"

# 保存变化记录
amass track -d example.com -o changes.txt

# 持续监控（配合cron）
*/6 * * * amass track -d example.com -o /var/log/amass/changes_$(date +\%Y\%m\%d).txt
```

### 5. db - 数据库管理

**查询和管理Amass数据库**

```bash
# 基本语法
amass db [选项]
```

#### 参数说明

| 参数 | 说明 | 示例 |
|------|------|------|
| `-d` | 域名 | `-d example.com` |
| `-dir` | 数据库目录 | `-dir ./amass_db` |
| `-list` | 列出所有枚举 | `-list` |
| `-names` | 显示域名 | `-names` |
| `-show` | 显示枚举详情 | `-show` |
| `-import` | 导入数据 | `-import file.json` |
| `-since` | 时间过滤 | `-since "2024-01-01"` |

#### 使用示例

```bash
# 列出所有枚举
amass db -list

# 查看域名数据
amass db -d example.com -names

# 显示详细信息
amass db -d example.com -show

# 导入数据
amass db -import amass_results.json

# 按时间过滤
amass db -d example.com -since "2024-01-01" -names

# 指定数据库目录
amass db -dir /path/to/db -d example.com -names
```

---

## 基本用法

### 快速开始

```bash
# 最简单的使用
amass enum -d example.com

# 被动模式（快速，无DNS查询）
amass enum -passive -d example.com

# 主动模式（深度扫描）
amass enum -active -d example.com

# 暴力破解模式
amass enum -brute -d example.com
```

### 常用组合

```bash
# 被动 + 保存结果
amass enum -passive -d example.com -o passive_results.txt

# 主动 + IP信息
amass enum -active -d example.com -ip -o active_with_ip.txt

# 暴力 + 自定义字典
amass enum -brute -d example.com -w /opt/SecLists/Discovery/DNS/subdomains-top1million-5000.txt

# 多域名 + JSON输出
amass enum -df domains.txt -json results.json

# 完整扫描
amass enum -active -brute -d example.com -w wordlist.txt -ip -src -o full_scan.txt
```

---

## 高级功能

### 1. 递归枚举

```bash
# 启用递归（自动发现子域名的子域名）
amass enum -d example.com -r

# 限制递归深度
amass enum -d example.com -r -max-depth 2

# 设置递归触发阈值
amass enum -d example.com -r -min-for-recursive 5
```

### 2. 子域名变体

```bash
# 启用变体生成
amass enum -d example.com -alts

# 变体包括：
# - 字符替换（o -> 0）
# - 连字符变化
# - 常见前缀后缀
```

### 3. 网络范围限制

```bash
# 只显示特定网络范围内的结果
amass enum -d example.com -nf 192.168.0.0/16

# 排除特定IP范围
amass enum -d example.com -nf -10.0.0.0/8
```

### 4. 黑名单控制

```bash
# 排除特定子域名
amass enum -d example.com -bl test.example.com,dev.example.com

# 使用黑名单文件
cat > blacklist.txt << EOF
test.example.com
dev.example.com
staging.example.com
EOF

amass enum -d example.com -blf blacklist.txt
```

### 5. 自定义DNS解析器

```bash
# 使用特定DNS服务器
amass enum -d example.com -r 8.8.8.8,1.1.1.1,9.9.9.9

# 使用解析器列表文件
cat > resolvers.txt << EOF
8.8.8.8
1.1.1.1
9.9.9.9
208.67.222.222
EOF

amass enum -d example.com -rf resolvers.txt
```

---

## 数据源配置

### 配置文件

Amass 使用 YAML 配置文件管理API密钥和数据源。

#### 配置文件位置

```bash
# Linux/macOS
~/.config/amass/config.yaml

# Windows
%USERPROFILE%\AppData\Local\amass\config.yaml
```

#### 生成配置模板

```bash
# 生成默认配置
amass enum -config /dev/null -show
```

#### 配置文件示例

```yaml
# ~/.config/amass/config.yaml

# 解析器配置
resolvers:
  - 8.8.8.8
  - 1.1.1.1
  - 9.9.9.9

# 数据源配置
datasources:
  # AlienVault OTX
  - name: AlienVault
    key: your_api_key_here

  # BinaryEdge
  - name: BinaryEdge
    key: your_api_key_here

  # C99
  - name: C99
    key: your_api_key_here

  # Censys
  - name: Censys
    credentials:
      apikey: your_api_id
      secret: your_api_secret

  # Chaos
  - name: Chaos
    key: your_api_key_here

  # CIRCL
  - name: CIRCL
    credentials:
      username: your_username
      password: your_password

  # Facebook
  - name: FacebookCT
    key: your_app_id
    secret: your_app_secret

  # GitHub
  - name: GitHub
    key: your_personal_access_token

  # Hunter
  - name: Hunter
    key: your_api_key_here

  # IPinfo
  - name: IPinfo
    key: your_api_key_here

  # NetworksDB
  - name: NetworksDB
    key: your_api_key_here

  # PassiveTotal
  - name: PassiveTotal
    credentials:
      username: your_username
      apikey: your_api_key

  # SecurityTrails
  - name: SecurityTrails
    key: your_api_key_here

  # Shodan
  - name: Shodan
    key: your_api_key_here

  # Spyse
  - name: Spyse
    key: your_api_key_here

  # Twitter
  - name: Twitter
    credentials:
      apikey: your_api_key
      secret: your_api_secret
      bearer_token: your_bearer_token

  # URLScan
  - name: URLScan
    key: your_api_key_here

  # VirusTotal
  - name: VirusTotal
    key: your_api_key_here

  # WhoisXML API
  - name: WhoisXMLAPI
    key: your_api_key_here

  # ZETAlytics
  - name: ZETAlytics
    key: your_api_key_here

# 暴力破解配置
bruteforce:
  enabled: true
  recursive: true
  minimum_for_recursive: 3
  wordlists:
    - /path/to/wordlist1.txt
    - /path/to/wordlist2.txt

# 数据库配置
database: /path/to/amass_db

# 输出目录
output_directory: /path/to/output

# 最大DNS查询数
max_dns_queries: 20000

# 超时设置
timeout: 30
```

#### 使用配置文件

```bash
# 使用配置文件
amass enum -d example.com -config ~/.config/amass/config.yaml
```

### 免费API密钥获取

| 服务 | 免费额度 | 注册地址 |
|------|---------|---------|
| **VirusTotal** | 500/天 | https://www.virustotal.com/gui/join-us |
| **SecurityTrails** | 50/月 | https://securitytrails.com/app/signup |
| **Shodan** | 1次性100积分 | https://account.shodan.io/register |
| **Censys** | 250/月 | https://censys.io/register |
| **GitHub** | 5000/小时 | https://github.com/settings/tokens |
| **AlienVault** | 10000/月 | https://otx.alienvault.com/ |
| **Hunter** | 25/月 | https://hunter.io/users/sign_up |
| **URLScan** | 无限制 | https://urlscan.io/user/signup |

---

## 实战场景

### 场景1：Bug Bounty快速侦察

```bash
#!/bin/bash
# Bug Bounty 快速侦察脚本

TARGET=$1
OUTPUT_DIR="amass_${TARGET}_$(date +%Y%m%d)"

mkdir -p $OUTPUT_DIR
cd $OUTPUT_DIR

echo "[*] Starting Bug Bounty reconnaissance for $TARGET"

# 1. 被动枚举（快速）
echo "[+] Phase 1: Passive enumeration"
amass enum -passive -d $TARGET -o passive.txt -json passive.json

# 2. 主动枚举
echo "[+] Phase 2: Active enumeration"
amass enum -active -d $TARGET -o active.txt -json active.json

# 3. 暴力破解
echo "[+] Phase 3: Brute force"
amass enum -brute -d $TARGET \
  -w /opt/SecLists/Discovery/DNS/subdomains-top1million-20000.txt \
  -o brute.txt

# 4. 合并去重
echo "[+] Phase 4: Merging results"
cat passive.txt active.txt brute.txt | sort -u > all_subdomains.txt

# 5. 提取活跃子域名（需要httpx）
echo "[+] Phase 5: Finding live subdomains"
cat all_subdomains.txt | httpx -silent -o live_subdomains.txt

# 6. 生成可视化
echo "[+] Phase 6: Generating visualization"
amass viz -d $TARGET -d3 -o network_graph.html

echo "[+] Reconnaissance complete!"
echo "[+] Total subdomains: $(wc -l < all_subdomains.txt)"
echo "[+] Live subdomains: $(wc -l < live_subdomains.txt)"
```

### 场景2：企业资产发现

```bash
#!/bin/bash
# 企业资产发现脚本

ORG_NAME="Example Inc"
OUTPUT_DIR="intel_$(date +%Y%m%d)"

mkdir -p $OUTPUT_DIR

echo "[*] Enterprise Asset Discovery for: $ORG_NAME"

# 1. 情报收集 - 发现根域名
echo "[+] Step 1: Discovering root domains"
amass intel -org "$ORG_NAME" -whois -o $OUTPUT_DIR/domains.txt

# 2. 对每个域名进行深度枚举
echo "[+] Step 2: Deep enumeration of each domain"
while read domain; do
    echo "[*] Enumerating: $domain"
    amass enum -active -brute -d $domain \
      -o $OUTPUT_DIR/${domain}_results.txt \
      -json $OUTPUT_DIR/${domain}_results.json
done < $OUTPUT_DIR/domains.txt

# 3. 合并所有结果
cat $OUTPUT_DIR/*_results.txt | sort -u > $OUTPUT_DIR/all_assets.txt

# 4. ASN枚举
echo "[+] Step 3: ASN enumeration"
amass intel -org "$ORG_NAME" -asn -o $OUTPUT_DIR/asn_info.txt

# 5. IP范围收集
echo "[+] Step 4: IP range collection"
amass intel -org "$ORG_NAME" -cidr -o $OUTPUT_DIR/ip_ranges.txt

# 6. 生成报告
echo "[+] Step 5: Generating report"
cat > $OUTPUT_DIR/report.md << EOF
# Asset Discovery Report: $ORG_NAME

**Date:** $(date)

## Summary
- Root domains found: $(wc -l < $OUTPUT_DIR/domains.txt)
- Total subdomains: $(wc -l < $OUTPUT_DIR/all_assets.txt)

## Root Domains
$(cat $OUTPUT_DIR/domains.txt)

## ASN Information
$(cat $OUTPUT_DIR/asn_info.txt)

## IP Ranges
$(cat $OUTPUT_DIR/ip_ranges.txt)
EOF

echo "[+] Asset discovery complete! Report: $OUTPUT_DIR/report.md"
```

### 场景3：持续监控

```bash
#!/bin/bash
# 域名变化监控脚本

DOMAIN=$1
ALERT_EMAIL="admin@example.com"

# 执行新枚举
amass enum -d $DOMAIN -o /tmp/current_scan.txt

# 检查变化
CHANGES=$(amass track -d $DOMAIN)

if [ ! -z "$CHANGES" ]; then
    echo "New subdomains discovered:"
    echo "$CHANGES"

    # 发送告警邮件
    echo "$CHANGES" | mail -s "New subdomains found for $DOMAIN" $ALERT_EMAIL
fi
```

### 场景4：深度网络映射

```bash
#!/bin/bash
# 深度网络映射

TARGET=$1

echo "[*] Deep network mapping for $TARGET"

# 1. 完整枚举
amass enum -active -brute -alts -d $TARGET \
  -w /opt/SecLists/Discovery/DNS/dns-Jhaddix.txt \
  -ip -src \
  -json full_enum.json \
  -o full_enum.txt

# 2. ASN发现
echo "[+] Discovering ASNs"
cat full_enum.txt | while read subdomain; do
    ip=$(dig +short $subdomain | head -1)
    if [ ! -z "$ip" ]; then
        whois $ip | grep -i "origin" | awk '{print $NF}'
    fi
done | sort -u > asns.txt

# 3. 对每个ASN进行枚举
cat asns.txt | while read asn; do
    echo "[*] Enumerating ASN: $asn"
    amass intel -asn $asn -o asn_${asn}_domains.txt
done

# 4. 生成网络拓扑图
amass viz -d $TARGET -d3 -o network_topology.html

echo "[+] Network mapping complete!"
```

### 场景5：多目标批量扫描

```bash
#!/bin/bash
# 批量域名扫描

DOMAIN_LIST=$1
OUTPUT_BASE="batch_scan_$(date +%Y%m%d)"

mkdir -p $OUTPUT_BASE

# 被动模式批量扫描（快速）
echo "[*] Passive scan of all domains"
amass enum -passive -df $DOMAIN_LIST -o $OUTPUT_BASE/passive_all.txt

# 对每个域名进行深度扫描
cat $DOMAIN_LIST | while read domain; do
    echo "[*] Deep scan: $domain"

    amass enum -active -brute -d $domain \
      -w /opt/SecLists/Discovery/DNS/subdomains-top1million-5000.txt \
      -o $OUTPUT_BASE/${domain}_results.txt \
      -json $OUTPUT_BASE/${domain}_results.json &

    # 限制并发数
    if [[ $(jobs -r -p | wc -l) -ge 5 ]]; then
        wait -n
    fi
done

wait

# 统计结果
echo "[+] Scan complete!"
for file in $OUTPUT_BASE/*_results.txt; do
    domain=$(basename $file _results.txt)
    count=$(wc -l < $file)
    echo "  $domain: $count subdomains"
done
```

---

## API集成

### 配置API密钥

```bash
# 编辑配置文件
vim ~/.config/amass/config.yaml

# 或使用环境变量
export AMASS_CONFIG=~/.config/amass/config.yaml
```

### 推荐API服务

#### 必备API（免费）

```yaml
datasources:
  # VirusTotal - 必备
  - name: VirusTotal
    key: YOUR_VT_API_KEY

  # GitHub - 强烈推荐
  - name: GitHub
    key: YOUR_GITHUB_TOKEN

  # AlienVault - 推荐
  - name: AlienVault
    key: YOUR_OTX_KEY

  # URLScan - 推荐
  - name: URLScan
    key: YOUR_URLSCAN_KEY
```

#### 付费API（效果显著）

```yaml
  # SecurityTrails - 最有价值
  - name: SecurityTrails
    key: YOUR_ST_KEY

  # Censys - 推荐
  - name: Censys
    credentials:
      apikey: YOUR_CENSYS_ID
      secret: YOUR_CENSYS_SECRET

  # Shodan - 推荐
  - name: Shodan
    key: YOUR_SHODAN_KEY
```

---

## 数据库管理

### 数据库位置

```bash
# 默认位置
~/.config/amass/amass.sqlite

# 查看数据库
amass db -list
```

### 数据库操作

```bash
# 列出所有枚举
amass db -list

# 查看特定域名数据
amass db -d example.com -names

# 显示详细信息
amass db -d example.com -show

# 按时间过滤
amass db -d example.com -since "2024-01-01" -names

# 导出数据
amass db -d example.com -names -o exported_domains.txt

# 清理旧数据（小心！）
rm ~/.config/amass/amass.sqlite
```

### 导入导出

```bash
# 导出JSON
amass enum -d example.com -json export.json

# 导入数据
amass db -import export.json

# 导出可视化数据
amass viz -d example.com -gexf -o network.gexf
```

---

## 可视化分析

### D3.js交互图

```bash
# 生成HTML可视化
amass viz -d example.com -d3 -o graph.html

# 在浏览器中打开
open graph.html  # macOS
xdg-open graph.html  # Linux
```

### Graphviz图形

```bash
# 生成DOT文件
amass viz -d example.com -dot | dot -Tpng -o network.png

# 生成SVG
amass viz -d example.com -dot | dot -Tsvg -o network.svg

# 生成PDF
amass viz -d example.com -dot | dot -Tpdf -o network.pdf
```

### Maltego集成

```bash
# 导出Maltego CSV
amass viz -d example.com -maltego -o maltego_import.csv

# 在Maltego中导入
# Transform Hub -> Import -> CSV
```

### Gephi可视化

```bash
# 导出GEXF格式
amass viz -d example.com -gexf -o network.gexf

# 在Gephi中打开
# File -> Open -> network.gexf
```

---

## 性能优化

### 1. 并发控制

```bash
# 增加DNS查询限制（更快但可能被限速）
amass enum -d example.com -max-dns-queries 50000

# 减少查询（更慢但更稳定）
amass enum -d example.com -max-dns-queries 5000
```

### 2. 超时设置

```bash
# 设置DNS超时
amass enum -d example.com -timeout 10

# 被动模式（最快）
amass enum -passive -d example.com
```

### 3. 资源限制

```bash
# 限制递归深度
amass enum -d example.com -max-depth 1

# 禁用某些数据源
amass enum -d example.com -nolocaldb -noalts
```

### 4. 分布式扫描

```bash
# 分割域名列表
split -l 10 domains.txt chunk_

# 并行执行
for chunk in chunk_*; do
    amass enum -df $chunk -o ${chunk}_results.txt &
done
wait

# 合并结果
cat chunk_*_results.txt | sort -u > final_results.txt
```

---

## 输出格式

### 文本格式

```bash
# 基本文本输出
amass enum -d example.com -o results.txt

# 带IP地址
amass enum -d example.com -ip -o results_with_ip.txt

# 带数据源
amass enum -d example.com -src -o results_with_source.txt

# 详细输出
amass enum -d example.com -ip -src -v -o detailed_results.txt
```

### JSON格式

```bash
# JSON输出
amass enum -d example.com -json results.json

# JSON结构
{
  "name": "subdomain.example.com",
  "domain": "example.com",
  "addresses": [
    {
      "ip": "192.0.2.1",
      "cidr": "192.0.2.0/24",
      "asn": 15169,
      "desc": "GOOGLE"
    }
  ],
  "tag": "dns",
  "sources": ["Crtsh", "VirusTotal"]
}
```

### 处理JSON输出

```bash
# 提取所有子域名
cat results.json | jq -r '.name' | sort -u

# 提取IP地址
cat results.json | jq -r '.addresses[].ip' | sort -u

# 按数据源分类
cat results.json | jq -r '.sources[]' | sort | uniq -c

# 提取ASN
cat results.json | jq -r '.addresses[].asn' | sort -u

# 生成CSV
cat results.json | jq -r '[.name, .addresses[0].ip, .tag] | @csv'
```

---

## 实战案例

### 案例1：快速Bug Bounty侦察

```bash
# 15分钟快速侦察
amass enum -passive -d target.com -o quick_recon.txt
cat quick_recon.txt | httpx -silent -o live_targets.txt
cat live_targets.txt | nuclei -t cves/ -o vulnerabilities.txt
```

### 案例2：完整资产清点

```bash
# 企业完整资产发现（耗时较长）
# 1. 发现根域名
amass intel -org "Target Corp" -whois -o root_domains.txt

# 2. 深度枚举
cat root_domains.txt | while read domain; do
    amass enum -active -brute -alts -d $domain \
      -config ~/.config/amass/config.yaml \
      -o ${domain}_assets.txt
done

# 3. 网络映射
cat *_assets.txt | sort -u | while read sub; do
    amass viz -d $sub -d3
done
```

### 案例3：APT威胁情报

```bash
# 发现攻击者基础设施
# 已知IOC: attacker-domain.com

# 1. 枚举攻击域名
amass enum -active -d attacker-domain.com -o attacker_infra.txt

# 2. 发现关联ASN
cat attacker_infra.txt | while read domain; do
    ip=$(dig +short $domain | head -1)
    whois $ip | grep -i origin
done | sort -u

# 3. 扩展发现
amass intel -asn <ASN_NUMBER> -o related_domains.txt
```

---

## 与其他工具组合

### 与Subfinder组合

```bash
# Amass深度 + Subfinder速度
subfinder -d target.com -silent -o subfinder.txt
amass enum -passive -d target.com -o amass.txt
cat subfinder.txt amass.txt | sort -u > all_subs.txt
```

### 与httpx组合

```bash
# 发现 -> 验证活跃
amass enum -d target.com -o subs.txt
cat subs.txt | httpx -silent -title -tech-detect -o live_subs.txt
```

### 与Nuclei组合

```bash
# 侦察 -> 漏洞扫描
amass enum -d target.com -o targets.txt
cat targets.txt | httpx -silent | nuclei -t cves/
```

### 与Nmap组合

```bash
# 域名发现 -> 端口扫描
amass enum -d target.com -ip -o subs_with_ip.txt
cat subs_with_ip.txt | awk '{print $2}' | sort -u | nmap -iL - -p-
```

### 完整工作流

```bash
#!/bin/bash
# 完整Bug Bounty工作流

TARGET=$1

# 1. 子域名发现
amass enum -passive -d $TARGET -o subs.txt
subfinder -d $TARGET -silent >> subs.txt
cat subs.txt | sort -u > unique_subs.txt

# 2. 活跃性检测
cat unique_subs.txt | httpx -silent -title -tech-detect -o live.txt

# 3. 端口扫描
cat live.txt | awk '{print $1}' | nmap -iL - -p- -oA nmap_scan

# 4. 漏洞扫描
cat live.txt | nuclei -t cves/ -o vulns.txt

# 5. 截图
cat live.txt | aquatone -out screenshots/

echo "[+] Workflow complete!"
```

---

## 常见问题

### 1. 速度太慢怎么办？

```bash
# 使用被动模式
amass enum -passive -d example.com

# 增加DNS查询限制
amass enum -d example.com -max-dns-queries 50000

# 禁用递归和变体
amass enum -d example.com -norecursive -noalts
```

### 2. 内存占用过高

```bash
# 限制并发
amass enum -d example.com -max-dns-queries 5000

# 分批处理
# 不要一次扫描太多域名
```

### 3. API配额用尽

```bash
# 使用被动模式减少API调用
amass enum -passive -d example.com

# 禁用特定数据源
# 在config.yaml中注释掉已用尽的API
```

### 4. 如何提高结果准确性？

```bash
# 使用主动模式验证
amass enum -active -d example.com

# 配置多个DNS解析器
amass enum -d example.com -rf resolvers.txt

# 使用配置文件启用所有数据源
amass enum -d example.com -config config.yaml
```

### 5. 数据库错误

```bash
# 删除损坏的数据库
rm ~/.config/amass/amass.sqlite

# 重新初始化
amass enum -d example.com
```

---

## 最佳实践

### ✅ 推荐做法

1. **配置API密钥**
   - 至少配置VirusTotal、GitHub、AlienVault
   - 定期检查API配额

2. **分阶段扫描**
   ```bash
   # 第一阶段：被动快速扫描
   amass enum -passive -d target.com -o passive.txt

   # 第二阶段：主动验证
   amass enum -active -d target.com -o active.txt

   # 第三阶段：暴力破解
   amass enum -brute -d target.com -w wordlist.txt
   ```

3. **保存配置和数据**
   - 使用配置文件管理API
   - 定期备份数据库
   - 保存JSON格式便于分析

4. **监控变化**
   ```bash
   # 定期运行track命令
   amass track -d target.com
   ```

5. **结合其他工具**
   - Amass负责发现
   - httpx验证活跃性
   - Nuclei检测漏洞

### ❌ 避免错误

1. **不要**无配置直接使用
   - 效果会大打折扣
   - 配置API密钥很重要

2. **不要**忽略被动模式
   - 被动模式速度快
   - 适合初步侦察

3. **不要**过度依赖暴力破解
   - 结合多种技术
   - 暴力破解作为补充

4. **不要**忽略数据库
   - track功能很有用
   - 历史数据有价值

### 🎯 效率技巧

1. **使用别名**
   ```bash
   alias amass-quick='amass enum -passive'
   alias amass-deep='amass enum -active -brute'
   ```

2. **自动化脚本**
   - 编写wrapper脚本
   - 集成到工作流

3. **结果处理**
   ```bash
   # 提取子域名
   cat results.json | jq -r '.name' | sort -u

   # 统计数据源
   cat results.json | jq -r '.sources[]' | sort | uniq -c | sort -rn
   ```

---

## 参考资源

### 官方资源
- **GitHub**: https://github.com/owasp-amass/amass
- **文档**: https://github.com/owasp-amass/amass/blob/master/doc/user_guide.md
- **OWASP**: https://owasp.org/www-project-amass/

### 学习资源
- **HackTricks**: https://book.hacktricks.xyz/generic-methodologies-and-resources/external-recon-methodology
- **YouTube**: Search "Amass Tutorial"
- **Blog Posts**: https://hakluke.medium.com/

### 相关工具
- **Subfinder**: https://github.com/projectdiscovery/subfinder
- **Assetfinder**: https://github.com/tomnomnom/assetfinder
- **Findomain**: https://github.com/Findomain/Findomain

---

## 速查表

### 常用命令

| 场景 | 命令 |
|------|------|
| 快速被动扫描 | `amass enum -passive -d target.com` |
| 深度主动扫描 | `amass enum -active -brute -d target.com` |
| 使用配置 | `amass enum -d target.com -config config.yaml` |
| 显示IP | `amass enum -d target.com -ip` |
| JSON输出 | `amass enum -d target.com -json results.json` |
| 情报收集 | `amass intel -org "Company Name"` |
| 可视化 | `amass viz -d target.com -d3 -o graph.html` |
| 追踪变化 | `amass track -d target.com` |
| 查看数据库 | `amass db -d target.com -names` |

### 完整扫描模板

```bash
# Bug Bounty完整扫描
amass enum \
  -active \
  -brute \
  -d target.com \
  -w /opt/SecLists/Discovery/DNS/subdomains-top1million-20000.txt \
  -config ~/.config/amass/config.yaml \
  -ip \
  -src \
  -o results.txt \
  -json results.json \
  -dir ./amass_output \
  -v
```

---

**最后更新:** 2025-10-04
**Amass版本:** v4.2.0+
**适用场景:** 渗透测试、Bug Bounty、威胁情报、资产管理
**维护组织:** OWASP
