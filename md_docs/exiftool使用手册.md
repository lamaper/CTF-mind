# ExifTool 使用手册（元数据分析与处理）

> 免责声明：本手册仅用于合法的数字取证、CTF 竞赛和授权的元数据分析。尊重他人隐私，不要将工具用于非法目的。

---

## 目录
1. 概述
2. 安装方法
3. 基本命令结构
4. 核心功能详解
5. 常用命令与示例
6. 元数据类型详解
7. CTF 常见场景
8. 实战案例
9. 高级技巧
10. 常见问题与排查
11. 隐私与安全
12. 参考链接

---

## 1. 概述

ExifTool 是由 Phil Harvey 开发的强大元数据读写工具，支持 **200+ 种文件格式**。它能够：
- **读取元数据**：EXIF、IPTC、XMP、GPS、Maker Notes 等
- **修改元数据**：编辑、删除或创建元数据标签
- **格式转换**：在不同元数据格式间转换
- **批量处理**：对整个目录进行批量操作

**适用场景**：
- CTF 隐写分析（从图片元数据中找 flag）
- 数字取证（照片来源、拍摄设备、地理位置）
- 隐私保护（清除敏感元数据）
- 媒体管理（批量重命名、标签整理）

**支持格式**：JPEG、PNG、GIF、TIFF、RAW（CR2/NEF/ARW）、PDF、MP4、AVI、DOCX、MP3、AAC、FLAC 等

**版本信息**：本手册基于 ExifTool v12.0+

---

## 2. 安装方法

### macOS 安装
```bash
# 使用 Homebrew 安装（推荐）
brew install exiftool

# 验证安装
exiftool -ver
```

### Kali Linux / Debian / Ubuntu
```bash
# 使用 APT 安装
sudo apt update
sudo apt install libimage-exiftool-perl

# 或使用包管理器名称
sudo apt install exiftool

# 验证安装
exiftool -ver
```

### 从源码安装（最新版本）
```bash
# 下载最新版本
wget https://exiftool.org/Image-ExifTool-12.70.tar.gz

# 解压安装
tar -xzf Image-ExifTool-12.70.tar.gz
cd Image-ExifTool-12.70
perl Makefile.PL
make
sudo make install

# 验证
exiftool -ver
```

---

## 3. 基本命令结构

```bash
exiftool [选项] [文件/目录]

# 查看所有元数据
exiftool image.jpg

# 查看特定标签
exiftool -Make -Model image.jpg

# 修改元数据
exiftool -Artist="John Doe" image.jpg

# 删除所有元数据
exiftool -all= image.jpg
```

### 核心选项说明

**读取选项**：
- `-a` / `--all` - 显示所有重复标签
- `-G` / `--group` - 按组显示标签
- `-s` / `--short` - 使用短标签名
- `-n` / `--numerical` - 显示数值而非描述
- `-j` / `--json` - 以 JSON 格式输出
- `-csv` - 以 CSV 格式输出

**写入选项**：
- `-<tag>=<value>` - 设置标签值
- `-<tag><op=<value>` - 使用操作符（+=、-=、<）
- `-all=` - 删除所有元数据
- `-overwrite_original` - 不保留原始文件备份
- `-P` - 保留文件修改时间

**文件选项**：
- `-r` - 递归处理子目录
- `-ext <ext>` - 仅处理特定扩展名
- `-o <outfile>` - 输出到文件
- `-w <ext>` - 输出为新文件（指定扩展名）

**其他选项**：
- `-v` / `--verbose` - 详细输出
- `-q` / `--quiet` - 安静模式
- `-lang <lang>` - 设置语言（如 zh_cn）

---

## 4. 核心功能详解

### 4.1 支持的元数据类型

**EXIF（Exchangeable Image File Format）**：
- 相机信息：制造商、型号、镜头
- 拍摄参数：ISO、光圈、快门速度、焦距
- 时间信息：拍摄时间、修改时间
- 图像设置：白平衡、闪光灯、曝光模式

**IPTC（International Press Telecommunications Council）**：
- 版权信息：作者、版权声明
- 描述信息：标题、说明、关键词
- 联系信息：创建者、来源

**XMP（Extensible Metadata Platform）**：
- Adobe 软件标签
- 评分、标签、收藏
- 编辑历史

**GPS（地理位置）**：
- 经纬度坐标
- 海拔高度
- 时间戳

**Maker Notes（制造商专有）**：
- 相机特定信息
- 镜头校正数据
- 自定义设置

---

## 5. 常用命令与示例

### 5.1 读取元数据

**查看所有元数据**：
```bash
exiftool image.jpg
```

**输出示例**：
```
ExifTool Version Number         : 12.70
File Name                       : image.jpg
Directory                       : .
File Size                       : 2.5 MB
File Modification Date/Time     : 2025:01:20 10:30:00
File Type                       : JPEG
MIME Type                       : image/jpeg
Image Width                     : 4032
Image Height                    : 3024
Make                            : Apple
Model                           : iPhone 13 Pro
Orientation                     : Horizontal (normal)
X Resolution                    : 72
Y Resolution                    : 72
Resolution Unit                 : inches
Software                        : iOS 17.2
Date/Time Original              : 2025:01:15 14:25:30
Create Date                     : 2025:01:15 14:25:30
GPS Latitude                    : 31 deg 13' 45.00" N
GPS Longitude                   : 121 deg 28' 30.00" E
```

**按组分类显示**：
```bash
exiftool -G image.jpg
```

**输出示例**：
```
[ExifTool]      ExifTool Version Number         : 12.70
[File]          File Name                       : image.jpg
[EXIF]          Make                            : Apple
[EXIF]          Model                           : iPhone 13 Pro
[GPS]           GPS Latitude                    : 31 deg 13' 45.00" N
```

**查看特定标签**：
```bash
# 查看相机型号
exiftool -Make -Model image.jpg

# 查看 GPS 信息
exiftool -GPS* image.jpg

# 查看拍摄时间
exiftool -CreateDate -DateTimeOriginal image.jpg
```

### 5.2 导出格式

**JSON 格式**：
```bash
exiftool -j image.jpg > metadata.json
```

**CSV 格式（批量文件）**：
```bash
exiftool -csv *.jpg > metadata.csv
```

**HTML 格式**：
```bash
exiftool -h image.jpg > metadata.html
```

**纯文本（特定字段）**：
```bash
# 仅输出值（无标签名）
exiftool -s -s -s -Make image.jpg
# 输出：Apple
```

### 5.3 修改元数据

**设置单个标签**：
```bash
# 设置作者
exiftool -Artist="John Doe" image.jpg

# 设置版权
exiftool -Copyright="Copyright 2025 John Doe" image.jpg

# 设置描述
exiftool -ImageDescription="Sunset at beach" image.jpg
```

**批量修改**：
```bash
# 修改所有 JPEG 文件的作者
exiftool -Artist="John Doe" *.jpg

# 递归修改整个目录
exiftool -r -Artist="John Doe" /path/to/photos/
```

**修改日期时间**：
```bash
# 设置拍摄时间
exiftool -DateTimeOriginal="2025:01:20 12:00:00" image.jpg

# 将所有照片时间前移 2 小时
exiftool "-AllDates+=2:0:0 0:0:0" *.jpg
```

**修改 GPS 信息**：
```bash
# 设置经纬度
exiftool -GPSLatitude="31.229167" -GPSLatitudeRef="N" \
         -GPSLongitude="121.475000" -GPSLongitudeRef="E" image.jpg

# 删除 GPS 信息
exiftool -GPS:all= image.jpg
```

### 5.4 删除元数据

**删除所有元数据**：
```bash
exiftool -all= image.jpg
```

**删除特定组**：
```bash
# 仅删除 EXIF
exiftool -EXIF:all= image.jpg

# 仅删除 GPS
exiftool -GPS:all= image.jpg

# 删除评论和缩略图
exiftool -Comment= -ThumbnailImage= image.jpg
```

**批量清理（保护隐私）**：
```bash
# 删除所有照片的 GPS 和作者信息
exiftool -GPS:all= -Artist= -Copyright= *.jpg

# 不保留备份文件
exiftool -overwrite_original -all= *.jpg
```

### 5.5 批量处理

**批量重命名（按拍摄日期）**：
```bash
# 重命名为 YYYYMMDD_HHMMSS.jpg
exiftool '-FileName<CreateDate' -d %Y%m%d_%H%M%S%%+c.%%e *.jpg
```

**按目录整理（按日期分类）**：
```bash
# 根据拍摄年月移动文件
exiftool '-Directory<DateTimeOriginal' -d %Y/%m *.jpg
```

**复制元数据到新文件**：
```bash
# 从 source.jpg 复制元数据到 target.jpg
exiftool -TagsFromFile source.jpg target.jpg
```

---

## 6. 元数据类型详解

### 6.1 EXIF 常用标签

| 标签名 | 说明 | 示例值 |
|--------|------|--------|
| Make | 相机制造商 | Apple, Canon, Nikon |
| Model | 相机型号 | iPhone 13 Pro, EOS 5D |
| DateTimeOriginal | 拍摄时间 | 2025:01:15 14:25:30 |
| ISO | ISO 感光度 | 100, 400, 1600 |
| FNumber | 光圈值 | f/1.8, f/2.8 |
| ExposureTime | 曝光时间 | 1/125, 1/60 |
| FocalLength | 焦距 | 24mm, 50mm |
| Flash | 闪光灯状态 | On, Off, Auto |
| Software | 编辑软件 | Adobe Photoshop, iOS 17 |
| Orientation | 图像方向 | Horizontal, Rotate 90 CW |

### 6.2 GPS 标签

| 标签名 | 说明 | 格式 |
|--------|------|------|
| GPSLatitude | 纬度 | 31 deg 13' 45.00" N |
| GPSLongitude | 经度 | 121 deg 28' 30.00" E |
| GPSAltitude | 海拔 | 10 m Above Sea Level |
| GPSDateStamp | GPS 日期 | 2025:01:15 |
| GPSTimeStamp | GPS 时间 | 14:25:30 |

### 6.3 IPTC/XMP 标签

| 标签名 | 说明 |
|--------|------|
| Artist / Creator | 作者 |
| Copyright | 版权声明 |
| ImageDescription / Caption | 图像描述 |
| Keywords | 关键词（数组） |
| Rating | 评分（1-5 星） |
| Title | 标题 |

---

## 7. CTF 常见场景

### 场景1：元数据中隐藏 Flag

**题目特征**：提供图片，提示"仔细看"

```bash
# 1. 查看所有元数据
exiftool challenge.jpg

# 2. 重点检查字段：
#    - Artist: flag{hidden_in_artist}
#    - Comment: The flag is flag{...}
#    - Copyright: flag{metadata_master}
#    - ImageDescription: flag{exif_data}
#    - UserComment: flag{user_comment}

# 3. 查看所有自定义字段
exiftool -a challenge.jpg | grep -i flag

# 4. 查看十六进制数据（隐藏在二进制中）
exiftool -b -UserComment challenge.jpg | xxd
```

### 场景2：GPS 坐标线索

**题目特征**：提示"找到拍摄地点"

```bash
# 1. 提取 GPS 坐标
exiftool -GPS* challenge.jpg

# 输出示例：
# GPS Latitude: 39 deg 54' 45.00" N
# GPS Longitude: 116 deg 23' 30.00" E

# 2. 转换为十进制格式
exiftool -n -GPS* challenge.jpg
# 输出：39.9125, 116.39167

# 3. 在 Google Maps 查找
# https://www.google.com/maps?q=39.9125,116.39167

# 4. 或使用命令行工具
curl "https://nominatim.openstreetmap.org/reverse?lat=39.9125&lon=116.39167&format=json"
```

### 场景3：拍摄时间编码

**题目特征**：时间戳异常或有规律

```bash
# 1. 查看所有时间字段
exiftool -Time:all challenge.jpg

# 2. 可能的编码方式：
#    - 时间戳：2025:01:06 06:18:05 → 20250106061805 → 解密
#    - Unix 时间戳：1736144285 → flag 的一部分
#    - 时间差异：多张图片时间差构成密码

# 3. 批量提取时间（多张图片）
exiftool -s -s -s -DateTimeOriginal *.jpg

# 4. 计算时间差
```

### 场景4：隐藏在缩略图中

**题目特征**：图片正常但 Exif 数据量大

```bash
# 1. 检查是否有缩略图
exiftool -ThumbnailImage challenge.jpg

# 2. 提取缩略图
exiftool -b -ThumbnailImage challenge.jpg > thumbnail.jpg

# 3. 查看缩略图内容
open thumbnail.jpg

# 4. 缩略图中可能隐藏不同内容或 flag
```

---

## 8. 实战案例

### 案例1：CTF Misc - 元数据隐写

**题目**：提供 `secret.jpg`，描述"Metadata matters"

```bash
# 1. 基础扫描
exiftool secret.jpg

# 输出（关键信息）：
# Artist: John
# Copyright: Doe
# Comment: flag{3x1f_m3t4d4t4}

# 2. 提取所有文本字段
exiftool secret.jpg | grep -E "(Artist|Copyright|Comment|Description)"

# 3. 检查二进制字段
exiftool -b -UserComment secret.jpg | strings

# 4. 查看完整输出（包括重复标签）
exiftool -a -G secret.jpg
```

### 案例2：数字取证 - 照片溯源

**目标**：确定照片拍摄设备和时间

```bash
# 1. 查看相机信息
exiftool -Make -Model -LensModel photo.jpg

# 输出：
# Make: Apple
# Camera Model Name: iPhone 13 Pro
# Lens Model: iPhone 13 Pro back triple camera 5.7mm f/1.5

# 2. 查看拍摄参数
exiftool -ISO -FNumber -ExposureTime -FocalLength photo.jpg

# 输出：
# ISO: 320
# F Number: f/1.5
# Exposure Time: 1/60
# Focal Length: 5.7 mm

# 3. 查看时间信息
exiftool -CreateDate -DateTimeOriginal -ModifyDate photo.jpg

# 输出：
# Create Date: 2025:01:15 14:25:30
# Date/Time Original: 2025:01:15 14:25:30
# Modify Date: 2025:01:20 09:10:15

# 4. 查看 GPS 信息
exiftool -GPS* photo.jpg

# 5. 生成完整报告
exiftool -h photo.jpg > forensic_report.html
```

### 案例3：批量清理隐私信息

**目标**：上传照片前删除敏感元数据

```bash
# 1. 创建测试目录
mkdir cleaned_photos

# 2. 复制照片并清理元数据
for img in *.jpg; do
  exiftool -all= "$img"
  cp "${img}_original" "cleaned_photos/$img"
done

# 3. 或使用批量命令（不保留原文件）
exiftool -overwrite_original -all= *.jpg

# 4. 验证清理效果
exiftool cleaned_photos/*.jpg
```

### 案例4：恢复被编辑的原始数据

**题目**：图片被 Photoshop 编辑过，但保留了原始 EXIF

```bash
# 1. 查看编辑历史
exiftool -History challenge.jpg

# 2. 查看原始数据
exiftool -OriginalTransmissionReference challenge.jpg

# 3. 对比编辑前后的元数据
exiftool -EXIF:all challenge.jpg > exif_data.txt
grep "Original" exif_data.txt

# 4. 提取原始缩略图（可能未被编辑）
exiftool -b -ThumbnailImage challenge.jpg > original_thumb.jpg
```

---

## 9. 高级技巧

### 9.1 自定义输出格式

**格式化输出**：
```bash
# 使用 -p 参数自定义格式
exiftool -p '$FileName: $Make $Model, ISO $ISO' *.jpg

# 输出示例：
# IMG_1234.jpg: Apple iPhone 13 Pro, ISO 320
# IMG_1235.jpg: Canon EOS 5D Mark IV, ISO 1600
```

**CSV 报表**：
```bash
# 生成详细 CSV 报表
exiftool -csv -FileName -Make -Model -DateTimeOriginal -GPS* *.jpg > report.csv
```

### 9.2 条件处理

**仅处理特定相机的照片**：
```bash
# 仅修改 iPhone 拍摄的照片
exiftool -if '$Make eq "Apple"' -Artist="iPhone User" *.jpg

# 仅处理高 ISO 照片
exiftool -if '$ISO > 1600' -directory=high_iso *.jpg
```

### 9.3 元数据复制与同步

**从原片复制元数据到编辑后的照片**：
```bash
# 复制所有元数据（除了图像数据）
exiftool -TagsFromFile original.jpg -all:all edited.jpg

# 仅复制特定标签
exiftool -TagsFromFile original.jpg -Artist -Copyright edited.jpg
```

### 9.4 脚本自动化

**批量分析脚本**：
```bash
#!/bin/bash
# CTF 元数据扫描脚本

for file in *.jpg *.png *.gif; do
  echo "=== Analyzing $file ==="

  # 搜索 flag
  exiftool "$file" | grep -i "flag{"

  # 提取 GPS
  exiftool -GPS* "$file"

  # 提取缩略图
  exiftool -b -ThumbnailImage "$file" > "thumb_$file" 2>/dev/null

  echo ""
done
```

### 9.5 十六进制分析

**查看原始二进制数据**：
```bash
# 提取 UserComment 的二进制数据
exiftool -b -UserComment image.jpg | xxd

# 查找隐藏的 ASCII 字符串
exiftool -b -UserComment image.jpg | strings

# 转换为 Base64
exiftool -b -UserComment image.jpg | base64
```

---

## 10. 常见问题与排查

### 问题1：无法修改元数据

**错误信息**：`Error: File is write protected`

**原因**：文件权限或文件系统只读

**解决方案**：
```bash
# 检查文件权限
ls -l image.jpg

# 修改权限
chmod 644 image.jpg

# 或使用 sudo
sudo exiftool -Artist="John" image.jpg
```

### 问题2：修改后图像损坏

**原因**：某些格式不支持特定标签

**解决方案**：
```bash
# 1. 总是先备份
cp image.jpg image.jpg.backup

# 2. 使用 -P 保留文件修改时间
exiftool -P -Artist="John" image.jpg

# 3. 验证文件完整性
file image.jpg
identify image.jpg  # 需要 ImageMagick
```

### 问题3：找不到预期的元数据

**排查步骤**：
```bash
# 1. 查看所有组
exiftool -G -a image.jpg

# 2. 使用详细模式
exiftool -v image.jpg

# 3. 检查是否有重复标签
exiftool -a image.jpg

# 4. 查看原始二进制数据
exiftool -b -a image.jpg | xxd | less
```

### 问题4：GPS 坐标格式转换

**度分秒 → 十进制**：
```bash
# 使用 -n 参数获取数值格式
exiftool -n -GPS* image.jpg

# 手动计算：度 + 分/60 + 秒/3600
# 例：39 deg 54' 45.00" = 39 + 54/60 + 45/3600 = 39.9125
```

---

## 11. 隐私与安全

### 11.1 元数据隐私风险

**敏感信息类型**：
- **GPS 坐标**：暴露家庭地址、工作地点
- **时间戳**：暴露作息规律
- **相机序列号**：可追踪设备
- **软件信息**：暴露使用的工具
- **作者信息**：真实姓名、联系方式

### 11.2 隐私保护最佳实践

**上传前清理元数据**：
```bash
# 创建清理后的副本
mkdir safe_uploads
exiftool -all= -o safe_uploads/ *.jpg

# 或使用在线工具
# https://www.verexif.com/
# https://jimpl.com/
```

**部分保留策略**：
```bash
# 仅删除 GPS 和个人信息
exiftool -GPS:all= -Artist= -Copyright= -SerialNumber= *.jpg
```

### 11.3 检测元数据泄露

**审查自己的照片**：
```bash
# 生成完整报告
exiftool -a -G -h *.jpg > privacy_audit.html

# 查找敏感信息
exiftool *.jpg | grep -E "(GPS|Serial|Owner|Artist|Copyright)"
```

---

## 12. 参考链接

### 官方资源
- **ExifTool 官网**：https://exiftool.org/
- **标签名称文档**：https://exiftool.org/TagNames/
- **FAQ**：https://exiftool.org/faq.html
- **论坛**：https://exiftool.org/forum/

### EXIF 标准
- **EXIF 2.32 规范**：https://www.cipa.jp/std/documents/e/DC-008-Translation-2019-E.pdf
- **IPTC 标准**：https://iptc.org/standards/photo-metadata/
- **XMP 规范**：https://www.adobe.com/devnet/xmp.html

### 隐私工具
- **MAT2（Metadata Anonymisation Toolkit）**：https://0xacab.org/jvoisin/mat2
- **ExifCleaner（GUI）**：https://exifcleaner.com/
- **ImageOptim（macOS）**：https://imageoptim.com/

### 教程和文章
- **ExifTool 完整教程**：https://ninedegreesbelow.com/photography/exiftool-commands.html
- **EXIF 隐私指南**：https://www.eff.org/deeplinks/2020/02/guide-protecting-your-photo-metadata
- **CTF Writeups - EXIF**：https://ctftime.org/

### 相关工具
- **ExifReader（JavaScript）**：https://github.com/mattiasw/ExifReader
- **Pillow（Python）**：https://pillow.readthedocs.io/
- **ImageMagick identify**：https://imagemagick.org/script/identify.php

---

## 快速参考卡片

### 最常用命令
```bash
# 查看所有元数据
exiftool image.jpg

# 查看特定标签
exiftool -Make -Model -DateTimeOriginal image.jpg

# 删除所有元数据
exiftool -all= image.jpg

# 删除 GPS 信息
exiftool -GPS:all= image.jpg

# 批量重命名（按日期）
exiftool '-FileName<CreateDate' -d %Y%m%d_%H%M%S.%%e *.jpg

# JSON 输出
exiftool -j image.jpg

# CSV 批量导出
exiftool -csv *.jpg > metadata.csv
```

### CTF 快速检查
```bash
# 搜索 flag
exiftool image.jpg | grep -i "flag"

# 查看所有文本字段
exiftool -s image.jpg | grep -v "^File\|^Directory\|^ExifTool"

# 提取缩略图
exiftool -b -ThumbnailImage image.jpg > thumb.jpg

# 查看十六进制数据
exiftool -b -UserComment image.jpg | xxd
```

---

**文档版本**：v1.0
**更新日期**：2025-01
**适用版本**：ExifTool v12.0 - v12.70+
