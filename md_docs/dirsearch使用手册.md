# Dirsearch 使用手册

## 目录
- [工具简介](#工具简介)
- [安装方法](#安装方法)
- [基本用法](#基本用法)
- [常用参数](#常用参数)
- [实战示例](#实战示例)
- [字典文件](#字典文件)
- [输出格式](#输出格式)
- [进阶技巧](#进阶技巧)
- [常见问题](#常见问题)

---

## 工具简介

**Dirsearch** 是一款功能强大的Web路径扫描工具，用于发现网站中的隐藏目录和文件。

**主要特性**：
- 多线程扫描，速度快
- 支持多种HTTP方法（GET、POST、HEAD等）
- 递归目录扫描
- 自定义字典和扩展名
- 支持代理和认证
- 多种输出格式（TXT、JSON、XML等）
- 支持通配符排除和包含规则

**适用场景**：
- Web安全测试
- 渗透测试
- CTF竞赛
- 网站结构分析

---

## 安装方法

### 方法1：使用pip安装（推荐）

```bash
# 安装
pip3 install dirsearch

# 验证安装
dirsearch --version
```

### 方法2：从GitHub克隆源码

```bash
# 克隆仓库
git clone https://github.com/maurosoria/dirsearch.git

# 进入目录
cd dirsearch

# 运行
python3 dirsearch.py -u http://example.com
```

### 方法3：使用包管理器

```bash
# Kali Linux / Debian / Ubuntu
sudo apt install dirsearch

# Arch Linux
sudo pacman -S dirsearch

# macOS (Homebrew)
brew install dirsearch
```

---

## 基本用法

### 最简单的扫描

```bash
dirsearch -u http://example.com
```

### 指定扩展名

```bash
dirsearch -u http://example.com -e php,html,js,txt
```

### 使用自定义字典

```bash
dirsearch -u http://example.com -w /path/to/wordlist.txt
```

### 递归扫描

```bash
dirsearch -u http://example.com -r
```

### 保存结果

```bash
dirsearch -u http://example.com -o results.txt
```

---

## 常用参数

### 目标设置

| 参数 | 说明 | 示例 |
|------|------|------|
| `-u URL` | 目标URL | `-u http://example.com` |
| `-l FILE` | URL列表文件 | `-l targets.txt` |
| `--cidr CIDR` | CIDR格式的IP范围 | `--cidr 192.168.1.0/24` |

### 字典和扩展名

| 参数 | 说明 | 示例 |
|------|------|------|
| `-w WORDLIST` | 字典文件路径 | `-w wordlist.txt` |
| `-e EXTENSIONS` | 文件扩展名（逗号分隔） | `-e php,html,js` |
| `-f` | 强制扩展名（不添加/） | `-f -e php` |
| `--remove-extensions` | 移除扩展名 | `--remove-extensions` |

### 扫描控制

| 参数 | 说明 | 示例 |
|------|------|------|
| `-t THREADS` | 线程数（默认30） | `-t 50` |
| `-r` | 递归扫描 | `-r` |
| `-R DEPTH` | 递归深度 | `-R 3` |
| `--subdirs SUBDIRS` | 指定子目录扫描 | `--subdirs admin,backup` |
| `--max-time TIME` | 最大扫描时间（秒） | `--max-time 3600` |

### 过滤选项

| 参数 | 说明 | 示例 |
|------|------|------|
| `--exclude-status CODES` | 排除HTTP状态码 | `--exclude-status 404,403` |
| `--include-status CODES` | 只包含HTTP状态码 | `--include-status 200,301,302` |
| `--exclude-sizes SIZES` | 排除响应大小 | `--exclude-sizes 0B,123KB` |
| `--exclude-texts TEXTS` | 排除包含特定文本的响应 | `--exclude-texts "Not Found"` |
| `--exclude-regex REGEX` | 排除匹配正则的响应 | `--exclude-regex "error.*"` |

### HTTP设置

| 参数 | 说明 | 示例 |
|------|------|------|
| `-m METHOD` | HTTP方法 | `-m POST` |
| `-H HEADERS` | 自定义HTTP头 | `-H "User-Agent: Mozilla/5.0"` |
| `--cookie COOKIE` | Cookie | `--cookie "session=abc123"` |
| `--user-agent UA` | User-Agent | `--user-agent "CustomBot/1.0"` |
| `--timeout TIMEOUT` | 超时时间（秒） | `--timeout 10` |
| `--proxy PROXY` | 代理服务器 | `--proxy http://127.0.0.1:8080` |
| `--auth USER:PASS` | HTTP认证 | `--auth admin:password` |

### 输出选项

| 参数 | 说明 | 示例 |
|------|------|------|
| `-o FILE` | 输出文件 | `-o results.txt` |
| `--format FORMAT` | 输出格式 | `--format json` |
| `--log FILE` | 日志文件 | `--log scan.log` |
| `-q` | 安静模式 | `-q` |

### 性能优化

| 参数 | 说明 | 示例 |
|------|------|------|
| `--random-agent` | 随机User-Agent | `--random-agent` |
| `--delay DELAY` | 请求延迟（秒） | `--delay 0.5` |
| `--max-rate RATE` | 最大请求速率 | `--max-rate 100` |

---

## 实战示例

### 示例1：基础扫描

```bash
# 扫描常见目录和文件
dirsearch -u http://example.com -e php,html,js,txt
```

### 示例2：使用自定义字典

```bash
# 使用DirBuster字典
dirsearch -u http://example.com \
  -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt \
  -e php,html
```

### 示例3：递归扫描并保存结果

```bash
# 递归3层深度，保存为JSON格式
dirsearch -u http://example.com \
  -e php,html,js \
  -r -R 3 \
  -o results.json \
  --format json
```

### 示例4：排除特定状态码

```bash
# 只显示200和301状态码
dirsearch -u http://example.com \
  -e php \
  --include-status 200,301
```

### 示例5：多线程高速扫描

```bash
# 使用100个线程进行快速扫描
dirsearch -u http://example.com \
  -e php,html \
  -t 100 \
  --random-agent
```

### 示例6：通过代理扫描

```bash
# 使用Burp Suite代理
dirsearch -u http://example.com \
  -e php \
  --proxy http://127.0.0.1:8080
```

### 示例7：批量扫描多个目标

```bash
# 创建目标列表文件 targets.txt
# 内容：
# http://example1.com
# http://example2.com
# http://example3.com

dirsearch -l targets.txt -e php,html -o batch_results.txt
```

### 示例8：扫描特定子目录

```bash
# 只扫描/admin和/backup目录
dirsearch -u http://example.com \
  --subdirs admin,backup \
  -e php,html
```

### 示例9：带认证的扫描

```bash
# HTTP基本认证
dirsearch -u http://example.com \
  --auth admin:password123 \
  -e php

# 使用Cookie认证
dirsearch -u http://example.com \
  --cookie "PHPSESSID=abc123; auth_token=xyz789" \
  -e php
```

### 示例10：CTF场景 - 寻找隐藏文件

```bash
# 常见CTF隐藏文件扫描
dirsearch -u http://ctf.example.com \
  -e txt,zip,bak,old,backup,sql,tar.gz \
  -w common.txt \
  --exclude-status 404 \
  -o ctf_scan.txt
```

---

## 字典文件

### 内置字典

Dirsearch自带多个字典文件，位于：
- `/opt/homebrew/lib/python3.11/site-packages/dirsearch/db/` (macOS Homebrew)
- `/usr/share/dirsearch/db/` (Linux)

常用内置字典：
- `dicc.txt` - 默认字典
- `common.txt` - 通用目录和文件
- `big.txt` - 大型字典

### 推荐外部字典

```bash
# SecLists (推荐)
git clone https://github.com/danielmiessler/SecLists.git

# 常用路径：
# SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt
# SecLists/Discovery/Web-Content/common.txt
# SecLists/Discovery/Web-Content/big.txt

# DirBuster字典（Kali Linux）
/usr/share/wordlists/dirbuster/
```

### 自定义字典示例

创建CTF专用字典 `ctf_wordlist.txt`：

```
flag
flag.txt
flag.php
admin
admin.php
login
login.php
upload
upload.php
backup
backup.zip
config
config.php
.git
.svn
robots.txt
sitemap.xml
index.bak
index.old
```

---

## 输出格式

### 支持的输出格式

- `plain` (默认) - 纯文本
- `json` - JSON格式
- `xml` - XML格式
- `md` - Markdown格式
- `csv` - CSV格式
- `html` - HTML报告

### 输出示例

```bash
# 纯文本输出
dirsearch -u http://example.com -o results.txt

# JSON格式
dirsearch -u http://example.com -o results.json --format json

# HTML报告
dirsearch -u http://example.com -o report.html --format html
```

### JSON输出示例

```json
{
  "target": "http://example.com",
  "results": [
    {
      "path": "/admin",
      "status": 200,
      "size": "4KB",
      "redirect": null
    },
    {
      "path": "/backup.zip",
      "status": 200,
      "size": "1.2MB",
      "redirect": null
    }
  ]
}
```

---

## 进阶技巧

### 1. 组合使用多个字典

```bash
# 合并多个字典
cat dict1.txt dict2.txt dict3.txt > combined.txt
dirsearch -u http://example.com -w combined.txt
```

### 2. 使用正则过滤

```bash
# 排除包含"error"的响应
dirsearch -u http://example.com \
  --exclude-regex "error|exception|warning"
```

### 3. 自定义HTTP头

```bash
# 设置多个自定义头
dirsearch -u http://example.com \
  -H "X-Forwarded-For: 127.0.0.1" \
  -H "X-Custom-Header: value"
```

### 4. 速率限制（避免被封禁）

```bash
# 每秒最多50个请求，每个请求延迟0.1秒
dirsearch -u http://example.com \
  --max-rate 50 \
  --delay 0.1
```

### 5. 结合其他工具

```bash
# 先用nmap发现Web服务，再用dirsearch扫描
nmap -p 80,443,8080 192.168.1.0/24 -oG - | \
  grep open | \
  awk '{print $2}' | \
  while read ip; do
    dirsearch -u http://$ip -e php,html -o scan_$ip.txt
  done
```

### 6. 断点续扫

```bash
# 使用会话保存进度
dirsearch -u http://example.com \
  --session-file session.txt
```

---

## 常见问题

### Q1: 扫描速度太慢怎么办？

**解决方案**：
```bash
# 增加线程数
dirsearch -u http://example.com -t 100

# 使用更小的字典
dirsearch -u http://example.com -w common.txt

# 减少扩展名
dirsearch -u http://example.com -e php
```

### Q2: 被目标网站封禁IP怎么办？

**解决方案**：
```bash
# 降低扫描速度
dirsearch -u http://example.com --delay 1 --max-rate 10

# 使用代理
dirsearch -u http://example.com --proxy http://proxy:8080

# 随机User-Agent
dirsearch -u http://example.com --random-agent
```

### Q3: 如何扫描HTTPS网站？

**解决方案**：
```bash
# 直接使用https://
dirsearch -u https://example.com

# 忽略SSL证书错误
dirsearch -u https://example.com --no-verify
```

### Q4: 如何只查看成功的结果？

**解决方案**：
```bash
# 只显示200状态码
dirsearch -u http://example.com --include-status 200

# 排除404和403
dirsearch -u http://example.com --exclude-status 404,403
```

### Q5: 如何扫描需要登录的网站？

**解决方案**：
```bash
# 使用Cookie
dirsearch -u http://example.com \
  --cookie "session_id=abc123; auth=xyz789"

# 使用HTTP认证
dirsearch -u http://example.com \
  --auth username:password
```

---

## CTF实战技巧

### 1. 寻找备份文件

```bash
dirsearch -u http://ctf.example.com \
  -e bak,old,backup,zip,tar,tar.gz,sql,db \
  -w backup_wordlist.txt
```

### 2. 寻找Git/SVN泄露

```bash
dirsearch -u http://ctf.example.com \
  -w git_svn.txt

# git_svn.txt内容：
# .git
# .git/config
# .git/HEAD
# .svn
# .svn/entries
```

### 3. 寻找管理后台

```bash
dirsearch -u http://ctf.example.com \
  -e php,html,jsp \
  --subdirs admin,manage,backend,console
```

### 4. 寻找上传点

```bash
dirsearch -u http://ctf.example.com \
  -w upload_wordlist.txt \
  -e php,jsp,asp,aspx

# upload_wordlist.txt内容：
# upload
# uploads
# uploader
# file
# files
# attachment
```

---

## 安全提示

⚠️ **重要提醒**：

1. **仅用于授权测试**：只能在获得明确授权的系统上使用
2. **遵守法律法规**：未经授权的扫描可能违法
3. **教育用途**：本手册仅供安全教育和CTF竞赛使用
4. **合理使用**：避免对目标系统造成过大负载
5. **数据保护**：妥善保管扫描结果，保护隐私数据

---

## 参考资源

- **官方GitHub**: https://github.com/maurosoria/dirsearch
- **SecLists字典**: https://github.com/danielmiessler/SecLists
- **OWASP测试指南**: https://owasp.org/www-project-web-security-testing-guide/

---

**文档版本**: 1.0
**更新日期**: 2025-01-21
**适用版本**: Dirsearch v0.4.3+

---

## 快速参考

```bash
# 最常用命令
dirsearch -u URL -e php,html,js -t 50 -o results.txt

# CTF场景
dirsearch -u URL -e txt,zip,bak -w common.txt --exclude-status 404

# 渗透测试
dirsearch -u URL -e php,jsp,asp -r -R 2 --random-agent --proxy http://127.0.0.1:8080
```
