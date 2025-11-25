# Git泄漏找Flag思路总结

## 概述
Git泄漏是CTF中常见的Misc/Web题型，通过分析.git目录或git仓库历史记录来寻找隐藏的flag。

---

## 核心思路分类

### 1. Git Stash（储藏区检查）⭐

**原理**: 开发者使用`git stash`临时保存未提交的修改，flag可能被储藏后删除

**检查命令**:
```bash
# 列出所有stash
git stash list

# 查看特定stash的详细内容
git stash show -p stash@{0}

# 应用stash查看文件
git stash apply stash@{0}
```

**实战示例**:
```bash
$ git stash list
stash@{0}: WIP on master: c35bb3e add flag

$ git stash show -p stash@{0}
diff --git a/file.txt b/file.txt
-where is flag
+ctfhub{68ad8199f5c4734638a08f6a}
```

---

### 2. Git Log & Reflog（历史记录分析）⭐⭐

**原理**: 查看完整的提交历史，包括已删除的提交

**检查命令**:
```bash
# 查看提交历史
git log --all --oneline
git log --all --graph --decorate

# 查看详细的引用日志（包括被删除的提交）
git reflog

# 查看所有分支的提交历史
git log --all --full-history --source --
```

**关键点**:
- 查找包含"flag"、"add"、"remove"等关键词的提交
- reflog能显示被reset/rebase删除的提交
- 使用`--all`参数查看所有分支

---

### 3. Git Show（查看特定提交）⭐⭐

**原理**: 检查特定提交的详细内容

**检查命令**:
```bash
# 查看特定提交的完整内容
git show <commit-hash>

# 查看特定提交的特定文件
git show <commit-hash>:path/to/file

# 列出提交中修改的所有文件
git show --name-only <commit-hash>
```

**实战技巧**:
```bash
# 遍历所有提交查找flag
for commit in $(git log --all --format=%H); do
    echo "=== $commit ==="
    git show $commit | grep -i "flag\|ctf"
done
```

---

### 4. Git Diff（差异对比）⭐

**原理**: 对比不同版本之间的差异，找到被删除的flag

**检查命令**:
```bash
# 对比两个提交
git diff <commit1> <commit2>

# 查看工作区与最新提交的差异
git diff HEAD

# 查看暂存区差异
git diff --cached

# 对比所有历史更改
git log -p
```

---

### 5. 分支检查（Branch Analysis）⭐

**原理**: flag可能在其他分支或已删除的分支中

**检查命令**:
```bash
# 查看所有分支（包括远程）
git branch -a

# 查看已删除分支的记录
git reflog | grep "checkout:"

# 恢复已删除的分支
git checkout -b <branch-name> <commit-hash>

# 检查所有分支的文件
git log --all --source -- <filename>
```

---

### 6. Git Objects直接分析⭐⭐⭐

**原理**: 直接检查.git/objects目录中的所有对象

**检查命令**:
```bash
# 查看所有git对象
find .git/objects -type f

# 遍历所有对象查找内容
for obj in $(find .git/objects -type f | sed 's/\.git\/objects\///g' | tr -d '/'); do
    git cat-file -p $obj 2>/dev/null | grep -i "flag\|ctf"
done

# 查看松散对象
git fsck --lost-found

# 检查未被引用的对象
git fsck --unreachable
```

---

### 7. .git目录泄漏利用（Web场景）⭐⭐⭐

**场景**: Web应用泄漏.git目录，需要下载并重建仓库

#### 工具1: GitHacker
```bash
githacker --url http://target.com/.git/ --output-folder result
cd result
git log --all
git stash list
```

#### 工具2: git-dumper
```bash
git-dumper http://target.com/.git/ result
cd result
git checkout .
git log --all
```

#### 工具3: GitTools
```bash
# 下载.git目录
./gitdumper.sh http://target.com/.git/ result

# 提取器
./extractor.sh result extract
cd extract
```

#### 手动下载关键文件
```bash
# 关键文件列表
.git/HEAD
.git/config
.git/index
.git/logs/HEAD
.git/refs/heads/master
.git/objects/xx/xxxxxx...
```

---

### 8. 特殊场景检查

#### 8.1 配置文件检查
```bash
# 查看git配置
cat .git/config

# 查看用户信息
git config --list
```

#### 8.2 标签检查
```bash
# 查看所有标签
git tag

# 查看标签详情
git show <tag-name>
```

#### 8.3 忽略文件检查
```bash
# 查看.gitignore
cat .gitignore

# 查找被忽略但存在的文件
git status --ignored
```

#### 8.4 子模块检查
```bash
# 查看子模块
cat .gitmodules
git submodule status
```

---

## 自动化脚本

### 综合检查脚本
```bash
#!/bin/bash

echo "[*] Git Repository Flag Hunter"
echo "================================"

# 1. Stash检查
echo -e "\n[+] Checking stashes..."
git stash list
for i in {0..10}; do
    git stash show -p stash@{$i} 2>/dev/null | grep -i "flag\|ctf"
done

# 2. 提交历史检查
echo -e "\n[+] Checking commit history..."
git log --all --oneline | grep -i "flag\|add\|remove\|delete"

# 3. Reflog检查
echo -e "\n[+] Checking reflog..."
git reflog | head -20

# 4. 遍历所有提交查找flag
echo -e "\n[+] Searching in all commits..."
for commit in $(git log --all --format=%H); do
    git show $commit | grep -iE "flag\{|ctf\{|flag:" && echo "Found in: $commit"
done

# 5. 检查所有分支
echo -e "\n[+] Checking all branches..."
git branch -a

# 6. 检查未引用对象
echo -e "\n[+] Checking unreachable objects..."
git fsck --unreachable

# 7. 搜索所有对象
echo -e "\n[+] Searching in all objects..."
find .git/objects -type f | while read obj; do
    hash=$(echo $obj | sed 's/\.git\/objects\///g' | tr -d '/')
    git cat-file -p $hash 2>/dev/null | grep -iE "flag\{|ctf\{" && echo "Found in object: $hash"
done

echo -e "\n[*] Scan complete!"
```

---

## 常见Flag位置

| 位置 | 概率 | 检查方法 |
|------|------|---------|
| git stash | ⭐⭐⭐⭐⭐ | `git stash list && git stash show -p` |
| 历史提交 | ⭐⭐⭐⭐ | `git log --all -p` |
| 已删除提交 | ⭐⭐⭐⭐ | `git reflog` |
| 其他分支 | ⭐⭐⭐ | `git branch -a` |
| git objects | ⭐⭐⭐ | 遍历.git/objects |
| 提交信息 | ⭐⭐ | `git log --all --oneline` |
| 标签 | ⭐⭐ | `git tag` |
| 配置文件 | ⭐ | `cat .git/config` |

---

## 快速检查清单

- [ ] `git stash list` - 检查储藏区
- [ ] `git log --all --oneline` - 查看所有提交
- [ ] `git reflog` - 查看引用日志
- [ ] `git branch -a` - 查看所有分支
- [ ] `git tag` - 查看标签
- [ ] `git fsck --unreachable` - 查看未引用对象
- [ ] 遍历所有提交内容查找flag字符串
- [ ] 遍历.git/objects查找flag
- [ ] 检查.git/config配置文件
- [ ] 使用自动化工具扫描

---

## 工具推荐

### Web Git泄漏工具
1. **GitHacker** - Python工具，自动化下载和分析
2. **git-dumper** - 快速下载.git目录
3. **GitTools** - 包含dumper和extractor
4. **dvcs-ripper** - 支持多种版本控制系统

### 本地分析工具
1. **git命令** - 原生强大
2. **tig** - 交互式git查看器
3. **gitk** - 图形化历史查看
4. **grep/rg** - 内容搜索

---

## 实战技巧

### 技巧1: 关键词搜索
```bash
# 搜索所有提交中的敏感词
git log --all -S "flag" -S "password" -S "key"

# 搜索提交信息
git log --all --grep="flag\|password"
```

### 技巧2: 时间线分析
```bash
# 按时间查看提交
git log --all --since="2024-01-01" --until="2024-12-31"

# 查看特定作者的提交
git log --all --author="admin"
```

### 技巧3: 文件历史追踪
```bash
# 追踪特定文件的所有历史
git log --all --full-history -- <filename>

# 查看文件的所有版本
git log --all --follow -p -- <filename>
```

### 技巧4: 二进制文件检查
```bash
# 查找所有二进制文件
git log --all --numstat | grep "^-"

# 提取特定提交的二进制文件
git show <commit>:<file> > extracted_file
```

---

## 防御建议（了解攻击方式）

1. **生产环境禁止.git目录**
   - Nginx: `location ~ /\.git { deny all; }`
   - Apache: `RedirectMatch 404 /\.git`

2. **使用.gitignore**
   - 添加敏感文件到.gitignore
   - 但注意：已提交的文件不会自动删除

3. **清理敏感历史**
   ```bash
   # 从所有历史中删除文件
   git filter-branch --tree-filter 'rm -f passwords.txt' HEAD

   # 使用git-filter-repo（推荐）
   git filter-repo --path passwords.txt --invert-paths
   ```

4. **使用环境变量**
   - 敏感信息不要硬编码
   - 使用.env文件（加入.gitignore）

---

## 总结

Git泄漏找flag的核心思路：
1. **优先检查stash** - 最常见的隐藏点
2. **完整历史分析** - log + reflog组合
3. **对象遍历** - 全面但耗时
4. **自动化工具** - 提高效率
5. **关键词搜索** - grep是好帮手

**记住**: Git的设计初衷是永久保存历史，"删除"往往只是引用删除，数据仍在.git/objects中。
