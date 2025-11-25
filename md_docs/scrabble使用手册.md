# Scrabble 使用手册（快速Git泄露扫描器）

> Scrabble 是 BishopFox 开发的高速 .git 泄露扫描工具，使用 Go 语言编写，速度极快。

---

## 目录
1. [概述](#概述)
2. [安装方法](#安装方法)
3. [基本使用](#基本使用)
4. [高级功能](#高级功能)
5. [CTF应用场景](#ctf应用场景)
6. [实战案例](#实战案例)
7. [常见问题](#常见问题)
8. [参考资源](#参考资源)

---

## 概述

### 工具特点

Scrabble 的主要优势:
- ✅ **极速扫描**: Go 语言并发,比 Python 工具快10倍+
- ✅ **智能检测**: 自动检测常见路径变体
- ✅ **简单易用**: 单个二进制文件,无需依赖
- ✅ **跨平台**: 支持 Linux、macOS、Windows
- ✅ **现代工具**: 活跃维护,定期更新

### 与其他工具的区别

| 特性 | Scrabble | GitHacker | git-dumper |
|------|----------|-----------|------------|
| **主要功能** | 扫描检测 | 下载恢复 | 下载恢复 |
| **速度** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ |
| **语言** | Go | Python | Python |
| **依赖** | 无 | 多个 | 多个 |
| **安装** | 单文件 | pip | pip |

### 适用场景

- 🎯 快速批量扫描大量目标
- 🎯 验证 .git 目录是否可访问
- 🎯 路径模糊测试
- 🎯 CI/CD 自动化检测
- 🎯 红队侦察阶段

### 工作原理

```
1. 尝试访问常见的 .git 路径
   - /.git/
   - /.git/config
   - /.git/HEAD
   - /.git/index

2. 检测路径变体
   - /test/.git/
   - /backup/.git/
   - /old/.git/

3. 并发请求
   - 使用 Go 协程
   - 高效并发处理

4. 报告结果
   - 返回可访问的路径
   - 显示 HTTP 状态码
```

---

## 安装方法

### 方法1: Go Install（推荐）

```bash
# 需要 Go 1.16+
go install github.com/BishopFox/scrabble@latest

# 验证安装
scrabble --version

# 查看帮助
scrabble --help
```

### 方法2: 下载预编译二进制

```bash
# 访问 Releases 页面下载
# https://github.com/BishopFox/scrabble/releases

# Linux
wget https://github.com/BishopFox/scrabble/releases/download/v1.0/scrabble-linux-amd64
chmod +x scrabble-linux-amd64
sudo mv scrabble-linux-amd64 /usr/local/bin/scrabble

# macOS
wget https://github.com/BishopFox/scrabble/releases/download/v1.0/scrabble-darwin-amd64
chmod +x scrabble-darwin-amd64
sudo mv scrabble-darwin-amd64 /usr/local/bin/scrabble

# Windows
# 下载 scrabble-windows-amd64.exe
```

### 方法3: 源码编译

```bash
# 克隆仓库
git clone https://github.com/BishopFox/scrabble.git
cd scrabble

# 编译
go build -o scrabble

# 安装
sudo mv scrabble /usr/local/bin/

# 验证
scrabble --version
```

### 系统要求

```
Go 1.16+ (仅编译时需要)
网络连接
```

---

## 基本使用

### 最简单的用法

```bash
# 扫描单个目标
scrabble http://example.com

# 扫描指定 .git 路径
scrabble http://example.com/.git/

# 扫描 HTTPS
scrabble https://secure.example.com
```

### 完整参数说明

```
位置参数:
  URL                   目标 URL

可选参数:
  -c, --concurrency N   并发数（默认:10）
  -t, --timeout N       超时时间（秒，默认:10）
  -o, --output FILE     输出文件
  -v, --verbose         详细输出
  -q, --quiet           静默模式
  -h, --help            显示帮助
  -V, --version         显示版本
```

### 基本示例

```bash
# 基本扫描
scrabble http://target.com

# 增加并发
scrabble http://target.com --concurrency 20

# 设置超时
scrabble http://target.com --timeout 5

# 详细输出
scrabble http://target.com --verbose

# 保存结果
scrabble http://target.com --output results.txt
```

---

## 高级功能

### 1. 并发控制

```bash
# 默认并发（10）
scrabble http://target.com

# 低并发（更隐蔽）
scrabble http://target.com -c 5

# 高并发（更快）
scrabble http://target.com -c 50

# 极限并发（大量目标时）
scrabble http://target.com -c 100
```

### 2. 超时设置

```bash
# 默认超时（10秒）
scrabble http://target.com

# 短超时（快速扫描）
scrabble http://target.com -t 3

# 长超时（慢速网络）
scrabble http://target.com -t 30
```

### 3. 输出控制

```bash
# 详细输出（显示所有请求）
scrabble http://target.com --verbose

# 静默模式（只显示发现的泄露）
scrabble http://target.com --quiet

# 保存到文件
scrabble http://target.com -o findings.txt
```

### 4. 批量扫描

```bash
# 方法1: Shell 循环
for domain in $(cat domains.txt); do
    echo "[*] Scanning: $domain"
    scrabble "http://$domain" -q
done

# 方法2: xargs 并行
cat domains.txt | xargs -I {} -P 10 scrabble "http://{}" -q

# 方法3: GNU Parallel
parallel -j 10 scrabble "http://{}" -q :::: domains.txt
```

---

## CTF应用场景

### 场景1: 快速验证

```bash
# CTF 题目给出 URL
scrabble http://challenge.com

# 如果发现泄露，使用其他工具下载
# scrabble 只负责检测，不负责下载
githacker -u http://challenge.com/.git/ -o source
```

### 场景2: 批量扫描子域名

```bash
# 准备子域名列表
cat > subdomains.txt << 'EOF'
www.example.com
admin.example.com
dev.example.com
test.example.com
api.example.com
EOF

# 批量扫描
cat subdomains.txt | while read domain; do
    echo "[*] $domain"
    scrabble "http://$domain" -q -o results.txt
done

# 查看发现的泄露
cat results.txt
```

### 场景3: 路径变体扫描

Scrabble 自动尝试常见路径:

```bash
# 扫描
scrabble http://example.com -v

# 会自动检测:
# http://example.com/.git/
# http://example.com/.git/config
# http://example.com/.git/HEAD
# http://example.com/.git/index
# http://example.com/backup/.git/
# http://example.com/test/.git/
# ...
```

### 场景4: 与其他工具配合

```bash
#!/bin/bash
# 完整工作流脚本

TARGET="$1"

echo "[1/2] 检测 .git 泄露..."
if scrabble "$TARGET" -q | grep -q "Found"; then
    echo "[+] 发现泄露！"

    echo "[2/2] 下载源码..."
    githacker -u "$TARGET/.git/" -o source

    echo "[+] 分析..."
    cd source
    grep -r "flag{" .
else
    echo "[-] 未发现泄露"
fi
```

### 场景5: CI/CD 集成

```yaml
# GitHub Actions 示例
name: Git Leak Scanner

on: [push]

jobs:
  scan:
    runs-on: ubuntu-latest
    steps:
      - name: Install Scrabble
        run: go install github.com/BishopFox/scrabble@latest

      - name: Scan Target
        run: scrabble https://example.com --output results.txt

      - name: Check Results
        run: |
          if grep -q "Found" results.txt; then
            echo "::error::Git leak detected!"
            exit 1
          fi
```

---

## 实战案例

### 案例1: 快速单目标检测

```bash
# CTF 比赛中快速检测
$ scrabble http://challenge.com

[+] Checking http://challenge.com/.git/config
[+] Found: http://challenge.com/.git/config (200 OK)
[+] Found: http://challenge.com/.git/HEAD (200 OK)

# 确认泄露后立即下载
$ githacker -u http://challenge.com/.git/ -o source
$ cd source && grep -r "flag{" .
```

### 案例2: 批量域名扫描

```bash
# 扫描多个子域名
cat > targets.txt << 'EOF'
www.target.com
admin.target.com
dev.target.com
test.target.com
backup.target.com
old.target.com
staging.target.com
EOF

# 并行扫描（10个并发）
cat targets.txt | parallel -j 10 scrabble "http://{}" -q -o findings.txt

# 提取成功的
grep "Found" findings.txt

# 输出示例:
# [+] Found: http://dev.target.com/.git/config (200 OK)
# [+] Found: http://backup.target.com/.git/HEAD (200 OK)
```

### 案例3: 大规模扫描

```bash
# 扫描大量目标（1000+）
# 使用高并发

cat large_targets.txt | xargs -P 50 -I {} sh -c '
    echo "[*] Scanning: {}"
    scrabble "http://{}" -q -t 5 >> mass_scan_results.txt 2>&1
'

# 统计结果
echo "Total found: $(grep -c "Found" mass_scan_results.txt)"

# 提取 URL
grep "Found" mass_scan_results.txt | awk '{print $3}' > vulnerable.txt
```

### 案例4: 路径模糊测试

```bash
# 测试不同路径
paths=(
    ""
    "/admin"
    "/test"
    "/backup"
    "/old"
    "/dev"
    "/staging"
    "/_old"
    "/.backup"
)

for path in "${paths[@]}"; do
    url="http://target.com${path}"
    echo "[*] Testing: $url"
    scrabble "$url" -q
done
```

### 案例5: 集成到渗透测试流程

```bash
#!/bin/bash
# 自动化渗透测试脚本

DOMAIN="$1"

# 步骤1: 子域名枚举
echo "[*] Enumerating subdomains..."
subfinder -d "$DOMAIN" -silent > subdomains.txt

# 步骤2: Git 泄露扫描
echo "[*] Scanning for git leaks..."
cat subdomains.txt | parallel -j 20 \
    scrabble "http://{}" -q -o git_leaks.txt

# 步骤3: 下载发现的仓库
grep "Found" git_leaks.txt | awk '{print $3}' | while read url; do
    base_url=$(dirname "$url")
    output_dir="dumps/$(echo $base_url | sed 's/[^a-zA-Z0-9]/_/g')"

    echo "[*] Downloading: $base_url"
    githacker -u "$base_url" -o "$output_dir" -t 10
done

# 步骤4: 分析
echo "[*] Analyzing..."
grep -r "password\|secret\|api_key" dumps/ > sensitive.txt
```

---

## 常见问题

### 问题1: 找不到命令

**症状**: `scrabble: command not found`

**解决**:
```bash
# 检查 Go bin 路径
echo $GOPATH
ls $GOPATH/bin

# 添加到 PATH
export PATH=$PATH:$GOPATH/bin

# 或使用完整路径
$GOPATH/bin/scrabble http://target.com
```

### 问题2: 超时错误

**症状**: 大量请求超时

**解决**:
```bash
# 增加超时时间
scrabble http://target.com -t 30

# 减少并发
scrabble http://target.com -c 5

# 检查网络连接
curl -I http://target.com/.git/config
```

### 问题3: 误报

**症状**: 检测到但实际不存在

**解决**:
```bash
# 手动验证
curl -I http://target.com/.git/config

# 使用详细输出查看详情
scrabble http://target.com -v

# 尝试直接下载验证
wget http://target.com/.git/config
```

### 问题4: 被 WAF 拦截

**症状**: HTTP 403 或 429

**解决**:
```bash
# 减少并发和请求频率
scrabble http://target.com -c 1 -t 10

# 添加延迟（修改源码）
# 或使用其他工具

# 尝试不同路径变体
curl http://target.com/.git%2f/config
curl http://target.com/%2e%67%69%74/config
```

### 问题5: SSL 证书错误

**症状**: 自签名证书导致失败

**解决**:
```bash
# Scrabble 目前不支持 --insecure 选项
# 临时解决方案:

# 方法1: 使用 HTTP 而非 HTTPS
scrabble http://target.com

# 方法2: 使用其他工具
git-dumper https://target.com/.git/ output --insecure
```

---

## 性能对比

### 速度测试（扫描100个目标）

| 工具 | 时间 | 并发 |
|------|------|------|
| **Scrabble** | 8秒 | 10 |
| GitFinder (Python 2) | 45秒 | 20 |
| 手写 Bash 脚本 | 120秒 | 1 |

### 资源占用

```
Scrabble: ~10MB 内存, <5% CPU
Python 工具: ~50MB 内存, ~15% CPU
```

---

## 进阶技巧

### 1. 集成到 Nuclei

```yaml
# git-leak-scrabble.yaml
id: git-leak-detection

info:
  name: Git Configuration Disclosure
  severity: high

requests:
  - method: GET
    path:
      - "{{BaseURL}}/.git/config"
      - "{{BaseURL}}/.git/HEAD"

    matchers:
      - type: word
        words:
          - "[core]"
          - "repositoryformatversion"
```

### 2. 自动化报告

```bash
#!/bin/bash
# 生成扫描报告

TARGETS="targets.txt"
OUTPUT="report_$(date +%Y%m%d).html"

cat > "$OUTPUT" << 'HTML'
<!DOCTYPE html>
<html>
<head><title>Git Leak Scan Report</title></head>
<body>
<h1>Git Leak Scan Results</h1>
<table border="1">
<tr><th>URL</th><th>Status</th></tr>
HTML

cat "$TARGETS" | while read domain; do
    result=$(scrabble "http://$domain" -q 2>&1)

    if echo "$result" | grep -q "Found"; then
        status="<span style='color:red'>VULNERABLE</span>"
    else
        status="<span style='color:green'>SAFE</span>"
    fi

    echo "<tr><td>$domain</td><td>$status</td></tr>" >> "$OUTPUT"
done

echo "</table></body></html>" >> "$OUTPUT"
echo "Report: $OUTPUT"
```

### 3. 与 Masscan 集成

```bash
#!/bin/bash
# 先用 masscan 发现 Web 服务，再扫描 Git 泄露

# 步骤1: 扫描端口
masscan -p 80,443,8080,8443 10.0.0.0/24 \
    --rate 10000 \
    -oG masscan.txt

# 步骤2: 提取 IP:Port
cat masscan.txt | grep "Ports:" | awk '{print $2":"$5}' | sed 's/\/.*//g' > targets.txt

# 步骤3: 扫描 Git
cat targets.txt | parallel -j 50 scrabble "http://{}" -q -o git_findings.txt
```

---

## 参考资源

### 官方资源
- **GitHub**: https://github.com/BishopFox/scrabble
- **Releases**: https://github.com/BishopFox/scrabble/releases
- **Issues**: https://github.com/BishopFox/scrabble/issues

### BishopFox 其他工具
- **Sliver**: C2 框架
- **CloudFox**: AWS 枚举工具
- **Dnsmonster**: DNS 监控工具

### 替代工具
- **GitHacker**: Python Git 泄露利用
- **git-dumper**: Python Git 恢复
- **GitTools**: Bash 工具集

### 学习资源
- **Go 并发编程**: https://go.dev/tour/concurrency
- **CTF Wiki**: https://ctf-wiki.org/
- **OWASP**: https://owasp.org/

---

**文档版本**: v1.0
**更新日期**: 2025-01
**适用版本**: Scrabble v1.0+
