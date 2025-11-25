# Pillow 使用手册（Python图像处理库）

> Pillow（PIL Fork）是Python最流行的图像处理库,在CTF中广泛用于图像隐写分析、图像修复和数据提取。

---

## 目录
1. [概述](#概述)
2. [安装方法](#安装方法)
3. [基础操作](#基础操作)
4. [图像读写](#图像读写)
5. [像素操作](#像素操作)
6. [图像处理](#图像处理)
7. [LSB隐写](#lsb隐写)
8. [二维码处理](#二维码处理)
9. [CTF常见场景](#ctf常见场景)
10. [实战案例](#实战案例)
11. [高级技巧](#高级技巧)
12. [参考资源](#参考资源)

---

## 概述

Pillow 是 PIL (Python Imaging Library) 的维护分支,提供:
- **图像I/O**: 支持30+图像格式读写
- **像素操作**: 直接访问和修改像素数据
- **图像处理**: 滤镜、变换、增强等操作
- **格式转换**: 不同图像格式间转换
- **通道分离**: RGB/RGBA通道独立操作

**CTF应用场景**:
- LSB隐写分析
- 图像拼接和修复
- 二维码识别
- 像素数据提取
- 图层分析

---

## 安装方法

```bash
# 安装 Pillow
pip install Pillow

# 安装额外依赖(可选)
pip install pillow[imagingft]  # 字体渲染支持

# 验证安装
python3 -c "from PIL import Image; print(Image.__version__)"

# 查看支持的格式
python3 -c "from PIL import features; features.pilinfo()"
```

---

## 基础操作

### 打开和显示图像

```python
from PIL import Image

# 打开图像
img = Image.open('image.png')

# 查看基本信息
print(f"格式: {img.format}")  # PNG, JPEG, GIF等
print(f"尺寸: {img.size}")  # (width, height)
print(f"模式: {img.mode}")  # RGB, RGBA, L, P等
print(f"像素总数: {img.width * img.height}")

# 显示图像(会调用系统默认查看器)
img.show()

# 关闭图像
img.close()
```

### 图像模式说明

```python
# 常见模式
# 1: 1位像素,黑白
# L: 8位像素,灰度
# P: 8位像素,调色板模式
# RGB: 3x8位像素,真彩色
# RGBA: 4x8位像素,带透明通道
# CMYK: 4x8位像素,色彩分离
# I: 32位整型像素
# F: 32位浮点型像素

# 模式转换
img_rgb = img.convert('RGB')
img_gray = img.convert('L')
img_rgba = img.convert('RGBA')
```

---

## 图像读写

### 保存图像

```python
from PIL import Image

img = Image.open('input.png')

# 保存为不同格式
img.save('output.jpg')  # 自动识别格式
img.save('output.png', 'PNG')  # 指定格式
img.save('output.gif', 'GIF')

# 保存时指定参数
img.save('output.jpg', quality=95)  # JPEG质量
img.save('output.png', optimize=True)  # PNG优化
```

### 创建新图像

```python
from PIL import Image

# 创建空白图像
img = Image.new('RGB', (800, 600), color='white')
img = Image.new('RGBA', (400, 300), color=(255, 0, 0, 128))

# 从数组创建
import numpy as np
arr = np.zeros((100, 100, 3), dtype=np.uint8)
arr[25:75, 25:75] = [255, 0, 0]  # 红色方块
img = Image.fromarray(arr)
img.save('array_image.png')
```

### 批量处理

```python
from PIL import Image
import os

# 批量转换格式
def batch_convert(input_dir, output_dir, target_format='PNG'):
    os.makedirs(output_dir, exist_ok=True)

    for filename in os.listdir(input_dir):
        if filename.lower().endswith(('.jpg', '.jpeg', '.png', '.bmp')):
            img_path = os.path.join(input_dir, filename)
            img = Image.open(img_path)

            output_filename = os.path.splitext(filename)[0] + f'.{target_format.lower()}'
            output_path = os.path.join(output_dir, output_filename)

            img.save(output_path, target_format)
            print(f"Converted: {filename} -> {output_filename}")

batch_convert('input/', 'output/', 'PNG')
```

---

## 像素操作

### 读取像素值

```python
from PIL import Image

img = Image.open('image.png')

# 获取单个像素
pixel = img.getpixel((10, 20))  # (x, y)
print(f"像素值: {pixel}")  # RGB: (r, g, b) 或 RGBA: (r, g, b, a)

# 遍历所有像素
width, height = img.size
for y in range(height):
    for x in range(width):
        pixel = img.getpixel((x, y))
        # 处理像素...

# 使用load()加速访问
pixels = img.load()
for y in range(height):
    for x in range(width):
        pixel = pixels[x, y]
```

### 修改像素值

```python
from PIL import Image

img = Image.open('image.png')
pixels = img.load()

# 修改单个像素
pixels[10, 20] = (255, 0, 0)  # 设置为红色

# 批量修改
width, height = img.size
for y in range(height):
    for x in range(width):
        r, g, b = pixels[x, y]
        # 反色效果
        pixels[x, y] = (255 - r, 255 - g, 255 - b)

img.save('modified.png')
```

### 转换为NumPy数组

```python
from PIL import Image
import numpy as np

img = Image.open('image.png')

# PIL转NumPy
arr = np.array(img)
print(f"数组形状: {arr.shape}")  # (height, width, channels)
print(f"数据类型: {arr.dtype}")  # uint8

# 批量像素操作
arr[:, :, 0] = 255  # 所有像素的红色通道设为255

# NumPy转PIL
img_new = Image.fromarray(arr)
img_new.save('numpy_modified.png')
```

---

## 图像处理

### 几何变换

```python
from PIL import Image

img = Image.open('image.png')

# 缩放
img_resized = img.resize((400, 300))  # 指定尺寸
img_thumbnail = img.copy()
img_thumbnail.thumbnail((128, 128))  # 保持比例缩放

# 旋转
img_rotated = img.rotate(45)  # 逆时针旋转45度
img_rotated = img.rotate(45, expand=True)  # 扩展画布以包含全部内容

# 翻转
img_flipped_h = img.transpose(Image.FLIP_LEFT_RIGHT)  # 水平翻转
img_flipped_v = img.transpose(Image.FLIP_TOP_BOTTOM)  # 垂直翻转

# 裁剪
box = (100, 100, 400, 400)  # (left, top, right, bottom)
img_cropped = img.crop(box)
```

### 颜色调整

```python
from PIL import Image, ImageEnhance

img = Image.open('image.png')

# 亮度调整
enhancer = ImageEnhance.Brightness(img)
img_bright = enhancer.enhance(1.5)  # 1.5倍亮度

# 对比度调整
enhancer = ImageEnhance.Contrast(img)
img_contrast = enhancer.enhance(2.0)

# 饱和度调整
enhancer = ImageEnhance.Color(img)
img_saturated = enhancer.enhance(0.5)  # 0.5倍饱和度

# 锐度调整
enhancer = ImageEnhance.Sharpness(img)
img_sharp = enhancer.enhance(2.0)
```

### 滤镜效果

```python
from PIL import Image, ImageFilter

img = Image.open('image.png')

# 模糊
img_blur = img.filter(ImageFilter.BLUR)
img_gaussian = img.filter(ImageFilter.GaussianBlur(radius=5))

# 锐化
img_sharp = img.filter(ImageFilter.SHARPEN)
img_unsharp = img.filter(ImageFilter.UnsharpMask(radius=2, percent=150))

# 边缘检测
img_edges = img.filter(ImageFilter.FIND_EDGES)
img_contour = img.filter(ImageFilter.CONTOUR)

# 浮雕
img_emboss = img.filter(ImageFilter.EMBOSS)

# 平滑
img_smooth = img.filter(ImageFilter.SMOOTH)
```

---

## LSB隐写

### LSB信息隐藏

```python
from PIL import Image

def hide_data_lsb(image_path, data, output_path):
    """在图像最低有效位(LSB)中隐藏数据"""
    img = Image.open(image_path)
    img = img.convert('RGB')
    pixels = img.load()

    # 将数据转换为二进制
    data_binary = ''.join(format(ord(char), '08b') for char in data)
    data_binary += '1111111111111110'  # 结束标记

    data_index = 0
    width, height = img.size

    for y in range(height):
        for x in range(width):
            if data_index >= len(data_binary):
                break

            r, g, b = pixels[x, y]

            # 在红色通道LSB中隐藏数据
            if data_index < len(data_binary):
                r = (r & 0xFE) | int(data_binary[data_index])
                data_index += 1

            pixels[x, y] = (r, g, b)

        if data_index >= len(data_binary):
            break

    img.save(output_path, 'PNG')
    print(f"数据已隐藏到 {output_path}")

# 使用示例
hide_data_lsb('cover.png', 'flag{hidden_message}', 'stego.png')
```

### LSB信息提取

```python
from PIL import Image

def extract_data_lsb(image_path):
    """从图像LSB中提取隐藏数据"""
    img = Image.open(image_path)
    img = img.convert('RGB')
    pixels = img.load()

    binary_data = ''
    width, height = img.size

    for y in range(height):
        for x in range(width):
            r, g, b = pixels[x, y]

            # 提取红色通道LSB
            binary_data += str(r & 1)

    # 将二进制转换为文本
    message = ''
    for i in range(0, len(binary_data), 8):
        byte = binary_data[i:i+8]
        if len(byte) == 8:
            char_code = int(byte, 2)
            if char_code == 0:  # 结束标记
                break
            if 32 <= char_code <= 126:  # 可打印字符
                message += chr(char_code)

    return message

# 使用示例
hidden_message = extract_data_lsb('stego.png')
print(f"提取的数据: {hidden_message}")
```

### 多通道LSB分析

```python
from PIL import Image
import numpy as np

def analyze_lsb_all_channels(image_path, output_prefix='lsb'):
    """分析所有颜色通道的LSB"""
    img = Image.open(image_path)
    img = img.convert('RGB')
    arr = np.array(img)

    channels = ['R', 'G', 'B']

    for i, channel_name in enumerate(channels):
        # 提取LSB
        lsb = (arr[:, :, i] & 1) * 255

        # 保存LSB图像
        lsb_img = Image.fromarray(lsb.astype(np.uint8), mode='L')
        lsb_img.save(f'{output_prefix}_{channel_name}.png')
        print(f"保存了 {channel_name} 通道 LSB: {output_prefix}_{channel_name}.png")

    # 组合所有通道的LSB
    lsb_combined = np.zeros_like(arr)
    for i in range(3):
        lsb_combined[:, :, i] = (arr[:, :, i] & 1) * 255

    lsb_combined_img = Image.fromarray(lsb_combined.astype(np.uint8))
    lsb_combined_img.save(f'{output_prefix}_combined.png')
    print(f"保存了组合LSB: {output_prefix}_combined.png")

# 使用示例
analyze_lsb_all_channels('suspicious.png')
```

---

## 二维码处理

### 生成二维码

```python
from PIL import Image, ImageDraw

def create_simple_qr_pattern(size=200, block_size=10):
    """创建简单的二维码样式图案"""
    img = Image.new('RGB', (size, size), 'white')
    draw = ImageDraw.Draw(img)

    # 绘制定位点
    positions = [(0, 0), (size - block_size * 7, 0), (0, size - block_size * 7)]

    for x, y in positions:
        # 外框
        draw.rectangle([x, y, x + block_size * 7, y + block_size * 7],
                      outline='black', width=block_size)
        # 内部方块
        draw.rectangle([x + block_size * 2, y + block_size * 2,
                       x + block_size * 5, y + block_size * 5], fill='black')

    img.save('qr_pattern.png')
    print("二维码图案已生成")

create_simple_qr_pattern()

# 实际二维码生成需要使用 qrcode 库
# pip install qrcode[pil]
# import qrcode
# qr = qrcode.QRCode(version=1, box_size=10, border=4)
# qr.add_data('https://example.com')
# qr.make(fit=True)
# img = qr.make_image(fill_color="black", back_color="white")
# img.save('qrcode.png')
```

### 二维码修复

```python
from PIL import Image
import numpy as np

def repair_qr_code(damaged_path, output_path):
    """尝试修复损坏的二维码"""
    img = Image.open(damaged_path)
    img = img.convert('L')  # 转为灰度
    arr = np.array(img)

    # 二值化处理
    threshold = 128
    arr[arr < threshold] = 0
    arr[arr >= threshold] = 255

    # 降噪
    from scipy import ndimage
    arr = ndimage.median_filter(arr, size=3)

    repaired_img = Image.fromarray(arr.astype(np.uint8))
    repaired_img.save(output_path)
    print(f"修复后的二维码已保存到: {output_path}")

# 使用示例
repair_qr_code('damaged_qr.png', 'repaired_qr.png')
```

---

## CTF常见场景

### 场景1: 图像拼接

```python
from PIL import Image

def stitch_images_horizontal(image_paths, output_path):
    """水平拼接多张图像"""
    images = [Image.open(path) for path in image_paths]

    # 计算总宽度和最大高度
    total_width = sum(img.width for img in images)
    max_height = max(img.height for img in images)

    # 创建新图像
    result = Image.new('RGB', (total_width, max_height))

    # 拼接
    x_offset = 0
    for img in images:
        result.paste(img, (x_offset, 0))
        x_offset += img.width

    result.save(output_path)
    print(f"拼接完成: {output_path}")

# 使用示例
image_files = ['part1.png', 'part2.png', 'part3.png']
stitch_images_horizontal(image_files, 'flag.png')
```

### 场景2: 通道分离

```python
from PIL import Image

def split_channels(image_path, output_prefix='channel'):
    """分离RGB通道并保存"""
    img = Image.open(image_path)
    img = img.convert('RGB')

    r, g, b = img.split()

    # 保存各通道
    r.save(f'{output_prefix}_R.png')
    g.save(f'{output_prefix}_G.png')
    b.save(f'{output_prefix}_B.png')

    print("通道分离完成")

    # 通道可视化(将单通道显示为灰度图)
    Image.merge('RGB', (r, Image.new('L', img.size), Image.new('L', img.size))).save(f'{output_prefix}_R_visual.png')
    Image.merge('RGB', (Image.new('L', img.size), g, Image.new('L', img.size))).save(f'{output_prefix}_G_visual.png')
    Image.merge('RGB', (Image.new('L', img.size), Image.new('L', img.size), b)).save(f'{output_prefix}_B_visual.png')

# 使用示例
split_channels('flag.png')
```

### 场景3: 异或图像恢复

```python
from PIL import Image
import numpy as np

def xor_images(img1_path, img2_path, output_path):
    """对两张图像进行异或操作"""
    img1 = Image.open(img1_path).convert('RGB')
    img2 = Image.open(img2_path).convert('RGB')

    arr1 = np.array(img1)
    arr2 = np.array(img2)

    # 确保尺寸相同
    if arr1.shape != arr2.shape:
        print("警告: 图像尺寸不同,调整中...")
        img2 = img2.resize(img1.size)
        arr2 = np.array(img2)

    # 异或操作
    result_arr = np.bitwise_xor(arr1, arr2)

    result_img = Image.fromarray(result_arr.astype(np.uint8))
    result_img.save(output_path)
    print(f"异或结果已保存: {output_path}")

# 使用示例
xor_images('encrypted1.png', 'encrypted2.png', 'flag.png')
```

### 场景4: 盲水印检测

```python
from PIL import Image
import numpy as np

def detect_watermark(image_path, output_path, alpha=3.0):
    """检测图像中的盲水印"""
    img = Image.open(image_path).convert('RGB')
    arr = np.array(img, dtype=np.float32)

    # 增强低频信号差异
    enhanced = np.clip(arr * alpha, 0, 255)

    # 提取差异
    diff = enhanced - arr
    diff = np.abs(diff)
    diff = np.clip(diff * 10, 0, 255)

    result_img = Image.fromarray(diff.astype(np.uint8))
    result_img.save(output_path)
    print(f"水印检测结果: {output_path}")

# 使用示例
detect_watermark('watermarked.png', 'watermark_detected.png')
```

### 场景5: 图像差分分析

```python
from PIL import Image
import numpy as np

def image_diff(img1_path, img2_path, output_path, threshold=10):
    """计算两张图像的差异"""
    img1 = Image.open(img1_path).convert('RGB')
    img2 = Image.open(img2_path).convert('RGB')

    arr1 = np.array(img1, dtype=np.int16)
    arr2 = np.array(img2, dtype=np.int16)

    # 计算差异
    diff = np.abs(arr1 - arr2)

    # 突出显示差异
    mask = np.any(diff > threshold, axis=2)
    result = np.zeros_like(arr1)
    result[mask] = [255, 0, 0]  # 差异部分标红
    result[~mask] = arr1[~mask]

    result_img = Image.fromarray(result.astype(np.uint8))
    result_img.save(output_path)
    print(f"差异分析完成: {output_path}")

# 使用示例
image_diff('original.png', 'modified.png', 'diff.png')
```

---

## 实战案例

### 案例1: 提取PNG块中的隐藏数据

```python
from PIL import Image
import struct

def extract_png_chunks(png_path):
    """提取PNG文件的所有数据块"""
    with open(png_path, 'rb') as f:
        # 验证PNG签名
        signature = f.read(8)
        if signature != b'\x89PNG\r\n\x1a\n':
            print("不是有效的PNG文件")
            return

        print("PNG数据块列表:")
        print("-" * 50)

        while True:
            # 读取块长度
            length_data = f.read(4)
            if len(length_data) < 4:
                break

            length = struct.unpack('>I', length_data)[0]

            # 读取块类型
            chunk_type = f.read(4).decode('ascii', errors='ignore')

            # 读取块数据
            chunk_data = f.read(length)

            # 读取CRC
            crc = f.read(4)

            print(f"类型: {chunk_type}, 长度: {length} 字节")

            # 检查自定义块(CTF常用隐藏位置)
            if chunk_type not in ['IHDR', 'PLTE', 'IDAT', 'IEND', 'tRNS',
                                  'gAMA', 'cHRM', 'sRGB', 'iCCP', 'tEXt',
                                  'zTXt', 'iTXt', 'bKGD', 'pHYs', 'sBIT',
                                  'sPLT', 'hIST', 'tIME']:
                print(f"  [!] 发现可疑块: {chunk_type}")
                print(f"  数据: {chunk_data[:100]}")

                # 尝试解码为文本
                try:
                    text = chunk_data.decode('utf-8', errors='ignore')
                    if text.isprintable():
                        print(f"  解码文本: {text}")
                except:
                    pass

            if chunk_type == 'IEND':
                break

        # 检查IEND后是否有额外数据
        extra_data = f.read()
        if extra_data:
            print(f"\n[!] IEND块后发现额外数据: {len(extra_data)} 字节")
            print(f"数据: {extra_data[:100]}")

# 使用示例
extract_png_chunks('challenge.png')
```

### 案例2: 图像频谱分析

```python
from PIL import Image
import numpy as np
import matplotlib.pyplot as plt

def frequency_analysis(image_path, output_path='spectrum.png'):
    """对图像进行频谱分析"""
    img = Image.open(image_path).convert('L')
    arr = np.array(img)

    # 傅里叶变换
    f_transform = np.fft.fft2(arr)
    f_shift = np.fft.fftshift(f_transform)

    # 计算幅度谱
    magnitude_spectrum = 20 * np.log(np.abs(f_shift) + 1)

    # 可视化
    plt.figure(figsize=(12, 6))

    plt.subplot(121)
    plt.imshow(arr, cmap='gray')
    plt.title('原始图像')
    plt.axis('off')

    plt.subplot(122)
    plt.imshow(magnitude_spectrum, cmap='gray')
    plt.title('频谱')
    plt.axis('off')

    plt.tight_layout()
    plt.savefig(output_path)
    print(f"频谱分析结果: {output_path}")

# 使用示例
frequency_analysis('challenge.png')
```

### 案例3: 像素值统计分析

```python
from PIL import Image
import numpy as np
import matplotlib.pyplot as plt

def pixel_statistics(image_path):
    """统计图像像素值分布"""
    img = Image.open(image_path).convert('RGB')
    arr = np.array(img)

    # 分离通道
    r, g, b = arr[:, :, 0], arr[:, :, 1], arr[:, :, 2]

    # 绘制直方图
    plt.figure(figsize=(15, 5))

    plt.subplot(131)
    plt.hist(r.ravel(), bins=256, range=(0, 256), color='red', alpha=0.7)
    plt.title('Red Channel')
    plt.xlabel('Pixel Value')
    plt.ylabel('Frequency')

    plt.subplot(132)
    plt.hist(g.ravel(), bins=256, range=(0, 256), color='green', alpha=0.7)
    plt.title('Green Channel')
    plt.xlabel('Pixel Value')
    plt.ylabel('Frequency')

    plt.subplot(133)
    plt.hist(b.ravel(), bins=256, range=(0, 256), color='blue', alpha=0.7)
    plt.title('Blue Channel')
    plt.xlabel('Pixel Value')
    plt.ylabel('Frequency')

    plt.tight_layout()
    plt.savefig('pixel_statistics.png')
    print("像素统计完成: pixel_statistics.png")

    # 查找异常
    print("\n统计信息:")
    print(f"Red   - Min: {r.min()}, Max: {r.max()}, Mean: {r.mean():.2f}, Std: {r.std():.2f}")
    print(f"Green - Min: {g.min()}, Max: {g.max()}, Mean: {g.mean():.2f}, Std: {g.std():.2f}")
    print(f"Blue  - Min: {b.min()}, Max: {b.max()}, Mean: {b.mean():.2f}, Std: {b.std():.2f}")

# 使用示例
pixel_statistics('challenge.png')
```

---

## 高级技巧

### 图像元数据操作

```python
from PIL import Image
from PIL.ExifTags import TAGS

def extract_exif(image_path):
    """提取EXIF元数据"""
    img = Image.open(image_path)
    exif_data = img._getexif()

    if exif_data:
        print("EXIF数据:")
        for tag_id, value in exif_data.items():
            tag = TAGS.get(tag_id, tag_id)
            print(f"{tag}: {value}")
    else:
        print("没有EXIF数据")

# 使用示例
extract_exif('photo.jpg')
```

### 自定义图像格式处理

```python
from PIL import Image
import struct

def parse_custom_format(file_path):
    """解析自定义图像格式"""
    with open(file_path, 'rb') as f:
        # 读取头部
        magic = f.read(4)
        width, height = struct.unpack('<II', f.read(8))

        print(f"Magic: {magic}")
        print(f"尺寸: {width}x{height}")

        # 读取像素数据
        pixel_data = f.read(width * height * 3)

        # 创建图像
        img = Image.frombytes('RGB', (width, height), pixel_data)
        img.save('reconstructed.png')
        print("重建完成: reconstructed.png")

# 使用示例
parse_custom_format('custom.dat')
```

---

## 常见问题解决

### 问题1: 图像无法打开

```python
from PIL import Image

try:
    img = Image.open('corrupted.png')
except Exception as e:
    print(f"错误: {e}")

    # 尝试修复
    with open('corrupted.png', 'rb') as f:
        data = f.read()
        # 检查文件头
        if not data.startswith(b'\x89PNG'):
            print("PNG文件头损坏")
```

### 问题2: 内存不足

```python
from PIL import Image

# 使用lazy loading
Image.MAX_IMAGE_PIXELS = None  # 移除像素限制

# 分块处理大图像
def process_large_image(path):
    img = Image.open(path)
    width, height = img.size
    chunk_size = 1000

    for y in range(0, height, chunk_size):
        for x in range(0, width, chunk_size):
            box = (x, y, min(x + chunk_size, width), min(y + chunk_size, height))
            chunk = img.crop(box)
            # 处理chunk...
```

---

## 参考资源

### 官方文档
- **Pillow官网**: https://python-pillow.org/
- **文档**: https://pillow.readthedocs.io/
- **GitHub**: https://github.com/python-pillow/Pillow

### CTF工具
- **Stegsolve**: 图像隐写分析工具
- **zsteg**: LSB隐写检测
- **steghide**: 图像隐写工具

### 学习资源
- **CTF Wiki**: https://ctf-wiki.org/misc/picture/
- **隐写术教程**: https://0xrick.github.io/lists/stego/

---

**文档版本**: v1.0
**更新日期**: 2025-01
**适用版本**: Pillow v10.0+
