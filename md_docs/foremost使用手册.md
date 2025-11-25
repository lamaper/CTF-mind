# Foremost 使用手册（文件恢复与数据雕刻）

> 免责声明：本手册仅用于合法的数字取证、CTF 竞赛和授权的数据恢复。未经授权恢复他人数据可能违法。

---

## 目录
1. 概述
2. 安装方法
3. 基本命令结构
4. 核心功能详解
5. 常用命令与示例
6. 支持的文件类型
7. CTF 常见场景
8. 实战案例
9. 高级技巧
10. 常见问题与排查
11. 参考链接

---

## 1. 概述

Foremost 是一个基于文件头和文件尾的数据雕刻（Data Carving）工具，由美国空军特别调查办公室开发。它能够：
- **文件恢复**：从磁盘镜像、内存dump、损坏文件中恢复数据
- **数据雕刻**：不依赖文件系统，通过文件签名直接提取
- **隐写分析**：提取隐藏在文件中的其他文件
- **取证调查**：恢复被删除或隐藏的文件

**工作原理**：
- 扫描数据块，寻找已知文件类型的头部签名（Magic Bytes）
- 根据文件尾签名或最大文件大小确定文件边界
- 提取并保存完整文件到指定目录

**适用场景**：
- CTF MISC 题目（文件隐写、多层嵌套）
- 数字取证（恢复删除文件）
- 磁盘恢复（格式化后数据恢复）
- 固件分析（提取固件中的文件）

**版本信息**：本手册基于 Foremost v1.5.7

---

## 2. 安装方法

### macOS 安装
```bash
# 使用 Homebrew 安装
brew install foremost

# 验证安装
foremost -V
```

### Kali Linux / Debian / Ubuntu
```bash
# Kali 自带，如需更新
sudo apt update
sudo apt install foremost

# 验证安装
foremost -V
```

### 从源码编译
```bash
# 下载源码
wget http://foremost.sourceforge.net/pkg/foremost-1.5.7.tar.gz

# 解压编译
tar -xzf foremost-1.5.7.tar.gz
cd foremost-1.5.7
make
sudo make install

# 验证
foremost -V
```

---

## 3. 基本命令结构

```bash
foremost [选项] -i [输入文件] -o [输出目录]

# 基本用法
foremost -i disk.img -o recovered/

# 指定文件类型
foremost -t jpg,png,pdf -i disk.img -o recovered/

# 详细输出
foremost -v -i disk.img -o recovered/
```

### 核心选项说明

**必需选项**：
- `-i <file>` - 输入文件（磁盘镜像、内存dump、任意二进制文件）
- `-o <dir>` - 输出目录（恢复的文件保存位置）

**文件类型选项**：
- `-t <types>` - 指定文件类型（逗号分隔），如：`jpg,png,pdf`
- `-c <config>` - 使用自定义配置文件（默认：`/etc/foremost.conf`）

**输出选项**：
- `-v` - 详细模式（显示扫描进度）
- `-q` - 安静模式（仅输出结果）
- `-w` - 仅写入审计文件（audit.txt），不提取文件

**高级选项**：
- `-d` - 间接块检测（用于文件系统恢复）
- `-T` - 设置时间戳为文件创建时间
- `-b <size>` - 块大小（默认512字节）
- `-k <size>` - 单个文件最大大小（KB）
- `-s <offset>` - 从指定偏移开始扫描
- `-a` - 写入所有头部（包括损坏的）

---

## 4. 核心功能详解

### 4.1 文件签名识别

Foremost 基于 **Magic Bytes（文件头签名）** 识别文件类型：

| 文件类型 | 头部签名（十六进制） | 尾部签名 |
|---------|-------------------|----------|
| JPEG | `FF D8 FF` | `FF D9` |
| PNG | `89 50 4E 47 0D 0A 1A 0A` | `49 45 4E 44 AE 42 60 82` |
| GIF | `47 49 46 38 (37|39) 61` | `00 3B` |
| PDF | `25 50 44 46` | `25 25 45 4F 46` |
| ZIP | `50 4B 03 04` | `50 4B 05 06` |
| RAR | `52 61 72 21 1A 07` | N/A |
| EXE | `4D 5A` | N/A |
| MP3 | `FF FB` / `49 44 33` | N/A |

### 4.2 数据雕刻流程

1. **扫描阶段**：逐字节扫描输入文件，查找文件头签名
2. **边界确定**：
   - 方法1：查找文件尾签名
   - 方法2：使用配置的最大文件大小
3. **文件提取**：将识别的数据块保存为单独文件
4. **审计报告**：生成 `audit.txt` 记录恢复详情

### 4.3 配置文件结构

**默认配置文件位置**：
- macOS：`/usr/local/etc/foremost.conf`
- Linux：`/etc/foremost.conf`

**配置格式**：
```
# 格式：文件类型  区分大小写  最大大小  头部签名  尾部签名  [反向搜索]
jpg     y   10000000    \xff\xd8\xff    \xff\xd9
png     y   10000000    \x89\x50\x4e\x47\x0d\x0a\x1a\x0a    \x49\x45\x4e\x44\xae\x42\x60\x82
```

---

## 5. 常用命令与示例

### 5.1 基本文件恢复

**恢复所有支持的文件类型**：
```bash
foremost -i disk.img -o recovered/
```

**恢复特定文件类型**：
```bash
# 仅恢复图片
foremost -t jpg,png,gif -i challenge.bin -o images/

# 仅恢复文档
foremost -t pdf,doc,docx -i disk.img -o documents/

# 仅恢复压缩包
foremost -t zip,rar,gz -i firmware.bin -o archives/
```

**详细输出模式**：
```bash
foremost -v -i disk.img -o recovered/
```

**输出示例**：
```
Foremost version 1.5.7 by Jesse Kornblum, Kris Kendall, and Nick Mikus
Audit File

Foremost started at Mon Jan 20 10:30:00 2025
Invocation: foremost -v -i disk.img -o recovered/
Output directory: recovered
Configuration file: /etc/foremost.conf
------------------------------------------------------------------
File: disk.img
Start: Mon Jan 20 10:30:00 2025
Length: 100 MB (104857600 bytes)

Num      Name (bs=512)         Size      File Offset     Comment
0:       00000128.jpg          156 KB            65536
1:       00000384.png          2.3 MB           196608
2:       00002048.zip          45 KB           1048576
...

*|
------------------------------------------------------------------
Finish: Mon Jan 20 10:35:00 2025

3 FILES EXTRACTED

jpg:= 1
png:= 1
zip:= 1
------------------------------------------------------------------

Foremost finished at Mon Jan 20 10:35:00 2025
```

### 5.2 从指定偏移开始

**场景**：已知文件大致位置，加快扫描

```bash
# 从 1MB 偏移开始扫描
foremost -s 1048576 -i disk.img -o recovered/

# 从十六进制偏移开始
foremost -s 0x100000 -i disk.img -o recovered/
```

### 5.3 限制文件大小

```bash
# 单个文件最大 5MB（5000 KB）
foremost -k 5000 -i disk.img -o recovered/

# 结合文件类型限制
foremost -t jpg -k 2000 -i challenge.bin -o images/
```

### 5.4 自定义配置文件

**创建自定义配置**：
```bash
# 创建配置文件
cat > custom.conf <<EOF
# 仅提取 JPEG 和 PNG
jpg     y   10000000    \xff\xd8\xff    \xff\xd9
png     y   10000000    \x89\x50\x4e\x47    \x49\x45\x4e\x44\xae\x42\x60\x82
EOF

# 使用自定义配置
foremost -c custom.conf -i disk.img -o recovered/
```

### 5.5 批量处理

```bash
#!/bin/bash
# 批量恢复多个镜像文件

for img in *.img; do
  echo "Processing $img..."
  mkdir -p "recovered_${img%.img}"
  foremost -i "$img" -o "recovered_${img%.img}/"
done
```

---

## 6. 支持的文件类型

### 6.1 默认支持类型

| 类别 | 文件类型 |
|------|----------|
| 图像 | jpg, gif, png, bmp |
| 文档 | pdf, doc, ole, ppt, xls |
| 压缩 | zip, rar, gz, bz2 |
| 音视频 | avi, mov, mpg, wav, wmv |
| 可执行 | exe, dll |
| 其他 | html, cpp, mp4 |

### 6.2 查看所有支持类型

```bash
# 查看配置文件
cat /usr/local/etc/foremost.conf  # macOS
cat /etc/foremost.conf             # Linux

# 或使用 grep 过滤注释
grep -v "^#" /etc/foremost.conf | grep -v "^$"
```

### 6.3 添加自定义文件类型

**示例：添加 SQLite 数据库**：
```bash
# 编辑配置文件
sudo nano /etc/foremost.conf

# 添加规则
sqlite  y  100000000  \x53\x51\x4c\x69\x74\x65\x20\x66\x6f\x72\x6d\x61\x74\x20\x33\x00
```

---

## 7. CTF 常见场景

### 场景1：图片隐写（嵌入ZIP）

**题目特征**：提供一张图片，但 `binwalk` 显示有嵌入文件

```bash
# 1. Binwalk 扫描确认
binwalk challenge.jpg

# 输出：
# 0         0x0         JPEG image
# 54321     0xD431      Zip archive data

# 2. 使用 Foremost 提取
foremost -i challenge.jpg -o extracted/

# 3. 查看结果
cd extracted
ls -lh
```

### 场景2：损坏的压缩包恢复

**题目特征**：ZIP 文件无法解压，提示损坏

```bash
# 1. 尝试 Foremost 恢复
foremost -t zip -i corrupted.zip -o recovered/

# 2. 查看恢复的文件
cd recovered/zip
unzip *.zip
```

### 场景3：内存 Dump 分析

**题目特征**：提供内存镜像文件 `memory.dmp`

```bash
# 1. 提取所有文件
foremost -i memory.dmp -o memory_files/

# 2. 重点查找图片和文档
cd memory_files
ls -R

# 3. 搜索 flag
find . -type f -exec grep -l "flag{" {} \;
```

### 场景4：固件镜像文件提取

**题目特征**：固件文件 `firmware.bin`

```bash
# 1. 提取固件中的所有文件
foremost -v -i firmware.bin -o firmware_extracted/

# 2. 查看提取的文件
cd firmware_extracted
tree

# 3. 分析提取的文件
file */*
```

---

## 8. 实战案例

### 案例1：CTF - Misc 多层隐写

**题目**：提供 `challenge.png`，提示"层层深入"

```bash
# 1. 查看文件基本信息
file challenge.png
# Output: PNG image data, 1920 x 1080

# 2. Binwalk 扫描
binwalk challenge.png
# Output:
# 0         0x0         PNG image
# 524288    0x80000     Zip archive data

# 3. Foremost 提取
foremost -v -i challenge.png -o layer1/

# 4. 检查提取的 ZIP
cd layer1/zip
unzip *.zip
# 解压出 hidden.jpg

# 5. 继续分析 hidden.jpg
foremost -i hidden.jpg -o ../layer2/

# 6. 重复直到找到 flag
```

### 案例2：删除文件恢复

**场景**：磁盘镜像中有被删除的重要文件

```bash
# 1. 创建磁盘镜像（如果需要）
sudo dd if=/dev/sdb of=disk.img bs=4M

# 2. Foremost 恢复所有文件
foremost -v -i disk.img -o recovered/

# 3. 查看审计报告
cat recovered/audit.txt

# 4. 分类查看恢复的文件
cd recovered
for dir in */; do
  echo "=== $dir ==="
  ls -lh "$dir"
done

# 5. 搜索特定内容
find recovered -type f -exec grep -l "password" {} \;
```

### 案例3：USB 设备取证

**场景**：需要从 USB 设备恢复删除的照片

```bash
# 1. 识别 USB 设备
lsblk
# 假设是 /dev/sdc

# 2. 创建镜像（避免进一步损坏）
sudo dd if=/dev/sdc of=usb_image.img bs=4M status=progress

# 3. 恢复照片
foremost -t jpg,png,gif,raw -i usb_image.img -o usb_photos/

# 4. 查看恢复结果
cd usb_photos
ls -lhR

# 5. 按修改时间排序（如果需要）
find . -type f -exec stat -f "%Sm %N" {} \; | sort
```

### 案例4：固件逆向 - 提取嵌入文件

**场景**：路由器固件分析

```bash
# 1. Foremost 提取所有文件
foremost -v -i router_firmware.bin -o firmware_files/

# 2. 查看文件系统镜像
cd firmware_files
file */*

# 3. 如果提取出文件系统镜像，进一步挂载
mkdir /tmp/firmware_fs
sudo mount -o loop,ro squashfs-root/00012345.ext4 /tmp/firmware_fs

# 4. 分析固件内容
ls /tmp/firmware_fs
grep -r "password" /tmp/firmware_fs

# 5. 卸载
sudo umount /tmp/firmware_fs
```

---

## 9. 高级技巧

### 9.1 结合 Binwalk 使用

**工作流程**：Binwalk 识别 → Foremost 提取

```bash
# 1. Binwalk 扫描获取偏移
binwalk challenge.bin > binwalk_result.txt

# 2. 使用 Foremost 从特定偏移提取
foremost -s <offset> -i challenge.bin -o extracted/

# 3. 或直接提取整个文件
foremost -i challenge.bin -o extracted/
```

### 9.2 自定义文件签名

**场景**：提取非标准文件类型

```bash
# 1. 确定文件签名（使用 xxd）
xxd custom_file | head

# 2. 编辑配置文件
sudo nano /etc/foremost.conf

# 3. 添加自定义规则
# 格式：类型  大小写  最大大小  头部签名  尾部签名
custom  y  5000000  \x43\x55\x53\x54\x4f\x4d  \x45\x4e\x44

# 4. 使用自定义配置
foremost -c /etc/foremost.conf -i data.bin -o extracted/
```

### 9.3 批量分析脚本

```bash
#!/bin/bash
# 批量恢复和分析脚本

INPUT_DIR="images"
OUTPUT_BASE="extracted"

for file in "$INPUT_DIR"/*; do
  filename=$(basename "$file")
  output_dir="${OUTPUT_BASE}/${filename%.img}"

  echo "[+] Processing $filename..."
  foremost -v -i "$file" -o "$output_dir"

  # 统计恢复文件数
  total=$(find "$output_dir" -type f | wc -l)
  echo "[+] Recovered $total files from $filename"

  # 生成报告
  echo "$filename: $total files" >> recovery_summary.txt
done

echo "[+] All done! Summary:"
cat recovery_summary.txt
```

### 9.4 内存 Dump 分析

```bash
# 1. 提取内存中的文件
foremost -i memory.dmp -o memory_files/

# 2. 提取特定类型（如密码文件）
foremost -t pdf,doc,txt -i memory.dmp -o documents/

# 3. 搜索敏感信息
find memory_files -type f -exec strings {} \; | grep -E "(password|token|key)"
```

---

## 10. 常见问题与排查

### 问题1：无法创建输出目录

**错误信息**：`ERROR: Unable to create output directory`

**原因**：权限不足或目录已存在

**解决方案**：
```bash
# 1. 检查权限
ls -ld output_dir

# 2. 创建目录
mkdir -p output_dir

# 3. 使用 sudo（如果需要）
sudo foremost -i disk.img -o /root/recovered/

# 4. 或删除已存在的目录
rm -rf output_dir
```

### 问题2：提取的文件损坏

**原因**：
1. 文件尾签名匹配错误
2. 文件大小超过配置的最大值
3. 文件本身在源文件中就已损坏

**解决方案**：
```bash
# 1. 增加最大文件大小
foremost -k 50000 -i disk.img -o recovered/

# 2. 使用 -a 选项（写入所有头部）
foremost -a -i disk.img -o recovered/

# 3. 编辑配置文件调整文件大小限制
sudo nano /etc/foremost.conf
# 修改对应类型的最大大小（第三列）

# 4. 验证提取的文件
file recovered/*/*.jpg
```

### 问题3：恢复文件数量过少

**排查步骤**：
```bash
# 1. 使用详细模式查看扫描过程
foremost -v -i disk.img -o recovered/

# 2. 检查配置文件是否正确
cat /etc/foremost.conf

# 3. 尝试不指定文件类型（恢复所有）
foremost -i disk.img -o recovered/

# 4. 尝试从不同偏移开始
foremost -s 0 -i disk.img -o recovered1/
foremost -s 1048576 -i disk.img -o recovered2/

# 5. 结合其他工具验证
binwalk disk.img
strings disk.img | grep -i "jpg\|png\|pdf"
```

### 问题4：速度太慢

**优化方法**：
```bash
# 1. 指定文件类型（减少扫描范围）
foremost -t jpg,png -i large_disk.img -o images/

# 2. 从已知偏移开始
foremost -s <offset> -i disk.img -o recovered/

# 3. 使用 SSD 存储输出目录
foremost -i disk.img -o /path/to/ssd/recovered/

# 4. 增大块大小（默认512字节）
foremost -b 4096 -i disk.img -o recovered/
```

---

## 11. 参考链接

### 官方资源
- **Foremost 官网**：http://foremost.sourceforge.net/
- **项目页面**：https://sourceforge.net/projects/foremost/
- **Man 手册**：`man foremost`

### 数据恢复工具对比
- **Scalpel**：https://github.com/sleuthkit/scalpel（Foremost 的改进版）
- **PhotoRec**：https://www.cgsecurity.org/wiki/PhotoRec（支持更多格式）
- **TestDisk**：https://www.cgsecurity.org/wiki/TestDisk（分区恢复）

### 数字取证资源
- **The Sleuth Kit**：https://www.sleuthkit.org/
- **Autopsy**：https://www.autopsy.com/（取证平台）
- **SANS Digital Forensics**：https://www.sans.org/cyber-security-courses/digital-forensics-essentials/

### CTF 相关
- **CTF Wiki - Misc**：https://ctf-wiki.org/misc/archive/
- **PicoCTF Forensics**：https://picoctf.org/
- **CTFtime Writeups**：https://ctftime.org/writeups

### 文件签名参考
- **File Signatures Database**：https://www.garykessler.net/library/file_sigs.html
- **Wikipedia - Magic Number**：https://en.wikipedia.org/wiki/Magic_number_(programming)

---

## 快速参考

### 最常用命令
```bash
# 恢复所有文件类型
foremost -i disk.img -o recovered/

# 恢复图片
foremost -t jpg,png,gif -i image.jpg -o images/

# 恢复文档和压缩包
foremost -t pdf,zip,rar -i data.bin -o files/

# 详细输出
foremost -v -i disk.img -o recovered/

# 从指定偏移开始
foremost -s 1048576 -i disk.img -o recovered/
```

### CTF 快速检查
```bash
# 提取所有文件
foremost -i challenge.bin -o extracted/

# 查看提取结果
cd extracted
tree
ls -lhR

# 搜索 flag
find . -type f -exec grep -l "flag{" {} \;
```

---

**文档版本**：v1.0
**更新日期**：2025-01
**适用版本**：Foremost v1.5.7+
