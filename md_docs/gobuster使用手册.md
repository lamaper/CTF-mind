# Gobuster 使用手册

## 目录
- [简介](#简介)
- [安装](#安装)
- [基本用法](#基本用法)
- [模式详解](#模式详解)
  - [dir 模式 - 目录/文件枚举](#dir-模式---目录文件枚举)
  - [dns 模式 - DNS子域名枚举](#dns-模式---dns子域名枚举)
  - [vhost 模式 - 虚拟主机枚举](#vhost-模式---虚拟主机枚举)
  - [s3 模式 - AWS S3存储桶枚举](#s3-模式---aws-s3存储桶枚举)
  - [gcs 模式 - Google Cloud存储枚举](#gcs-模式---google-cloud存储枚举)
  - [tftp 模式 - TFTP服务器枚举](#tftp-模式---tftp服务器枚举)
- [常用参数](#常用参数)
- [实战示例](#实战示例)
- [字典文件推荐](#字典文件推荐)
- [技巧和最佳实践](#技巧和最佳实践)

---

## 简介

Gobuster 是一个用 Go 语言编写的高性能目录/文件、DNS和虚拟主机枚举工具。它通过暴力破解的方式发现隐藏的目录、文件、子域名等资源。

**主要特点：**
- 速度快：Go语言编写，多线程并发
- 多种扫描模式：支持目录、DNS、虚拟主机、S3等
- 灵活配置：支持自定义字典、代理、认证等
- 输出友好：支持多种输出格式

---

## 安装

### macOS
```bash
brew install gobuster
```

### Linux (Debian/Ubuntu)
```bash
sudo apt update
sudo apt install gobuster
```

### Linux (通用 - 使用 Go)
```bash
go install github.com/OJ/gobuster/v3@latest
```

### Kali Linux
```bash
# Kali 预装 gobuster
gobuster version
```

### 验证安装
```bash
gobuster version
# 输出: Gobuster v3.x.x
```

---

## 基本用法

```bash
gobuster [模式] [选项]
```

**可用模式：**
- `dir` - 目录/文件枚举
- `dns` - DNS子域名枚举
- `vhost` - 虚拟主机枚举
- `s3` - AWS S3存储桶枚举
- `gcs` - Google Cloud存储枚举
- `tftp` - TFTP服务器枚举

---

## 模式详解

### dir 模式 - 目录/文件枚举

最常用的模式，用于发现Web应用中的隐藏目录和文件。

#### 基本语法
```bash
gobuster dir -u <URL> -w <字典文件>
```

#### 常用参数

| 参数 | 说明 | 示例 |
|------|------|------|
| `-u, --url` | 目标URL（必需） | `-u https://example.com` |
| `-w, --wordlist` | 字典文件路径（必需） | `-w /path/wordlist.txt` |
| `-t, --threads` | 并发线程数（默认10） | `-t 50` |
| `-x, --extensions` | 文件扩展名（逗号分隔） | `-x php,html,txt` |
| `-s, --status-codes` | 显示的状态码（默认200,204,301,302,307,401,403） | `-s 200,204,301` |
| `-b, --status-codes-blacklist` | 排除的状态码 | `-b 404,403` |
| `-k, --no-tls-validation` | 跳过SSL证书验证 | `-k` |
| `-a, --useragent` | 自定义User-Agent | `-a "Mozilla/5.0..."` |
| `-H, --headers` | 自定义请求头 | `-H "Authorization: Bearer token"` |
| `-c, --cookies` | 自定义Cookie | `-c "session=abc123"` |
| `-U, --username` | HTTP基本认证用户名 | `-U admin` |
| `-P, --password` | HTTP基本认证密码 | `-P password123` |
| `-p, --proxy` | 代理服务器 | `-p http://127.0.0.1:8080` |
| `-r, --follow-redirect` | 跟随重定向 | `-r` |
| `-e, --expanded` | 显示完整URL | `-e` |
| `-n, --no-status` | 不显示状态码 | `-n` |
| `-q, --quiet` | 安静模式（只显示结果） | `-q` |
| `-o, --output` | 输出到文件 | `-o results.txt` |
| `--timeout` | 请求超时时间 | `--timeout 30s` |
| `--delay` | 请求延迟 | `--delay 100ms` |
| `--no-error` | 不显示错误信息 | `--no-error` |

#### 示例

**1. 基本目录扫描**
```bash
gobuster dir -u https://example.com -w /usr/share/wordlists/dirb/common.txt
```

**2. 扫描指定扩展名文件**
```bash
gobuster dir -u https://example.com -w wordlist.txt -x php,html,txt,js
```

**3. 跳过SSL证书验证**
```bash
gobuster dir -u https://example.com -w wordlist.txt -k
```

**4. 排除特定状态码**
```bash
gobuster dir -u https://example.com -w wordlist.txt -b 404,403
```

**5. 增加线程数加快扫描**
```bash
gobuster dir -u https://example.com -w wordlist.txt -t 50
```

**6. 使用代理**
```bash
gobuster dir -u https://example.com -w wordlist.txt -p http://127.0.0.1:8080
```

**7. 自定义请求头**
```bash
gobuster dir -u https://example.com -w wordlist.txt -H "Authorization: Bearer token123"
```

**8. 完整扫描示例（跳过SSL + 排除404 + 多线程 + 指定扩展名）**
```bash
gobuster dir -u https://example.com \
  -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt \
  -x php,html,txt,js,zip,bak \
  -t 50 \
  -k \
  -b 404 \
  -e \
  -o scan_results.txt
```

---

### dns 模式 - DNS子域名枚举

用于发现目标域名的子域名。

#### 基本语法
```bash
gobuster dns -d <域名> -w <字典文件>
```

#### 常用参数

| 参数 | 说明 | 示例 |
|------|------|------|
| `-d, --domain` | 目标域名（必需） | `-d example.com` |
| `-w, --wordlist` | 字典文件（必需） | `-w subdomains.txt` |
| `-r, --resolver` | 自定义DNS服务器 | `-r 8.8.8.8` |
| `-i, --show-ips` | 显示IP地址 | `-i` |
| `-c, --show-cname` | 显示CNAME记录 | `-c` |
| `-t, --threads` | 并发线程数 | `-t 50` |

#### 示例

**1. 基本子域名扫描**
```bash
gobuster dns -d example.com -w /usr/share/wordlists/subdomains.txt
```

**2. 显示IP和CNAME**
```bash
gobuster dns -d example.com -w subdomains.txt -i -c
```

**3. 使用自定义DNS服务器**
```bash
gobuster dns -d example.com -w subdomains.txt -r 8.8.8.8
```

**4. 多线程扫描**
```bash
gobuster dns -d example.com -w subdomains.txt -t 50 -i
```

---

### vhost 模式 - 虚拟主机枚举

用于发现同一IP上托管的不同虚拟主机。

#### 基本语法
```bash
gobuster vhost -u <URL> -w <字典文件>
```

#### 常用参数

| 参数 | 说明 | 示例 |
|------|------|------|
| `-u, --url` | 目标URL（必需） | `-u https://example.com` |
| `-w, --wordlist` | 字典文件（必需） | `-w vhosts.txt` |
| `-t, --threads` | 并发线程数 | `-t 50` |
| `-k, --no-tls-validation` | 跳过SSL验证 | `-k` |

#### 示例

**1. 基本虚拟主机扫描**
```bash
gobuster vhost -u https://example.com -w vhosts.txt
```

**2. 跳过SSL验证**
```bash
gobuster vhost -u https://example.com -w vhosts.txt -k
```

---

### s3 模式 - AWS S3存储桶枚举

用于发现公开的AWS S3存储桶。

#### 基本语法
```bash
gobuster s3 -w <字典文件>
```

#### 示例

```bash
gobuster s3 -w bucket-names.txt
```

---

### gcs 模式 - Google Cloud存储枚举

用于发现Google Cloud存储桶。

#### 基本语法
```bash
gobuster gcs -w <字典文件>
```

---

### tftp 模式 - TFTP服务器枚举

用于枚举TFTP服务器上的文件。

#### 基本语法
```bash
gobuster tftp -s <服务器IP> -w <字典文件>
```

---

## 常用参数

### 全局参数（适用所有模式）

| 参数 | 说明 |
|------|------|
| `-t, --threads` | 并发线程数（默认10） |
| `-w, --wordlist` | 字典文件路径 |
| `-o, --output` | 输出结果到文件 |
| `-q, --quiet` | 安静模式 |
| `-v, --verbose` | 详细输出 |
| `--delay` | 请求之间的延迟 |
| `--no-error` | 不显示错误 |
| `-h, --help` | 显示帮助 |

---

## 实战示例

### 场景1：Web渗透测试 - 目录扫描

```bash
# 1. 快速扫描常见目录
gobuster dir -u https://target.com -w /usr/share/wordlists/dirb/common.txt -t 30 -k

# 2. 深度扫描（包含文件扩展名）
gobuster dir -u https://target.com \
  -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt \
  -x php,html,txt,js,zip,bak,old,sql \
  -t 50 \
  -k \
  -b 404,403 \
  -e \
  -o deep_scan.txt

# 3. 扫描特定目录下的子目录
gobuster dir -u https://target.com/admin -w wordlist.txt -t 30 -k

# 4. 扫描时添加认证头
gobuster dir -u https://target.com \
  -w wordlist.txt \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -t 30 -k
```

### 场景2：子域名枚举

```bash
# 1. 基本子域名扫描
gobuster dns -d target.com -w /usr/share/wordlists/subdomains-top1million-5000.txt -i

# 2. 使用多个DNS服务器
gobuster dns -d target.com \
  -w subdomains.txt \
  -r 8.8.8.8 \
  -i -c \
  -t 50 \
  -o subdomains.txt

# 3. 扫描并显示详细信息
gobuster dns -d target.com -w subdomains.txt -i -c --wildcard -v
```

### 场景3：CTF比赛扫描

```bash
# 1. 快速发现隐藏页面
gobuster dir -u http://ctf.challenge.com \
  -w /usr/share/wordlists/dirb/common.txt \
  -x php,txt,html \
  -t 50 \
  -b 404

# 2. 查找备份文件
gobuster dir -u http://ctf.challenge.com \
  -w wordlist.txt \
  -x bak,old,backup,zip,tar.gz,sql \
  -t 50

# 3. 扫描robots.txt和sitemap.xml中提到的路径
# 先查看这些文件，然后针对性扫描
curl http://ctf.challenge.com/robots.txt
curl http://ctf.challenge.com/sitemap.xml
```

### 场景4：API端点发现

```bash
# 扫描API端点
gobuster dir -u https://api.target.com \
  -w /usr/share/wordlists/api-endpoints.txt \
  -x json \
  -t 50 \
  -k \
  -H "Content-Type: application/json"
```

---

## 字典文件推荐

### 常见字典位置（Kali/ParrotOS）

```bash
# 目录枚举
/usr/share/wordlists/dirb/common.txt           # 小型，快速扫描
/usr/share/wordlists/dirb/big.txt              # 中型
/usr/share/wordlists/dirbuster/directory-list-2.3-small.txt    # 小型
/usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt   # 中型（推荐）
/usr/share/wordlists/dirbuster/directory-list-2.3-big.txt      # 大型

# 子域名枚举
/usr/share/wordlists/subdomains-top1million-5000.txt
/usr/share/wordlists/subdomains-top1million-20000.txt

# SecLists（推荐下载）
https://github.com/danielmiessler/SecLists
```

### 下载SecLists

```bash
# Kali Linux
sudo apt install seclists

# 手动下载
git clone https://github.com/danielmiessler/SecLists.git /opt/SecLists
```

### 创建自定义字典

```bash
# 创建常见目录字典
cat > custom_dirs.txt << EOF
admin
login
dashboard
api
upload
backup
config
.git
.env
robots.txt
sitemap.xml
EOF
```

---

## 技巧和最佳实践

### 1. 性能优化

```bash
# 增加线程数（但不要太高，避免被Ban）
gobuster dir -u https://target.com -w wordlist.txt -t 50

# 设置超时时间
gobuster dir -u https://target.com -w wordlist.txt --timeout 10s

# 减少请求延迟（小心被WAF拦截）
gobuster dir -u https://target.com -w wordlist.txt --delay 0
```

### 2. 绕过WAF

```bash
# 使用随机User-Agent
gobuster dir -u https://target.com -w wordlist.txt \
  -a "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36"

# 添加延迟避免触发速率限制
gobuster dir -u https://target.com -w wordlist.txt --delay 200ms

# 减少并发线程
gobuster dir -u https://target.com -w wordlist.txt -t 5
```

### 3. 组合使用

```bash
# 第一步：快速扫描
gobuster dir -u https://target.com -w /usr/share/wordlists/dirb/common.txt -t 30 -k -o quick.txt

# 第二步：深度扫描发现的目录
cat quick.txt | grep "Status: 200" | awk '{print $1}' | while read dir; do
  gobuster dir -u "https://target.com/$dir" -w wordlist.txt -t 30 -k
done
```

### 4. 过滤和排序结果

```bash
# 只显示200状态码
gobuster dir -u https://target.com -w wordlist.txt -s 200

# 排除常见错误码
gobuster dir -u https://target.com -w wordlist.txt -b 404,403,500

# 保存结果并排序
gobuster dir -u https://target.com -w wordlist.txt -o results.txt
cat results.txt | sort | uniq
```

### 5. 递归扫描

```bash
# Gobuster本身不支持递归，但可以用脚本实现
#!/bin/bash
URL="https://target.com"
WORDLIST="wordlist.txt"

function scan_dir() {
    local base_url=$1
    echo "[*] Scanning: $base_url"

    gobuster dir -u "$base_url" -w "$WORDLIST" -t 30 -k -q | while read line; do
        if [[ $line == *"Status: 200"* ]] || [[ $line == *"Status: 301"* ]]; then
            found_path=$(echo $line | awk '{print $1}')
            echo "[+] Found: $found_path"

            # 递归扫描新发现的目录
            if [[ $line == *"Status: 301"* ]]; then
                scan_dir "$base_url$found_path"
            fi
        fi
    done
}

scan_dir "$URL"
```

### 6. 日志记录

```bash
# 保存详细日志
gobuster dir -u https://target.com -w wordlist.txt -o results.txt -v 2>&1 | tee scan.log

# 保存不同格式
gobuster dir -u https://target.com -w wordlist.txt -o results.json --output-format json
```

### 7. 与其他工具配合

```bash
# 先用nmap发现Web服务
nmap -p- -A target.com -oN nmap.txt

# 然后用gobuster扫描发现的端口
gobuster dir -u https://target.com:8443 -w wordlist.txt

# 结合ffuf进行参数模糊测试
ffuf -u https://target.com/api/FUZZ -w wordlist.txt
```

---

## 故障排除

### 问题1：请求过快被Ban

```bash
# 解决：降低并发数和增加延迟
gobuster dir -u https://target.com -w wordlist.txt -t 5 --delay 500ms
```

### 问题2：SSL证书错误

```bash
# 解决：跳过SSL验证
gobuster dir -u https://target.com -w wordlist.txt -k
```

### 问题3：403 Forbidden

```bash
# 解决：修改User-Agent和Headers
gobuster dir -u https://target.com -w wordlist.txt \
  -a "Mozilla/5.0..." \
  -H "X-Forwarded-For: 127.0.0.1"
```

### 问题4：扫描速度慢

```bash
# 解决：增加线程数（注意不要触发WAF）
gobuster dir -u https://target.com -w wordlist.txt -t 100 --timeout 5s
```

---

## 总结

Gobuster是渗透测试和CTF比赛中不可或缺的工具，掌握其各种模式和参数可以大大提高信息收集的效率。

**核心要点：**

1. 选择合适的字典文件
2. 根据目标调整线程数和延迟
3. 注意绕过WAF和速率限制
4. 结合其他工具进行深度挖掘
5. 保存和分析扫描结果

**学习资源：**
- 官方文档：https://github.com/OJ/gobuster
- SecLists字典：https://github.com/danielmiessler/SecLists
- HackTricks：https://book.hacktricks.xyz/

---

#### 示例脚本

```shell
gobuster dir --url http://10.201.12.107/ -w /Volumes/D/ctf/wordlist/SecLists/Discovery/Web-Content/common.txt

```



**最后更新：** 2025-10-03
**版本：** Gobuster v3.6+
