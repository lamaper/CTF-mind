# Binwalk 使用手册（固件分析与文件提取）

> 免责声明：本手册仅用于合法的安全研究、CTF 竞赛和授权的固件分析。未经授权分析他人设备固件可能违法。

---

## 目录
1. 概述
2. 安装方法
3. 基本命令结构
4. 核心功能详解
5. 常用命令与示例
6. 文件签名识别
7. CTF 常见场景
8. 实战案例
9. 高级技巧
10. 常见问题与排查
11. 结合其他工具
12. 参考链接

---

## 1. 概述

Binwalk 是一个专门用于固件分析和文件提取的工具，由 Craig Heffner 开发。它能够：
- **识别文件签名**：扫描二进制文件中的嵌入文件和文件系统
- **自动提取文件**：递归提取嵌入的压缩包、文件系统和固件镜像
- **熵分析**：检测加密、压缩和随机数据
- **固件解包**：支持各种固件格式（UBI、JFFS2、SquashFS 等）

**适用场景**：
- CTF MISC 题目（文件隐写、嵌入文件）
- IoT 固件逆向分析
- 文件格式分析与数据恢复
- 隐写分析（图片、音频中隐藏数据）

**版本信息**：本手册基于 Binwalk v2.3.0+

---

## 2. 安装方法

### macOS 安装
```bash
# 使用 Homebrew 安装
brew install binwalk

# 验证安装
binwalk --help

# 安装额外依赖（完整功能）
brew install p7zip squashfs

# Python 依赖
pip3 install binwalk
```

### Kali Linux / Debian / Ubuntu
```bash
# Kali 自带，如需更新
sudo apt update
sudo apt install binwalk

# 安装额外工具（用于提取更多格式）
sudo apt install mtd-utils gzip bzip2 tar arj lhasa p7zip \
  p7zip-full cabextract cramfsprogs cramfsswap squashfs-tools \
  sleuthkit default-jdk lzop srecord
```

### 从源码安装（最新版本）
```bash
# 克隆仓库
git clone https://github.com/ReFirmLabs/binwalk.git
cd binwalk

# 安装
sudo python3 setup.py install

# 安装依赖
sudo ./deps.sh
```

---

## 3. 基本命令结构

```bash
binwalk [选项] [文件]

# 基本扫描
binwalk firmware.bin

# 提取文件
binwalk -e firmware.bin

# 递归提取
binwalk -Me firmware.bin
```

### 核心选项说明

**扫描选项**：
- `-B` / `--signature` - 扫描常见文件签名（默认）
- `-R` / `--raw=<string>` - 搜索原始字符串
- `-A` / `--opcodes` - 扫描可执行代码
- `-E` / `--entropy` - 计算熵值（检测加密/压缩）

**提取选项**：
- `-e` / `--extract` - 自动提取已识别文件
- `-M` / `--matryoshka` - 递归提取（俄罗斯套娃模式）
- `-D` / `--dd=<type:ext:cmd>` - 自定义提取规则
- `--run-as=root` - 以 root 权限运行（某些文件系统需要）

**输出选项**：
- `-v` / `--verbose` - 显示详细信息
- `-q` / `--quiet` - 安静模式
- `-C` / `--directory=<dir>` - 指定提取目录
- `-o` / `--offset=<offset>` - 从指定偏移开始扫描
- `-l` / `--length=<length>` - 扫描指定长度

**过滤选项**：
- `-y` / `--include-invalid` - 显示无效结果
- `-I` / `--show-invalid` - 仅显示无效结果
- `-x` / `--exclude=<filter>` - 排除特定签名

---

## 4. 核心功能详解

### 4.1 文件签名扫描

Binwalk 内置 **500+ 文件签名**，包括：
- 压缩格式：ZIP、GZIP、BZIP2、LZMA、7Z
- 文件系统：ext2/3/4、JFFS2、CRAMFS、SquashFS、YAFFS2、UBI
- 图像格式：JPEG、PNG、GIF、BMP
- 音视频：MP3、AVI、MP4
- 固件：U-Boot、ARM/MIPS 代码、Android Boot

**查看支持的签名**：
```bash
binwalk -B --list
```

### 4.2 熵分析

**熵值范围**：0（完全有序） - 1（完全随机）
- **低熵（0-0.3）**：纯文本、重复数据
- **中熵（0.3-0.7）**：普通文件（代码、文档）
- **高熵（0.7-1.0）**：加密数据、压缩数据、随机数据

**用途**：
- 检测加密区域
- 识别压缩算法
- 发现隐藏数据

### 4.3 自动提取

**提取模式**：
- **单层提取（-e）**：仅提取直接识别的文件
- **递归提取（-M）**：逐层解包，直到无法继续提取

**提取目录结构**：
```
_firmware.bin.extracted/
├── 0.zip           # 偏移 0 处的 ZIP 文件
├── 1000.gzip       # 偏移 1000 处的 GZIP 文件
├── _0.zip.extracted/  # ZIP 内容
└── ...
```

---

## 5. 常用命令与示例

### 5.1 基本扫描

**扫描文件签名**：
```bash
binwalk firmware.bin
```

**输出示例**：
```
DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
0             0x0             JPEG image data
54272         0xD400          PNG image
131072        0x20000         gzip compressed data
262144        0x40000         Squashfs filesystem
```

**详细扫描（显示更多信息）**：
```bash
binwalk -v firmware.bin
```

### 5.2 文件提取

**自动提取（单层）**：
```bash
binwalk -e firmware.bin
```

**递归提取（完全解包）**：
```bash
binwalk -Me firmware.bin
```
- `-M` = `--matryoshka`（俄罗斯套娃模式）
- 会自动解压 ZIP、GZIP、TAR 等，并继续扫描内层文件

**指定提取目录**：
```bash
binwalk -e -C /tmp/extracted firmware.bin
```

### 5.3 熵分析

**生成熵图**：
```bash
binwalk -E firmware.bin
```

**保存熵图为 PNG**：
```bash
binwalk -E -J firmware.bin
```
- 生成 `firmware.bin.png`
- 高熵区域（加密/压缩）显示为波动
- 低熵区域（文本/重复数据）显示为平坦

**结合签名扫描和熵分析**：
```bash
binwalk -B -E firmware.bin
```

### 5.4 搜索特定字符串

**搜索原始字符串**：
```bash
binwalk -R "flag{" challenge.bin
```

**搜索十六进制模式**：
```bash
binwalk -R "\x89PNG" image.bin
```

### 5.5 自定义提取规则

**提取特定类型文件**：
```bash
# 提取所有 JPEG 图片
binwalk -D 'jpeg:jpg' firmware.bin

# 提取 PNG 图片
binwalk -D 'png:png' firmware.bin

# 提取 ZIP 文件并执行自定义命令
binwalk -D 'zip:zip:unzip %e' firmware.bin
```

**格式说明**：
```
-D '<类型>:<扩展名>:<命令>'
```
- `<类型>`：签名类型（如 jpeg、png、zip）
- `<扩展名>`：保存文件的扩展名
- `<命令>`：提取后执行的命令（`%e` 代表提取的文件）

### 5.6 扫描特定偏移范围

**从指定偏移开始**：
```bash
binwalk -o 1024 firmware.bin
```

**扫描指定长度**：
```bash
binwalk -o 1024 -l 4096 firmware.bin
```
- 从偏移 1024 开始，扫描 4096 字节

---

## 6. 文件签名识别

### 6.1 常见文件签名（Magic Bytes）

| 文件类型 | 十六进制签名 | 偏移 |
|---------|------------|------|
| JPEG | `FF D8 FF` | 0 |
| PNG | `89 50 4E 47` | 0 |
| GIF | `47 49 46 38` | 0 |
| ZIP | `50 4B 03 04` | 0 |
| GZIP | `1F 8B 08` | 0 |
| BZIP2 | `42 5A 68` | 0 |
| 7Z | `37 7A BC AF 27 1C` | 0 |
| ELF (Linux) | `7F 45 4C 46` | 0 |
| PDF | `25 50 44 46` | 0 |
| RAR | `52 61 72 21` | 0 |

### 6.2 固件特定签名

| 类型 | 签名 | 描述 |
|------|------|------|
| U-Boot | `27 05 19 56` | U-Boot uImage |
| Squashfs | `68 73 71 73` / `73 71 73 68` | Squashfs 文件系统 |
| JFFS2 | `19 85` / `85 19` | JFFS2 文件系统 |
| CRAMFS | `45 3D CD 28` | CRAMFS 文件系统 |
| YAFFS2 | N/A | YAFFS2（无固定签名） |

### 6.3 手动查看文件签名

**使用 hexdump**：
```bash
hexdump -C firmware.bin | head -n 10
```

**使用 xxd**：
```bash
xxd firmware.bin | head -n 20
```

**使用 file 命令**：
```bash
file firmware.bin
```

---

## 7. CTF 常见场景

### 场景1：图片隐写（嵌入 ZIP）

**题目特征**：提供一张图片，但大小异常（几 MB）

**解题步骤**：
```bash
# 1. 扫描图片
binwalk image.png

# 输出示例：
# DECIMAL       HEXADECIMAL     DESCRIPTION
# 0             0x0             PNG image
# 54321         0xD431          Zip archive data

# 2. 提取隐藏文件
binwalk -e image.png

# 3. 查看提取的内容
cd _image.png.extracted
ls
```

### 场景2：固件镜像解包

**题目特征**：提供固件文件（.bin/.img）

```bash
# 1. 扫描固件结构
binwalk firmware.bin

# 输出示例：
# 0             0x0             U-Boot uImage
# 131072        0x20000         LZMA compressed data
# 1048576       0x100000        Squashfs filesystem

# 2. 递归提取
binwalk -Me firmware.bin

# 3. 进入文件系统查找 flag
cd _firmware.bin.extracted/squashfs-root
grep -r "flag{" .
```

### 场景3：音频文件隐写

**题目特征**：提供音频文件（MP3/WAV）

```bash
# 1. 扫描音频文件
binwalk music.mp3

# 输出示例：
# 0             0x0             MP3 audio
# 2048000       0x1F4000        PNG image

# 2. 提取隐藏图片
binwalk -e music.mp3

# 3. 查看提取的图片
cd _music.mp3.extracted
open 1F4000.png
```

### 场景4：多层嵌套提取

**题目特征**：文件套娃（ZIP 套 GZIP 套 TAR）

```bash
# 使用 -M 递归提取
binwalk -Me nested.bin

# 自动完成所有解包，最终找到 flag
find _nested.bin.extracted -name "*flag*"
```

---

## 8. 实战案例

### 案例1：解包 TP-Link 路由器固件

```bash
# 1. 下载固件
wget http://example.com/tplink_firmware.bin

# 2. 扫描固件结构
binwalk tplink_firmware.bin

# 输出：
# DECIMAL       HEXADECIMAL     DESCRIPTION
# 0             0x0             TP-Link firmware header
# 512           0x200           LZMA compressed data
# 1048576       0x100000        Squashfs filesystem

# 3. 递归提取
binwalk -Me tplink_firmware.bin

# 4. 分析文件系统
cd _tplink_firmware.bin.extracted/squashfs-root
ls -la
cat etc/passwd

# 5. 查找硬编码密码
grep -r "password" .
grep -r "admin" .
```

### 案例2：CTF - Misc 隐写题

**题目**：提供 `challenge.jpg`，提示"图片中有秘密"

```bash
# 1. 查看文件基本信息
file challenge.jpg
# Output: JPEG image data, JFIF standard 1.01

# 2. 检查文件大小（异常大）
ls -lh challenge.jpg
# Output: 3.5M （正常 JPEG 应该 <1MB）

# 3. Binwalk 扫描
binwalk challenge.jpg

# 输出：
# DECIMAL       HEXADECIMAL     DESCRIPTION
# 0             0x0             JPEG image data
# 156432        0x26310         Zip archive data, encrypted

# 4. 提取 ZIP
binwalk -e challenge.jpg

# 5. 破解 ZIP 密码（如果加密）
cd _challenge.jpg.extracted
fcrackzip -u -D -p /usr/share/wordlists/rockyou.txt 26310.zip

# 6. 解压获取 flag
unzip 26310.zip
cat flag.txt
```

### 案例3：IoT 设备固件分析

**目标**：分析智能摄像头固件，查找默认密码

```bash
# 1. 扫描固件
binwalk camera_firmware.bin

# 输出：
# 0             0x0             U-Boot uImage
# 64            0x40            LZMA compressed data
# 2097152       0x200000        JFFS2 filesystem

# 2. 提取文件系统（需要 root 权限）
sudo binwalk -Me --run-as=root camera_firmware.bin

# 3. 查找配置文件
cd _camera_firmware.bin.extracted/jffs2-root
find . -name "*.conf"
find . -name "*.cfg"

# 4. 搜索敏感信息
grep -r "password" .
grep -r "admin" .
grep -r "default" .

# 5. 分析启动脚本
cat etc/init.d/rcS
```

---

## 9. 高级技巧

### 9.1 结合 dd 提取特定区域

**场景**：Binwalk 识别了文件，但自动提取失败

```bash
# 1. Binwalk 扫描获取偏移
binwalk firmware.bin
# 输出：131072  0x20000  Squashfs filesystem

# 2. 使用 dd 手动提取
dd if=firmware.bin of=squashfs.img bs=1 skip=131072

# 3. 挂载文件系统
mkdir /tmp/squashfs
sudo mount -t squashfs squashfs.img /tmp/squashfs

# 4. 分析内容
ls /tmp/squashfs

# 5. 卸载
sudo umount /tmp/squashfs
```

### 9.2 熵分析定位加密区域

```bash
# 1. 生成熵图
binwalk -E firmware.bin

# 2. 分析输出（查找高熵区域）
# 高熵区域（0.8+）可能是加密数据

# 3. 结合签名扫描验证
binwalk -B -E firmware.bin

# 4. 如果是加密，尝试识别算法（使用其他工具）
```

### 9.3 自定义签名

**创建自定义签名文件**：
```bash
# 1. 创建签名文件 custom.magic
echo '0   string   CUSTOM   Custom file format' > custom.magic

# 2. 使用自定义签名
binwalk --magic=custom.magic firmware.bin
```

### 9.4 批量分析

**批量扫描多个固件**：
```bash
#!/bin/bash
for firmware in *.bin; do
  echo "Analyzing $firmware..."
  binwalk "$firmware" > "${firmware}.report"
  binwalk -Me "$firmware"
done
```

---

## 10. 常见问题与排查

### 问题1：提取失败（Permission Denied）

**错误信息**：`WARNING: Extractor.execute failed to run external extractor`

**原因**：某些文件系统需要 root 权限

**解决方案**：
```bash
# 使用 sudo 并启用 --run-as=root
sudo binwalk -Me --run-as=root firmware.bin
```

### 问题2：识别了文件但未提取

**原因**：缺少对应的解压工具

**解决方案**：
```bash
# macOS 安装额外工具
brew install p7zip squashfs

# Linux 安装
sudo apt install p7zip-full squashfs-tools cramfsswap mtd-utils

# 验证工具可用
which unsquashfs
which 7z
```

### 问题3：文件系统损坏

**错误信息**：`unsquashfs: Filesystem corruption detected`

**排查步骤**：
```bash
# 1. 确认偏移正确
binwalk firmware.bin

# 2. 手动提取到单独文件
dd if=firmware.bin of=fs.img bs=1 skip=<offset>

# 3. 使用文件系统专用工具修复
fsck.ext4 -f fs.img  # 对于 ext4
# 或
sudo unsquashfs -f fs.img  # 强制提取 Squashfs
```

### 问题4：识别错误（误报）

**原因**：随机数据碰巧匹配文件签名

**解决方案**：
```bash
# 1. 使用 -y 显示无效结果
binwalk -y firmware.bin

# 2. 手动验证可疑偏移
xxd -s <offset> -l 64 firmware.bin

# 3. 结合熵分析
binwalk -B -E firmware.bin
```

---

## 11. 结合其他工具

### 11.1 与 Foremost 结合

**Binwalk 用于扫描，Foremost 用于恢复**：
```bash
# 1. Binwalk 扫描
binwalk firmware.bin

# 2. Foremost 恢复删除的文件
foremost -i firmware.bin -o recovered/

# 3. 对比结果
diff -r _firmware.bin.extracted recovered/
```

### 11.2 与 7z 结合

**处理复杂压缩包**：
```bash
# 1. Binwalk 提取
binwalk -e firmware.bin

# 2. 使用 7z 进一步解压
cd _firmware.bin.extracted
7z x -r *.zip
7z x -r *.7z
```

### 11.3 与 Strings 结合

**查找文本线索**：
```bash
# 1. 提取可读字符串
strings firmware.bin > strings.txt

# 2. 搜索关键字
grep -i "password" strings.txt
grep -i "admin" strings.txt
grep -i "key" strings.txt

# 3. 结合 Binwalk 结果定位
```

### 11.4 与 Hexdump/xxd 结合

**手动分析可疑区域**：
```bash
# 1. Binwalk 找到偏移
binwalk firmware.bin
# 输出：12345  0x3039  Suspicious data

# 2. 查看该区域十六进制
xxd -s 12345 -l 256 firmware.bin

# 3. 或使用 hexdump
hexdump -C -s 12345 -n 256 firmware.bin
```

---

## 12. 参考链接

### 官方资源
- **Binwalk GitHub**：https://github.com/ReFirmLabs/binwalk
- **官方 Wiki**：https://github.com/ReFirmLabs/binwalk/wiki
- **签名数据库**：https://github.com/ReFirmLabs/binwalk/blob/master/src/binwalk/magic

### 固件分析资源
- **Firmware Analysis Toolkit (FAT)**：https://github.com/attify/firmware-analysis-toolkit
- **Firmwalker**：https://github.com/craigz28/firmwalker
- **Firmware.re 数据库**：https://www.firmware.re/

### 教程和文章
- **Binwalk 完整指南**：https://www.refirmlabs.com/binwalk/
- **固件逆向入门**：https://www.pentestpartners.com/security-blog/
- **CTF Wiki - Misc**：https://ctf-wiki.org/misc/archive/

### 相关工具
- **Foremost**：https://github.com/korczis/foremost（文件恢复）
- **Scalpel**：https://github.com/sleuthkit/scalpel（数据雕刻）
- **Firmware Mod Kit (FMK)**：https://github.com/rampageX/firmware-mod-kit
- **Jefferson**：https://github.com/sviehb/jefferson（JFFS2 提取）

---

## 文件签名速查表

```python
# 常用文件签名（Python 字典）
FILE_SIGNATURES = {
    'JPEG': b'\xFF\xD8\xFF',
    'PNG': b'\x89\x50\x4E\x47',
    'GIF': b'\x47\x49\x46\x38',
    'ZIP': b'\x50\x4B\x03\x04',
    'GZIP': b'\x1F\x8B\x08',
    'BZIP2': b'\x42\x5A\x68',
    '7Z': b'\x37\x7A\xBC\xAF\x27\x1C',
    'RAR': b'\x52\x61\x72\x21',
    'ELF': b'\x7F\x45\x4C\x46',
    'PDF': b'\x25\x50\x44\x46',
    'MP3': b'\xFF\xFB',
    'MP4': b'\x00\x00\x00\x18\x66\x74\x79\x70',
    'Squashfs': b'\x68\x73\x71\x73',
}
```

---

**文档版本**：v1.0
**更新日期**：2025-01
**适用版本**：Binwalk v2.3.0 - v2.4.0+
