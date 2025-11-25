# SecLists 字典分类详解

## 目录
- [简介](#简介)
- [目录结构总览](#目录结构总览)
- [Discovery - 信息发现](#discovery---信息发现)
- [Fuzzing - 模糊测试](#fuzzing---模糊测试)
- [Passwords - 密码字典](#passwords---密码字典)
- [Usernames - 用户名字典](#usernames---用户名字典)
- [Payloads - 攻击载荷](#payloads---攻击载荷)
- [Web-Shells - Web后门](#web-shells---web后门)
- [Miscellaneous - 杂项](#miscellaneous---杂项)
- [AI - 人工智能测试](#ai---人工智能测试)
- [常用字典推荐](#常用字典推荐)
- [使用示例](#使用示例)

---

## 简介

**SecLists** 是安全测试人员的字典集合，由 Daniel Miessler 创建和维护，包含各种类型的字典文件，用于：

- 🔍 信息发现（子域名、目录、文件）
- 🎯 模糊测试（XSS、SQL注入、LFI等）
- 🔑 密码破解
- 👤 用户名枚举
- 💣 攻击载荷测试

**安装位置**: `/Volumes/D/ctf/wordlists/SecLists`

---

## 目录结构总览

```
SecLists/
├── Ai/                      # AI/LLM 测试
├── Discovery/               # 信息发现
│   ├── DNS/                # 子域名枚举
│   ├── Web-Content/        # Web内容发现
│   ├── File-System/        # 文件系统路径
│   ├── Infrastructure/     # 基础设施
│   ├── SNMP/              # SNMP OID
│   └── Variables/         # 变量名
├── Fuzzing/                # 模糊测试
│   ├── XSS/               # XSS Payload
│   ├── SQLi/              # SQL注入
│   ├── LFI/               # 本地文件包含
│   ├── Databases/         # 数据库相关
│   └── ...
├── Passwords/              # 密码字典
│   ├── Common-Credentials/ # 通用凭证
│   ├── Default-Credentials/# 默认凭证
│   ├── Leaked-Databases/  # 泄露数据库
│   └── ...
├── Usernames/              # 用户名字典
├── Payloads/               # 攻击载荷
├── Web-Shells/             # Web后门
├── Miscellaneous/          # 杂项
└── Pattern-Matching/       # 模式匹配
```

---

## Discovery - 信息发现

### 📂 Discovery/DNS - 子域名枚举

| 文件名 | 大小 | 用途 | 推荐场景 |
|--------|------|------|----------|
| `subdomains-top1million-5000.txt` | 5K | 最常见5000个子域名 | ⭐⭐⭐⭐⭐ 快速扫描 |
| `subdomains-top1million-20000.txt` | 20K | 常见20000个子域名 | ⭐⭐⭐⭐ 中等扫描 |
| `subdomains-top1million-110000.txt` | 110K | 大型字典 | ⭐⭐⭐ 深度扫描 |
| `dns-Jhaddix.txt` | ~2M | Jhaddix整理的综合字典 | ⭐⭐⭐⭐⭐ Bug Bounty |
| `combined_subdomains.txt` | ~487K | 合并多个来源 | ⭐⭐⭐⭐ 全面扫描 |
| `deepmagic.com-prefixes-top500.txt` | 500 | 常见前缀 | ⭐⭐⭐ 快速测试 |
| `deepmagic.com-prefixes-top50000.txt` | 50K | 大型前缀字典 | ⭐⭐⭐ 深度测试 |
| `bitquark-subdomains-top100000.txt` | 100K | Bitquark收集 | ⭐⭐⭐⭐ 全面扫描 |
| `fierce-hostlist.txt` | ~2K | Fierce工具字典 | ⭐⭐⭐ 传统扫描 |
| `bug-bounty-program-subdomains-trickest-inventory.txt` | ~8K | Bug Bounty子域名 | ⭐⭐⭐⭐⭐ Bug Bounty |

**使用示例**:
```bash
# DNSRecon
dnsrecon -d target.com -t brt -D /Volumes/D/ctf/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt

# Amass
amass enum -brute -d target.com -w /Volumes/D/ctf/wordlists/SecLists/Discovery/DNS/dns-Jhaddix.txt

# ffuf
ffuf -u https://FUZZ.target.com -w /Volumes/D/ctf/wordlists/SecLists/Discovery/DNS/subdomains-top1million-20000.txt
```

### 📂 Discovery/Web-Content - Web内容发现

| 文件名 | 条目数 | 用途 | 推荐场景 |
|--------|--------|------|----------|
| `common.txt` | 4,613 | 常见目录/文件 | ⭐⭐⭐⭐⭐ 快速扫描 |
| `big.txt` | 20,469 | 大型目录字典 | ⭐⭐⭐⭐ 标准扫描 |
| `raft-large-directories.txt` | 62,284 | 大型目录 | ⭐⭐⭐⭐ 深度扫描 |
| `raft-large-files.txt` | 37,050 | 大型文件 | ⭐⭐⭐⭐ 文件发现 |
| `directory-list-2.3-medium.txt` | 220,560 | DirBuster中等字典 | ⭐⭐⭐⭐⭐ 标准扫描 |
| `directory-list-2.3-big.txt` | 1,273,833 | DirBuster大字典 | ⭐⭐⭐ 完整扫描 |
| `directory-list-lowercase-2.3-medium.txt` | 207,629 | 小写版本 | ⭐⭐⭐⭐ Linux系统 |
| `RobotsDisallowed-Top1000.txt` | 1,000 | robots.txt常见禁止项 | ⭐⭐⭐⭐ 敏感路径 |
| `api/api-endpoints.txt` | ~200 | API端点 | ⭐⭐⭐⭐⭐ API测试 |
| `Apache.fuzz.txt` | 8,724 | Apache特定路径 | ⭐⭐⭐ Apache服务器 |
| `nginx.txt` | 457 | Nginx特定路径 | ⭐⭐⭐ Nginx服务器 |
| `spring-boot.txt` | 372 | Spring Boot路径 | ⭐⭐⭐⭐ Java应用 |
| `graphql.txt` | ~30 | GraphQL端点 | ⭐⭐⭐⭐ GraphQL API |
| `swagger.txt` | ~20 | Swagger文档路径 | ⭐⭐⭐⭐ API文档 |

**使用示例**:
```bash
# Gobuster
gobuster dir -u https://target.com -w /Volumes/D/ctf/wordlists/SecLists/Discovery/Web-Content/common.txt

# ffuf
ffuf -u https://target.com/FUZZ -w /Volumes/D/ctf/wordlists/SecLists/Discovery/Web-Content/raft-large-directories.txt

# dirb
dirb https://target.com /Volumes/D/ctf/wordlists/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt
```

### 📂 Discovery/Infrastructure

| 文件名 | 用途 |
|--------|------|
| `common-ports.txt` | 常见端口列表 |
| `cloud-metadata.txt` | 云元数据端点 |
| `IP-Ranges-AWS.txt` | AWS IP范围 |
| `IP-Ranges-Azure.txt` | Azure IP范围 |

### 📂 Discovery/Variables

| 文件名 | 用途 |
|--------|------|
| `generic-variables.txt` | 通用变量名 |
| `secret-keywords.txt` | 敏感关键词 |

---

## Fuzzing - 模糊测试

### 📂 Fuzzing/XSS

| 文件名 | 条目数 | 用途 | 场景 |
|--------|--------|------|------|
| `XSS-Bypass-Strings-BruteLogic.txt` | ~134 | XSS绕过字符串 | ⭐⭐⭐⭐⭐ WAF绕过 |
| `XSS-Cheat-Sheet-PortSwigger.txt` | ~200 | PortSwigger XSS备忘单 | ⭐⭐⭐⭐⭐ 全面测试 |
| `XSS-RSNAKE.txt` | 681 | RSnake XSS向量 | ⭐⭐⭐⭐ 经典测试 |
| `XSS-BruteLogic.txt` | 528 | BruteLogic收集 | ⭐⭐⭐⭐ 实战测试 |
| `XSS-Somdev.txt` | 1,114 | Somdev收集 | ⭐⭐⭐⭐ 综合测试 |

### 📂 Fuzzing/Databases/SQLi

| 文件名 | 用途 | 数据库类型 |
|--------|------|-----------|
| `Generic-SQLi.txt` | 通用SQL注入 | 全部 |
| `MySQL.fuzzdb.txt` | MySQL注入 | MySQL |
| `MSSQL.fuzzdb.txt` | MSSQL注入 | MS SQL Server |
| `Oracle.fuzzdb.txt` | Oracle注入 | Oracle |
| `Generic-BlindSQLi.fuzzdb.txt` | 盲注测试 | 全部 |
| `sqli.auth.bypass.txt` | 认证绕过 | 全部 |
| `SQLi-Polyglots.txt` | SQL注入多语言payload | 全部 |
| `quick-SQLi.txt` | 快速测试 | 全部 |
| `NoSQL.txt` | NoSQL注入 | MongoDB等 |
| `time-based-sqlinjection.txt` | 时间盲注 | 全部 |

**使用示例**:
```bash
# SQL注入测试
ffuf -u "https://target.com/search?q=FUZZ" \
  -w /Volumes/D/ctf/wordlists/SecLists/Fuzzing/Databases/SQLi/Generic-SQLi.txt \
  -mr "error|sql|syntax"

# XSS测试
ffuf -u "https://target.com/comment?text=FUZZ" \
  -w /Volumes/D/ctf/wordlists/SecLists/Fuzzing/XSS/XSS-Bypass-Strings-BruteLogic.txt \
  -mr "<script>|onerror"
```

### 📂 Fuzzing/LFI

| 文件名 | 用途 |
|--------|------|
| `LFI-gracefulsecurity-linux.txt` | Linux LFI路径 |
| `LFI-gracefulsecurity-windows.txt` | Windows LFI路径 |
| `LFI-Jhaddix.txt` | Jhaddix LFI收集 |
| `LFI-InterestingFiles-linux.txt` | Linux敏感文件 |
| `LFI-InterestingFiles-windows.txt` | Windows敏感文件 |

### 📂 Fuzzing/通用

| 文件名 | 用途 |
|--------|------|
| `special-chars.txt` | 特殊字符 |
| `file-extensions.txt` | 文件扩展名 |
| `http-request-methods.txt` | HTTP方法 |
| `User-Agents/` | User-Agent字符串 |
| `big-list-of-naughty-strings.txt` | 恶意字符串合集 |

---

## Passwords - 密码字典

### 📂 Passwords/Common-Credentials

| 文件名 | 条目数 | 用途 | 推荐场景 |
|--------|--------|------|----------|
| `10k-most-common.txt` | 10,000 | 最常见10K密码 | ⭐⭐⭐⭐⭐ 快速测试 |
| `100k-most-used-passwords-NCSC.txt` | 100,000 | NCSC统计 | ⭐⭐⭐⭐ 标准测试 |
| `500-worst-passwords.txt` | 500 | 最弱密码 | ⭐⭐⭐⭐⭐ 快速检查 |
| `10-million-password-list-top-1000000.txt` | 1,000,000 | 百万级密码 | ⭐⭐⭐ 深度爆破 |
| `2023-200_most_used_passwords.txt` | 200 | 2023年最常用 | ⭐⭐⭐⭐⭐ 最新数据 |

### 📂 Passwords/Default-Credentials

| 文件名 | 用途 |
|--------|------|
| `default-passwords.csv` | 默认密码大全 |
| `ssh-betterdefaultpasslist.txt` | SSH默认密码 |
| `telnet-betterdefaultpasslist.txt` | Telnet默认密码 |
| `ftp-betterdefaultpasslist.txt` | FTP默认密码 |
| `postgres-betterdefaultpasslist.txt` | PostgreSQL默认密码 |
| `mysql-betterdefaultpasslist.txt` | MySQL默认密码 |
| `mssql-betterdefaultpasslist.txt` | MSSQL默认密码 |
| `tomcat-betterdefaultpasslist.txt` | Tomcat默认凭证 |

### 📂 Passwords/Leaked-Databases

| 文件名 | 来源 | 用途 |
|--------|------|------|
| `rockyou.txt` | RockYou泄露 | 最著名的密码字典 |
| `Ashley-Madison.txt` | Ashley Madison | 真实泄露数据 |
| `alleged-gmail-passwords.txt` | Gmail泄露 | Gmail密码 |

### 📂 Passwords/其他

| 目录 | 用途 |
|------|------|
| `Keyboard-Walks/` | 键盘走位模式 |
| `Software/` | 软件默认密码 |
| `WiFi-WPA/` | WiFi密码字典 |
| `Permutations/` | 密码变体 |
| `Books/` | 书籍密码（如哈利波特） |

**使用示例**:
```bash
# Hydra密码爆破
hydra -l admin -P /Volumes/D/ctf/wordlists/SecLists/Passwords/Common-Credentials/10k-most-common.txt ssh://target.com

# 默认凭证测试
hydra -C /Volumes/D/ctf/wordlists/SecLists/Passwords/Default-Credentials/ssh-betterdefaultpasslist.txt ssh://target.com
```

---

## Usernames - 用户名字典

| 文件名 | 条目数 | 用途 | 场景 |
|--------|--------|------|------|
| `top-usernames-shortlist.txt` | 17 | 最常见用户名 | ⭐⭐⭐⭐⭐ 快速测试 |
| `xato-net-10-million-usernames.txt` | 8,295,455 | 大型用户名库 | ⭐⭐⭐ 深度枚举 |
| `Names/names.txt` | 10,177 | 常见姓名 | ⭐⭐⭐⭐ 用户枚举 |
| `cirt-default-usernames.txt` | 828 | 默认用户名 | ⭐⭐⭐⭐ 默认账户 |
| `CommonAdminBase64.txt` | 60 | Base64编码管理员 | ⭐⭐⭐ 特殊场景 |

**使用示例**:
```bash
# 用户名枚举
ffuf -u "https://target.com/login" \
  -X POST \
  -d "username=FUZZ&password=test" \
  -w /Volumes/D/ctf/wordlists/SecLists/Usernames/top-usernames-shortlist.txt \
  -fs 2500
```

---

## Payloads - 攻击载荷

### 📂 Payloads/File-Names

| 文件名 | 用途 |
|--------|------|
| `file-extensions.txt` | 危险扩展名 |
| `sensitive-filenames.txt` | 敏感文件名 |

### 📂 Payloads/其他

| 目录 | 用途 |
|------|------|
| `Anti-Virus/` | 反病毒测试文件 |
| `Images/` | 恶意图片样本 |
| `Zip-Bombs/` | Zip炸弹 |
| `Flash/` | Flash漏洞利用 |

---

## Web-Shells - Web后门

| 目录 | 语言 | 用途 |
|------|------|------|
| `PHP/` | PHP | PHP后门 |
| `JSP/` | JSP | Java后门 |
| `ASP/` | ASP | ASP后门 |
| `CFM/` | ColdFusion | CF后门 |
| `WordPress/` | PHP | WordPress后门 |

---

## Miscellaneous - 杂项

| 目录 | 内容 |
|------|------|
| `Web/` | 各种Web相关字典 |
| `Words/` | 英文单词列表 |
| `Security-Question-Answers/` | 安全问题答案 |
| `Danish-Wordlists-n0kovo/` | 丹麦语字典 |

---

## AI - 人工智能测试

### 📂 Ai/LLM_Testing

| 文件名 | 用途 |
|--------|------|
| `LLM-Injection/` | LLM注入测试 |
| `prompt-injection.txt` | 提示词注入 |
| `jailbreak-prompts.txt` | 越狱提示词 |

---

## 常用字典推荐

### 🌟 必备字典（Top 10）

| 序号 | 字典路径 | 用途 | 优先级 |
|------|---------|------|--------|
| 1 | `Discovery/DNS/subdomains-top1million-5000.txt` | 子域名快速扫描 | ⭐⭐⭐⭐⭐ |
| 2 | `Discovery/Web-Content/common.txt` | 目录文件快速发现 | ⭐⭐⭐⭐⭐ |
| 3 | `Discovery/Web-Content/directory-list-2.3-medium.txt` | 标准目录扫描 | ⭐⭐⭐⭐⭐ |
| 4 | `Passwords/Common-Credentials/10k-most-common.txt` | 密码爆破 | ⭐⭐⭐⭐⭐ |
| 5 | `Usernames/top-usernames-shortlist.txt` | 用户名测试 | ⭐⭐⭐⭐⭐ |
| 6 | `Fuzzing/XSS/XSS-Bypass-Strings-BruteLogic.txt` | XSS测试 | ⭐⭐⭐⭐⭐ |
| 7 | `Fuzzing/Databases/SQLi/Generic-SQLi.txt` | SQL注入 | ⭐⭐⭐⭐⭐ |
| 8 | `Discovery/DNS/dns-Jhaddix.txt` | 综合子域名 | ⭐⭐⭐⭐⭐ |
| 9 | `Passwords/Default-Credentials/default-passwords.csv` | 默认凭证 | ⭐⭐⭐⭐⭐ |
| 10 | `Discovery/Web-Content/api/api-endpoints.txt` | API端点 | ⭐⭐⭐⭐⭐ |

### 📊 按场景分类

#### Bug Bounty必备
```bash
# 子域名枚举
Discovery/DNS/dns-Jhaddix.txt
Discovery/DNS/bug-bounty-program-subdomains-trickest-inventory.txt

# Web内容发现
Discovery/Web-Content/raft-large-directories.txt
Discovery/Web-Content/api/api-endpoints.txt

# 模糊测试
Fuzzing/XSS/XSS-Bypass-Strings-BruteLogic.txt
Fuzzing/Databases/SQLi/Generic-SQLi.txt
```

#### CTF比赛必备
```bash
# 快速扫描
Discovery/Web-Content/common.txt
Discovery/DNS/subdomains-top1million-5000.txt

# 常见漏洞
Fuzzing/LFI/LFI-Jhaddix.txt
Fuzzing/Databases/SQLi/quick-SQLi.txt
```

#### 渗透测试必备
```bash
# 信息收集
Discovery/Web-Content/directory-list-2.3-medium.txt
Discovery/DNS/subdomains-top1million-20000.txt

# 凭证爆破
Passwords/Common-Credentials/10k-most-common.txt
Passwords/Default-Credentials/default-passwords.csv
Usernames/xato-net-10-million-usernames.txt
```

---

## 使用示例

### 场景1：Web应用快速扫描

```bash
#!/bin/bash
TARGET="https://target.com"
SECLISTS="/Volumes/D/ctf/wordlists/SecLists"

# 1. 目录扫描
gobuster dir -u $TARGET \
  -w $SECLISTS/Discovery/Web-Content/common.txt \
  -x php,html,txt \
  -o dirs.txt

# 2. API端点发现
ffuf -u $TARGET/FUZZ \
  -w $SECLISTS/Discovery/Web-Content/api/api-endpoints.txt \
  -mc 200,401,403

# 3. 备份文件查找
ffuf -u $TARGET/FUZZ.FUZ2Z \
  -w $SECLISTS/Discovery/Web-Content/raft-large-files.txt:FUZZ \
  -w $SECLISTS/Discovery/Web-Content/web-extensions.txt:FUZ2Z
```

### 场景2：子域名完整枚举

```bash
#!/bin/bash
DOMAIN="target.com"
SECLISTS="/Volumes/D/ctf/wordlists/SecLists"

# 1. 快速扫描
amass enum -passive -d $DOMAIN -o passive.txt

# 2. 暴力破解（小字典）
amass enum -brute -d $DOMAIN \
  -w $SECLISTS/Discovery/DNS/subdomains-top1million-5000.txt \
  -o brute_small.txt

# 3. 深度扫描（大字典）
amass enum -brute -d $DOMAIN \
  -w $SECLISTS/Discovery/DNS/dns-Jhaddix.txt \
  -o brute_large.txt

# 4. 合并去重
cat passive.txt brute_small.txt brute_large.txt | sort -u > all_subs.txt
```

### 场景3：登录爆破

```bash
#!/bin/bash
TARGET="https://target.com/login"
SECLISTS="/Volumes/D/ctf/wordlists/SecLists"

# 1. 用户名枚举
ffuf -u $TARGET \
  -X POST \
  -d "username=FUZZ&password=test123" \
  -w $SECLISTS/Usernames/top-usernames-shortlist.txt \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -fs 2500

# 2. 默认凭证测试
hydra -C $SECLISTS/Passwords/Default-Credentials/ssh-betterdefaultpasslist.txt \
  target.com http-post-form "/login:username=^USER^&password=^PASS^:F=incorrect"

# 3. 常见密码爆破
hydra -l admin \
  -P $SECLISTS/Passwords/Common-Credentials/10k-most-common.txt \
  target.com http-post-form "/login:username=^USER^&password=^PASS^:F=incorrect"
```

### 场景4：漏洞模糊测试

```bash
#!/bin/bash
TARGET="https://target.com"
SECLISTS="/Volumes/D/ctf/wordlists/SecLists"

# 1. XSS测试
ffuf -u "$TARGET/search?q=FUZZ" \
  -w $SECLISTS/Fuzzing/XSS/XSS-Bypass-Strings-BruteLogic.txt \
  -mr "<script>|onerror|onload"

# 2. SQL注入测试
ffuf -u "$TARGET/product?id=FUZZ" \
  -w $SECLISTS/Fuzzing/Databases/SQLi/Generic-SQLi.txt \
  -mr "error|sql|syntax|mysql"

# 3. LFI测试
ffuf -u "$TARGET/file?path=FUZZ" \
  -w $SECLISTS/Fuzzing/LFI/LFI-Jhaddix.txt \
  -mr "root:|admin:|password"

# 4. 命令注入测试
ffuf -u "$TARGET/ping?host=FUZZ" \
  -w $SECLISTS/Fuzzing/command-injection-commix.txt \
  -mr "uid=|root|www-data"
```

### 场景5：API安全测试

```bash
#!/bin/bash
API_URL="https://api.target.com"
SECLISTS="/Volumes/D/ctf/wordlists/SecLists"

# 1. API端点发现
ffuf -u $API_URL/FUZZ \
  -w $SECLISTS/Discovery/Web-Content/api/api-endpoints.txt \
  -mc 200,201,401,403,405

# 2. API版本枚举
ffuf -u $API_URL/FUZZ/users \
  -w $SECLISTS/Discovery/Web-Content/api/api-versions.txt \
  -mc 200,201

# 3. 参数模糊测试
ffuf -u "$API_URL/v1/user?FUZZ=test" \
  -w $SECLISTS/Discovery/Web-Content/burp-parameter-names.txt \
  -mc 200,400,500
```

---

## 快速查找命令

```bash
# 查找特定类型的字典
find /Volumes/D/ctf/wordlists/SecLists -name "*subdomain*"
find /Volumes/D/ctf/wordlists/SecLists -name "*password*"
find /Volumes/D/ctf/wordlists/SecLists -name "*xss*"

# 查看字典大小
wc -l /Volumes/D/ctf/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt

# 搜索包含特定内容的字典
grep -r "admin" /Volumes/D/ctf/wordlists/SecLists/Discovery/Web-Content/ | head

# 列出某个目录下所有字典
ls -lh /Volumes/D/ctf/wordlists/SecLists/Passwords/Common-Credentials/
```

---

## 字典组合技巧

### 合并多个字典
```bash
# 合并并去重
cat wordlist1.txt wordlist2.txt | sort -u > combined.txt

# 合并子域名字典
cat /Volumes/D/ctf/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt \
    /Volumes/D/ctf/wordlists/SecLists/Discovery/DNS/deepmagic.com-prefixes-top500.txt \
    | sort -u > my_subdomain_list.txt
```

### 创建自定义字典
```bash
# 提取特定长度的密码
awk 'length($0) >= 8 && length($0) <= 12' passwords.txt > 8-12char.txt

# 转换大小写
tr '[:upper:]' '[:lower:]' < wordlist.txt > lowercase.txt

# 添加前缀后缀
sed 's/^/admin-/' wordlist.txt > prefixed.txt
sed 's/$/-2024/' wordlist.txt > suffixed.txt
```

---

## 字典更新

```bash
# 更新SecLists到最新版本
cd /Volumes/D/ctf/wordlists/SecLists
git pull

# 查看更新内容
git log --oneline -10
```

---

## 参考资源

- **官方仓库**: https://github.com/danielmiessler/SecLists
- **作者博客**: https://danielmiessler.com
- **贡献指南**: https://github.com/danielmiessler/SecLists/blob/master/CONTRIBUTING.md

---

**最后更新**: 2025-10-04
**SecLists路径**: `/Volumes/D/ctf/wordlists/SecLists`
**版本**: 持续更新中
