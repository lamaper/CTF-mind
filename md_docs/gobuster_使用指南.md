# Gobuster 使用文档

## 简介

Gobuster 是一个用 Go 语言编写的高性能目录/文件暴力破解工具，常用于 Web 安全测试和渗透测试中。它支持多种扫描模式，包括目录扫描、DNS 子域名枚举和虚拟主机发现。

## 安装

### macOS 安装

**使用 Homebrew（推荐）：**
```bash
brew install gobuster
```

**使用 Go：**
```bash
go install github.com/OJ/gobuster/v3@latest
```

**验证安装：**
```bash
gobuster version
```

### Linux 安装

**Debian/Ubuntu：**
```bash
sudo apt update
sudo apt install gobuster
```

**使用 Go：**
```bash
go install github.com/OJ/gobuster/v3@latest
```

## 基本用法

Gobuster 有四种主要模式：

- `dir` - 目录/文件暴力破解模式
- `dns` - DNS 子域名枚举模式
- `vhost` - 虚拟主机发现模式
- `s3` - AWS S3 存储桶枚举模式

## 1. 目录/文件扫描模式 (dir)

### 基础命令

```bash
gobuster dir -u http://target.com -w /path/to/wordlist.txt
```

### 常用参数

| 参数 | 说明 | 示例 |
|------|------|------|
| `-u, --url` | 目标 URL（必需） | `-u http://example.com` |
| `-w, --wordlist` | 字典文件路径（必需） | `-w wordlist.txt` |
| `-t, --threads` | 线程数（默认 10） | `-t 50` |
| `-x, --extensions` | 文件扩展名 | `-x php,html,txt` |
| `-o, --output` | 输出结果到文件 | `-o results.txt` |
| `-s, --status-codes` | 指定状态码 | `-s 200,204,301,302,307,401,403` |
| `-b, --status-codes-blacklist` | 排除的状态码 | `-b 404,403` |
| `-k, --no-tls-validation` | 跳过 SSL 证书验证 | `-k` |
| `-a, --useragent` | 自定义 User-Agent | `-a "Mozilla/5.0"` |
| `-c, --cookies` | 添加 Cookie | `-c "session=abc123"` |
| `-H, --headers` | 自定义 HTTP 头 | `-H "Authorization: Bearer token"` |
| `-r, --follow-redirect` | 跟随重定向 | `-r` |
| `-e, --expanded` | 显示完整 URL | `-e` |
| `-n, --no-status` | 不显示状态码 | `-n` |
| `-q, --quiet` | 安静模式 | `-q` |
| `-z, --no-progress` | 不显示进度 | `-z` |
| `-d, --debug` | 调试模式 | `-d` |
| `--timeout` | HTTP 超时时间 | `--timeout 10s` |
| `--delay` | 请求延迟 | `--delay 1s` |
| `-p, --proxy` | 代理设置 | `-p http://127.0.0.1:8080` |

### 实用示例

**基础目录扫描：**
```bash
gobuster dir -u http://example.com -w /usr/share/wordlists/dirb/common.txt
```

**扫描特定文件扩展名：**
```bash
gobuster dir -u http://example.com -w wordlist.txt -x php,html,js,txt
```

**增加线程数加速扫描：**
```bash
gobuster dir -u http://example.com -w wordlist.txt -t 50
```

**显示完整 URL：**
```bash
gobuster dir -u http://example.com -w wordlist.txt -e
```

**保存结果到文件：**
```bash
gobuster dir -u http://example.com -w wordlist.txt -o output.txt
```

**跳过 SSL 验证：**
```bash
gobuster dir -u https://example.com -w wordlist.txt -k
```

**使用代理：**
```bash
gobuster dir -u http://example.com -w wordlist.txt -p http://127.0.0.1:8080
```

**添加自定义请求头：**
```bash
gobuster dir -u http://example.com -w wordlist.txt -H "Authorization: Bearer token123"
```

**扫描特定状态码：**
```bash
gobuster dir -u http://example.com -w wordlist.txt -s 200,204,301,302,307
```

**排除特定状态码：**
```bash
gobuster dir -u http://example.com -w wordlist.txt -b 404,403
```

**跟随重定向：**
```bash
gobuster dir -u http://example.com -w wordlist.txt -r
```

**添加 Cookie：**
```bash
gobuster dir -u http://example.com -w wordlist.txt -c "PHPSESSID=abc123; token=xyz"
```

**设置延迟（防止被封）：**
```bash
gobuster dir -u http://example.com -w wordlist.txt --delay 500ms
```

## 2. DNS 子域名枚举模式 (dns)

### 基础命令

```bash
gobuster dns -d target.com -w /path/to/wordlist.txt
```

### 常用参数

| 参数 | 说明 | 示例 |
|------|------|------|
| `-d, --domain` | 目标域名（必需） | `-d example.com` |
| `-w, --wordlist` | 字典文件路径（必需） | `-w subdomains.txt` |
| `-t, --threads` | 线程数 | `-t 50` |
| `-o, --output` | 输出结果到文件 | `-o dns-results.txt` |
| `-r, --resolver` | 自定义 DNS 服务器 | `-r 8.8.8.8` |
| `-c, --show-cname` | 显示 CNAME 记录 | `-c` |
| `-i, --show-ips` | 显示 IP 地址 | `-i` |
| `--timeout` | 超时时间 | `--timeout 5s` |
| `--wildcard` | 强制继续（即使存在通配符） | `--wildcard` |

### 实用示例

**基础子域名扫描：**
```bash
gobuster dns -d example.com -w subdomains.txt
```

**显示 IP 地址：**
```bash
gobuster dns -d example.com -w subdomains.txt -i
```

**显示 CNAME 记录：**
```bash
gobuster dns -d example.com -w subdomains.txt -c
```

**使用自定义 DNS 服务器：**
```bash
gobuster dns -d example.com -w subdomains.txt -r 8.8.8.8
```

**增加线程数：**
```bash
gobuster dns -d example.com -w subdomains.txt -t 100
```

**保存结果：**
```bash
gobuster dns -d example.com -w subdomains.txt -o subdomains-found.txt
```

**绕过通配符检测：**
```bash
gobuster dns -d example.com -w subdomains.txt --wildcard
```

## 3. 虚拟主机发现模式 (vhost)

### 基础命令

```bash
gobuster vhost -u http://target.com -w /path/to/wordlist.txt
```

### 常用参数

| 参数 | 说明 | 示例 |
|------|------|------|
| `-u, --url` | 目标 URL（必需） | `-u http://example.com` |
| `-w, --wordlist` | 字典文件路径（必需） | `-w vhosts.txt` |
| `-t, --threads` | 线程数 | `-t 50` |
| `-o, --output` | 输出结果到文件 | `-o vhosts.txt` |
| `-k, --no-tls-validation` | 跳过 SSL 验证 | `-k` |
| `-H, --headers` | 自定义 HTTP 头 | `-H "Authorization: Bearer token"` |
| `--domain` | 附加域名 | `--domain example.com` |

### 实用示例

**基础虚拟主机扫描：**
```bash
gobuster vhost -u http://example.com -w vhosts.txt
```

**使用特定域名：**
```bash
gobuster vhost -u http://192.168.1.100 -w vhosts.txt --domain example.com
```

**跳过 SSL 验证：**
```bash
gobuster vhost -u https://example.com -w vhosts.txt -k
```

## 4. S3 存储桶枚举模式 (s3)

### 基础命令

```bash
gobuster s3 -w /path/to/wordlist.txt
```

### 常用参数

| 参数 | 说明 | 示例 |
|------|------|------|
| `-w, --wordlist` | 字典文件路径（必需） | `-w buckets.txt` |
| `-t, --threads` | 线程数 | `-t 50` |
| `-o, --output` | 输出结果到文件 | `-o s3-results.txt` |
| `--max-files` | 列出的最大文件数 | `--max-files 100` |

### 实用示例

**基础 S3 扫描：**
```bash
gobuster s3 -w bucket-names.txt
```

**保存结果：**
```bash
gobuster s3 -w bucket-names.txt -o s3-found.txt
```

## 常用字典文件

### macOS/Linux 系统自带

**SecLists（推荐安装）：**
```bash
# 安装 SecLists
git clone https://github.com/danielmiessler/SecLists.git

# 目录扫描字典
SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt
SecLists/Discovery/Web-Content/common.txt
SecLists/Discovery/Web-Content/big.txt

# 子域名字典
SecLists/Discovery/DNS/subdomains-top1million-5000.txt
SecLists/Discovery/DNS/subdomains-top1million-20000.txt
```

**Kali Linux 自带：**
```bash
/usr/share/wordlists/dirb/common.txt
/usr/share/wordlists/dirb/big.txt
/usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

**macOS Homebrew 安装 SecLists：**
```bash
brew install seclists
```

## 实战场景示例

### 场景 1：Web 应用目录扫描

```bash
# 基础扫描
gobuster dir -u http://target.com -w common.txt

# 深度扫描（多扩展名）
gobuster dir -u http://target.com -w medium.txt -x php,html,js,txt,bak

# 使用代理（Burp Suite）
gobuster dir -u http://target.com -w wordlist.txt -p http://127.0.0.1:8080

# 认证后扫描
gobuster dir -u http://target.com -w wordlist.txt -c "session=abc123"
```

### 场景 2：子域名枚举

```bash
# 标准子域名扫描
gobuster dns -d target.com -w subdomains-top1million-5000.txt -i

# 使用多个 DNS 服务器
gobuster dns -d target.com -w subdomains.txt -r 8.8.8.8 -r 1.1.1.1

# 显示详细信息
gobuster dns -d target.com -w subdomains.txt -i -c
```

### 场景 3：API 端点发现

```bash
# 扫描 API 端点
gobuster dir -u http://api.target.com -w api-endpoints.txt -e

# JSON API 扫描
gobuster dir -u http://api.target.com/v1 -w wordlist.txt -x json
```

### 场景 4：虚拟主机发现

```bash
# 基于 IP 的虚拟主机扫描
gobuster vhost -u http://192.168.1.100 -w vhosts.txt --domain target.com

# HTTPS 虚拟主机扫描
gobuster vhost -u https://target.com -w vhosts.txt -k
```

## 性能优化建议

### 线程数调整

```bash
# 低速网络或目标服务器性能较差
gobuster dir -u http://target.com -w wordlist.txt -t 10

# 高速网络和稳定服务器
gobuster dir -u http://target.com -w wordlist.txt -t 50

# 本地测试或性能极好的环境
gobuster dir -u http://target.com -w wordlist.txt -t 100
```

### 添加延迟防止被封

```bash
# 添加延迟
gobuster dir -u http://target.com -w wordlist.txt --delay 100ms

# 减少线程并添加延迟
gobuster dir -u http://target.com -w wordlist.txt -t 5 --delay 500ms
```

### 超时设置

```bash
# 设置较长超时（针对慢速服务器）
gobuster dir -u http://target.com -w wordlist.txt --timeout 30s

# 设置较短超时（快速扫描）
gobuster dir -u http://target.com -w wordlist.txt --timeout 3s
```

## 结果分析

### 状态码含义

| 状态码 | 含义 |
|--------|------|
| 200 | 成功访问 |
| 301 | 永久重定向 |
| 302 | 临时重定向 |
| 401 | 需要认证 |
| 403 | 禁止访问（但路径存在） |
| 404 | 未找到 |
| 500 | 服务器错误 |

### 重点关注

- **200** - 直接访问的资源
- **301/302** - 可能存在的目录或重定向
- **401/403** - 存在但需要权限的资源（重点测试）
- **500** - 可能存在漏洞的端点

## 常见问题

### 1. 扫描速度慢

**解决方案：**
- 增加线程数：`-t 50`
- 使用更小的字典
- 检查网络连接

### 2. 被目标服务器封禁

**解决方案：**
- 减少线程数：`-t 5`
- 添加延迟：`--delay 500ms`
- 更换 User-Agent：`-a "Mozilla/5.0"`
- 使用代理：`-p http://proxy:port`

### 3. SSL 证书错误

**解决方案：**
```bash
gobuster dir -u https://target.com -w wordlist.txt -k
```

### 4. 通配符域名干扰

**解决方案：**
```bash
gobuster dns -d target.com -w wordlist.txt --wildcard
```

## 安全与合规

### 重要提醒

⚠️ **警告：仅在授权情况下使用**

- 只对您拥有或获得明确授权的目标进行扫描
- 未经授权的扫描可能违反法律
- 遵守道德黑客准则和渗透测试规范
- 在生产环境使用前先在测试环境验证

### 合法使用场景

✅ 自己的网站或应用
✅ 获得书面授权的渗透测试项目
✅ Bug Bounty 项目（遵守规则）
✅ 安全研究和学习（使用合法靶场）

## 高级技巧

### 1. 递归扫描

```bash
# 发现目录后手动递归
gobuster dir -u http://target.com -w wordlist.txt -o results.txt
# 然后对发现的目录再次扫描
gobuster dir -u http://target.com/admin -w wordlist.txt
```

### 2. 组合使用

```bash
# 先做子域名枚举
gobuster dns -d target.com -w subdomains.txt -o subdomains-found.txt

# 对发现的子域名做目录扫描
for subdomain in $(cat subdomains-found.txt | awk '{print $1}'); do
    gobuster dir -u http://$subdomain -w wordlist.txt -o scan-$subdomain.txt
done
```

### 3. 使用正则表达式过滤

```bash
# 结合 grep 过滤结果
gobuster dir -u http://target.com -w wordlist.txt | grep -E "Status: (200|301|302)"
```

### 4. 自动化脚本

```bash
#!/bin/bash
TARGET="http://example.com"
WORDLIST="wordlist.txt"
OUTPUT_DIR="results"

mkdir -p $OUTPUT_DIR

# 目录扫描
gobuster dir -u $TARGET -w $WORDLIST -o $OUTPUT_DIR/dir-scan.txt

# 常见扩展名扫描
gobuster dir -u $TARGET -w $WORDLIST -x php,html,js,txt -o $OUTPUT_DIR/files-scan.txt

echo "扫描完成，结果保存在 $OUTPUT_DIR 目录"
```

## 总结

Gobuster 是一个功能强大且高效的目录和子域名暴力破解工具。合理使用各种参数和技巧，可以大大提高渗透测试的效率。

### 核心要点

- ✅ 选择合适的字典文件
- ✅ 根据目标调整线程数和延迟
- ✅ 使用适当的文件扩展名
- ✅ 关注 403 和 401 状态码
- ✅ 结合其他工具进行深度测试
- ⚠️ 始终在授权范围内使用

## 参考资源

- [Gobuster GitHub 仓库](https://github.com/OJ/gobuster)
- [SecLists 字典集合](https://github.com/danielmiessler/SecLists)
- [OWASP 测试指南](https://owasp.org/www-project-web-security-testing-guide/)

---

**文档版本：** 1.0  
**最后更新：** 2025年9月

**免责声明：** 本文档仅供学习和合法安全测试使用。使用者需自行承担所有法律责任。