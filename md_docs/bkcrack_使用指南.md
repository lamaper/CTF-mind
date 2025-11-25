# bkcrack 使用指南

## 概述

bkcrack 是一个强大的ZIP文件明文攻击工具，基于Biham和Kocher的已知明文攻击算法。它可以利用已知的明文内容来破解传统ZIP加密，恢复内部密钥并解密整个ZIP文件。

## 安装

### macOS (Homebrew)
```bash
brew install bkcrack
```

### 验证安装
```bash
bkcrack --help
```

## 核心概念

### 明文攻击原理
- **要求**: 需要至少12字节的已知明文内容
- **目标**: 恢复ZIP加密的三个32位内部密钥
- **适用**: ZIP 2.0传统加密 (不适用于AES加密)

### 关键参数
- **密文ZIP** (`-C`): 包含加密文件的ZIP档案
- **明文内容** (`-p`): 已知的明文文件或内容
- **目标文件** (`-c`): 要攻击的加密文件名

## 基本用法

### 1. 明文攻击基础命令
```bash
bkcrack -C encrypted.zip -c target_file.txt -p plaintext_file.txt
```

**参数说明**:
- `-C encrypted.zip`: 加密的ZIP文件
- `-c target_file.txt`: ZIP中要攻击的目标文件
- `-p plaintext_file.txt`: 本地的已知明文文件

### 2. 使用ZIP中的明文
```bash
bkcrack -C encrypted.zip -c secret.txt -P plaintext.zip -p known.txt
```

**参数说明**:
- `-P plaintext.zip`: 包含明文的ZIP文件
- `-p known.txt`: ZIP中对应的明文文件

### 3. 指定偏移量
```bash
bkcrack -C encrypted.zip -c target.txt -p plain.txt -o 10
```

**参数说明**:
- `-o 10`: 明文相对于密文的偏移量(字节)

## 高级用法

### 1. 使用十六进制数据
```bash
bkcrack -C encrypted.zip -c target.txt -x 0 504b0304
```

**参数说明**:
- `-x 0 504b0304`: 在偏移0处指定十六进制明文数据

### 2. 截断明文
```bash
bkcrack -C encrypted.zip -c target.txt -p plain.txt -t 100
```

**参数说明**:
- `-t 100`: 只使用前100字节的明文

### 3. 继续之前的攻击
```bash
bkcrack -C encrypted.zip -c target.txt -p plain.txt --continue-attack 12345
```

## 密钥恢复后的操作

### 1. 创建新的可解密ZIP
```bash
bkcrack -C encrypted.zip -k 12345678 87654321 abcdefab -U decrypted.zip newpassword
```

**参数说明**:
- `-k`: 三个恢复的内部密钥
- `-U decrypted.zip`: 输出的新ZIP文件
- `newpassword`: 为新ZIP设置的密码

### 2. 直接解密文件
```bash
bkcrack -C encrypted.zip -k 12345678 87654321 abcdefab -c target.txt -d output.txt
```

**参数说明**:
- `-d output.txt`: 直接解密输出到文件

### 3. 恢复原始密码
```bash
bkcrack -k 12345678 87654321 abcdefab -r 8 ?a
```

**参数说明**:
- `-r 8`: 密码长度
- `?a`: 字符集(a=所有ASCII可打印字符)

## 实战案例：破解download.zip

### 背景
我们有两个文件：
- `download.zip`: 加密的ZIP文件，包含`secret1.txt`和`secret2.txt`
- `download.txt.zip`: 未加密的ZIP文件，包含已知明文`download.txt`

### 步骤1: 分析目标文件
```bash
# 检查加密ZIP的内容
unzip -l /Users/ss/Downloads/download.zip
```
输出：
```
Archive:  /Users/ss/Downloads/download.zip
  Length      Date    Time    Name
---------  ---------- -----   ----
       42  04-23-2024 14:43   secret1.txt
       44  04-23-2024 14:47   secret2.txt
---------                     -------
       86                     2 files
```

```bash
# 检查明文ZIP的内容
unzip -l /Users/ss/Downloads/download.txt.zip
```
输出：
```
Archive:  /Users/ss/Downloads/download.txt.zip
  Length      Date    Time    Name
---------  ---------- -----   ----
       44  09-28-2025 14:20   download.txt
      427  09-28-2025 14:20   __MACOSX/._download.txt
---------                     -------
       471                     2 files
```

### 步骤2: 提取已知明文
```bash
# 提取明文内容
mkdir -p /tmp/plaintext_attack
unzip /Users/ss/Downloads/download.txt.zip -d /tmp/plaintext_attack/

# 查看明文内容
cat /tmp/plaintext_attack/download.txt
```
输出：
```
Hello, but what you're looking for isn't me.
```

### 步骤3: 关键发现
注意到两个文件的大小：
- `secret2.txt`: 44字节
- `download.txt`: 44字节

大小相同！这意味着`secret2.txt`很可能包含相同的内容。

### 步骤4: 准备明文文件
```bash
# 创建与目标文件同名的明文文件
mkdir -p /tmp/plaintext_files
echo -n "Hello, but what you're looking for isn't me." > /tmp/plaintext_files/secret2.txt

# 验证文件大小
wc -c /tmp/plaintext_files/secret2.txt
```
输出：
```
44 /tmp/plaintext_files/secret2.txt
```

### 步骤5: 执行明文攻击
```bash
bkcrack -C /Users/ss/Downloads/download.zip -c secret2.txt -p /tmp/plaintext_files/secret2.txt
```

攻击过程输出：
```
bkcrack 1.8.0 - 2025-08-18
[14:24:39] Z reduction using 37 bytes of known plaintext
100.0 % (37 / 37)
[14:24:39] Attack on 207115 Z values at index 6
...
Keys: e0c271a4 cbd76d08 8d707128
Found a solution. Stopping.
[14:25:11] Keys
e0c271a4 cbd76d08 8d707128
```

**成功！** 恢复的密钥：`e0c271a4 cbd76d08 8d707128`

### 步骤6: 解密文件
```bash
# 使用恢复的密钥创建新的可解密ZIP
bkcrack -C /Users/ss/Downloads/download.zip -k e0c271a4 cbd76d08 8d707128 -U /tmp/decrypted_files.zip newpassword
```
输出：
```
[14:25:29] Writing unlocked archive /tmp/decrypted_files.zip with password "newpassword"
100.0 % (2 / 2)
Wrote unlocked archive.
```

```bash
# 提取解密后的文件
unzip -P newpassword /tmp/decrypted_files.zip -d /tmp/decrypted_contents/
```

### 步骤7: 查看破解结果
```bash
# 查看secret1.txt
cat /tmp/decrypted_contents/secret1.txt
```
输出：
```
flag{70854278-ea0c-462e-bc18-468c7a04a505}
```

```bash
# 验证secret2.txt
cat /tmp/decrypted_contents/secret2.txt
```
输出：
```
Hello, but what you're looking for isn't me.
```

### 破解总结
🎉 **攻击成功！**
- **攻击时间**: 约32秒
- **使用明文**: 37字节有效明文
- **恢复密钥**: `e0c271a4 cbd76d08 8d707128`
- **真正目标**: `secret1.txt`中的flag值
- **攻击效率**: 10.0%完成时就找到解决方案

## 其他实战案例

### 案例1: 利用ZIP文件头信息

ZIP文件通常以固定的字节序列开始，可以利用这个信息：

```bash
# ZIP文件头: 504b0304
bkcrack -C encrypted.zip -c any_file.txt -x 0 504b0304
```

### 案例2: 使用部分已知内容

如果你知道文件的部分内容（比如文件格式、常见开头等）：

```bash
# 文本文件常见开头
echo -n "This is a " > partial_plain.txt
bkcrack -C encrypted.zip -c document.txt -p partial_plain.txt -o 0
```

## 性能优化

### 1. 使用更多明文
- 明文越多，攻击越快
- 建议至少37字节明文以获得最佳性能

### 2. 并行处理
bkcrack自动使用多核处理器进行并行计算

### 3. 内存使用
大型ZIP文件可能需要更多内存，确保系统有足够的RAM

## 常见问题与解决方案

### 1. "could not find end of central directory signature"
```bash
# 检查ZIP文件完整性
unzip -t encrypted.zip

# 确认文件没有损坏
hexdump -C encrypted.zip | tail
```

### 2. "Attack failed"
可能的原因：
- 明文内容不匹配
- 文件使用AES加密而非传统加密
- 明文长度不足

### 3. 攻击时间过长
- 增加明文长度
- 确保明文正确对齐
- 使用更快的硬件

## 安全注意事项

### 合法使用
- 仅在合法授权的情况下使用
- 用于安全研究和教育目的
- 不得用于非法破解他人文件

### 防护措施
- 使用现代AES加密代替传统ZIP加密
- 使用强密码
- 避免在ZIP中包含可预测的内容

## 相关工具

### 其他ZIP破解工具
- **pkcrack**: 传统的明文攻击工具
- **fcrackzip**: 字典和暴力破解
- **john**: 通用密码破解工具

### 配合使用
```bash
# 使用john生成密码字典
john --wordlist=rockyou.txt --rules --stdout | head -1000 > custom.txt

# 使用fcrackzip进行字典攻击
fcrackzip -D -p custom.txt -u encrypted.zip
```

## 命令参考

### 主要参数
| 参数 | 长参数 | 描述 |
|------|--------|------|
| `-C` | `--cipher-zip` | 加密的ZIP文件 |
| `-c` | `--cipher-file` | 目标加密文件 |
| `-P` | `--plain-zip` | 明文ZIP文件 |
| `-p` | `--plain-file` | 明文文件 |
| `-k` | `--keys` | 内部密钥 |
| `-U` | `--unlock` | 创建解锁的ZIP |
| `-d` | `--decipher` | 解密文件 |
| `-o` | `--offset` | 偏移量 |
| `-x` | `--extra` | 十六进制明文 |
| `-t` | `--truncate` | 截断大小 |
| `-r` | `--recover-password` | 恢复原始密码 |

### 字符集选项 (用于密码恢复)
- `?l`: 小写字母 (a-z)
- `?u`: 大写字母 (A-Z)
- `?d`: 数字 (0-9)
- `?s`: 特殊字符
- `?a`: 所有ASCII可打印字符

## 总结

bkcrack是一个功能强大的ZIP明文攻击工具，在有已知明文的情况下可以高效地破解传统ZIP加密。我们的实际案例展示了：

1. **文件分析的重要性** - 通过对比文件大小找到关键线索
2. **明文准备的技巧** - 正确创建匹配的明文文件
3. **攻击的高效性** - 在短时间内成功恢复密钥
4. **实际应用价值** - 成功获取目标flag内容

正确使用该工具需要：

1. **充分的明文内容** (≥12字节，推荐≥37字节)
2. **正确的文件对应关系**
3. **适当的参数配置**
4. **合法的使用授权**

通过本指南和实际案例，你应该能够有效地使用bkcrack进行安全研究和教育目的的ZIP文件分析。记住始终在合法授权的前提下使用这些技术！