# zsteg 使用手册（PNG/BMP隐写分析）

> 免责声明：本手册仅用于合法的隐写分析、CTF 竞赛和授权的图像取证。

---

## 目录
1. 概述
2. 安装方法
3. 基本命令结构
4. 核心功能详解
5. 常用命令与示例
6. 隐写技术详解
7. CTF 常见场景
8. 实战案例
9. 高级技巧
10. 常见问题与排查
11. 参考链接

---

## 1. 概述

zsteg 是一个专门用于检测 PNG 和 BMP 图像中 LSB（Least Significant Bit）隐写的 Ruby 工具，由 zed-0xff 开发。它能够：
- **LSB 隐写检测**：自动检测最低有效位隐写
- **多种编码支持**：支持各种位序和字节序
- **元数据提取**：提取隐藏的文本和数据
- **多种载体**：RGB 通道、Alpha 通道、调色板

**适用场景**：
- CTF 图片隐写题目（最常用工具之一）
- LSB 隐写分析和检测
- PNG/BMP 图像取证
- 隐写术研究

**版本信息**：本手册基于 zsteg v0.2.x

---

## 2. 安装方法

### macOS / Linux 安装
```bash
# 安装 Ruby（如果未安装）
# macOS 自带 Ruby
ruby -v

# 安装 zsteg
gem install zsteg

# 验证安装
zsteg --help
```

### 权限问题解决
```bash
# 如果遇到权限错误
gem install --user-install zsteg

# 添加到 PATH（macOS）
echo 'export PATH="$HOME/.gem/ruby/2.6.0/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc

# 验证
which zsteg
```

### Kali Linux
```bash
# Kali 通常已包含
zsteg --help

# 如需安装
sudo gem install zsteg
```

---

## 3. 基本命令结构

```bash
zsteg [选项] [图像文件]

# 基本扫描
zsteg image.png

# 显示所有结果
zsteg -a image.png

# 提取特定载体的数据
zsteg -E "b1,rgb,lsb,xy" image.png > output.txt
```

### 核心选项说明

**扫描选项**：
- `-a` / `--all` - 显示所有检测结果（包括负面结果）
- `-v` / `--verbose` - 详细输出
- `-q` / `--quiet` - 安静模式（仅显示发现的数据）

**提取选项**：
- `-E <format>` - 提取指定格式的数据
- `-s <size>` - 最小有效数据大小（字节）
- `-l <limit>` - 输出限制（字节）

**格式选项**：
- `-b <bits>` - 指定位数（1,2,4,8）
- `-o <order>` - 指定字节序（xy, yx, XY, YX）
- `-P <prime>` - 使用素数间隔

---

## 4. 核心功能详解

### 4.1 LSB 隐写原理

**最低有效位（Least Significant Bit）隐写**：

```
原始像素：
R: 11010110 (214)
G: 10101100 (172)
B: 01110011 (115)

隐藏数据："A" (01000001)

修改后像素（每个通道1bit）：
R: 11010110 → 11010110 (0, 无变化)
G: 10101100 → 10101101 (1)
B: 01110011 → 01110010 (0)

视觉变化：几乎不可见（每个通道最多±1）
```

**容量计算**：
```
图像大小：1920x1080 像素
RGB 图像：3 个通道
LSB-1：(1920 * 1080 * 3 * 1) / 8 = 777,600 字节 ≈ 760 KB
LSB-2：(1920 * 1080 * 3 * 2) / 8 = 1,555,200 字节 ≈ 1.5 MB
```

### 4.2 支持的载体类型

**PNG 载体**：
- RGB 通道（R、G、B）
- Alpha 通道
- 灰度通道
- 调色板索引

**BMP 载体**：
- RGB 通道
- 调色板（索引颜色）

### 4.3 检测方法

zsteg 会尝试多种组合：
- **位数**：1, 2, 4, 8 bits
- **通道**：R, G, B, RGB, RGBA, Gray
- **顺序**：LSB first, MSB first
- **方向**：xy（行优先）, yx（列优先）

---

## 5. 常用命令与示例

### 5.1 基本扫描

**快速扫描**：
```bash
zsteg image.png
```

**输出示例**：
```
imagedata           .. text: "Hello, this is hidden text!"
b1,r,lsb,xy         .. text: "flag{1sb_st3g0}"
b1,g,msb,xy         .. file: PGP RSA encrypted session key
b1,bgr,lsb,xy       .. text: "Another message"
```

**解读输出**：
- `b1,r,lsb,xy` - 1位，红色通道，LSB，行优先
- `b1,g,msb,xy` - 1位，绿色通道，MSB，行优先
- `b1,bgr,lsb,xy` - 1位，BGR顺序，LSB，行优先

### 5.2 显示所有结果

**包括无意义的结果**：
```bash
zsteg -a image.png
```

**详细输出**：
```bash
zsteg -v image.png
```

### 5.3 提取特定数据

**提取文本**：
```bash
# 提取 b1,r,lsb,xy 的数据
zsteg -E "b1,r,lsb,xy" image.png

# 保存到文件
zsteg -E "b1,r,lsb,xy" image.png > hidden.txt
```

**提取二进制文件**：
```bash
# 提取隐藏的 ZIP
zsteg -E "b1,rgb,lsb,xy" image.png > hidden.zip

# 验证文件类型
file hidden.zip
```

### 5.4 限制输出

**设置最小有效数据大小**：
```bash
# 只显示 ≥100 字节的结果
zsteg -s 100 image.png
```

**限制输出长度**：
```bash
# 只显示前 200 字节
zsteg -l 200 image.png
```

### 5.5 指定扫描参数

**仅扫描 LSB-1**：
```bash
zsteg -b 1 image.png
```

**仅扫描特定通道**：
```bash
zsteg -E "b1,r,lsb,xy" image.png  # 红色通道
zsteg -E "b1,g,lsb,xy" image.png  # 绿色通道
zsteg -E "b1,b,lsb,xy" image.png  # 蓝色通道
```

---

## 6. 隐写技术详解

### 6.1 LSB 隐写变体

**按位数分类**：
- **LSB-1**：最低1位（最常用，容量小，隐蔽性高）
- **LSB-2**：最低2位（容量翻倍，但可能可见）
- **LSB-4**：最低4位（容量大，但明显影响图像）

**按通道分类**：
- **单通道**：仅使用 R、G 或 B
- **多通道**：RGB 组合，容量最大
- **Alpha 通道**：PNG 特有，隐蔽性极高

**按顺序分类**：
- **LSB First**：从最低位开始（常见）
- **MSB First**：从最高位开始（较少见）

### 6.2 检测模式

zsteg 使用的检测模式：
```
b<bits>,<channels>,<lsb|msb>,<xy|yx>

参数说明：
- bits: 1,2,4,8
- channels: r, g, b, rgb, bgr, rgba, gray
- lsb/msb: 最低位优先/最高位优先
- xy/yx: 行优先/列优先
```

**示例**：
```
b1,r,lsb,xy    - 1位，红色，LSB，行优先
b2,rgb,lsb,xy  - 2位，RGB，LSB，行优先
b1,g,msb,yx    - 1位，绿色，MSB，列优先
```

---

## 7. CTF 常见场景

### 场景1：纯文本隐写

**题目特征**：提供 PNG 图片，提示"有秘密"

```bash
# 1. 基本扫描
zsteg image.png

# 输出示例：
# b1,r,lsb,xy .. text: "flag{z5t3g_1s_c00l}"

# 2. 直接提取
zsteg -E "b1,r,lsb,xy" image.png
```

### 场景2：隐藏文件提取

**题目特征**：图片中嵌入了 ZIP 或其他文件

```bash
# 1. 扫描
zsteg image.png

# 输出示例：
# b1,bgr,lsb,xy .. file: Zip archive data

# 2. 提取 ZIP
zsteg -E "b1,bgr,lsb,xy" image.png > hidden.zip

# 3. 解压
unzip hidden.zip
```

### 场景3：多层隐写

**题目特征**：需要尝试不同的通道和位数

```bash
# 1. 查看所有结果
zsteg -a image.png

# 2. 逐一尝试可疑的载体
zsteg -E "b1,r,lsb,xy" image.png > try1.txt
zsteg -E "b1,g,lsb,xy" image.png > try2.txt
zsteg -E "b1,b,lsb,xy" image.png > try3.txt
zsteg -E "b2,rgb,lsb,xy" image.png > try4.txt

# 3. 检查每个文件
for f in try*.txt; do
  echo "=== $f ==="
  file $f
  head $f
done
```

### 场景4：BMP 隐写

**题目特征**：提供 BMP 文件

```bash
# 1. BMP 扫描（与 PNG 相同）
zsteg image.bmp

# 2. 提取数据
zsteg -E "b1,rgb,lsb,xy" image.bmp > hidden.txt
```

---

## 8. 实战案例

### 案例1：CTF Misc - LSB 文本隐写

**题目**：提供 `secret.png`，描述"Look deeper"

```bash
# 1. 基本扫描
zsteg secret.png

# 输出：
# imagedata           .. text: "\n\n\n\n\n"
# b1,r,lsb,xy         .. text: "The flag is: flag{l5b_m4st3r}"
# b1,g,msb,xy         .. text: "................"
# b1,bgr,lsb,xy       .. text: "DJJDJDJDJD"

# 2. 发现关键信息：b1,r,lsb,xy
# 3. 提取完整数据
zsteg -E "b1,r,lsb,xy" secret.png

# 输出完整 flag
```

### 案例2：隐藏 ZIP 文件

**题目**：`challenge.png` 中隐藏了文件

```bash
# 1. 扫描
zsteg challenge.png

# 输出：
# b1,rgb,lsb,xy       .. file: Zip archive data, at least v2.0

# 2. 提取 ZIP
zsteg -E "b1,rgb,lsb,xy" challenge.png > hidden.zip

# 3. 验证文件
file hidden.zip
# Output: Zip archive data, at least v2.0 to extract

# 4. 解压
unzip hidden.zip

# 5. 查看内容
cat flag.txt
```

### 案例3：多通道组合隐写

**场景**：需要组合多个通道的数据

```bash
# 1. 扫描发现线索
zsteg image.png

# 输出：
# b1,r,lsb,xy         .. text: "Part1: flag{"
# b1,g,lsb,xy         .. text: "Part2: m4ny"
# b1,b,lsb,xy         .. text: "Part3: _ch4nn3ls}"

# 2. 分别提取
zsteg -E "b1,r,lsb,xy" image.png > part1.txt
zsteg -E "b1,g,lsb,xy" image.png > part2.txt
zsteg -E "b1,b,lsb,xy" image.png > part3.txt

# 3. 合并
cat part1.txt part2.txt part3.txt
# 输出：flag{m4ny_ch4nn3ls}
```

### 案例4：MSB 隐写（非常见）

**场景**：LSB 没有发现，尝试 MSB

```bash
# 1. LSB 扫描无果
zsteg image.png | grep -i "flag"
# 无结果

# 2. 查看所有结果（包括 MSB）
zsteg -a image.png | grep msb

# 输出：
# b1,r,msb,xy         .. text: "flag{m5b_h1dd3n}"

# 3. 提取 MSB 数据
zsteg -E "b1,r,msb,xy" image.png
```

---

## 9. 高级技巧

### 9.1 自动化脚本

**批量扫描所有图片**：
```bash
#!/bin/bash
# zsteg 批量扫描脚本

for img in *.png *.bmp; do
  [ -f "$img" ] || continue

  echo "=== Analyzing $img ==="
  result=$(zsteg -a "$img" 2>&1)

  # 搜索 flag
  if echo "$result" | grep -qi "flag{"; then
    echo "[!] Possible flag found in $img!"
    echo "$result" | grep -i "flag{"
  fi

  # 搜索文件
  if echo "$result" | grep -qi "file:"; then
    echo "[!] Hidden file detected in $img!"
    echo "$result" | grep "file:"
  fi

  echo ""
done
```

### 9.2 结合其他工具

**工作流程**：
```bash
# 1. Binwalk 检查嵌入文件
binwalk image.png

# 2. ExifTool 查看元数据
exiftool image.png

# 3. pngcheck 检查结构
pngcheck -v image.png

# 4. zsteg 检测 LSB 隐写
zsteg image.png

# 5. stegsolve（GUI 工具）手动分析
# 下载：https://github.com/eugenekolo/sec-tools/tree/master/stego/stegsolve
```

### 9.3 提取所有可能的数据

```bash
#!/bin/bash
# 提取所有可能的载体数据

formats=(
  "b1,r,lsb,xy"
  "b1,g,lsb,xy"
  "b1,b,lsb,xy"
  "b1,rgb,lsb,xy"
  "b1,bgr,lsb,xy"
  "b1,rgba,lsb,xy"
  "b2,rgb,lsb,xy"
  "b1,r,msb,xy"
  "b1,g,msb,xy"
  "b1,b,msb,xy"
)

mkdir -p extracted

for format in "${formats[@]}"; do
  safe_name=$(echo "$format" | tr ',' '_')
  echo "Extracting $format..."
  zsteg -E "$format" "$1" > "extracted/${safe_name}.bin" 2>/dev/null

  # 检查文件类型
  if [ -s "extracted/${safe_name}.bin" ]; then
    file_type=$(file -b "extracted/${safe_name}.bin")
    echo "  -> $file_type"
  fi
done

echo "Extraction complete! Check extracted/ directory"
```

### 9.4 图像比较分析

**对比原图和隐写图**：
```bash
# 如果有原图和隐写图
# 计算差异

# 方法1：使用 ImageMagick
compare -metric AE original.png stego.png diff.png
# 输出不同像素数量

# 方法2：提取差异
convert original.png stego.png -compose difference -composite diff.png

# 方法3：使用 Python
python3 << 'EOF'
from PIL import Image
import numpy as np

orig = np.array(Image.open('original.png'))
stego = np.array(Image.open('stego.png'))

diff = np.abs(orig.astype(int) - stego.astype(int))
print(f"Max difference: {diff.max()}")
print(f"Affected pixels: {(diff > 0).sum()}")
EOF
```

---

## 10. 常见问题与排查

### 问题1：gem install 失败

**错误**：`Permission denied`

**解决方案**：
```bash
# 使用用户安装
gem install --user-install zsteg

# 或使用 sudo
sudo gem install zsteg
```

### 问题2：未发现任何数据

**排查步骤**：
```bash
# 1. 确认是 PNG 或 BMP
file image.png

# 2. 查看所有结果（包括负面）
zsteg -a image.png

# 3. 尝试其他工具
binwalk image.png
strings image.png | grep -i flag
exiftool image.png

# 4. 使用 Stegsolve（GUI）手动查看
```

### 问题3：提取的数据损坏

**原因**：可能选错了载体格式

**解决方案**：
```bash
# 1. 查看所有可能的载体
zsteg image.png

# 2. 逐一尝试
zsteg -E "b1,r,lsb,xy" image.png > try1.bin
zsteg -E "b1,g,lsb,xy" image.png > try2.bin
zsteg -E "b1,b,lsb,xy" image.png > try3.bin
zsteg -E "b1,rgb,lsb,xy" image.png > try4.bin

# 3. 检查每个文件
for f in try*.bin; do
  file $f
done
```

---

## 11. 参考链接

### 官方资源
- **zsteg GitHub**：https://github.com/zed-0xff/zsteg
- **Ruby Gems**：https://rubygems.org/gems/zsteg

### 隐写术资源
- **Stegsolve**：https://github.com/eugenekolo/sec-tools/tree/master/stego/stegsolve
- **StegExpose**：https://github.com/b3dk7/StegExpose
- **OpenStego**：https://www.openstego.com/

### 教程和文章
- **CTF Wiki - Steganography**：https://ctf-wiki.org/misc/picture/png/
- **LSB 隐写原理**：https://en.wikipedia.org/wiki/Bit_numbering#Least_significant_bit

### 相关工具
- **StegSolve（GUI）**：https://github.com/eugenekolo/sec-tools
- **Stegpy（Python）**：https://github.com/dhsdshdhk/stegpy
- **StegCracker**：https://github.com/Paradoxis/StegCracker

---

## 快速参考

### 最常用命令
```bash
# 快速扫描
zsteg image.png

# 显示所有结果
zsteg -a image.png

# 提取文本
zsteg -E "b1,r,lsb,xy" image.png

# 提取文件
zsteg -E "b1,rgb,lsb,xy" image.png > hidden.zip

# 限制输出
zsteg -s 100 image.png  # 只显示 ≥100 字节
```

### CTF 快速检查
```bash
# 搜索 flag
zsteg image.png | grep -i "flag"

# 查看文件类型
zsteg image.png | grep "file:"

# 批量扫描
for img in *.png; do echo "=== $img ==="; zsteg "$img" | grep -i "flag\|file"; done
```

---

**文档版本**：v1.0
**更新日期**：2025-01
**适用版本**：zsteg v0.2.x+
