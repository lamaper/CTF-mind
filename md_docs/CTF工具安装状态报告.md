# CTF工具安装状态报告

生成时间：2025-10-24

---

## 📊 总体统计

| 类别 | 已安装 | 未安装 | 总计 |
|------|--------|--------|------|
| **命令行工具** | 16 | 9 | 25 |
| **Python库** | 2 | 7 | 9 |
| **GUI应用** | 2 | 3 | 5 |
| **隐写工具** | 0 | 4 | 4 |
| **逆向工具** | 0 | 5 | 5 |
| **其他工具** | 2 | 4 | 6 |
| **总计** | 22 | 32 | 54 |

**安装率：40.7%**

---

## ✅ 已安装工具列表

### 命令行工具（16个）

| 工具名 | 类别 | 说明 |
|--------|------|------|
| **amass** | 信息收集 | OSINT信息收集工具 |
| **bkcrack** | 密码破解 | Zip密码破解工具 |
| **dig** | 网络工具 | DNS查询工具 |
| **dirsearch** | Web扫描 | Web目录扫描工具 |
| **dnsrecon** | 信息收集 | DNS枚举工具 |
| **ffuf** | Web扫描 | Web模糊测试工具 |
| **gobuster** | Web扫描 | 目录/DNS爆破工具 |
| **hashcat** | 密码破解 | 密码哈希破解工具 |
| **nmap** | 网络扫描 | 网络扫描工具 |
| **sqlmap** | Web渗透 | SQL注入工具 |
| **john** | 密码破解 | John the Ripper密码破解工具 |
| **strings** | 二进制分析 | 字符串提取工具 |
| **file** | 文件分析 | 文件类型识别工具 |
| **xxd** | 十六进制 | 十六进制查看工具 |
| **hexdump** | 十六进制 | 十六进制转储工具 |
| **objdump** | 二进制分析 | 对象文件分析工具 |

### Python库（2个）

| 库名 | 版本 | 说明 |
|------|------|------|
| **requests** | 2.31.0 | HTTP请求库 |
| **pycryptodome** | 3.23.0 | 密码学库 |

### GUI应用（2个）

| 应用名 | 说明 |
|--------|------|
| **Wireshark.app** | 网络协议分析工具 |
| **Burp Suite Professional.app** | Web安全测试工具 |

### 其他工具（2个）

| 工具名 | 说明 |
|--------|------|
| **docker** | 容器化平台 |
| **git** | 版本控制工具 |

---

## ❌ 未安装工具列表

### 命令行工具（9个）

#### 🔴 高优先级（必装）

| 工具名 | 类别 | 安装命令 | 说明 |
|--------|------|----------|------|
| **hydra** | 密码爆破 | `brew install hydra` | 在线密码爆破工具，CTF必备 |
| **binwalk** | 固件分析 | `brew install binwalk` | 固件分析和提取工具 |
| **exiftool** | 元数据分析 | `brew install exiftool` | 图片元数据分析 |
| **steghide** | 隐写工具 | `brew install steghide` | 经典隐写工具 |
| **foremost** | 数据恢复 | `brew install foremost` | 文件恢复和提取 |

#### 🟡 中优先级（推荐）

| 工具名 | 类别 | 安装命令 | 说明 |
|--------|------|----------|------|
| **aircrack-ng** | 无线安全 | `brew install aircrack-ng` | 无线网络安全工具 |
| **radare2** | 逆向工程 | `brew install radare2` | 强大的逆向工程框架 |
| **gdb** | 调试器 | `brew install gdb` | GNU调试器 |

#### 🟢 低优先级（可选）

| 工具名 | 说明 |
|--------|------|
| - | 当前已覆盖主要工具 |

---

### Python库（7个）

#### 🔴 高优先级（必装）

| 库名 | 安装命令 | 说明 |
|------|----------|------|
| **pwntools** | `pip3 install pwntools` | Pwn题目必备工具库 |
| **z3-solver** | `pip3 install z3-solver` | 约束求解器，密码学题常用 |
| **ROPgadget** | `pip3 install ROPgadget` | ROP链构造工具 |

#### 🟡 中优先级（推荐）

| 库名 | 安装命令 | 说明 |
|------|----------|------|
| **capstone** | `pip3 install capstone` | 反汇编框架 |
| **unicorn** | `pip3 install unicorn` | CPU模拟器 |
| **keystone-engine** | `pip3 install keystone-engine` | 汇编器 |

#### 🟢 低优先级（可选）

| 库名 | 安装命令 | 说明 |
|------|----------|------|
| **angr** | `pip3 install angr` | 二进制分析框架（体积大，谨慎安装） |

---

### GUI应用（3个）

#### 🔴 高优先级（必装）

| 应用名 | 下载地址 | 说明 |
|--------|----------|------|
| **Ghidra** | https://ghidra-sre.org/ | NSA开源逆向工程工具，免费 |

#### 🟡 中优先级（推荐）

| 应用名 | 说明 |
|--------|------|
| **IDA Pro** | 商业逆向工程工具（昂贵） |
| **Hopper Disassembler** | Mac平台反汇编器 |

---

### 隐写工具（4个）

#### 🔴 高优先级（必装）

| 工具名 | 安装命令 | 说明 |
|--------|----------|------|
| **zsteg** | `gem install zsteg` | PNG/BMP隐写检测 |
| **pngcheck** | `brew install pngcheck` | PNG文件完整性检查 |
| **stegseek** | 手动安装 | Steghide密码爆破工具 |

#### 🟡 中优先级（推荐）

| 工具名 | 说明 |
|--------|------|
| **stegsolve** | Java图片隐写分析工具 |

---

### 逆向工具（5个）

#### 🔴 高优先级（必装）

| 工具名 | 安装方式 | 说明 |
|--------|----------|------|
| **ghidra** | 下载GUI应用 | 免费强大的逆向工具 |
| **r2 (radare2)** | `brew install radare2` | 命令行逆向框架 |

#### 🟡 中优先级（推荐）

| 工具名 | 说明 |
|--------|------|
| **cutter** | Radare2的GUI前端 |
| **ida** | IDA Pro |
| **hopper** | Hopper Disassembler |

---

### 其他实用工具（4个）

#### 🔴 高优先级（必装）

| 工具名 | 安装命令 | 说明 |
|--------|----------|------|
| **checksec** | `brew install checksec` | 二进制安全检查工具 |
| **qemu** | `brew install qemu` | CPU模拟器 |

#### 🟡 中优先级（推荐）

| 工具名 | 安装命令 | 说明 |
|--------|----------|------|
| **one_gadget** | `gem install one_gadget` | One Gadget查找工具 |
| **patchelf** | `brew install patchelf` | ELF文件修改工具 |

---

## 🚀 快速安装方案

### 方案A：核心必备工具（适合快速开始）

```bash
# 1. 命令行工具（5个必备）
brew install hydra binwalk exiftool steghide foremost

# 2. Python库（3个必备）
pip3 install pwntools z3-solver ROPgadget

# 3. 隐写工具（3个必备）
gem install zsteg
brew install pngcheck

# 4. 逆向工具（1个必备）
brew install radare2

# 5. 其他工具（2个必备）
brew install checksec qemu
```

**预计安装时间**：15-30分钟
**磁盘占用**：约2-3GB

---

### 方案B：完整工具包（适合深度使用）

```bash
#!/bin/bash
# 完整CTF工具安装脚本

echo "开始安装CTF完整工具包..."

# 1. 更新Homebrew
brew update

# 2. 命令行工具
echo "[1/6] 安装命令行工具..."
brew install hydra binwalk exiftool steghide foremost aircrack-ng radare2 gdb checksec qemu patchelf pngcheck

# 3. Python库
echo "[2/6] 安装Python库..."
pip3 install pwntools z3-solver ROPgadget capstone unicorn keystone-engine

# 4. Ruby工具
echo "[3/6] 安装Ruby工具..."
gem install zsteg one_gadget

# 5. 下载Ghidra
echo "[4/6] 准备下载Ghidra..."
echo "请访问: https://ghidra-sre.org/"
echo "下载后拖到Applications文件夹"

# 6. 下载stegsolve
echo "[5/6] 准备下载Stegsolve..."
echo "请访问: https://github.com/eugenekolo/sec-tools/tree/master/stego/stegsolve"

# 7. 验证安装
echo "[6/6] 验证安装..."
command -v hydra && echo "✓ hydra"
command -v binwalk && echo "✓ binwalk"
command -v radare2 && echo "✓ radare2"
python3 -c "import pwn" && echo "✓ pwntools"

echo "安装完成！"
```

保存为 `install_all_ctf_tools.sh`，然后执行：
```bash
chmod +x install_all_ctf_tools.sh
./install_all_ctf_tools.sh
```

**预计安装时间**：30-60分钟
**磁盘占用**：约5-8GB

---

### 方案C：分类安装（按需选择）

#### Web渗透工具包
```bash
brew install hydra sqlmap gobuster ffuf dirsearch
```

#### 密码破解工具包
```bash
brew install hashcat john hydra
pip3 install hashid
```

#### 隐写分析工具包
```bash
brew install steghide binwalk foremost exiftool pngcheck
gem install zsteg
```

#### 二进制分析工具包
```bash
brew install radare2 gdb checksec patchelf qemu
pip3 install pwntools ROPgadget capstone unicorn
```

#### 逆向工程工具包
```bash
brew install radare2 gdb
# 手动下载：Ghidra, IDA, Hopper
```

---

## 📋 安装优先级建议

### 第一批（立即安装，20分钟）
```bash
# 最常用的CTF工具
brew install hydra binwalk exiftool steghide
pip3 install pwntools z3-solver
gem install zsteg
```

### 第二批（本周内安装，30分钟）
```bash
# 扩展功能工具
brew install foremost radare2 checksec qemu pngcheck
pip3 install ROPgadget capstone unicorn
```

### 第三批（有需要时安装，1小时）
```bash
# 高级和专业工具
brew install aircrack-ng gdb patchelf
gem install one_gadget
# 下载Ghidra
```

---

## 🎯 按CTF题型分类

### Misc类题目
**已有**：✅ binwalk, exiftool, strings, file, xxd
**缺少**：❌ foremost, steghide, zsteg, stegsolve, pngcheck

**推荐安装**：
```bash
brew install foremost steghide pngcheck
gem install zsteg
```

### Crypto类题目
**已有**：✅ hashcat, john, pycryptodome
**缺少**：❌ z3-solver

**推荐安装**：
```bash
pip3 install z3-solver
```

### Web类题目
**已有**：✅ sqlmap, gobuster, ffuf, dirsearch, nmap
**缺少**：❌ hydra

**推荐安装**：
```bash
brew install hydra
```

### Pwn类题目
**已有**：✅ objdump
**缺少**：❌ pwntools, ROPgadget, radare2, gdb, checksec

**推荐安装**：
```bash
brew install radare2 gdb checksec
pip3 install pwntools ROPgadget
```

### Reverse类题目
**已有**：✅ objdump, xxd
**缺少**：❌ ghidra, radare2, gdb

**推荐安装**：
```bash
brew install radare2 gdb
# 下载Ghidra: https://ghidra-sre.org/
```

---

## 💾 磁盘空间需求

| 安装方案 | 磁盘占用 | 安装时间 |
|----------|----------|----------|
| **方案A（核心）** | ~2-3 GB | 15-30分钟 |
| **方案B（完整）** | ~5-8 GB | 30-60分钟 |
| **Ghidra** | ~500 MB | 5分钟 |
| **IDA Pro** | ~1-2 GB | 10分钟 |
| **angr（Python）** | ~800 MB | 15-30分钟 |

---

## 🔧 安装注意事项

### 1. Homebrew相关

```bash
# 如果遇到权限问题
sudo chown -R $(whoami) /usr/local/Homebrew

# 如果下载慢，使用国内镜像
export HOMEBREW_BOTTLE_DOMAIN=https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles
```

### 2. Python库安装

```bash
# 如果pip安装失败，尝试使用--user
pip3 install --user pwntools

# 如果需要特定版本
pip3 install pwntools==4.11.0
```

### 3. Ruby Gems安装

```bash
# 如果gem安装慢，切换国内源
gem sources --add https://gems.ruby-china.com/ --remove https://rubygems.org/
gem sources -l
```

### 4. 特殊工具安装

#### Ghidra安装
```bash
# 1. 下载：https://ghidra-sre.org/
# 2. 解压到Applications
# 3. 需要Java 17+
brew install openjdk@17
```

#### Stegsolve安装
```bash
# 下载jar文件
curl -L -o ~/Desktop/stegsolve.jar https://github.com/eugenekolo/sec-tools/raw/master/stego/stegsolve/stegsolve/stegsolve.jar

# 运行
java -jar ~/Desktop/stegsolve.jar
```

#### Stegseek安装（Linux/WSL）
```bash
# 仅支持Linux，Mac上需要使用Docker
docker pull rickdejager/stegseek
```

---

## 📝 验证安装

安装完成后，使用以下脚本验证：

```bash
#!/bin/bash
echo "验证CTF工具安装..."

tools=(
    "hydra" "binwalk" "exiftool" "steghide" "foremost"
    "radare2" "gdb" "checksec" "qemu-system-x86_64"
)

for tool in "${tools[@]}"; do
    if command -v "$tool" &> /dev/null; then
        echo "✅ $tool"
    else
        echo "❌ $tool - 未安装"
    fi
done

echo ""
echo "Python库检查："
python3 -c "import pwn" 2>/dev/null && echo "✅ pwntools" || echo "❌ pwntools"
python3 -c "import z3" 2>/dev/null && echo "✅ z3-solver" || echo "❌ z3-solver"

echo ""
echo "Ruby工具检查："
command -v zsteg &> /dev/null && echo "✅ zsteg" || echo "❌ zsteg"
```

---

## 🎓 学习建议

### 新手优先掌握（前3个月）
1. **Web工具**：sqlmap, gobuster, burpsuite
2. **密码破解**：hashcat, john, hydra
3. **信息收集**：nmap, amass, dnsrecon
4. **隐写分析**：binwalk, exiftool, steghide, zsteg

### 进阶必学（3-6个月）
1. **Pwn工具**：pwntools, ROPgadget, checksec
2. **逆向工程**：radare2, gdb, Ghidra
3. **密码学**：z3-solver, pycryptodome
4. **二进制分析**：capstone, unicorn

### 高级选修（6个月+）
1. **高级逆向**：IDA Pro, angr
2. **无线安全**：aircrack-ng
3. **固件分析**：binwalk, foremost, qemu

---

## 📚 相关文档

本目录下的工具使用手册：
- [x] amass使用手册.md
- [x] bkcrack_使用指南.md
- [x] CyberChef使用指南.md
- [x] dig_使用详尽手册.md
- [x] dirsearch使用手册.md
- [x] dnsrecon使用手册.md
- [x] ffuf使用手册.md
- [x] gobuster使用手册.md
- [x] hashcat_使用手册.md
- [x] nmap使用手册.md
- [x] sqlmap使用手册.md
- [x] Wireshark使用手册.md

待添加的工具手册：
- [ ] hydra使用手册.md
- [ ] binwalk使用手册.md
- [ ] steghide使用手册.md
- [ ] pwntools使用手册.md
- [ ] radare2使用手册.md
- [ ] Ghidra使用手册.md

---

## 🔄 更新日志

- **2025-10-24**: 初始报告生成
  - 检查25个命令行工具
  - 检查9个Python库
  - 检查5个GUI应用
  - 检查10个专用工具
  - 总计54个工具

---

**报告生成者**: 上海理工大学信息安全战队
**下次更新**: 每月或工具包更新时
**维护**: 定期检查工具更新和新工具添加

---

## 📞 工具问题求助

如果安装或使用遇到问题：
1. 查看本目录下对应工具的使用手册
2. 搜索GitHub Issues
3. 咨询战队成员
4. CTF论坛求助

**Good Luck in CTF! 🚩**
