# CyberChef 详细使用指南

## 📋 目录
- [CyberChef简介](#cyberchef简介)
- [安装与启动](#安装与启动)
- [界面介绍](#界面介绍)
- [基本操作](#基本操作)
- [功能模块详解](#功能模块详解)
- [CTF实战应用](#ctf实战应用)
- [典型案例分析](#典型案例分析)
- [高级技巧](#高级技巧)
- [常用Recipe收藏](#常用recipe收藏)

---

## 🔍 CyberChef简介

### 什么是CyberChef？

**CyberChef** 是由英国政府通信总部（GCHQ）开发的一款开源Web应用，被称为"网络安全的瑞士军刀"。它能够对数据进行各种编码、解码、加密、解密、格式化、分析等操作。

### 核心特点

- **300+ 操作功能**：涵盖编码、加密、压缩、数据分析等
- **可视化拖拽**：直观的"配方（Recipe）"操作方式
- **链式处理**：多个操作可串联执行
- **实时预览**：即时查看每步操作结果
- **完全离线**：单一HTML文件，无需网络
- **开源免费**：MIT许可证，完全开源

### 主要用途

✅ **CTF竞赛**：快速解密编码混淆的flag
✅ **逆向工程**：分析二进制数据、提取字符串
✅ **渗透测试**：编码payload、绕过WAF
✅ **取证分析**：处理日志、解析数据
✅ **密码学研究**：测试加密算法、破解简单加密
✅ **数据处理**：格式转换、数据清洗

---

## 💻 安装与启动

### 离线版安装（推荐）

**文件位置**：
```
/Users/ss/Documents/github/CyberChef/CyberChef_v10.19.4.html
```

**启动方式**：

#### 方法1：应用程序启动（最便捷）
```bash
# 从Applications启动
open /Applications/CyberChef.app

# 或使用Spotlight搜索"CyberChef"
```

#### 方法2：浏览器直接打开
```bash
# 默认浏览器
open /Users/ss/Documents/github/CyberChef/CyberChef_v10.19.4.html

# Chrome浏览器
open -a "Google Chrome" /Users/ss/Documents/github/CyberChef/CyberChef_v10.19.4.html

# Safari浏览器
open -a "Safari" /Users/ss/Documents/github/CyberChef/CyberChef_v10.19.4.html
```

#### 方法3：Launchpad快速启动
1. 打开Launchpad（F4或触控板四指捏合）
2. 搜索"CyberChef"
3. 点击图标启动

### 在线版

**官方地址**：https://gchq.github.io/CyberChef/

**优点**：
- 无需下载安装
- 自动更新到最新版本
- 可分享配方链接

**缺点**：
- 需要网络连接
- 数据经过在线处理
- 比赛现场可能网络不稳定

**建议**：CTF竞赛优先使用离线版！

---

## 🎨 界面介绍

### 主界面布局

```
┌─────────────────────────────────────────────────────────────┐
│  CyberChef Logo                    [Save] [Load] [Clear]    │
├──────────────┬────────────────────┬─────────────────────────┤
│              │                    │                         │
│   Operations │      Recipe        │        Output          │
│   (操作列表)  │     (配方区)        │       (输出区)          │
│              │                    │                         │
│  • Data      │  1. From Base64    │  Hello World           │
│  • Encryption│  2. ROT13          │                         │
│  • Encoding  │  3. To Hex         │  [Bake!] [Auto Bake]   │
│  • Compression│                   │                         │
│  • ...       │  [Clear Recipe]    │                         │
│              │                    │                         │
├──────────────┴────────────────────┴─────────────────────────┤
│  Input (输入区)                                              │
│  SGVsbG8gV29ybGQ=                                           │
└─────────────────────────────────────────────────────────────┘
```

### 四大核心区域

#### 1. **Operations（操作列表）**
- 左侧边栏，包含所有可用操作
- 分类清晰：Data、Encryption、Encoding等
- 搜索框可快速定位功能
- 拖拽到Recipe区使用

#### 2. **Recipe（配方区）**
- 中间区域，当前操作流程
- 支持多个操作串联
- 可调整操作顺序
- 每个操作可配置参数

#### 3. **Input（输入区）**
- 底部区域，输入原始数据
- 支持文本、文件、URL
- 可粘贴、拖拽输入

#### 4. **Output（输出区）**
- 右侧区域，显示处理结果
- 实时预览（Auto Bake）
- 可下载、复制结果
- 显示执行时间和错误

### 工具栏功能

| 按钮 | 功能 | 说明 |
|------|------|------|
| **Bake!** | 执行配方 | 手动执行当前Recipe |
| **Auto Bake** | 自动执行 | 输入改变时自动执行 |
| **Save** | 保存配方 | 保存当前Recipe为JSON |
| **Load** | 加载配方 | 导入已保存的Recipe |
| **Clear** | 清空 | 清空Recipe和输入 |

---

## 🔧 基本操作

### 1. 创建第一个Recipe

**示例：Base64解码**

1. **输入数据**
   ```
   在Input区输入：SGVsbG8gQ1RG
   ```

2. **选择操作**
   - 在Operations搜索框输入"base64"
   - 找到"From Base64"
   - 拖拽到Recipe区

3. **执行操作**
   - 点击"Bake!"按钮
   - 或启用"Auto Bake"自动执行

4. **查看结果**
   ```
   Output区显示：Hello CTF
   ```

### 2. 链式操作

**示例：Base64解码 → ROT13 → 转十六进制**

```
Recipe:
  1. From Base64
  2. ROT13 (Amount: 13)
  3. To Hex (Delimiter: Space)
```

**操作步骤**：
1. 依次拖拽三个操作到Recipe区
2. 调整ROT13的Amount参数为13
3. 设置To Hex的分隔符为空格
4. 点击Bake执行

### 3. 调整操作顺序

- **上下拖动**：拖动操作卡片调整顺序
- **禁用操作**：点击操作左侧的开关图标
- **删除操作**：点击操作右侧的×图标
- **复制操作**：拖动时按住Alt键

### 4. 配置操作参数

每个操作都有可配置的参数：

- **文本输入框**：如密钥、分隔符
- **下拉菜单**：如字符编码、算法类型
- **数字输入**：如ROT位移量、密钥长度
- **开关选项**：如是否显示空格

### 5. 保存和分享配方

#### 保存配方

```bash
# 1. 点击"Save"按钮
# 2. 选择"Save to file"
# 3. 下载JSON文件
```

#### 加载配方

```bash
# 1. 点击"Load"按钮
# 2. 选择"Load recipe from file"
# 3. 上传之前保存的JSON文件
```

#### 分享配方（在线版）

```bash
# 在线版会生成唯一URL
https://gchq.github.io/CyberChef/#recipe=From_Base64()ROT13(true,true,13)
```

---

## 📚 功能模块详解

### 一、数据格式（Data Format）

#### 1. **To/From Base64**
- **用途**：Base64编码/解码
- **CTF场景**：最常见的编码方式
- **参数**：
  - Alphabet：标准/URL安全/文件名安全
  - Remove non-alphabet chars：移除非字母字符

**示例**：
```
Input: Hello CTF
Operation: To Base64
Output: SGVsbG8gQ1RG
```

#### 2. **To/From Hex**
- **用途**：十六进制编码/解码
- **参数**：
  - Delimiter：分隔符（空格、冒号、无）
  - Byte Length：每次读取字节数

**示例**：
```
Input: Hello
Operation: To Hex (Delimiter: Space)
Output: 48 65 6c 6c 6f
```

#### 3. **From Hexdump**
- **用途**：解析hexdump格式输出
- **场景**：处理二进制文件dump

**示例**：
```
Input:
00000000  48 65 6c 6c 6f 20 43 54  46 21 0a                 |Hello CTF!.|

Operation: From Hexdump
Output: Hello CTF!
```

#### 4. **URL Encode/Decode**
- **用途**：URL编码/解码
- **参数**：Encode all special chars

**示例**：
```
Input: Hello CTF!
Operation: URL Encode
Output: Hello%20CTF%21
```

#### 5. **HTML Entity Encode/Decode**
- **用途**：HTML实体编码/解码
- **场景**：处理Web页面数据

**示例**：
```
Input: <script>alert('XSS')</script>
Operation: HTML Entity Encode
Output: &lt;script&gt;alert(&#39;XSS&#39;)&lt;/script&gt;
```

#### 6. **Unicode**
- **To/From Unicode**：Unicode编码转换
- **Unescape Unicode Characters**：解码\uXXXX格式

**示例**：
```
Input: Hello
Operation: To Unicode (Encoding: \u)
Output: \u0048\u0065\u006c\u006c\u006f
```

---

### 二、加密解密（Encryption/Encoding）

#### 1. **AES加密/解密**

**操作**：`AES Encrypt` / `AES Decrypt`

**参数**：
- Key：密钥（十六进制或UTF8）
- IV：初始化向量
- Mode：加密模式（CBC、ECB、CTR、GCM等）
- Input/Output format：输入输出格式

**示例**：
```
Input: SecretMessage
Key: 0123456789abcdef0123456789abcdef
IV: 0123456789abcdef
Mode: CBC
Operation: AES Encrypt
Output: (加密后的Base64或十六进制)
```

**CTF提示**：
- ECB模式没有IV
- Key长度：128位(16字节)、192位(24字节)、256位(32字节)
- 注意Padding模式

#### 2. **DES/Triple DES**

**操作**：`DES Encrypt` / `Triple DES Encrypt`

**参数**：
- Key：密钥
- Mode：ECB/CBC
- IV：初始化向量（CBC模式需要）

**示例**：
```
Operation: DES Encrypt
Key: 0123456789abcdef
Mode: ECB
Input: HelloCTF
```

#### 3. **RSA**

**操作**：`RSA Encrypt` / `RSA Decrypt`

**参数**：
- Public/Private Key：PEM格式密钥
- Encryption Scheme：PKCS#1或OAEP

**CTF应用**：
- 小指数攻击
- 公钥解析
- 模数分解

#### 4. **Blowfish**

**操作**：`Blowfish Encrypt` / `Blowfish Decrypt`

**参数**：
- Key：密钥（4-56字节）
- Mode：ECB/CBC/CFB等
- IV：初始化向量

---

### 三、哈希函数（Hashing）

#### 常用哈希算法

| 操作 | 输出长度 | CTF用途 |
|------|---------|---------|
| **MD5** | 128位(32字符) | 常见哈希、碰撞攻击 |
| **SHA1** | 160位(40字符) | Git、文件校验 |
| **SHA2 (224/256/384/512)** | 可变 | 现代哈希标准 |
| **SHA3** | 可变 | 最新SHA标准 |
| **BLAKE2** | 可变 | 高性能哈希 |

**示例**：
```
Input: Hello CTF
Operation: MD5
Output: 8b1a9953c4611296a827abf8c47804d7

Operation: SHA256
Output: 3c7a8c6b9d5f2e1a4b8c7d6e5f4a3b2c1d0e9f8a7b6c5d4e3f2a1b0c9d8e7f6
```

#### HMAC系列

**操作**：`HMAC (MD5/SHA1/SHA256/etc)`

**参数**：
- Key：密钥
- Hash function：使用的哈希算法

**用途**：消息认证码、API签名验证

**示例**：
```
Input: message
Key: secret
Operation: HMAC (SHA256)
```

#### CRC校验

**操作**：`CRC32` / `CRC16`

**用途**：文件完整性校验、网络传输校验

---

### 四、编码转换（Encoding）

#### 1. **ROT13 / ROT47**

**操作**：`ROT13` / `ROT47`

**参数**：
- Amount：位移量（标准ROT13为13）
- Lowercase/Uppercase：是否转换大小写

**示例**：
```
Input: Hello
Operation: ROT13
Output: Uryyb
```

**CTF技巧**：尝试所有位移量（0-25）寻找可读文本

#### 2. **XOR**

**操作**：`XOR` / `XOR Brute Force`

**参数**：
- Key：XOR密钥（十六进制或UTF8）
- Key Type：单字节/多字节
- Output format：输出格式

**示例**：
```
Input: Hello
Key: 0x41 (单字节)
Operation: XOR
Output: )$--.
```

**XOR Brute Force**：
```
Operation: XOR Brute Force
Key length: 1
Output: 显示所有可能的单字节XOR结果
```

**CTF提示**：
- 单字节XOR：仅256种可能
- 已知明文攻击：明文⊕密文=密钥
- 密钥重用攻击

#### 3. **Base系列**

- **Base32**：使用A-Z和2-7
- **Base58**：比特币地址编码
- **Base62**：使用0-9、a-z、A-Z
- **Base85/Ascii85**：更高效的编码

**示例 - Base58**：
```
Input: Hello World
Operation: To Base58
Output: JxF12TrwUP45BMd
```

#### 4. **Punycode**

**用途**：国际化域名编码（IDN）

**示例**：
```
Input: münchen.de
Operation: To Punycode
Output: mnchen-3ya.de
```

---

### 五、压缩与解压（Compression）

#### 1. **Gzip / Gunzip**

**操作**：`Gzip` / `Gunzip`

**用途**：处理.gz压缩文件

**示例**：
```
Input: (二进制gzip数据)
Operation: Gunzip
Output: 解压后的内容
```

#### 2. **Zip / Unzip**

**操作**：`Unzip`

**参数**：
- Password：解压密码
- File filter：文件过滤器

**CTF应用**：
- 提取ZIP文件内容
- 查看ZIP文件结构
- 密码保护的ZIP

#### 3. **Bzip2 / Raw Deflate / Raw Inflate**

**用途**：处理各种压缩格式

---

### 六、数据提取（Extraction）

#### 1. **Extract Strings**

**操作**：`Strings`

**参数**：
- Encoding：字符编码（ASCII、UTF8、UTF16等）
- Minimum length：最小字符串长度
- Display as：显示格式

**用途**：从二进制文件提取可读字符串

**CTF应用**：
- 分析二进制文件
- 查找隐藏的flag
- 提取URL、邮箱等

**示例**：
```
Input: (二进制数据混合文本)
Operation: Strings
Minimum length: 4
Output: 所有长度≥4的可读字符串
```

#### 2. **Extract URLs**

**操作**：`Extract URLs`

**用途**：从文本中提取所有URL

#### 3. **Extract IP addresses**

**操作**：`Extract IP addresses`

**用途**：提取IPv4和IPv6地址

#### 4. **Extract Email addresses**

**操作**：`Extract email addresses`

**用途**：从文本提取邮箱地址

#### 5. **Regular expression**

**操作**：`Regular expression`

**参数**：
- Regex：正则表达式
- Output format：输出格式
- Flags：i（忽略大小写）、m（多行）、s（单行）

**示例**：
```
Input: flag{hello_world_123}
Regex: flag\{(.+?)\}
Output: hello_world_123
```

---

### 七、图像处理（Image）

#### 1. **Render Image**

**操作**：`Render Image`

**用途**：显示图片数据

**支持格式**：PNG、JPEG、GIF、BMP、SVG

#### 2. **Detect File Type**

**操作**：`Detect File Type`

**用途**：识别文件类型（Magic Number）

**示例**：
```
Input: (二进制文件头)
89 50 4E 47 0D 0A 1A 0A...
Output: PNG image
```

#### 3. **Extract LSB / LSB Steganography**

**操作**：`Extract LSB`

**用途**：图片隐写术分析

**参数**：
- Red、Green、Blue：提取通道
- LSB位数：提取最低几位

---

### 八、网络分析（Networking）

#### 1. **Parse IP Range**

**操作**：`Parse IP Range`

**用途**：解析IP范围，生成IP列表

**示例**：
```
Input: 192.168.1.1-192.168.1.10
Output: (列出所有IP)
```

#### 2. **Parse IPv6 address**

**操作**：`Parse IPv6 address`

**用途**：解析和格式化IPv6地址

#### 3. **Convert distance**

**操作**：`HTTP request` / `DNS over HTTPS`

**用途**：发起网络请求（仅在线版）

---

### 九、公钥和证书（Public Key）

#### 1. **Parse X.509 certificate**

**操作**：`Parse X.509 certificate`

**用途**：解析SSL/TLS证书

**输入格式**：PEM或DER

**输出信息**：
- Subject、Issuer
- 有效期
- 公钥信息
- 扩展字段

#### 2. **Parse SSH Host Key**

**操作**：`Parse SSH Host Key`

**用途**：解析SSH主机密钥

#### 3. **PEM to Hex / Hex to PEM**

**用途**：PEM格式转换

---

### 十、其他实用功能

#### 1. **Magic**

**操作**：`Magic`

**说明**：CyberChef的智能分析功能

**功能**：
- 自动检测输入数据类型
- 推荐合适的操作
- 尝试多种解码方式

**使用场景**：
- 不确定数据编码方式
- 快速尝试常见解码
- CTF盲解题目

**示例**：
```
Input: SGVsbG8gQ1RG
Operation: Magic
Output:
  可能是Base64编码
  建议操作：From Base64
  解码结果：Hello CTF
```

#### 2. **Fork / Merge**

**操作**：`Fork` / `Merge`

**用途**：分支处理，对多段数据分别处理后合并

**示例**：
```
Recipe:
  1. Fork (Split delimiter: \n)
  2. From Base64
  3. Merge
```

#### 3. **Register**

**操作**：`Register`

**用途**：保存中间结果，供后续操作使用

**应用**：复杂的多步骤处理

#### 4. **Subsection**

**操作**：`Subsection`

**用途**：仅对输入的一部分进行处理

**参数**：
- Section：选择范围（字节偏移）
- Operations：在该范围内执行的操作

#### 5. **Comment**

**操作**：`Comment`

**用途**：在Recipe中添加注释说明

---

## 🎯 CTF实战应用

### 场景1：多重编码识别与解密

**题目类型**：Misc、Crypto

**常见编码组合**：
- Base64 → Hex → Base64
- URL编码 → Base64 → ROT13
- Hex → XOR → Base64

**解题思路**：

1. **使用Magic自动识别**
   ```
   Input: (未知编码数据)
   Operation: Magic
   → 查看推荐的解码方式
   ```

2. **逐层解码**
   ```
   Recipe:
     1. From Base64
     2. From Hex
     3. ROT13
     4. From Base64
   ```

3. **寻找Flag特征**
   - 搜索"flag{"
   - 搜索"ctf{"
   - 搜索"FLAG"

### 场景2：XOR密钥破解

**题目类型**：Crypto

**情况A：单字节XOR**

```
Recipe:
  1. From Hex
  2. XOR Brute Force
     Key length: 1
     Output as: Raw
  3. Find / Replace
     Find: flag{
```

**情况B：已知明文攻击**

```
已知：
  密文: 1a2b3c4d5e...
  明文开头: flag{

Recipe:
  1. XOR (使用已知明文)
  → 获取密钥
  2. 使用密钥XOR完整密文
```

**情况C：多字节XOR（字典爆破）**

```
Recipe:
  1. From Hex
  2. XOR Brute Force
     Key length: 4-8
     Sample length: 100
```

### 场景3：AES解密

**题目类型**：Crypto

**已知信息**：
- 密文（Base64或Hex）
- 密钥
- 可能的IV
- 加密模式

**解题步骤**：

```
Recipe:
  1. From Base64 (或 From Hex)
  2. AES Decrypt
     Key: (已知密钥，转为Hex格式)
     IV: (如果是CBC模式)
     Mode: CBC/ECB/CTR
     Input format: Raw
     Output format: Raw
```

**常见错误**：
- Key格式错误（UTF8 vs Hex）
- IV错误或缺失
- Mode选择错误
- Padding不匹配

**调试技巧**：
- 尝试所有常见Mode
- 尝试全0的IV
- 检查密钥长度（16/24/32字节）

### 场景4：图片隐写

**题目类型**：Misc、Stego

**LSB隐写提取**：

```
Recipe:
  1. Render Image (查看图片)
  2. Extract LSB
     Red LSB: true
     Green LSB: true
     Blue LSB: true
     Order: RGB
  3. From Hex (如果是十六进制)
  4. Gunzip (如果压缩了)
```

**文件头分析**：

```
Recipe:
  1. To Hex
  2. Take first bytes (16)
  → 查看文件魔术头
  3. Detect File Type
```

### 场景5：流量包分析

**题目类型**：Misc、Web

**HTTP流提取**：

```
Recipe:
  1. Extract URLs
  2. Filter (包含flag关键字)

或

  1. Regular expression
     Regex: flag\{[^}]+\}
```

**Base64数据提取**：

```
Recipe:
  1. Regular expression
     Regex: [A-Za-z0-9+/]{20,}={0,2}
  2. From Base64
```

### 场景6：JWT分析

**题目类型**：Web

**解析JWT**：

```
Recipe:
  1. JWT Decode
  → 查看Header和Payload

  2. 修改Payload
  3. JWT Sign (如果知道密钥)
```

**JWT爆破**（需结合其他工具）：

```
Recipe:
  1. JWT Decode (查看算法)
  2. HMAC (尝试弱密钥)
```

### 场景7：编码混淆

**题目类型**：Misc

**Brainfuck解密**：

```
Input: ++++++[>++++++<-]>...
Recipe:
  1. Brainfuck (需使用在线版或脚本)
```

**Ook解密**：

```
Input: Ook. Ook? Ook. Ook...
Recipe:
  → 使用专门的Ook解释器
```

### 场景8：文件格式转换

**ZIP文件分析**：

```
Recipe:
  1. From Hex (如果是十六进制)
  2. Unzip
     → 查看文件列表
     → 提取文件内容
```

**压缩数据解压**：

```
Recipe:
  1. From Base64
  2. Gunzip / Bzip2 Decompress / Zlib Inflate
  3. From Hex (可能还有编码)
```

---

## 💡 典型案例分析

### 案例1：多层Base64嵌套

**题目描述**：给定一个长字符串，flag被多层Base64编码

**原始数据**：
```
VTFObGJHOGdRMVJHSUNFaA==
```

**解题过程**：

```
Recipe:
  1. From Base64
  Output: U1NlbGcgQ1RGICEh

  2. From Base64 (再次)
  Output: SGVsbG8gQ1RGICEh

  3. From Base64 (第三次)
  Output: Hello CTF !!
```

**优化方案**：使用循环

```
Recipe:
  1. Label ('loop')
  2. From Base64
  3. Jump (Label: 'loop', Max jumps: 10)
     Condition: 输入包含"flag{"
```

### 案例2：ROT13+Base64组合

**题目描述**：flag先ROT13再Base64编码

**原始数据**：
```
syntz7uryyb_jbeyq321=
```

**错误尝试**：直接Base64解码（失败）

**正确解法**：

```
Recipe:
  1. ROT13
  Output: flag{hello_world321}
```

**分析**：
- Base64字符集：A-Z、a-z、0-9、+、/、=
- ROT13后可能产生非Base64字符
- 先尝试ROT13

### 案例3：XOR单字节爆破

**题目描述**：flag使用单字节XOR加密

**原始数据（Hex）**：
```
29292e2d2d382e3f382c212038272c293821
```

**解题步骤**：

```
Recipe:
  1. From Hex
  2. XOR Brute Force
     Key length: 1
     Sample length: 100
     Output: Raw
```

**结果分析**：
```
Key: 0x48
Output: flag{xor_is_easy}
```

**识别技巧**：
- 查找可读的ASCII文本
- 寻找"flag{"模式
- 高频字符（e、t、a、空格）

### 案例4：AES-CBC解密

**题目描述**：使用AES-CBC加密flag

**已知信息**：
```
密文(Base64): U2FsdGVkX1+6J3m...
Key: MySecretKey12345
IV: 1234567890123456
```

**解题过程**：

```
Recipe:
  1. From Base64

  2. AES Decrypt
     Key: MySecretKey12345 (UTF8)
     IV: 1234567890123456 (UTF8)
     Mode: CBC
     Input: Raw
     Output: Raw

  Output: flag{aes_crypto_fun}
```

**调试步骤**：
1. 检查密文格式（Base64/Hex）
2. 确认Key长度（16/24/32字节）
3. 尝试不同Mode（CBC最常见）
4. 检查IV（可能是全0或省略）

### 案例5：图片LSB隐写

**题目描述**：flag隐藏在图片的LSB中

**解题步骤**：

```
Recipe:
  1. Render Image
     → 查看图片（可能有提示）

  2. Extract LSB
     Red LSB: true
     Green LSB: true
     Blue LSB: true
     Order: RGB

  3. From Hex (LSB提取结果可能是十六进制)

  4. Strings (提取可读字符串)
     Minimum length: 4

  Output: flag{1sb_st3g0}
```

**进阶分析**：
- 仅提取单个颜色通道
- 调整LSB位数（1-8位）
- 查看Alpha通道
- 检查图片元数据（EXIF）

### 案例6：JWT伪造

**题目描述**：修改JWT的payload并重新签名

**原始JWT**：
```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyIjoiZ3Vlc3QifQ.xxx
```

**目标**：将user改为admin

**解题步骤**：

```
Step 1: 解析JWT
Recipe:
  1. JWT Decode

Output:
  Header: {"alg":"HS256","typ":"JWT"}
  Payload: {"user":"guest"}

Step 2: 修改Payload
  修改为: {"user":"admin"}

Step 3: 重新签名
Recipe:
  1. JWT Sign
     Key: (已知或爆破的密钥)
     Algorithm: HS256

  Output: 新的JWT
```

### 案例7：Gzip压缩数据

**题目描述**：flag被Gzip压缩后Base64编码

**原始数据**：
```
H4sIAAAAAAAAA0vOz01VKM7IL1VIzCvJLEoFABmWoU4OAAAA
```

**解题过程**：

```
Recipe:
  1. From Base64

  2. Gunzip

  Output: flag{c0mpr3ss10n}
```

**识别技巧**：
- Gzip文件头：1F 8B 08
- Base64后识别：H4sI开头（常见）
- 使用Detect File Type识别

### 案例8：正则表达式提取

**题目描述**：从大量数据中提取特定格式的flag

**原始数据**：
```
asdlkfjasldfj flag{h3ll0_w0rld} asdfasdfasdf
other data here...
flag{f4k3_fl4g} more data
the real one: flag{r34l_fl4g_h3r3}
```

**解题过程**：

```
Recipe:
  1. Regular expression
     Regex: flag\{[a-z0-9_]{10,}\}
     Output format: List matches

  2. Find / Replace
     Find: flag\{f4k3.*?\}
     Replace: (空)

  Output: flag{r34l_fl4g_h3r3}
```

**常用正则**：
```
Flag: flag\{[^}]+\}
Email: [a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}
URL: https?://[^\s]+
IP: \d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}
```

---

## 🚀 高级技巧

### 1. 批量处理（Fork）

**场景**：处理多个相同编码的数据

```
Input:
line1_base64_data
line2_base64_data
line3_base64_data

Recipe:
  1. Fork (Split delimiter: \n)
  2. From Base64
  3. Merge
```

### 2. 条件跳转（Jump）

**场景**：重复执行直到满足条件

```
Recipe:
  1. Label ('decode')
  2. From Base64
  3. Jump (Label: 'decode', Max jumps: 10)
     Condition: 输入 不包含 "flag{"
```

### 3. 使用Register保存中间结果

**场景**：需要多次使用中间结果

```
Recipe:
  1. From Base64
  2. Register (Store in: $R0)
  3. MD5
  4. To Hex
  5. Merge ($R0)
     → 同时显示原文和MD5
```

### 4. 子区间处理（Subsection）

**场景**：只处理数据的一部分

```
Recipe:
  1. Subsection (0-10)
     Operations:
       - From Hex
       - XOR (key: 0x41)
  2. Subsection (10-20)
     Operations:
       - From Base64
```

### 5. 自动化Recipe分享

**创建Recipe库**：

```json
{
  "name": "Base64+ROT13",
  "recipe": [
    {"op": "From Base64", "args": []},
    {"op": "ROT13", "args": [true, true, 13]}
  ]
}
```

**常用Recipe集合**：
- Base64解码链
- XOR爆破模板
- AES解密模板
- 数据提取模板

### 6. 快捷键

| 快捷键 | 功能 |
|--------|------|
| **Ctrl/Cmd + Enter** | Bake（执行） |
| **Ctrl/Cmd + Shift + S** | 保存Recipe |
| **Ctrl/Cmd + Shift + L** | 加载Recipe |
| **Ctrl/Cmd + Shift + C** | 清空Recipe |
| **Esc** | 取消当前操作 |

### 7. 性能优化

**处理大文件**：
- 禁用Auto Bake
- 使用Subsection处理片段
- 调整Sample length

**避免浏览器卡死**：
- 分段处理
- 使用外部工具预处理
- 限制循环次数

### 8. 组合使用外部工具

**CyberChef + Python**：
```python
import base64
import requests

# CyberChef在线API
url = "https://gchq.github.io/CyberChef/"
recipe = [{"op": "From Base64"}, {"op": "ROT13"}]
```

**CyberChef + BurpSuite**：
- 拦截请求
- 复制数据到CyberChef
- 解码/编码
- 粘贴回BurpSuite

### 9. 调试复杂Recipe

**步骤化调试**：
1. 逐个添加操作
2. 检查每步输出
3. 使用Comment标注
4. 禁用可疑操作

**常见错误**：
- 编码格式不匹配
- 密钥格式错误
- 操作顺序错误
- 参数配置错误

---

## 📖 常用Recipe收藏

### 1. 多层Base64解码

```json
{
  "name": "Multi-Layer Base64 Decode",
  "recipe": [
    {"op": "Label", "args": ["loop"]},
    {"op": "From Base64", "args": ["A-Za-z0-9+/=", true]},
    {"op": "Jump", "args": ["loop", 10, "flag{"]}
  ]
}
```

### 2. XOR单字节爆破

```json
{
  "name": "XOR Single Byte Bruteforce",
  "recipe": [
    {"op": "From Hex", "args": ["Auto"]},
    {"op": "XOR Brute Force", "args": [1, 100, false, "", false, false, false]}
  ]
}
```

### 3. 提取所有编码字符串

```json
{
  "name": "Extract All Encoded Strings",
  "recipe": [
    {"op": "Regular expression", "args": ["[A-Za-z0-9+/]{20,}={0,2}", true, true, false, false, false, "List matches"]},
    {"op": "Fork", "args": ["\\n", "\\n", false]},
    {"op": "From Base64", "args": ["A-Za-z0-9+/=", true]},
    {"op": "Merge", "args": []}
  ]
}
```

### 4. AES-CBC标准解密

```json
{
  "name": "AES-CBC Decrypt Template",
  "recipe": [
    {"op": "From Base64", "args": ["A-Za-z0-9+/=", true]},
    {"op": "AES Decrypt", "args": [
      {"option": "Hex", "string": "00112233445566778899aabbccddeeff"},
      {"option": "Hex", "string": "00000000000000000000000000000000"},
      "CBC", "Raw", "Raw"
    ]}
  ]
}
```

### 5. 图片LSB提取

```json
{
  "name": "Image LSB Extraction",
  "recipe": [
    {"op": "Render Image", "args": ["Raw"]},
    {"op": "Extract LSB", "args": [[0,0,0], "RGB", "Row", 1]},
    {"op": "From Hex", "args": ["Auto"]},
    {"op": "Strings", "args": ["All printable chars", 4, true]}
  ]
}
```

### 6. JWT解析

```json
{
  "name": "JWT Decode and Verify",
  "recipe": [
    {"op": "JWT Decode", "args": []}
  ]
}
```

### 7. URL参数提取

```json
{
  "name": "Extract URL Parameters",
  "recipe": [
    {"op": "URL Decode", "args": []},
    {"op": "Regular expression", "args": ["[?&]([^=]+)=([^&]+)", true, true, false, false, false, "List capture groups"]},
    {"op": "From Base64", "args": ["A-Za-z0-9+/=", true]}
  ]
}
```

### 8. 文件头识别

```json
{
  "name": "File Type Detection",
  "recipe": [
    {"op": "To Hex", "args": ["Space"]},
    {"op": "Take bytes", "args": [0, 16, false]},
    {"op": "Detect File Type", "args": []}
  ]
}
```

### 9. ROT全量爆破

```json
{
  "name": "ROT All Rotations",
  "recipe": [
    {"op": "ROT13", "args": [true, true, 1]},
    {"op": "ROT13", "args": [true, true, 2]},
    ...
    {"op": "ROT13", "args": [true, true, 25]}
  ]
}
```

### 10. 压缩数据自动解压

```json
{
  "name": "Auto Decompress",
  "recipe": [
    {"op": "From Base64", "args": ["A-Za-z0-9+/=", true]},
    {"op": "Gunzip", "args": []},
    {"op": "OR", "args": []},
    {"op": "Bzip2 Decompress", "args": []},
    {"op": "OR", "args": []},
    {"op": "Zlib Inflate", "args": [0, 0, "Adaptive", false, false]}
  ]
}
```

---

## 🎓 学习资源与参考

### 官方资源

- **GitHub仓库**：https://github.com/gchq/CyberChef
- **在线版本**：https://gchq.github.io/CyberChef/
- **Wiki文档**：https://github.com/gchq/CyberChef/wiki

### CTF平台

- **NSSCTF**：https://www.nssctf.cn/
- **CTFHub**：https://www.ctfhub.com/
- **BugKu**：https://ctf.bugku.com/
- **攻防世界**：https://adworld.xctf.org.cn/

### 推荐练习

1. **Crypto类题目**：
   - 各种编码识别
   - 经典加密破解
   - RSA相关题目

2. **Misc类题目**：
   - 文件分析
   - 隐写术
   - 流量分析

3. **Web类题目**：
   - JWT伪造
   - 编码绕过
   - 加密通信

---

## 🔧 故障排除

### 常见问题

#### 1. **操作失败或报错**

**原因**：
- 输入格式不正确
- 参数配置错误
- 密钥格式不匹配

**解决**：
- 检查输入格式（Base64/Hex/Raw）
- 验证操作参数
- 尝试不同编码格式

#### 2. **浏览器卡死**

**原因**：
- 数据量过大
- Auto Bake处理复杂Recipe
- 无限循环

**解决**：
- 禁用Auto Bake
- 分段处理大文件
- 限制Jump次数

#### 3. **解密结果乱码**

**原因**：
- 密钥错误
- 加密模式错误
- 编码转换错误

**解决**：
- 验证密钥正确性
- 尝试所有加密模式
- 检查字符编码设置

#### 4. **无法加载Recipe**

**原因**：
- JSON格式错误
- 版本不兼容
- 操作名称变更

**解决**：
- 验证JSON格式
- 更新到最新版本
- 手动重建Recipe

---

## 📝 最佳实践

### CTF竞赛建议

1. **赛前准备**
   - 安装离线版CyberChef
   - 准备常用Recipe集合
   - 测试所有功能正常

2. **解题流程**
   - 先用Magic快速识别
   - 记录每步操作
   - 保存有效Recipe

3. **团队协作**
   - 分享有效Recipe
   - 记录常见编码模式
   - 建立团队Recipe库

### 安全提醒

⚠️ **数据安全**：
- 敏感数据使用离线版
- 不要在在线版处理机密信息
- 比赛题目优先使用离线版

⚠️ **工具限制**：
- CyberChef不是万能的
- 复杂算法需要专用工具
- 大数据处理性能有限

---

## 📅 更新记录

- **2025-10-24**：创建文档，覆盖CyberChef v10.19.4所有主要功能
- 包含300+操作说明
- 提供CTF实战案例
- 整理常用Recipe模板

---

**文档维护**：上海理工大学信息安全战队
**工具版本**：CyberChef v10.19.4
**最后更新**：2025年10月24日
**适用对象**：CTF竞赛选手、网络安全学习者、渗透测试人员

---

## 🎯 快速参考卡片

### 最常用操作速查

| 类别 | 操作 | 典型用途 |
|------|------|----------|
| 🔤 **编码** | From Base64 | Base64解码 |
| 🔤 **编码** | From Hex | 十六进制解码 |
| 🔤 **编码** | URL Decode | URL解码 |
| 🔒 **加密** | AES Decrypt | AES解密 |
| 🔒 **加密** | XOR Brute Force | XOR暴力破解 |
| 🔒 **加密** | ROT13 | 凯撒密码 |
| #️⃣ **哈希** | MD5 | MD5哈希 |
| #️⃣ **哈希** | SHA256 | SHA256哈希 |
| 📦 **压缩** | Gunzip | Gzip解压 |
| 📦 **压缩** | Unzip | ZIP解压 |
| 🔍 **提取** | Strings | 提取字符串 |
| 🔍 **提取** | Regular expression | 正则提取 |
| 🖼️ **图片** | Extract LSB | LSB隐写提取 |
| 🖼️ **图片** | Render Image | 显示图片 |
| ✨ **其他** | Magic | 智能识别 |
| ✨ **其他** | Detect File Type | 文件类型识别 |

**记住这16个操作，解决80%的CTF Misc题目！**

---

**Good Luck in CTF! 🚩**
