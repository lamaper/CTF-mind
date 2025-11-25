# pngcheck 使用手册（PNG文件完整性验证）

> 免责声明：本手册仅用于合法的文件分析、CTF 竞赛和授权的图像取证。

---

## 目录
1. 概述
2. 安装方法
3. 基本命令结构
4. 核心功能详解
5. 常用命令与示例
6. PNG 文件结构详解
7. CTF 常见场景
8. 实战案例
9. 高级技巧
10. 常见问题与排查
11. 参考链接

---

## 1. 概述

pngcheck 是由 Greg Roelofs 开发的 PNG 文件验证工具，专门用于检测 PNG、JNG、MNG 文件的完整性和合法性。它能够：
- **验证文件结构**：检查 PNG 文件格式是否符合标准
- **检测损坏块**：识别损坏或非法的数据块（Chunk）
- **提取元数据**：显示所有数据块的详细信息
- **隐写检测**：发现隐藏或附加的数据

**适用场景**：
- CTF 图片隐写分析（检测异常数据块）
- PNG 文件损坏修复（定位错误位置）
- 图像取证（验证文件完整性）
- 开发调试（验证 PNG 生成器输出）

**版本信息**：本手册基于 pngcheck v3.0.3

---

## 2. 安装方法

### macOS 安装
```bash
# 使用 Homebrew 安装
brew install pngcheck

# 验证安装
pngcheck -h
```

### Kali Linux / Debian / Ubuntu
```bash
# 使用 APT 安装
sudo apt update
sudo apt install pngcheck

# 验证安装
pngcheck -h
```

### 从源码编译
```bash
# 下载源码
wget http://www.libpng.org/pub/png/src/pngcheck-3.0.3.tar.gz

# 解压编译
tar -xzf pngcheck-3.0.3.tar.gz
cd pngcheck-3.0.3
make
sudo make install

# 验证
pngcheck -h
```

---

## 3. 基本命令结构

```bash
pngcheck [选项] [PNG文件]

# 基本检查
pngcheck image.png

# 详细输出
pngcheck -v image.png

# 强制详细输出（即使文件有错）
pngcheck -vv image.png

# 检查多个文件
pngcheck *.png
```

### 核心选项说明

**输出选项**：
- `-v` - 详细输出（显示所有数据块）
- `-vv` - 超详细输出（包括错误文件的详细信息）
- `-q` - 安静模式（仅显示错误和警告）
- `-t` - 测试模式（仅输出 OK 或错误）
- `-7` - 7-bit ASCII 模式（用于纯文本输出）

**检查选项**：
- `-c` - 启用颜色类型和位深度检查
- `-p` - 部分检查（检查前几个数据块）
- `-s` - 搜索 PNG 签名（用于恢复嵌入文件）
- `-f` - 强制继续（即使遇到错误）

**格式选项**：
- `-x` - 十六进制输出（显示原始数据）
- `-e` - 提取文本块（tEXt、zTXt、iTXt）

---

## 4. 核心功能详解

### 4.1 PNG 文件结构

**PNG 文件组成**：
```
1. PNG 签名（8 字节）：89 50 4E 47 0D 0A 1A 0A
2. 数据块（Chunks）：
   - 关键数据块（Critical Chunks）：必须支持
     * IHDR - 图像头部（Image Header）
     * PLTE - 调色板（Palette）
     * IDAT - 图像数据（Image Data）
     * IEND - 图像结束（Image End）

   - 辅助数据块（Ancillary Chunks）：可选
     * tRNS - 透明度（Transparency）
     * gAMA - 伽马值（Gamma）
     * tEXt - 文本信息（Text）
     * tIME - 修改时间（Time）
     * ...
```

**数据块结构**：
```
+--------+--------+--------+--------+
| Length (4 bytes) - 数据长度      |
+--------+--------+--------+--------+
| Type   (4 bytes) - 块类型        |
+--------+--------+--------+--------+
| Data   (Length bytes) - 数据     |
+--------+--------+--------+--------+
| CRC    (4 bytes) - 校验和        |
+--------+--------+--------+--------+
```

### 4.2 关键数据块详解

**IHDR（Image Header）- 必须是第一个数据块**：
```
Width:      4 bytes  - 图像宽度
Height:     4 bytes  - 图像高度
Bit depth:  1 byte   - 位深度（1,2,4,8,16）
Color type: 1 byte   - 颜色类型（0,2,3,4,6）
Compression: 1 byte  - 压缩方法（必须为0）
Filter:     1 byte   - 滤波方法（必须为0）
Interlace:  1 byte   - 隔行扫描（0=无,1=Adam7）
```

**颜色类型**：
- 0 = Grayscale（灰度）
- 2 = Truecolor（RGB）
- 3 = Indexed（索引颜色，需要 PLTE）
- 4 = Grayscale + Alpha
- 6 = Truecolor + Alpha（RGBA）

---

## 5. 常用命令与示例

### 5.1 基本检查

**快速检查**：
```bash
pngcheck image.png
```

**输出示例（正常文件）**：
```
OK: image.png (1920x1080, 24-bit RGB, non-interlaced, 95.2%).
```

**输出示例（有问题文件）**：
```
image.png  CRC error in chunk IDAT (computed 3a2b1c4d, expected 5e6f7a8b)
ERROR: image.png
```

### 5.2 详细输出

**查看所有数据块**：
```bash
pngcheck -v image.png
```

**输出示例**：
```
File: image.png (524288 bytes)
  chunk IHDR at offset 0x0000c, length 13
    1920 x 1080 image, 24-bit RGB, non-interlaced
  chunk gAMA at offset 0x00025, length 4: 0.45455
  chunk cHRM at offset 0x00035, length 32
    White x = 0.3127 y = 0.329,  Red x = 0.64 y = 0.33
    Green x = 0.3 y = 0.6,  Blue x = 0.15 y = 0.06
  chunk IDAT at offset 0x00061, length 65536
  chunk IDAT at offset 0x10069, length 65536
  chunk IDAT at offset 0x20071, length 45678
  chunk IEND at offset 0x2b32f, length 0
No errors detected in image.png (8 chunks, 95.2% compression).
```

**超详细输出（即使有错误）**：
```bash
pngcheck -vv broken.png
```

### 5.3 批量检查

**检查目录下所有 PNG**：
```bash
pngcheck *.png
```

**递归检查**：
```bash
find . -name "*.png" -exec pngcheck {} \;
```

**仅显示错误文件**：
```bash
pngcheck *.png | grep ERROR
```

**统计正常和错误文件**：
```bash
#!/bin/bash
ok=0
error=0

for file in *.png; do
  if pngcheck -q "$file" > /dev/null 2>&1; then
    ((ok++))
  else
    ((error++))
    echo "Error: $file"
  fi
done

echo "OK: $ok, Errors: $error"
```

### 5.4 搜索嵌入的 PNG

**场景**：文件中可能嵌入了多个 PNG

```bash
# 搜索 PNG 签名
pngcheck -s file.bin

# 输出可能的 PNG 起始位置
```

### 5.5 提取文本块

**提取 tEXt/zTXt/iTXt 数据块**：
```bash
pngcheck -t image.png
```

**输出示例**：
```
File: image.png
  chunk tEXt at offset 0x00025, length 42
    keyword: Comment
    text: Created with GIMP
  chunk tEXt at offset 0x00055, length 25
    keyword: Author
    text: John Doe
```

---

## 6. PNG 文件结构详解

### 6.1 PNG 签名（8 字节）

```
Hex:  89 50 4E 47 0D 0A 1A 0A
Dec:  137 80 78 71 13 10 26 10

含义：
- 89: 非 ASCII，防止文本传输损坏
- 50 4E 47: "PNG" ASCII 字符
- 0D 0A: DOS 行结束符（CRLF）
- 1A: DOS 文件结束符
- 0A: Unix 行结束符（LF）
```

**验证签名**：
```bash
# 使用 xxd 查看前 8 字节
xxd -l 8 image.png

# 使用 file 命令
file image.png

# 使用 pngcheck
pngcheck -v image.png | head -n 1
```

### 6.2 关键数据块类型

| 块类型 | 必需 | 说明 |
|--------|-----|------|
| IHDR | ✓ | 图像头部，必须是第一个 |
| PLTE | * | 调色板（索引色必需） |
| IDAT | ✓ | 图像数据（可多个） |
| IEND | ✓ | 图像结束，必须是最后一个 |

### 6.3 辅助数据块类型

| 块类型 | 说明 | 内容 |
|--------|-----|------|
| bKGD | 背景颜色 | 默认背景色 |
| cHRM | 色度 | 白点和主要色度 |
| gAMA | 伽马值 | 图像伽马 |
| hIST | 直方图 | 调色板直方图 |
| iCCP | ICC 配置文件 | 嵌入的 ICC |
| pHYs | 物理像素尺寸 | DPI 信息 |
| sBIT | 有效位数 | 原始位深度 |
| sPLT | 建议调色板 | 建议颜色 |
| sRGB | 标准 RGB | sRGB 渲染意图 |
| tEXt | 文本 | 未压缩文本 |
| tIME | 修改时间 | 最后修改时间 |
| tRNS | 透明度 | 透明颜色 |
| zTXt | 压缩文本 | 压缩的文本 |

---

## 7. CTF 常见场景

### 场景1：PNG 高度/宽度被修改

**题目特征**：PNG 显示不完整或无法打开

```bash
# 1. 检查文件
pngcheck -v challenge.png

# 输出可能显示：
# IHDR CRC error
# 或：图像尺寸异常

# 2. 查看 IHDR 数据块
xxd -l 33 challenge.png

# 3. 修复 IHDR（手动或使用工具）
# 宽度：字节 16-19
# 高度：字节 20-23

# 4. 重新计算 CRC（使用 Python）
python3 << 'EOF'
import zlib
import struct

with open('challenge.png', 'rb') as f:
    data = f.read()

# IHDR 数据（跳过长度和类型）
ihdr_data = data[12:29]  # Type + Data

# 计算新的 CRC
crc = zlib.crc32(ihdr_data)
print(f"Correct CRC: {hex(crc)}")
EOF
```

### 场景2：IEND 后隐藏数据

**题目特征**：文件大小异常，pngcheck 显示有额外数据

```bash
# 1. 检查文件
pngcheck -v challenge.png

# 输出可能显示：
# chunk IEND at offset 0x12345, length 0
# Additional data after IEND

# 2. 查找 IEND 位置
xxd challenge.png | grep "49 45 4E 44"
# IEND 的十六进制：49 45 4E 44 AE 42 60 82

# 3. 提取 IEND 后的数据
IEND_OFFSET=74789  # 从 xxd 输出获取
IEND_SIZE=12       # IEND 块总大小（4+4+0+4）

dd if=challenge.png of=hidden_data.bin bs=1 skip=$((IEND_OFFSET + IEND_SIZE))

# 4. 分析隐藏数据
file hidden_data.bin
strings hidden_data.bin
xxd hidden_data.bin | head
```

### 场景3：自定义数据块隐写

**题目特征**：pngcheck 显示未知数据块

```bash
# 1. 详细检查
pngcheck -vv challenge.png

# 输出可能显示：
# chunk fLAg at offset 0x5678, length 48 (unknown)

# 2. 提取自定义数据块
# 使用 Python 脚本
python3 << 'EOF'
import struct

with open('challenge.png', 'rb') as f:
    data = f.read()

# 查找自定义块（例如 "fLAg"）
pos = data.find(b'fLAg')
if pos > 0:
    # 读取长度（前4字节）
    length = struct.unpack('>I', data[pos-4:pos])[0]
    # 提取数据
    chunk_data = data[pos+4:pos+4+length]
    print(chunk_data.decode('utf-8', errors='ignore'))
EOF
```

### 场景4：CRC 校验失败

**题目特征**：pngcheck 报告 CRC 错误

```bash
# 1. 检查错误
pngcheck -v challenge.png

# 输出：
# CRC error in chunk IDAT (computed 12345678, expected 87654321)

# 2. 两种可能：
#    a) 数据被修改（需要恢复）
#    b) CRC 被故意修改（隐藏信息）

# 3. 提取预期的 CRC 值（可能是 flag 的一部分）
xxd challenge.png | grep -A 10 "49 44 41 54"  # IDAT

# 4. 或使用 Python 提取
python3 << 'EOF'
with open('challenge.png', 'rb') as f:
    data = f.read()

# 查找 IDAT
pos = data.find(b'IDAT')
while pos > 0:
    length = struct.unpack('>I', data[pos-4:pos])[0]
    crc = struct.unpack('>I', data[pos+4+length:pos+8+length])[0]
    print(f"IDAT at {pos}, CRC: {hex(crc)}")
    pos = data.find(b'IDAT', pos+1)
EOF
```

---

## 8. 实战案例

### 案例1：修复损坏的 PNG 高度

**问题**：PNG 图片只显示上半部分

```bash
# 1. 检查文件
pngcheck -v challenge.png
# 输出：1920 x 540 image

# 2. 怀疑高度被修改，尝试恢复
# 使用十六进制编辑器或脚本

# 3. 查看 IHDR
xxd -l 33 challenge.png
# 00000000: 8950 4e47 0d0a 1a0a 0000 000d 4948 4452  .PNG........IHDR
# 00000010: 0000 0780 0000 021c 0806 0000 00xx xxxx  ................

# 宽度：0x0780 = 1920 ✓
# 高度：0x021c = 540   可能应该是 1080 (0x0438)

# 4. 修复高度（使用Python）
python3 << 'EOF'
import zlib
import struct

with open('challenge.png', 'rb') as f:
    data = bytearray(f.read())

# 修改高度为 1080 (0x0438)
data[20:24] = struct.pack('>I', 1080)

# 重新计算 IHDR CRC
ihdr_data = data[12:29]
crc = zlib.crc32(ihdr_data)
data[29:33] = struct.pack('>I', crc)

with open('fixed.png', 'wb') as f:
    f.write(data)

print("Fixed! New height: 1080")
EOF

# 5. 验证修复
pngcheck fixed.png
```

### 案例2：提取 IEND 后的隐藏 ZIP

```bash
# 1. 检查文件
pngcheck -v hidden.png
# 输出显示：Additional data after IEND

# 2. 查找 IEND 位置
grep -abo "IEND" hidden.png
# 输出：74789:IEND

# 3. 提取后续数据
tail -c +74802 hidden.png > hidden.zip
# 74802 = 74789 + 12 + 1 (IEND块长度+偏移调整)

# 4. 验证提取的文件
file hidden.zip
# 输出：Zip archive data

# 5. 解压
unzip hidden.zip
```

### 案例3：批量检测隐写文件

```bash
#!/bin/bash
# 批量检测 PNG 隐写脚本

echo "=== PNG Steganography Scanner ==="
echo ""

for file in *.png; do
  echo "Checking $file..."

  # 基本检查
  result=$(pngcheck -v "$file" 2>&1)

  # 检查1：IEND 后有数据
  if echo "$result" | grep -q "additional data after IEND"; then
    echo "  [!] Found data after IEND"
  fi

  # 检查2：未知数据块
  if echo "$result" | grep -q "unknown"; then
    echo "  [!] Found unknown chunks"
    echo "$result" | grep "unknown"
  fi

  # 检查3：CRC 错误
  if echo "$result" | grep -q "CRC error"; then
    echo "  [!] CRC error detected"
  fi

  # 检查4：文件大小异常
  size=$(wc -c < "$file")
  if [ $size -gt 10000000 ]; then  # >10MB
    echo "  [!] Unusually large file: $size bytes"
  fi

  echo ""
done
```

---

## 9. 高级技巧

### 9.1 结合其他工具

**与 Binwalk 结合**：
```bash
# 1. pngcheck 检测异常
pngcheck -v image.png

# 2. Binwalk 提取嵌入文件
binwalk -e image.png

# 3. 对比结果
```

**与 ExifTool 结合**：
```bash
# 1. pngcheck 检查结构
pngcheck -v image.png

# 2. ExifTool 查看元数据
exiftool image.png

# 3. 综合分析
```

### 9.2 自动化修复脚本

**修复常见 PNG 错误**：
```python
#!/usr/bin/env python3
import zlib
import struct
import sys

def fix_png_crc(filename):
    with open(filename, 'rb') as f:
        data = bytearray(f.read())

    # 验证 PNG 签名
    if data[:8] != b'\x89PNG\r\n\x1a\n':
        print("Not a PNG file!")
        return False

    pos = 8
    fixed = False

    while pos < len(data):
        # 读取数据块
        if pos + 12 > len(data):
            break

        length = struct.unpack('>I', data[pos:pos+4])[0]
        chunk_type = data[pos+4:pos+8]
        chunk_data = data[pos+8:pos+8+length]
        stored_crc = struct.unpack('>I', data[pos+8+length:pos+12+length])[0]

        # 计算正确的 CRC
        calculated_crc = zlib.crc32(chunk_type + chunk_data)

        if calculated_crc != stored_crc:
            print(f"Fixing CRC for {chunk_type.decode('latin1')} chunk")
            data[pos+8+length:pos+12+length] = struct.pack('>I', calculated_crc)
            fixed = True

        pos += 12 + length

        if chunk_type == b'IEND':
            break

    if fixed:
        with open(filename + '.fixed.png', 'wb') as f:
            f.write(data)
        print(f"Saved fixed file as {filename}.fixed.png")

    return fixed

if __name__ == '__main__':
    if len(sys.argv) < 2:
        print(f"Usage: {sys.argv[0]} <png_file>")
        sys.exit(1)

    fix_png_crc(sys.argv[1])
```

---

## 10. 常见问题与排查

### 问题1：CRC error in chunk IHDR

**原因**：图像头部数据被修改

**解决方案**：
```bash
# 重新计算 CRC
python3 -c "
import zlib, struct
data = open('image.png', 'rb').read()
ihdr = data[12:29]
crc = zlib.crc32(ihdr)
print(f'Correct CRC: {hex(crc)}')
print(f'Current CRC: {hex(struct.unpack(\">I\", data[29:33])[0])}')
"
```

### 问题2：Additional data after IEND

**原因**：IEND 后有附加数据（可能是隐写）

**解决方案**：提取 IEND 后的数据进行分析

---

## 11. 参考链接

- **官网**：http://www.libpng.org/pub/png/apps/pngcheck.html
- **PNG 规范**：https://www.w3.org/TR/PNG/
- **CTF Wiki**：https://ctf-wiki.org/misc/picture/png/

---

**文档版本**：v1.0
**更新日期**：2025-01
**适用版本**：pngcheck v3.0.3+
