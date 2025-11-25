# QEMU 使用手册（虚拟化与CPU模拟）

> 免责声明：本手册仅用于合法的系统测试、CTF 竞赛和授权的软件分析。

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

QEMU（Quick Emulator）是一个开源的通用虚拟化和模拟器，支持多种架构。在 CTF 中主要用于：
- **用户模式模拟**：运行不同架构的二进制文件（ARM、MIPS、PowerPC等）
- **系统模拟**：运行完整的操作系统（测试固件、分析恶意软件）
- **调试支持**：结合 GDB 进行跨架构调试

**两种主要模式**：
1. **用户模式（User Mode）**：`qemu-<arch>`
   - 仅模拟 CPU 和系统调用
   - 用于运行单个二进制文件
   - 速度快，适合 CTF 逆向题

2. **系统模式（System Mode）**：`qemu-system-<arch>`
   - 模拟完整系统（CPU、内存、设备）
   - 运行完整操作系统
   - 用于固件分析、IoT 逆向

**支持的架构**：
- x86/x86_64（PC）
- ARM/ARM64（移动设备、嵌入式）
- MIPS/MIPS64（路由器、嵌入式）
- PowerPC（Apple旧设备）
- RISC-V（新兴架构）
- SPARC、Alpha、M68K 等

**版本信息**：本手册基于 QEMU v8.0+

---

## 2. 安装方法

### macOS 安装
```bash
# 使用 Homebrew 安装
brew install qemu

# 验证安装（检查用户模式）
qemu-x86_64 --version
qemu-arm --version
qemu-mips --version

# 验证系统模式
qemu-system-x86_64 --version
qemu-system-arm --version
```

### Kali Linux / Debian / Ubuntu
```bash
# 安装用户模式（所有架构）
sudo apt update
sudo apt install qemu-user qemu-user-static

# 安装系统模式
sudo apt install qemu-system

# 验证
qemu-arm --version
qemu-system-arm --version
```

### 检查已安装的架构
```bash
# 列出所有 qemu 用户模式二进制
ls /usr/bin/qemu-*

# 或使用 which
which qemu-arm
which qemu-mips
which qemu-aarch64
```

---

## 3. 基本命令结构

### 用户模式（User Mode）
```bash
# 基本运行
qemu-<arch> <binary> [args]

# 带库路径
qemu-<arch> -L <sysroot> <binary>

# 调试模式
qemu-<arch> -g <port> <binary>
```

### 系统模式（System Mode）
```bash
# 基本启动
qemu-system-<arch> [选项]

# 常用选项
qemu-system-<arch> -kernel <kernel> -initrd <initrd> -hda <disk.img>
```

---

## 4. 核心功能详解

### 4.1 用户模式核心选项

| 选项 | 说明 |
|------|------|
| `-L <path>` | 指定库根目录（sysroot） |
| `-g <port>` | 启用 GDB 调试（监听端口） |
| `-E <var>=<value>` | 设置环境变量 |
| `-strace` | 跟踪系统调用（类似 strace） |
| `-d <items>` | 启用调试日志 |
| `-cpu <model>` | 指定 CPU 型号 |

### 4.2 常用架构二进制

| 架构 | QEMU 命令 | 常见用途 |
|------|-----------|---------|
| ARM (32-bit) | `qemu-arm` | 移动设备、嵌入式 |
| ARM64 (64-bit) | `qemu-aarch64` | 现代 ARM 设备 |
| MIPS (32-bit) | `qemu-mips` | 路由器固件 |
| MIPS64 | `qemu-mips64` | 64位 MIPS |
| PowerPC | `qemu-ppc` | 旧 Apple 设备 |
| RISC-V | `qemu-riscv64` | 新兴架构 |

---

## 5. 常用命令与示例

### 5.1 运行 ARM 二进制

**场景**：运行 ARM ELF 文件

```bash
# 1. 确认文件架构
file binary_arm
# Output: ELF 32-bit LSB executable, ARM, version 1 (SYSV)

# 2. 运行（无库依赖）
qemu-arm ./binary_arm

# 3. 如果有库依赖，指定 sysroot
qemu-arm -L /usr/arm-linux-gnueabihf ./binary_arm

# 4. 传递参数
qemu-arm ./binary_arm arg1 arg2
```

### 5.2 运行 MIPS 二进制

```bash
# 1. 确认架构
file binary_mips
# Output: ELF 32-bit MSB executable, MIPS, MIPS-I version 1 (SYSV)

# 2. 运行
qemu-mips ./binary_mips

# 3. 指定库路径（如果需要）
qemu-mips -L /usr/mips-linux-gnu ./binary_mips
```

### 5.3 系统调用跟踪

**类似 strace 功能**：
```bash
# 跟踪所有系统调用
qemu-arm -strace ./binary_arm

# 输出示例：
# 3 brk(0x00000000) = 0x00021000
# 3 uname(0xf6fff8e0) = 0
# 3 readlink("/proc/self/exe",0xf6fff180,4096) = 12
# 3 open("/etc/ld.so.cache",O_RDONLY|O_CLOEXEC) = 3
# 3 fstat64(3,0xf6fff714) = 0
```

### 5.4 GDB 调试

**启动调试服务器**：
```bash
# 1. 启动 QEMU 并监听 1234 端口
qemu-arm -g 1234 ./binary_arm

# 2. 在另一个终端启动 GDB
gdb-multiarch ./binary_arm
(gdb) target remote localhost:1234
(gdb) break main
(gdb) continue
```

**使用 pwndbg/gef 增强**：
```bash
# 1. 安装 pwndbg 或 gef
git clone https://github.com/pwndbg/pwndbg
cd pwndbg && ./setup.sh

# 2. 启动调试
gdb-multiarch ./binary_arm
pwndbg> target remote :1234
pwndbg> break *0x10400
pwndbg> continue
```

---

## 6. CTF 常见场景

### 场景1：运行 ARM Crackme

**题目特征**：提供 ARM 二进制文件

```bash
# 1. 确认架构
file crackme_arm
# Output: ELF 32-bit LSB executable, ARM

# 2. 尝试直接运行
qemu-arm ./crackme_arm
# Enter password:

# 3. 如果报错缺少库
qemu-arm -L /usr/arm-linux-gnueabihf ./crackme_arm

# 4. 跟踪系统调用查找线索
qemu-arm -strace ./crackme_arm 2>&1 | grep "flag\|password"
```

### 场景2：调试 MIPS 二进制

**题目特征**：需要动态分析 MIPS 程序

```bash
# 1. 启动调试服务器
qemu-mips -g 1234 -L /usr/mips-linux-gnu ./challenge_mips

# 2. 在另一个终端
gdb-multiarch ./challenge_mips
(gdb) set architecture mips
(gdb) target remote localhost:1234
(gdb) break main
(gdb) continue
(gdb) info registers
(gdb) x/10i $pc
```

### 场景3：固件仿真（系统模式）

**题目特征**：提供路由器固件

```bash
# 1. 提取固件（使用 binwalk）
binwalk -e firmware.bin

# 2. 找到 kernel 和 rootfs
cd _firmware.bin.extracted
ls
# vmlinux  rootfs.squashfs

# 3. 挂载 rootfs
mkdir rootfs
sudo mount -t squashfs rootfs.squashfs rootfs

# 4. 启动 QEMU 系统模式
qemu-system-mips \
  -M malta \
  -kernel vmlinux \
  -hda rootfs.ext4 \
  -append "root=/dev/sda console=ttyS0" \
  -nographic
```

### 场景4：ARM64 (aarch64) 程序

```bash
# 1. 确认是 64 位 ARM
file binary_arm64
# Output: ELF 64-bit LSB executable, ARM aarch64

# 2. 使用 qemu-aarch64
qemu-aarch64 ./binary_arm64

# 3. 指定库路径
qemu-aarch64 -L /usr/aarch64-linux-gnu ./binary_arm64
```

---

## 7. 实战案例

### 案例1：ARM Crackme 密码破解

**题目**：ARM 二进制，需要找到正确密码

```bash
# 1. 运行程序
$ qemu-arm ./crackme_arm
Enter password: test
Wrong!

# 2. 查看字符串
$ strings crackme_arm | grep -i "flag\|password\|correct"
Enter password:
Correct! flag{arm_r3v3rs3}
Wrong!

# 3. 或使用 strace 跟踪
$ qemu-arm -strace ./crackme_arm 2>&1 | grep -A 5 "strcmp"
3 strcmp(0xf6fff8a0,0x00010400) = -1
# 0x00010400 可能是正确密码地址

# 4. 提取密码
$ echo "0x00010400" | xxd -r -p
SecretPass123

# 5. 验证
$ qemu-arm ./crackme_arm
Enter password: SecretPass123
Correct! flag{arm_r3v3rs3}
```

### 案例2：GDB 动态调试找 Flag

**场景**：需要动态分析 MIPS 程序

```bash
# Terminal 1: 启动 QEMU
$ qemu-mips -g 1234 -L /usr/mips-linux-gnu ./challenge

# Terminal 2: GDB 调试
$ gdb-multiarch ./challenge
(gdb) set architecture mips
(gdb) target remote localhost:1234
Remote debugging using localhost:1234

(gdb) break main
Breakpoint 1 at 0x400800

(gdb) continue
Continuing.
Breakpoint 1, 0x00400800 in main ()

# 查看反汇编
(gdb) disas main
...
0x00400840 <+64>:   lw      $v0, -32544($gp)
0x00400844 <+68>:   addiu   $a0, $v0, 1024     # flag 字符串地址
0x00400848 <+72>:   lw      $v0, -32460($gp)
0x0040084c <+76>:   jalr    $v0                # 调用函数
...

# 打印 flag 字符串
(gdb) x/s $a0
0x400a400: "flag{m1p5_d3bug}"

# 或设置断点在关键函数
(gdb) break *0x400848
(gdb) continue
(gdb) x/20i $pc
```

### 案例3：系统调用分析

**场景**：程序有隐藏的系统调用

```bash
# 1. 使用 strace 模式
$ qemu-arm -strace ./mystery_arm 2>&1 | tee strace.log

# 2. 查看日志
$ cat strace.log

# 发现可疑调用：
# open("/tmp/.secret",O_RDONLY) = 3
# read(3,0xf6fff8a0,256) = 32
# close(3) = 0

# 3. 检查读取的内容
$ qemu-arm -strace ./mystery_arm 2>&1 | grep -A 2 "read(3"
read(3,0xf6fff8a0,256) = 32
# 数据：flag{sysc4ll_tr4c1ng}
```

---

## 8. 高级技巧

### 8.1 安装交叉编译工具链

**ARM**：
```bash
# Ubuntu/Debian
sudo apt install gcc-arm-linux-gnueabihf g++-arm-linux-gnueabihf
sudo apt install libc6-armhf-cross libc6-dev-armhf-cross

# macOS
brew install arm-linux-gnueabihf-binutils
```

**MIPS**：
```bash
sudo apt install gcc-mips-linux-gnu g++-mips-linux-gnu
```

### 8.2 自定义 CPU 模型

```bash
# 列出支持的 CPU 型号
qemu-arm -cpu help

# 使用特定 CPU
qemu-arm -cpu cortex-a53 ./binary_arm
```

### 8.3 环境变量设置

```bash
# 设置 LD_LIBRARY_PATH
qemu-arm -E LD_LIBRARY_PATH=/custom/libs ./binary_arm

# 设置自定义环境变量
qemu-arm -E FLAG="flag{env_var}" ./binary_arm
```

### 8.4 网络配置（系统模式）

```bash
# 用户模式网络（端口转发）
qemu-system-arm \
  -kernel vmlinuz \
  -initrd initrd.img \
  -hda disk.img \
  -netdev user,id=net0,hostfwd=tcp::2222-:22 \
  -device virtio-net-pci,netdev=net0
```

---

## 9. 常见问题与排查

### 问题1：缺少共享库

**错误信息**：`error while loading shared libraries: libxxx.so.1`

**解决方案**：
```bash
# 方法1：指定 sysroot
qemu-arm -L /usr/arm-linux-gnueabihf ./binary_arm

# 方法2：安装对应架构的库
sudo apt install libc6-armhf-cross

# 方法3：静态编译（如果有源码）
arm-linux-gnueabihf-gcc -static -o binary_static binary.c
qemu-arm ./binary_static
```

### 问题2：架构不匹配

**错误信息**：`Invalid ELF image for this architecture`

**排查**：
```bash
# 1. 确认文件架构
file binary
readelf -h binary | grep Machine

# 2. 使用对应的 qemu
# ARM 32-bit → qemu-arm
# ARM 64-bit → qemu-aarch64
# MIPS 32-bit → qemu-mips
# MIPS 64-bit → qemu-mips64
```

### 问题3：GDB 调试连接失败

**排查步骤**：
```bash
# 1. 确认 QEMU 监听端口
netstat -an | grep 1234

# 2. 确认防火墙允许连接
sudo ufw allow 1234

# 3. 使用正确的 GDB
# 多架构：gdb-multiarch
# ARM：arm-linux-gnueabihf-gdb
# MIPS：mips-linux-gnu-gdb

# 4. 设置架构
(gdb) set architecture arm
(gdb) target remote localhost:1234
```

### 问题4：系统模式无法启动

**常见原因**：
- Kernel 与架构不匹配
- 缺少必要的启动参数
- 磁盘镜像格式错误

**排查**：
```bash
# 1. 使用 -nographic 查看输出
qemu-system-arm -nographic ...

# 2. 启用调试日志
qemu-system-arm -d guest_errors ...

# 3. 使用标准配置测试
qemu-system-arm -M virt -kernel zImage -nographic
```

---

## 10. 参考链接

### 官方资源
- **QEMU 官网**：https://www.qemu.org/
- **文档**：https://www.qemu.org/docs/master/
- **用户模式文档**：https://www.qemu.org/docs/master/user/main.html
- **系统模式文档**：https://www.qemu.org/docs/master/system/index.html

### CTF 相关
- **CTF Wiki - Reverse**：https://ctf-wiki.org/reverse/linux/
- **Azeria Labs ARM 教程**：https://azeria-labs.com/writing-arm-assembly-part-1/
- **MIPS 汇编教程**：https://www.mips.com/

### 工具资源
- **pwndbg**：https://github.com/pwndbg/pwndbg
- **GEF**：https://github.com/hugsy/gef
- **Ghidra**：https://ghidra-sre.org/
- **Unicorn Engine**：https://www.unicorn-engine.org/

---

## 快速参考

### 最常用命令
```bash
# 运行不同架构
qemu-arm ./binary_arm
qemu-mips ./binary_mips
qemu-aarch64 ./binary_arm64

# 指定库路径
qemu-arm -L /usr/arm-linux-gnueabihf ./binary_arm

# 系统调用跟踪
qemu-arm -strace ./binary_arm

# GDB 调试
qemu-arm -g 1234 ./binary_arm
gdb-multiarch ./binary_arm
(gdb) target remote :1234
```

### CTF 快速检查
```bash
# 确认架构
file binary

# 运行程序
qemu-<arch> ./binary

# 跟踪系统调用
qemu-<arch> -strace ./binary 2>&1 | grep flag

# 查看字符串
strings binary | grep flag
```

---

**文档版本**：v1.0
**更新日期**：2025-01
**适用版本**：QEMU v8.0+
