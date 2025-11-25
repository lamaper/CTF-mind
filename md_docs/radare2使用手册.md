# Radare2 使用手册（逆向工程框架）

> 免责声明：本手册仅用于合法的逆向工程、CTF 竞赛和授权的软件分析。未经授权逆向他人软件可能违法。

---

## 目录
1. 概述
2. 安装方法
3. 基本命令结构
4. 核心功能详解
5. 常用命令与示例
6. CTF 常见场景
7. 实战案例
8. 高级技巧
9. 常见问题与排查
10. 参考链接

---

## 1. 概述

Radare2（简称 r2）是一个开源的逆向工程框架和二进制分析工具，功能强大且完全命令行驱动。它能够：
- **反汇编**：支持多种架构（x86、ARM、MIPS、PowerPC 等）
- **调试**：本地和远程调试
- **补丁**：修改二进制文件
- **分析**：自动识别函数、字符串、交叉引用
- **可视化**：图形化控制流（CFG）

**适用场景**：
- CTF 逆向题目（Reverse Engineering）
- 恶意软件分析
- 二进制漏洞挖掘
- 固件逆向分析

**核心组件**：
- `r2` - 主程序
- `rabin2` - 二进制信息提取
- `radiff2` - 二进制对比
- `rafind2` - 模式搜索
- `ragg2` - 编译器
- `r2pm` - 包管理器

**版本信息**：本手册基于 Radare2 v5.0+

---

## 2. 安装方法

### macOS 安装
```bash
# 使用 Homebrew 安装
brew install radare2

# 验证安装
r2 -v
```

### Kali Linux / Debian / Ubuntu
```bash
# 使用 APT 安装（可能不是最新版）
sudo apt update
sudo apt install radare2

# 或从源码安装（推荐，获取最新版本）
git clone https://github.com/radareorg/radare2
cd radare2
sys/install.sh
```

### 从源码安装（所有平台）
```bash
# 克隆仓库
git clone https://github.com/radareorg/radare2
cd radare2

# 编译安装
sys/install.sh

# 验证
r2 -v
```

### 安装 Cutter（GUI 版本）
```bash
# macOS
brew install cutter

# 或下载：https://github.com/rizinorg/cutter/releases
```

---

## 3. 基本命令结构

```bash
# 打开文件
r2 <binary>

# 以写模式打开
r2 -w <binary>

# 调试模式
r2 -d <binary>

# 分析模式
r2 -A <binary>
```

### r2 内部命令结构

**命令格式**：`[命令][子命令][参数]`

**核心命令前缀**：
- `?` - 帮助（最重要的命令！）
- `i` - 信息（info）
- `a` - 分析（analyze）
- `s` - 跳转（seek）
- `p` - 打印（print）
- `d` - 调试（debug）
- `w` - 写入（write）
- `V` - 可视化模式
- `/` - 搜索
- `f` - 标志（flags）

**命令修饰符**：
- `j` - JSON 输出（如 `ij`）
- `*` - r2 命令输出（如 `i*`）
- `~` - grep 过滤（如 `pdf~flag`）
- `@` - 临时跳转（如 `pd 10 @ main`）

---

## 4. 核心功能详解

### 4.1 信息提取（i 命令）

| 命令 | 说明 |
|------|------|
| `i` | 基本信息 |
| `ii` | 导入函数（Imports） |
| `ie` | 入口点（Entry points） |
| `is` | 符号（Symbols） |
| `iz` | 字符串（.data 段） |
| `izz` | 所有字符串 |
| `iS` | 段信息（Sections） |
| `ij` | JSON 格式输出 |

### 4.2 分析（a 命令）

| 命令 | 说明 |
|------|------|
| `aa` | 基本分析 |
| `aaa` | 深度分析（推荐） |
| `aaaa` | 最深度分析（较慢） |
| `afl` | 列出所有函数 |
| `afn <name>` | 重命名函数 |
| `afx` | 列出交叉引用 |

### 4.3 反汇编（p 命令）

| 命令 | 说明 |
|------|------|
| `pd <n>` | 反汇编 n 条指令 |
| `pdf` | 反汇编当前函数 |
| `pdf @ <func>` | 反汇编指定函数 |
| `px <n>` | 十六进制 dump |
| `ps` | 打印字符串 |
| `pI` | 打印指令字节 |

### 4.4 跳转（s 命令）

| 命令 | 说明 |
|------|------|
| `s <addr>` | 跳转到地址 |
| `s main` | 跳转到 main 函数 |
| `s entry0` | 跳转到入口点 |
| `s-` | 撤销跳转 |

---

## 5. 常用命令与示例

### 5.1 打开和分析二进制文件

**基本工作流程**：
```bash
# 1. 打开文件并自动分析
r2 -A binary

# 在 r2 中执行：
[0x00401000]> aaa          # 深度分析
[0x00401000]> afl          # 列出所有函数
[0x00401000]> s main       # 跳转到 main 函数
[0x00401000]> pdf          # 反汇编 main 函数
```

### 5.2 查看基本信息

```bash
[0x00401000]> i            # 基本信息
arch     x86
baddr    0x400000
binsz    8192
bits     64
canary   false
class    ELF64
compiler GCC 9.3.0
crypto   false
endian   little
havecode true
lang     c
linenum  false
lsyms    false
machine  AMD x86-64
nx       true
os       linux
pic      false
relocs   false
relro    partial
rpath    NONE
static   false
stripped false
subsys   linux
va       true

[0x00401000]> ii           # 导入函数
[Imports]
nth vaddr      bind   type   lib name
―――――――――――――――――――――――――――――――――――――
1   0x00401030 GLOBAL FUNC       puts

[0x00401000]> ie           # 入口点
[Entrypoints]
vaddr=0x00401040 paddr=0x00001040 haddr=0x00000018 hvaddr=0x00400018 type=program

[0x00401000]> is           # 符号表
[Symbols]
nth paddr      vaddr      bind   type   size lib name
――――――――――――――――――――――――――――――――――――――――――――――――――――――
0   0x00001150 0x00401150 GLOBAL FUNC   23       main
```

### 5.3 查找字符串

```bash
[0x00401000]> iz            # .data 段字符串
[Strings]
nth paddr      vaddr      len size section type  string
――――――――――――――――――――――――――――――――――――――――――――――――――
0   0x00002000 0x00402000 13  14   .rodata ascii flag{r2_is_cool}
1   0x00002010 0x00402010 25  26   .rodata ascii Enter password:

[0x00401000]> izz           # 所有字符串（包括代码段）

[0x00401000]> / flag        # 搜索字符串 "flag"
0x00402000 hit0_0 flag{r2_is_cool}

[0x00401000]> /x 666c6167   # 搜索十六进制 "flag"
```

### 5.4 分析和反汇编函数

```bash
[0x00401000]> afl           # 列出所有函数
0x00401040    1 6            entry0
0x00401050    4 41           sym.deregister_tm_clones
0x00401080    4 57           sym.register_tm_clones
0x004010c0    3 34           entry.fini0
0x004010f0    4 45           entry.init0
0x00401150    1 95           main
0x004011b0    1 5            sym.__libc_csu_fini
0x004011b8    1 13           sym._fini

[0x00401000]> s main        # 跳转到 main
[0x00401150]> pdf           # 反汇编 main 函数
            ;-- main:
┌ 95: int main (int argc, char **argv, char **envp);
│           0x00401150      55             push rbp
│           0x00401151      4889e5         mov rbp, rsp
│           0x00401154      4883ec10       sub rsp, 0x10
│           0x00401158      c745fc000000.. mov dword [rbp - 4], 0
│       ┌─< 0x0040115f      eb42           jmp 0x4011a3
│       │   ; CODE XREF from main @ 0x4011a7
│      ┌──> 0x00401161      817dfc390500.. cmp dword [rbp - 4], 0x539
│      ╎│   0x00401168      7533           jne 0x40119d
│      ╎│   0x0040116a      488d3d8f0e00.. lea rdi, str.flag_r2_is_cool
│      ╎│   0x00401171      e8bafeffff     call sym.imp.puts
...
```

### 5.5 可视化模式

**进入可视化模式**：
```bash
[0x00401000]> V            # 基本可视化
[0x00401000]> VV           # 图形模式（控制流图）
```

**可视化模式快捷键**：
- `p` / `P` - 切换显示模式
- `hjkl` - 移动光标
- `q` - 退出可视化
- `?` - 帮助
- `x` - 查看交叉引用
- `:` - 命令模式

---

## 6. CTF 常见场景

### 场景1：查找 flag 字符串

```bash
# 1. 打开文件
r2 -A crackme

# 2. 搜索 "flag"
[0x00401000]> / flag
[0x00401000]> izz~flag     # 在所有字符串中搜索

# 3. 或搜索花括号
[0x00401000]> izz~{

# 4. 查看上下文
[0x00401000]> s <address>
[0x00401000]> px 100
```

### 场景2：密码比对逻辑

**题目特征**：需要找到正确密码

```bash
# 1. 分析 main 函数
[0x00401000]> s main
[0x00401150]> pdf

# 2. 查找 strcmp/strncmp 调用
[0x00401150]> /c strcmp
[0x00401150]> axt @ sym.imp.strcmp   # 查看交叉引用

# 3. 查看比对的字符串
# 找到类似代码：
#   lea rdi, [password_input]
#   lea rsi, str.correct_password
#   call strcmp

# 4. 查看 correct_password 地址的内容
[0x00401150]> ps @ str.correct_password
```

### 场景3：反混淆和修改二进制

```bash
# 1. 以写模式打开
r2 -w binary

# 2. 跳转到需要修改的位置
[0x00401000]> s 0x401234

# 3. 查看当前指令
[0x00401234]> pd 5

# 4. 修改指令（例如将 jne 改为 je）
[0x00401234]> wa je 0x401250

# 5. 或写入 NOP
[0x00401234]> wx 90909090

# 6. 验证修改
[0x00401234]> pd 5

# 7. 保存
[0x00401234]> q
```

### 场景4：动态调试

```bash
# 1. 以调试模式打开
r2 -d ./binary arg1 arg2

# 2. 分析
[0x00401000]> aaa

# 3. 设置断点
[0x00401000]> db main          # 在 main 设置断点
[0x00401000]> db 0x401234      # 在地址设置断点

# 4. 查看断点
[0x00401000]> dbl

# 5. 运行到断点
[0x00401000]> dc

# 6. 单步执行
[0x00401000]> ds               # step (into)
[0x00401000]> dso              # step over

# 7. 查看寄存器
[0x00401000]> dr               # 所有寄存器
[0x00401000]> dr rax           # 查看 rax

# 8. 查看内存
[0x00401000]> px @ rsp         # 查看栈
[0x00401000]> ps @ rdi         # 查看字符串指针

# 9. 继续执行
[0x00401000]> dc
```

---

## 7. 实战案例

### 案例1：简单 Crackme

**题目**：找到正确密码

```bash
# 1. 打开文件
$ r2 -A crackme

# 2. 列出函数
[0x00401000]> afl
...
0x00401150    1 95           main
...

# 3. 反汇编 main
[0x00401000]> s main
[0x00401150]> pdf

# 发现代码：
#   lea rdi, str.Enter_password:
#   call sym.imp.puts
#   lea rdi, [rbp - 0x20]
#   call sym.imp.gets
#   lea rdi, [rbp - 0x20]
#   lea rsi, str.MySecretPass123
#   call sym.imp.strcmp
#   test eax, eax
#   je success
#   jmp fail

# 4. 查看密码字符串
[0x00401150]> ps @ str.MySecretPass123
MySecretPass123

# 5. 验证
$ ./crackme
Enter password: MySecretPass123
Correct! flag{p455w0rd_cr4ck3d}
```

### 案例2：修改跳转逻辑

**题目**：绕过密码检查

```bash
# 1. 以写模式打开
$ r2 -w -A crackme

# 2. 找到比对逻辑
[0x00401000]> s main
[0x00401150]> pdf

# 发现：
#   0x004011a8      e873feffff     call sym.imp.strcmp
#   0x004011ad      85c0           test eax, eax
#   0x004011af      7507           jne 0x4011b8   # jump if not equal

# 3. 修改 jne 为 je（反转逻辑）
[0x00401150]> s 0x4011af
[0x004011af]> wa je 0x4011b8

# 或者 NOP 掉跳转
[0x004011af]> wx 9090

# 4. 保存退出
[0x004011af]> q

# 5. 测试
$ ./crackme
Enter password: wrong
Correct! flag{l0g1c_byp455}
```

### 案例3：提取加密的 flag

**场景**：flag 被简单异或加密

```bash
# 1. 分析找到加密函数
[0x00401000]> afl
...
0x00401200    1 50           sym.decrypt_flag
...

# 2. 反汇编 decrypt 函数
[0x00401000]> pdf @ sym.decrypt_flag

# 发现逻辑：
#   mov al, byte [rsi]     # 读取加密字节
#   xor al, 0x42           # XOR with 0x42
#   mov byte [rdi], al     # 写入解密字节

# 3. 找到加密数据
[0x00401000]> izz~encrypted
0x00402100 encrypted_flag: \x26\x2c\x27\x2d\x01\x34...

# 4. 提取数据
[0x00401000]> px 20 @ 0x402100
- offset -   0 1  2 3  4 5  6 7  8 9  A B  C D  E F
0x00402100  2626 272d 0134 3637 1c13 2e2f 3041 4200  &&'-.467..../0AB.

# 5. 解密（Python）
$ python3 -c "data = bytes.fromhex('2626272d0134363...'); print(''.join(chr(b ^ 0x42) for b in data))"
flag{x0r_1s_34sy}
```

---

## 8. 高级技巧

### 8.1 使用 rabin2 快速提取信息

```bash
# 基本信息
rabin2 -I binary

# 导入函数
rabin2 -i binary

# 符号
rabin2 -s binary

# 字符串
rabin2 -z binary

# 入口点
rabin2 -e binary

# 段信息
rabin2 -S binary
```

### 8.2 使用 rafind2 搜索模式

```bash
# 搜索字符串
rafind2 -s "flag{" binary

# 搜索十六进制
rafind2 -x 666c6167 binary

# 搜索正则表达式
rafind2 -r "flag{.*}" binary
```

### 8.3 使用 radiff2 比对文件

```bash
# 比对两个二进制
radiff2 binary1 binary2

# 详细模式
radiff2 -v binary1 binary2

# 十六进制diff
radiff2 -x binary1 binary2
```

### 8.4 脚本自动化

**r2pipe（Python）**：
```python
import r2pipe

# 打开二进制
r2 = r2pipe.open("binary")

# 分析
r2.cmd("aaa")

# 获取函数列表
functions = r2.cmdj("aflj")  # JSON 输出

# 搜索字符串
strings = r2.cmdj("izzj")

for s in strings:
    if "flag" in s['string']:
        print(f"Found at 0x{s['vaddr']:x}: {s['string']}")

# 反汇编 main
disasm = r2.cmd("pdf @ main")
print(disasm)
```

---

## 9. 常见问题与排查

### 问题1：无法分析函数

**解决方案**：
```bash
# 1. 尝试更深度的分析
[0x00401000]> aaaa

# 2. 手动分析函数
[0x00401000]> af @ 0x401234

# 3. 分析所有函数
[0x00401000]> af @@ sym.*
```

### 问题2：无法找到字符串

```bash
# 1. 使用 izz（包括所有段）
[0x00401000]> izz

# 2. 手动搜索
[0x00401000]> / flag

# 3. 查看所有段
[0x00401000]> iS

# 4. 在特定段搜索
[0x00401000]> e search.in=io.section.text
[0x00401000]> / flag
```

---

## 10. 参考链接

### 官方资源
- **官网**：https://rada.re/
- **GitHub**：https://github.com/radareorg/radare2
- **文档**：https://book.rada.re/
- **Cutter（GUI）**：https://cutter.re/

### 教程和书籍
- **Radare2 Book**：https://book.rada.re/
- **CTF Wiki - Reverse**：https://ctf-wiki.org/reverse/
- **LiveOverflow Radare2 教程**：https://www.youtube.com/playlist?list=PLhixgUqwRTjxglIswKp9mpkfPNfHkzyeN

### 相关工具
- **Ghidra**：https://ghidra-sre.org/
- **IDA Pro**：https://hex-rays.com/ida-pro/
- **Binary Ninja**：https://binary.ninja/
- **GDB + GEF**：https://github.com/hugsy/gef

---

## 快速参考

### 最常用命令
```bash
# 打开和分析
r2 -A binary

# 信息提取
i / ii / ie / is / iz / izz

# 分析
aaa / afl / pdf

# 跳转
s main / s 0x401000

# 搜索
/ flag / /x 666c6167

# 可视化
V / VV

# 调试
r2 -d binary
db main / dc / ds / dr
```

---

**文档版本**：v1.0
**更新日期**：2025-01
**适用版本**：Radare2 v5.0+
