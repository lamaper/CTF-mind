# FFUF 使用手册

## 目录
- [简介](#简介)
- [安装](#安装)
- [基本概念](#基本概念)
- [基本用法](#基本用法)
- [FUZZ关键字](#fuzz关键字)
- [常用参数](#常用参数)
- [过滤和匹配](#过滤和匹配)
- [实战场景](#实战场景)
  - [目录和文件枚举](#目录和文件枚举)
  - [子域名枚举](#子域名枚举)
  - [虚拟主机发现](#虚拟主机发现)
  - [参数模糊测试](#参数模糊测试)
  - [POST数据模糊测试](#post数据模糊测试)
  - [Header模糊测试](#header模糊测试)
  - [认证绕过](#认证绕过)
  - [SQL注入检测](#sql注入检测)
  - [XSS检测](#xss检测)
- [高级技巧](#高级技巧)
- [字典文件](#字典文件)
- [输出和报告](#输出和报告)
- [性能优化](#性能优化)
- [实战案例](#实战案例)
- [常见问题](#常见问题)

---

## 简介

**FFUF** (Fuzz Faster U Fool) 是一个用 Go 语言编写的快速 Web 模糊测试工具，主要用于Web应用的安全测试。

**主要特点：**
- ⚡ 速度极快：Go 并发，性能优异
- 🎯 灵活强大：支持多位置同时FUZZ
- 🔍 智能过滤：丰富的匹配和过滤选项
- 📊 输出多样：支持JSON、CSV、HTML等格式
- 🛠️ 功能全面：目录、参数、Header、POST数据等全方位模糊测试

**与其他工具对比：**
- vs Gobuster：FFUF更灵活，支持更多FUZZ场景
- vs wfuzz：FFUF速度更快，语法更简洁
- vs dirb/dirbuster：FFUF性能更好，功能更强

---

## 安装

### macOS
```bash
brew install ffuf
```

### Linux (使用 Go)
```bash
go install github.com/ffuf/ffuf/v2@latest
```

### Linux (预编译二进制)
```bash
# 下载最新版本
wget https://github.com/ffuf/ffuf/releases/download/v2.1.0/ffuf_2.1.0_linux_amd64.tar.gz

# 解压
tar -xzf ffuf_2.1.0_linux_amd64.tar.gz

# 移动到PATH
sudo mv ffuf /usr/local/bin/

# 验证安装
ffuf -V
```

### Kali Linux
```bash
sudo apt update
sudo apt install ffuf
```

### Docker
```bash
docker pull ffuf/ffuf
docker run --rm ffuf/ffuf -V
```

### 验证安装
```bash
ffuf -V
# 输出: ffuf version 2.x.x
```

---

## 基本概念

### FUZZ 关键字
FFUF使用 `FUZZ` 关键字标记需要替换的位置，可以使用多个FUZZ关键字（FUZZ, FUZ2Z, FUZ3Z等）。

```bash
# 单个FUZZ位置
ffuf -u https://target.com/FUZZ -w wordlist.txt

# 多个FUZZ位置
ffuf -u https://target.com/FUZZ/FUZ2Z -w wordlist1.txt:FUZZ -w wordlist2.txt:FUZ2Z
```

### 工作原理
1. 读取字典文件
2. 替换URL中的FUZZ关键字
3. 发送HTTP请求
4. 根据响应进行匹配/过滤
5. 输出符合条件的结果

---

## 基本用法

### 最简单的目录扫描
```bash
ffuf -u https://target.com/FUZZ -w wordlist.txt
```

### 完整格式
```bash
ffuf -u <URL> -w <字典文件> [选项]
```

### 快速示例
```bash
# 目录扫描
ffuf -u https://example.com/FUZZ -w /usr/share/wordlists/dirb/common.txt

# 子域名枚举
ffuf -u https://FUZZ.example.com -w subdomains.txt

# GET参数测试
ffuf -u https://example.com/?id=FUZZ -w numbers.txt

# POST参数测试
ffuf -u https://example.com/login -w wordlist.txt -X POST -d "username=admin&password=FUZZ"

# Header模糊测试
ffuf -u https://example.com -w headers.txt -H "X-Custom-Header: FUZZ"
```

---

## FUZZ关键字

### 单个FUZZ
```bash
ffuf -u https://target.com/FUZZ -w wordlist.txt
```

### 多个FUZZ（笛卡尔积）
```bash
# 使用两个字典
ffuf -u https://target.com/FUZZ/FUZ2Z \
  -w dirs.txt:FUZZ \
  -w files.txt:FUZ2Z

# 示例：
# dirs.txt: admin, api
# files.txt: config.php, index.html
# 结果：admin/config.php, admin/index.html, api/config.php, api/index.html
```

### 自定义FUZZ关键字
```bash
# 可以使用任意名称（区分大小写）
ffuf -u https://target.com/DIR/FILE \
  -w dirs.txt:DIR \
  -w files.txt:FILE
```

### 命令行模式（clusterbomb）
```bash
# 默认模式：笛卡尔积（所有组合）
ffuf -u https://target.com/FUZZ/FUZ2Z -w w1.txt:FUZZ -w w2.txt:FUZ2Z -mode clusterbomb

# pitchfork模式：一一对应
ffuf -u https://target.com/FUZZ/FUZ2Z -w w1.txt:FUZZ -w w2.txt:FUZ2Z -mode pitchfork
```

---

## 常用参数

### 核心参数

| 参数 | 说明 | 示例 |
|------|------|------|
| `-u` | 目标URL（必需，支持FUZZ） | `-u https://target.com/FUZZ` |
| `-w` | 字典文件（必需） | `-w wordlist.txt` |
| `-w` | 多字典（带标签） | `-w w1.txt:FUZZ -w w2.txt:FUZ2Z` |
| `-X` | HTTP方法 | `-X POST` |
| `-d` | POST数据 | `-d "user=admin&pass=FUZZ"` |
| `-H` | 请求头 | `-H "Authorization: Bearer FUZZ"` |
| `-b` | Cookie | `-b "session=FUZZ"` |
| `-r` | 跟随重定向 | `-r` |
| `-recursion` | 递归扫描 | `-recursion` |
| `-recursion-depth` | 递归深度 | `-recursion-depth 2` |
| `-e` | 扩展名 | `-e .php,.html,.txt` |
| `-t` | 并发线程数 | `-t 100` |
| `-p` | 延迟（秒） | `-p 0.1` |
| `-rate` | 每秒请求数 | `-rate 100` |
| `-timeout` | 超时时间（秒） | `-timeout 10` |
| `-v` | 详细输出 | `-v` |
| `-s` | 静默模式 | `-s` |
| `-c` | 彩色输出 | `-c` |
| `-o` | 输出文件 | `-o results.json` |
| `-of` | 输出格式 | `-of json` |

### HTTP参数

| 参数 | 说明 | 示例 |
|------|------|------|
| `-x` | 代理 | `-x http://127.0.0.1:8080` |
| `-replay-proxy` | 匹配结果重放代理 | `-replay-proxy http://127.0.0.1:8080` |
| `-k` | 跳过TLS验证 | `-k` |
| `-H` | 自定义请求头（可多次使用） | `-H "User-Agent: FFUF"` |
| `-b` | Cookie | `-b "session=abc123"` |
| `-d` | POST数据 | `-d "username=FUZZ"` |
| `-r` | 跟随重定向 | `-r` |
| `-recursion` | 递归模式 | `-recursion` |
| `-recursion-depth` | 递归深度 | `-recursion-depth 3` |
| `-u` | URL | `-u https://target.com/FUZZ` |
| `-X` | HTTP方法 | `-X POST` |

---

## 过滤和匹配

### 过滤器（Filter）- 排除不想要的结果

| 参数 | 说明 | 示例 |
|------|------|------|
| `-fc` | 过滤HTTP状态码 | `-fc 404,403` |
| `-fs` | 过滤响应大小（字节） | `-fs 4242` |
| `-fw` | 过滤响应字数 | `-fw 100` |
| `-fl` | 过滤响应行数 | `-fl 50` |
| `-ft` | 过滤响应时间（毫秒） | `-ft 1000` |
| `-fr` | 过滤正则表达式 | `-fr "error.*"` |

### 匹配器（Matcher）- 只显示想要的结果

| 参数 | 说明 | 示例 |
|------|------|------|
| `-mc` | 匹配HTTP状态码 | `-mc 200,301` |
| `-ms` | 匹配响应大小 | `-ms 1234` |
| `-mw` | 匹配响应字数 | `-mw 100` |
| `-ml` | 匹配响应行数 | `-ml 50` |
| `-mt` | 匹配响应时间 | `-mt 1000` |
| `-mr` | 匹配正则表达式 | `-mr "admin"` |

### 过滤示例

```bash
# 过滤404状态码
ffuf -u https://target.com/FUZZ -w wordlist.txt -fc 404

# 过滤特定大小的响应
ffuf -u https://target.com/FUZZ -w wordlist.txt -fs 4242

# 只显示200状态码
ffuf -u https://target.com/FUZZ -w wordlist.txt -mc 200

# 组合过滤
ffuf -u https://target.com/FUZZ -w wordlist.txt -fc 404,403 -fs 0 -fw 1

# 使用正则过滤错误页面
ffuf -u https://target.com/FUZZ -w wordlist.txt -fr "404|not found|error"

# 只显示包含特定内容的响应
ffuf -u https://target.com/FUZZ -w wordlist.txt -mr "admin|dashboard"
```

### 自动校准（Auto-Calibration）

```bash
# 自动过滤相同响应
ffuf -u https://target.com/FUZZ -w wordlist.txt -ac

# 自动校准+手动过滤
ffuf -u https://target.com/FUZZ -w wordlist.txt -ac -fc 404
```

---

## 实战场景

### 目录和文件枚举

#### 基本目录扫描
```bash
ffuf -u https://target.com/FUZZ -w /usr/share/wordlists/dirb/common.txt -fc 404
```

#### 扫描特定扩展名
```bash
# 方法1：使用-e参数
ffuf -u https://target.com/FUZZ -w wordlist.txt -e .php,.html,.txt,.js -fc 404

# 方法2：字典中包含扩展名
ffuf -u https://target.com/FUZZ.php -w wordlist.txt -fc 404

# 方法3：多个FUZZ位置
ffuf -u https://target.com/FUZZ.FUZ2Z \
  -w files.txt:FUZZ \
  -w extensions.txt:FUZ2Z \
  -fc 404
```

#### 递归目录扫描
```bash
# 递归扫描发现的目录
ffuf -u https://target.com/FUZZ \
  -w wordlist.txt \
  -recursion \
  -recursion-depth 3 \
  -fc 404 \
  -v
```

#### 扫描备份文件
```bash
# 创建备份文件字典
cat > backup_extensions.txt << EOF
.bak
.old
.backup
.zip
.tar.gz
.sql
~
.swp
.save
EOF

# 扫描备份文件
ffuf -u https://target.com/FUZZ.FUZ2Z \
  -w common_files.txt:FUZZ \
  -w backup_extensions.txt:FUZ2Z \
  -fc 404
```

### 子域名枚举

```bash
# 基本子域名扫描
ffuf -u https://FUZZ.target.com -w subdomains.txt -fc 404

# 使用大字典 + 过滤
ffuf -u https://FUZZ.target.com \
  -w /opt/SecLists/Discovery/DNS/subdomains-top1million-110000.txt \
  -fc 404 \
  -t 100

# 显示IP地址（需要配合其他工具）
ffuf -u https://FUZZ.target.com -w subdomains.txt -fc 404 -v | tee subdomains.txt

# 递归子域名（二级、三级）
ffuf -u https://FUZZ.FUZ2Z.target.com \
  -w subdomains1.txt:FUZZ \
  -w subdomains2.txt:FUZ2Z \
  -fc 404
```

### 虚拟主机发现

```bash
# 基本虚拟主机扫描
ffuf -u https://target.com -w vhosts.txt -H "Host: FUZZ.target.com" -fc 404

# 过滤默认虚拟主机响应
ffuf -u https://192.168.1.100 \
  -w vhosts.txt \
  -H "Host: FUZZ.target.com" \
  -fs 4242

# 使用自动校准
ffuf -u https://192.168.1.100 \
  -w vhosts.txt \
  -H "Host: FUZZ" \
  -ac
```

### 参数模糊测试

#### GET 参数
```bash
# 单个参数
ffuf -u https://target.com/search?q=FUZZ -w wordlist.txt -fc 404

# 多个参数
ffuf -u "https://target.com/api?user=FUZZ&role=FUZ2Z" \
  -w users.txt:FUZZ \
  -w roles.txt:FUZ2Z \
  -fc 404

# 参数名称模糊测试
ffuf -u "https://target.com/api?FUZZ=test" -w params.txt -fc 404

# 参数名称+值模糊测试
ffuf -u "https://target.com/api?FUZZ=FUZ2Z" \
  -w params.txt:FUZZ \
  -w values.txt:FUZ2Z \
  -fc 404
```

#### 数字范围测试
```bash
# 创建数字序列
seq 1 1000 > numbers.txt

# 测试ID参数
ffuf -u https://target.com/user?id=FUZZ -w numbers.txt -fc 404

# 或使用 -ic (ignore comments) 配合范围
for i in {1..1000}; do echo $i; done | ffuf -u https://target.com/user?id=FUZZ -w - -fc 404
```

### POST数据模糊测试

#### 表单登录测试
```bash
# 用户名模糊测试
ffuf -u https://target.com/login \
  -X POST \
  -d "username=FUZZ&password=test123" \
  -w usernames.txt \
  -fc 401 \
  -H "Content-Type: application/x-www-form-urlencoded"

# 密码模糊测试
ffuf -u https://target.com/login \
  -X POST \
  -d "username=admin&password=FUZZ" \
  -w passwords.txt \
  -fc 401

# 用户名和密码同时测试
ffuf -u https://target.com/login \
  -X POST \
  -d "username=FUZZ&password=FUZ2Z" \
  -w usernames.txt:FUZZ \
  -w passwords.txt:FUZ2Z \
  -mode clusterbomb \
  -fc 401
```

#### JSON POST数据
```bash
# JSON格式POST
ffuf -u https://target.com/api/login \
  -X POST \
  -d '{"username":"FUZZ","password":"test"}' \
  -w usernames.txt \
  -H "Content-Type: application/json" \
  -fc 401

# 复杂JSON结构
ffuf -u https://target.com/api/users \
  -X POST \
  -d '{"user":{"name":"FUZZ","role":"FUZ2Z"}}' \
  -w names.txt:FUZZ \
  -w roles.txt:FUZ2Z \
  -H "Content-Type: application/json" \
  -fc 404
```

### Header模糊测试

#### 常见Header测试
```bash
# User-Agent模糊测试
ffuf -u https://target.com -w user-agents.txt -H "User-Agent: FUZZ" -fc 403

# 自定义Header值
ffuf -u https://target.com -w values.txt -H "X-Forwarded-For: FUZZ" -fc 403

# Header名称模糊测试
ffuf -u https://target.com -w headers.txt -H "FUZZ: true" -fc 403

# Authorization Header
ffuf -u https://target.com/api -w tokens.txt -H "Authorization: Bearer FUZZ" -fc 401
```

#### 绕过访问控制
```bash
# IP绕过
cat > ip_headers.txt << EOF
X-Forwarded-For
X-Real-IP
X-Client-IP
X-Originating-IP
CF-Connecting-IP
True-Client-IP
EOF

# 测试不同Header
ffuf -u https://target.com/admin \
  -w ip_headers.txt:HEADER \
  -H "HEADER: 127.0.0.1" \
  -mode pitchfork \
  -fc 403
```

### 认证绕过

#### 基本认证
```bash
# HTTP Basic Auth
ffuf -u https://target.com/admin \
  -w users.txt:USER -w passwords.txt:PASS \
  -H "Authorization: Basic $(echo -n 'USER:PASS' | base64)" \
  -mode clusterbomb \
  -fc 401

# 简化方式（需要脚本处理）
```

#### Session/Cookie测试
```bash
# Cookie值模糊测试
ffuf -u https://target.com/admin \
  -w sessionids.txt \
  -b "session=FUZZ" \
  -fc 403

# 多个Cookie
ffuf -u https://target.com/admin \
  -w users.txt:USER -w sessions.txt:SESS \
  -b "user=USER; session=SESS" \
  -fc 403
```

### SQL注入检测

```bash
# 创建SQL注入payload字典
cat > sql_payloads.txt << EOF
'
"
' OR '1'='1
" OR "1"="1
' OR 1=1--
admin'--
' UNION SELECT NULL--
1' AND '1'='1
' AND 1=1--
EOF

# GET参数SQL注入
ffuf -u "https://target.com/user?id=FUZZ" \
  -w sql_payloads.txt \
  -fc 404 \
  -mr "error|sql|syntax|mysql|postgresql"

# POST SQL注入
ffuf -u https://target.com/login \
  -X POST \
  -d "username=admin&password=FUZZ" \
  -w sql_payloads.txt \
  -fc 401 \
  -mr "error|sql"
```

### XSS检测

```bash
# 创建XSS payload字典
cat > xss_payloads.txt << EOF
<script>alert(1)</script>
<img src=x onerror=alert(1)>
<svg/onload=alert(1)>
"><script>alert(1)</script>
'><script>alert(1)</script>
<iframe src=javascript:alert(1)>
EOF

# GET参数XSS
ffuf -u "https://target.com/search?q=FUZZ" \
  -w xss_payloads.txt \
  -fc 404 \
  -mr "<script>|onerror|onload"

# 反射型XSS检测
ffuf -u "https://target.com/comment?text=FUZZ" \
  -w xss_payloads.txt \
  -fc 404 \
  -v | grep -i "script\|onerror"
```

---

## 高级技巧

### 1. 使用stdin作为字典

```bash
# 从其他工具输出作为字典
cat wordlist.txt | ffuf -u https://target.com/FUZZ -w - -fc 404

# 生成数字序列
seq 1 100 | ffuf -u https://target.com/id/FUZZ -w - -fc 404

# 组合工具
subfinder -d target.com -silent | ffuf -u https://FUZZ -w - -fc 404
```

### 2. 递归扫描

```bash
# 自动递归
ffuf -u https://target.com/FUZZ \
  -w wordlist.txt \
  -recursion \
  -recursion-depth 2 \
  -fc 404 \
  -e .php,.html

# 递归策略
ffuf -u https://target.com/FUZZ \
  -w wordlist.txt \
  -recursion \
  -recursion-depth 3 \
  -recursion-strategy greedy \
  -fc 404
```

### 3. 重放到代理（Burp Suite）

```bash
# 所有请求通过代理
ffuf -u https://target.com/FUZZ -w wordlist.txt -x http://127.0.0.1:8080

# 只有匹配的请求通过代理
ffuf -u https://target.com/FUZZ \
  -w wordlist.txt \
  -replay-proxy http://127.0.0.1:8080 \
  -fc 404
```

### 4. 速率限制

```bash
# 限制每秒请求数
ffuf -u https://target.com/FUZZ -w wordlist.txt -rate 10 -fc 404

# 添加延迟（秒）
ffuf -u https://target.com/FUZZ -w wordlist.txt -p 0.5 -fc 404

# 限制并发线程
ffuf -u https://target.com/FUZZ -w wordlist.txt -t 10 -fc 404
```

### 5. 字典组合技巧

```bash
# 笛卡尔积（clusterbomb）- 所有组合
ffuf -u https://target.com/FUZZ/FUZ2Z \
  -w dirs.txt:FUZZ \
  -w files.txt:FUZ2Z \
  -mode clusterbomb \
  -fc 404

# Pitchfork - 一一对应
ffuf -u https://target.com/login \
  -X POST \
  -d "username=FUZZ&password=FUZ2Z" \
  -w users.txt:FUZZ \
  -w passwords.txt:FUZ2Z \
  -mode pitchfork \
  -fc 401
```

### 6. 响应分析

```bash
# 输出详细响应
ffuf -u https://target.com/FUZZ -w wordlist.txt -v -fc 404

# 输出响应到文件
ffuf -u https://target.com/FUZZ \
  -w wordlist.txt \
  -o results.json \
  -of json \
  -fc 404

# 彩色输出
ffuf -u https://target.com/FUZZ -w wordlist.txt -c -fc 404
```

### 7. 配置文件

```bash
# 创建配置文件
cat > ffuf.conf << EOF
[general]
threads: 100
timeout: 10
rate: 50

[http]
headers:
  User-Agent: FFUF Scanner
  X-Custom: Test
follow-redirects: true
EOF

# 使用配置文件
ffuf -config ffuf.conf -u https://target.com/FUZZ -w wordlist.txt
```

---

## 字典文件

### 常见字典位置

```bash
# SecLists (推荐)
/opt/SecLists/Discovery/Web-Content/common.txt
/opt/SecLists/Discovery/Web-Content/big.txt
/opt/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt
/opt/SecLists/Discovery/DNS/subdomains-top1million-110000.txt

# Dirb
/usr/share/wordlists/dirb/common.txt
/usr/share/wordlists/dirb/big.txt

# Dirbuster
/usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt

# FuzzDB
https://github.com/fuzzdb-project/fuzzdb
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
# API端点字典
cat > api_endpoints.txt << EOF
api
v1
v2
graphql
swagger
docs
admin
users
login
register
auth
token
EOF

# 备份文件扩展名
cat > backup_extensions.txt << EOF
.bak
.old
.backup
.zip
.tar.gz
.sql
.dump
~
.swp
EOF

# 常见子域名
cat > subdomains.txt << EOF
www
api
dev
staging
test
admin
mail
ftp
vpn
blog
shop
EOF
```

---

## 输出和报告

### 输出格式

```bash
# JSON格式
ffuf -u https://target.com/FUZZ -w wordlist.txt -o results.json -of json -fc 404

# CSV格式
ffuf -u https://target.com/FUZZ -w wordlist.txt -o results.csv -of csv -fc 404

# HTML格式
ffuf -u https://target.com/FUZZ -w wordlist.txt -o results.html -of html -fc 404

# EJSON格式（每行一个JSON对象）
ffuf -u https://target.com/FUZZ -w wordlist.txt -o results.ejson -of ejson -fc 404

# Markdown格式
ffuf -u https://target.com/FUZZ -w wordlist.txt -o results.md -of md -fc 404

# 标准输出（默认）
ffuf -u https://target.com/FUZZ -w wordlist.txt -fc 404
```

### 输出控制

```bash
# 详细模式
ffuf -u https://target.com/FUZZ -w wordlist.txt -v -fc 404

# 静默模式
ffuf -u https://target.com/FUZZ -w wordlist.txt -s -fc 404

# 彩色输出
ffuf -u https://target.com/FUZZ -w wordlist.txt -c -fc 404

# 不显示进度条
ffuf -u https://target.com/FUZZ -w wordlist.txt -noninteractive -fc 404
```

### 结果处理

```bash
# 保存并实时查看
ffuf -u https://target.com/FUZZ -w wordlist.txt -o results.json -of json -fc 404 | tee terminal.log

# 提取特定字段（JSON格式）
ffuf -u https://target.com/FUZZ -w wordlist.txt -o results.json -of json -fc 404
cat results.json | jq '.results[] | {url: .url, status: .status}'

# 转换为其他格式
# JSON -> CSV
cat results.json | jq -r '.results[] | [.url, .status, .length] | @csv' > results.csv
```

---

## 性能优化

### 线程和速率控制

```bash
# 增加线程数（默认40）
ffuf -u https://target.com/FUZZ -w wordlist.txt -t 100 -fc 404

# 限制速率（每秒请求数）
ffuf -u https://target.com/FUZZ -w wordlist.txt -rate 50 -fc 404

# 添加延迟（秒）
ffuf -u https://target.com/FUZZ -w wordlist.txt -p 0.1 -fc 404

# 超时设置
ffuf -u https://target.com/FUZZ -w wordlist.txt -timeout 10 -fc 404
```

### 性能建议

| 场景 | 推荐配置 | 说明 |
|------|---------|------|
| 快速扫描 | `-t 100 -rate 100` | 无WAF保护 |
| 中等速度 | `-t 50 -rate 50` | 一般网站 |
| 慢速扫描 | `-t 10 -rate 10 -p 0.2` | 有WAF或速率限制 |
| 大字典扫描 | `-t 100 -timeout 5` | 提高效率 |

### 内存优化

```bash
# 使用stdin减少内存占用
cat large_wordlist.txt | ffuf -u https://target.com/FUZZ -w - -fc 404

# 避免大量输出到内存
ffuf -u https://target.com/FUZZ -w wordlist.txt -o results.json -of json -fc 404
```

---

## 实战案例

### 案例1：CTF Web目录扫描

```bash
# 第一步：快速扫描常见目录
ffuf -u http://ctf.challenge.com/FUZZ \
  -w /usr/share/wordlists/dirb/common.txt \
  -fc 404 \
  -c

# 第二步：扫描特定扩展名
ffuf -u http://ctf.challenge.com/FUZZ \
  -w wordlist.txt \
  -e .php,.txt,.html,.bak,.old \
  -fc 404

# 第三步：扫描备份文件
ffuf -u http://ctf.challenge.com/index.FUZZ \
  -w backup_extensions.txt \
  -fc 404

# 第四步：参数模糊测试
ffuf -u "http://ctf.challenge.com/page?FUZZ=test" \
  -w params.txt \
  -fc 404 \
  -mr "error|flag|admin"
```

### 案例2：API端点发现

```bash
# 扫描API版本
ffuf -u https://api.target.com/FUZZ/users \
  -w api_versions.txt \
  -fc 404 \
  -H "Authorization: Bearer TOKEN"

# 扫描API端点
ffuf -u https://api.target.com/v1/FUZZ \
  -w /opt/SecLists/Discovery/Web-Content/api/api-endpoints.txt \
  -fc 404 \
  -mc 200,201,401,403

# 测试不同HTTP方法
for method in GET POST PUT DELETE PATCH; do
  echo "Testing $method"
  ffuf -u https://api.target.com/v1/users \
    -X $method \
    -fc 404,405 \
    -w /dev/null
done
```

### 案例3：子域名爆破

```bash
# 基本子域名扫描
ffuf -u https://FUZZ.target.com \
  -w /opt/SecLists/Discovery/DNS/subdomains-top1million-110000.txt \
  -fc 404 \
  -t 100

# 递归子域名（二级）
ffuf -u https://FUZZ.FUZ2Z.target.com \
  -w subdomains1.txt:FUZZ \
  -w subdomains2.txt:FUZ2Z \
  -fc 404

# 保存结果并解析IP
ffuf -u https://FUZZ.target.com \
  -w subdomains.txt \
  -fc 404 \
  -o subdomains.json \
  -of json

# 提取子域名
cat subdomains.json | jq -r '.results[].url' | sort -u > found_subdomains.txt
```

### 案例4：登录爆破

```bash
# 用户名枚举
ffuf -u https://target.com/login \
  -X POST \
  -d "username=FUZZ&password=test123" \
  -w usernames.txt \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -fc 401 \
  -fs 2500  # 过滤标准失败响应大小

# 密码爆破（已知用户名）
ffuf -u https://target.com/login \
  -X POST \
  -d "username=admin&password=FUZZ" \
  -w /usr/share/wordlists/rockyou.txt \
  -fc 401 \
  -mc 200,302

# 用户名+密码组合（慎用）
ffuf -u https://target.com/login \
  -X POST \
  -d "username=FUZZ&password=FUZ2Z" \
  -w users.txt:FUZZ \
  -w passwords.txt:FUZ2Z \
  -mode clusterbomb \
  -fc 401 \
  -t 5 \
  -p 1  # 慢速避免封锁
```

### 案例5：SSRF漏洞测试

```bash
# 创建SSRF payload
cat > ssrf_payloads.txt << EOF
http://localhost
http://127.0.0.1
http://0.0.0.0
http://169.254.169.254
http://[::1]
http://localhost:80
http://localhost:8080
file:///etc/passwd
file:///c:/windows/win.ini
EOF

# GET参数SSRF
ffuf -u "https://target.com/fetch?url=FUZZ" \
  -w ssrf_payloads.txt \
  -fc 404,403 \
  -mr "root:|localhost|metadata"

# POST SSRF
ffuf -u https://target.com/api/fetch \
  -X POST \
  -d '{"url":"FUZZ"}' \
  -w ssrf_payloads.txt \
  -H "Content-Type: application/json" \
  -fc 404
```

### 案例6：参数污染测试

```bash
# HTTP参数污染
ffuf -u "https://target.com/user?id=1&id=FUZZ" \
  -w ids.txt \
  -fc 404

# 测试参数覆盖
ffuf -u "https://target.com/api?role=user&role=FUZZ" \
  -w roles.txt \
  -fc 404 \
  -mr "admin|root"
```

---

## 常见问题

### 1. 如何处理WAF？

```bash
# 降低速率
ffuf -u https://target.com/FUZZ -w wordlist.txt -t 5 -rate 5 -p 1

# 随机User-Agent（需要字典）
ffuf -u https://target.com/FUZZ \
  -w wordlist.txt:FUZZ \
  -w user-agents.txt:UA \
  -H "User-Agent: UA" \
  -mode pitchfork

# 使用代理池（需要脚本）
```

### 2. 如何过滤动态内容？

```bash
# 使用自动校准
ffuf -u https://target.com/FUZZ -w wordlist.txt -ac

# 基于响应大小
ffuf -u https://target.com/FUZZ -w wordlist.txt -fs 4242

# 基于正则表达式
ffuf -u https://target.com/FUZZ -w wordlist.txt -fr "404|not found"
```

### 3. 如何提高扫描速度？

```bash
# 增加线程和速率
ffuf -u https://target.com/FUZZ -w wordlist.txt -t 200 -rate 200

# 减少超时
ffuf -u https://target.com/FUZZ -w wordlist.txt -timeout 3

# 使用更小的字典
ffuf -u https://target.com/FUZZ -w common.txt
```

### 4. 如何处理SSL错误？

```bash
# 跳过SSL验证
ffuf -u https://target.com/FUZZ -w wordlist.txt -k
```

### 5. 如何保存和恢复扫描？

```bash
# 保存状态
ffuf -u https://target.com/FUZZ \
  -w wordlist.txt \
  -o results.json \
  -of json

# FFUF不支持原生恢复，但可以通过字典分段实现
split -l 1000 wordlist.txt chunk_
ffuf -u https://target.com/FUZZ -w chunk_aa
```

---

## 最佳实践总结

### ✅ 推荐做法

1. **总是使用过滤器** - 避免误报和信息过载
2. **保存输出** - 便于后续分析
3. **使用自动校准** - 处理动态内容
4. **适当限速** - 避免被封禁
5. **分阶段扫描** - 先快速后深度
6. **记录过程** - 便于复现和报告

### ❌ 避免错误

1. 不要无限制高速扫描 - 会被WAF拦截
2. 不要忽略响应分析 - 可能错过重要信息
3. 不要只依赖一个字典 - 多字典组合效果更好
4. 不要忽略SSL错误 - 可能是中间人攻击
5. 不要暴力测试生产环境 - 获得授权后再测试

### 🎯 核心技巧

1. **组合使用FUZZ位置** - URL、Header、POST数据
2. **利用管道和重定向** - 与其他工具结合
3. **自定义字典** - 针对目标特点
4. **分析响应特征** - 大小、时间、内容
5. **递归思维** - 发现一个目录后继续深入

---

## 参考资源

- **官方仓库**: https://github.com/ffuf/ffuf
- **SecLists**: https://github.com/danielmiessler/SecLists
- **PayloadsAllTheThings**: https://github.com/swisskyrepo/PayloadsAllTheThings
- **HackTricks**: https://book.hacktricks.xyz/pentesting-web/web-tool-ffuf

---

**最后更新:** 2025-10-04
**FFUF版本:** v2.1.0+
**适用场景:** Web渗透测试、CTF、漏洞赏金、安全审计
