# Hashcat 使用手册（快速上手与进阶）

> 免责声明：请仅在你有权限的系统/样本上使用密码破解工具。未经授权破解他人密码是违法行为。

---

## 目录
1. 概述
2. 基本命令结构
3. 常见 hash 类型与 `-m` 模式编号（常用）
4. 常见攻击模式（`-a`）
5. 常用命令与示例（字典 / mask / hybrid / rules）
6. Mask 语法与例子
7. 规则（rules）简介与使用示例
8. 设备与性能（列设备 / benchmark / 调优）
9. 会话管理、状态与恢复
10. 常用实战案例（CTF 常见场景）
11. 常见问题与排查策略
12. 高级技巧与安全注意
13. 参考链接

---

## 1. 概述
Hashcat 是目前最流行、功能最全的密码恢复/破解工具之一，支持 CPU 与 GPU（OpenCL/Metal/CUDA）的高性能破解。它把“hash type”（通过 `-m` 指定）和“attack mode”（通过 `-a` 指定）分离，支持规则、mask、字典、组合等混合攻击方式。

---

## 2. 基本命令结构
```bash
hashcat [options] -m <hash-type> -a <attack-mode> <hashfile> <dict_or_mask_or_stdin>
# 或简写
hashcat -m 0 -a 0 hashes.txt rockyou.txt
```
常用选项：
- `-m, --hash-type`：hash 类型编号（如 MD5=0）。
- `-a, --attack-mode`：攻击模式编号（0=wordlist, 3=mask 等）。
- `-o <outfile>`：输出已破解结果文件。
- `--show`：显示已破解的密码（查看 potfile）。
- `-I`：列出可用设备信息（devices info）。
- `-b` 或 `--benchmark`：基准测试（benchmark）。
- `-w <1..4>`：工作负载（workload）驱动性能/响应权衡。
- `-O`：使用 optimized kernel（更快但限制最大密码长度）。
- `-d <devIDs>`：选择设备 ID（逗号分隔）。
- `--session <name>`：指定会话名以便恢复。
- `--status` / `--status-timer`：定时显示状态。
- `--restore`：从上次断点恢复会话。

更多帮助：`hashcat --help`。

---

## 3. 常见 hash 类型与 `-m` 模式编号（常用）
> 以下列出 CTF/渗透中最常见的 hash-mode（并非全表，更多请查看官方 wiki / cheatsheet）。

| 描述 | hash-mode (`-m`) |
|---|---:|
| MD5 | 0 |
| SHA1 | 100 |
| SHA256 | 1400 |
| SHA512 | 1700 |
| MD4 | 900 |
| NTLM | 1000 |
| bcrypt (Blowfish, crypt(3) $2y$) | 3200 |
| MD5(Unix: md5crypt) | 500 |
| LM | 3000 |
| MySQL323 | 200 |
| MySQL411 | 300 |

> 注：Hash 模式编号会随着版本扩展，遇到不确定 hash 时可以查官方 `example_hashes` 或 `hashcat --help`。

---

## 4. 常见攻击模式（`-a`）
| 编号 | attack mode | 说明 |
|---:|---|---|
| 0 | wordlist / straight | 字典/规则组合（最常用） |
| 1 | combination | 组合两个字典（dict1 + dict2） |
| 3 | brute-force / mask | mask 暴力（高效） |
| 6 | hybrid wordlist + mask | 字典前缀 + mask 后缀 |
| 7 | hybrid mask + wordlist | mask 前缀 + 字典后缀 |

例如：`-a 0` 表示用字典攻击；`-a 3` 表示用 mask 暴力。

---

## 5. 常用命令与示例

### 字典攻击（最常见）
```bash
hashcat -m 0 -a 0 hashes.txt /path/to/wordlists/rockyou.txt
```

### 带规则的字典攻击
```bash
hashcat -m 1000 -a 0 ntlm.hash rockyou.txt -r rules/best64.rule
```

### Mask / 暴力破解（指定字符集和长度）
```bash
# 尝试所有 8 位小写字母密码
hashcat -m 0 -a 3 hashes.txt ?l?l?l?l?l?l?l?l
# 更短的例子：6 位，字母+数字
hashcat -m 0 -a 3 hashes.txt ?l?l?l?l?d?d
```

### 混合（字典 + mask）
```bash
# word + 2 digits 后缀（常用于密码为单词+两位数字）
hashcat -m 0 -a 6 hashes.txt rockyou.txt ?d?d
```

### 组合字典（两个字典拼接）
```bash
hashcat -m 0 -a 1 hashes.txt dict1.txt dict2.txt
```

### 尝试 NTLM 示例并把结果输出
```bash
hashcat -m 1000 -a 0 ntlm.hash rockyou.txt -o cracked.txt --show
```

---

## 6. Mask 语法（常用占位符）
- `?l`：小写字母 (abcdefghijklmnopqrstuvwxyz)
- `?u`：大写字母 (ABCDEFGHIJKLMNOPQRSTUVWXYZ)
- `?d`：数字 (0123456789)
- `?s`：特殊字符（space + punctuation）
- `?a`：所有可打印字符（?l?u?d?s）
- `?b`：二进制（0x00..0xFF） — 少用
- `?1 ?2 ...`：自定义字符集（通过 `-1` 参数定义）

示例：
```bash
# 自定义第一个字符集为数字和小写字母
hashcat -m 0 -a 3 hash.txt -1 ?l?d ?1?1?1?1?1
```

---

## 7. 规则（rules）
- 规则文件存放在 `rules/` 目录（安装包自带常用规则文件 like `best64.rule`, `rockyou-30000.rule` 等）。
- 用法：`-r rules/best64.rule`，规则逐条应用到字典单词上，能快速扩大命中空间（例如加数字、字符替换、大小写变化等）。
- 示例：
```bash
hashcat -m 0 -a 0 hash.txt rockyou.txt -r rules/best64.rule
```

---

## 8. 设备、性能与调优

### 列出设备
```bash
hashcat -I
```

### 基准测试
```bash
hashcat -b        # 或 hashcat --benchmark
```

### 常用调优参数
- `-w <1..4>`：workload profile（1 最低、4 最高）。
- `-O`：启用 optimized kernel（更快但限制密码最大长度）。
- `-n <value>`：设置并行线程（tune for GPU）。
- `-u <value>`：设置迭代次数（loop-tuning）。
- `-d <devIDs>`：选择设备（多个设备时可指定）。
- `--force`：强制忽略某些警告（慎用）。

### 注意（mac / Apple Silicon）
在 macOS（尤其 Apple Silicon）上，OpenCL/Metal 驱动兼容性可能导致某些 kernel 编译失败或设备不被利用；遇到问题时可先用 CPU 模式或在 Linux + 专用 GPU 上执行高强度任务以稳定性能。

---

## 9. 会话管理、状态、恢复
- 指定 session 名：`--session myjob`
- 开启状态显示：`--status --status-timer=10`（每 10s 显示状态）
- 中断后恢复：使用 `--restore`（会读取 `.restore` 文件）
- 查看已破解结果：`hashcat --show -m <mode> hashes.txt`（从 potfile 显示）
- potfile 默认位置通常在当前工作目录或 `~/.hashcat`，可用 `--potfile-path` 指定

---

## 10. 常用实战案例（CTF 场景）

### 场景 A：已知是 MD5，直接用 rockyou
```bash
hashcat -m 0 -a 0 md5_hashes.txt rockyou.txt -o cracked.txt
```

### 场景 B：NTLM + 规则（windows 常见）
```bash
hashcat -m 1000 -a 0 ntlm.hash rockyou.txt -r rules/best64.rule -o ntlm_cracked.txt
```

### 场景 C：发现是 bcrypt（慢），先用字典尝试
```bash
hashcat -m 3200 -a 0 bcrypt.hash rockyou.txt --show
```
> bcrypt 属于慢哈希（成本高），使用 GPU 仍然慢，CTF 中通常通过字典与规则优先尝试。

### 场景 D：针对密码由“单词 + 2位数字”构成
```bash
hashcat -m 0 -a 6 hash.txt rockyou.txt ?d?d
```

---

## 11. 常见问题与排查
- **没列出 GPU / 设备不可用**：运行 `hashcat -I` 检查驱动和平台；mac 上可能因为 OpenCL/Metal 支持问题；在 Linux 上驱动与 CUDA/OpenCL 一般更稳定。
- **Kernel 编译或 CL_OUT_OF_RESOURCES 错误**：尝试降低 `-w`，使用 `-O` 亦可能触发限制，或使用 `--backend` 相关选项；在 Windows 上需关闭 GPU driver timeout（不推荐在生产机做）。
- **速度慢**：先用 `hashcat -b` 基准，确认是否启用优化内核和设备利用率；使用规则前先测试字典能否命中。
- **AX/特殊格式报错（line length exception）**：通常是 `-m` 模式与 hash 格式不匹配，检查 hash 长度/格式或用 `example_hashes` 对照。

---

## 12. 高级技巧与安全注意
- **分布式 / 多卡**：使用 `-d` 指定设备或在多台机器上分发不同 mask 区间。
- **限制风险**：在 Windows 上强制长时间 GPU 运行可能导致系统不稳定或黑屏（driver timeout）。
- **数据管理**：使用 `--session` 与 `--restore` 管理长任务，输出已破解用 `--show`，并合理备份 potfile。
- **效率优先策略**：先字典+规则、再 hybrid、最后 mask 暴力（mask 用于找结构明确的短密码）。
- **法律合规**：仅在授权范围内使用；CTF 实验环境除外。

---

## 13. 参考链接（便于你查更多细节）
- Hashcat 官方 wiki / example_hashes（hash 模式例子）。
- Hashcat Cheatsheet / 常用模式表（Black Hills InfoSec）—快速参考表。
- FreeCodeCamp Hashcat 教程（实战示例）。
- Hashcat 论坛（遇到特定错误可以搜索或发帖）。

---


如果你需要我把这个 Markdown 导出为 PDF，或生成一个一键运行的 bash 安装/检测脚本，我也可以直接做。