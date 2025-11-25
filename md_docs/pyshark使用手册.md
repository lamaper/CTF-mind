# Pyshark 使用手册（Python网络抓包分析库）

> Pyshark 是基于Wireshark的Python封装库,提供强大的网络数据包捕获和分析功能,在CTF中用于流量分析和取证。

---

## 目录
1. [概述](#概述)
2. [安装方法](#安装方法)
3. [基础概念](#基础概念)
4. [抓包操作](#抓包操作)
5. [PCAP文件分析](#pcap文件分析)
6. [协议解析](#协议解析)
7. [数据过滤](#数据过滤)
8. [数据提取](#数据提取)
9. [CTF常见场景](#ctf常见场景)
10. [实战案例](#实战案例)
11. [高级技巧](#高级技巧)
12. [参考资源](#参考资源)

---

## 概述

Pyshark 是Wireshark/Tshark的Python接口,提供:
- **实时抓包**: 捕获网络接口流量
- **离线分析**: 读取PCAP/PCAPNG文件
- **协议解析**: 支持数千种网络协议
- **数据过滤**: BPF/Display过滤器
- **深度解析**: 逐层分析数据包结构

**CTF应用场景**:
- 网络流量分析
- 协议还原
- 敏感信息提取
- 加密流量分析
- 恶意流量检测

---

## 安装方法

```bash
# 1. 安装Wireshark/Tshark (必需)
# macOS
brew install wireshark

# Ubuntu/Debian
sudo apt-get install tshark

# Windows: 下载安装包
# https://www.wireshark.org/download.html

# 2. 安装pyshark
pip install pyshark

# 验证安装
python3 -c "import pyshark; print(pyshark.__version__)"
tshark --version
```

### 权限配置

```bash
# Linux: 允许非root用户抓包
sudo usermod -aG wireshark $USER
sudo chmod +x /usr/bin/dumpcap

# 或设置SUID
sudo setcap cap_net_raw,cap_net_admin=eip /usr/bin/dumpcap

# 重新登录使权限生效
```

---

## 基础概念

### Capture对象类型

```python
import pyshark

# 1. LiveCapture - 实时抓包
capture = pyshark.LiveCapture(interface='eth0')

# 2. FileCapture - 读取PCAP文件
capture = pyshark.FileCapture('traffic.pcap')

# 3. RemoteCapture - 远程抓包
capture = pyshark.RemoteCapture('192.168.1.1', 'rpcapd')

# 4. InMemCapture - 内存抓包
capture = pyshark.InMemCapture()
```

### 数据包结构

```python
import pyshark

capture = pyshark.FileCapture('sample.pcap')
packet = capture[0]

# 数据包层次结构
print(packet.layers)  # 所有协议层

# 访问特定层
eth_layer = packet.eth  # 以太网层
ip_layer = packet.ip  # IP层
tcp_layer = packet.tcp  # TCP层

# 层的字段
print(packet.ip.src)  # 源IP
print(packet.ip.dst)  # 目标IP
print(packet.tcp.srcport)  # 源端口
```

---

## 抓包操作

### 实时抓包

```python
import pyshark

# 简单抓包
capture = pyshark.LiveCapture(interface='en0')

# 抓取前10个数据包
for packet in capture.sniff_continuously(packet_count=10):
    print(f'Time: {packet.sniff_time}')
    print(f'Protocol: {packet.highest_layer}')
    print(f'Length: {packet.length}')
    print('-' * 50)

capture.close()
```

### 带过滤的抓包

```python
import pyshark

# BPF过滤器
capture = pyshark.LiveCapture(
    interface='en0',
    bpf_filter='tcp port 80'  # 只抓HTTP流量
)

# 抓取指定时间
capture.sniff(timeout=30)  # 抓取30秒

# 遍历数据包
for packet in capture:
    try:
        print(f'HTTP: {packet.http.request_uri}')
    except AttributeError:
        pass
```

### 保存抓包结果

```python
import pyshark

# 抓包并保存
capture = pyshark.LiveCapture(
    interface='en0',
    output_file='captured.pcap'
)

# 抓取100个包
capture.sniff(packet_count=100)
capture.close()

print("抓包完成,已保存到 captured.pcap")
```

---

## PCAP文件分析

### 读取PCAP文件

```python
import pyshark

# 打开PCAP文件
capture = pyshark.FileCapture('traffic.pcap')

# 统计基本信息
packet_count = 0
protocols = {}

for packet in capture:
    packet_count += 1
    protocol = packet.highest_layer
    protocols[protocol] = protocols.get(protocol, 0) + 1

print(f"总数据包数: {packet_count}")
print(f"协议分布: {protocols}")

capture.close()
```

### 使用Display过滤器

```python
import pyshark

# Display过滤器(Wireshark语法)
capture = pyshark.FileCapture(
    'traffic.pcap',
    display_filter='http.request.method == "POST"'
)

# 只处理POST请求
for packet in capture:
    print(f'POST to: {packet.http.request_uri}')
    if hasattr(packet.http, 'file_data'):
        print(f'Data: {packet.http.file_data}')

capture.close()
```

### 常用Display过滤器

```python
# HTTP过滤
'http'                           # 所有HTTP
'http.request'                   # HTTP请求
'http.response'                  # HTTP响应
'http.request.method == "GET"'   # GET请求
'http.host == "example.com"'     # 特定主机

# TCP过滤
'tcp.port == 80'                 # 端口80
'tcp.flags.syn == 1'             # SYN包
'tcp.stream eq 0'                # TCP流0

# IP过滤
'ip.addr == 192.168.1.1'         # IP地址
'ip.src == 10.0.0.1'             # 源IP
'ip.dst == 10.0.0.2'             # 目标IP

# DNS过滤
'dns.qry.name == "example.com"'  # DNS查询
'dns.flags.response == 1'        # DNS响应

# 组合过滤
'http and ip.addr == 192.168.1.1'
'tcp.port == 80 or tcp.port == 443'
```

---

## 协议解析

### HTTP协议分析

```python
import pyshark

capture = pyshark.FileCapture('http_traffic.pcap', display_filter='http')

for packet in capture:
    try:
        # HTTP请求
        if hasattr(packet.http, 'request_method'):
            print(f"\n[REQUEST]")
            print(f"Method: {packet.http.request_method}")
            print(f"URI: {packet.http.request_uri}")
            print(f"Host: {packet.http.host}")

            if hasattr(packet.http, 'user_agent'):
                print(f"User-Agent: {packet.http.user_agent}")

            if hasattr(packet.http, 'cookie'):
                print(f"Cookie: {packet.http.cookie}")

        # HTTP响应
        if hasattr(packet.http, 'response_code'):
            print(f"\n[RESPONSE]")
            print(f"Code: {packet.http.response_code}")
            print(f"Phrase: {packet.http.response_phrase}")

            if hasattr(packet.http, 'content_type'):
                print(f"Content-Type: {packet.http.content_type}")

    except AttributeError:
        pass

capture.close()
```

### DNS协议分析

```python
import pyshark

capture = pyshark.FileCapture('dns_traffic.pcap', display_filter='dns')

for packet in capture:
    try:
        # DNS查询
        if hasattr(packet.dns, 'qry_name'):
            print(f"[Query] {packet.dns.qry_name}")

        # DNS响应
        if hasattr(packet.dns, 'a'):
            print(f"[Answer] {packet.dns.qry_name} -> {packet.dns.a}")

    except AttributeError:
        pass

capture.close()
```

### TCP流分析

```python
import pyshark

capture = pyshark.FileCapture('tcp_traffic.pcap')

# 分析TCP流
tcp_streams = {}

for packet in capture:
    if hasattr(packet, 'tcp'):
        stream_id = packet.tcp.stream

        if stream_id not in tcp_streams:
            tcp_streams[stream_id] = {
                'packets': [],
                'bytes': 0
            }

        tcp_streams[stream_id]['packets'].append(packet)
        tcp_streams[stream_id]['bytes'] += int(packet.length)

# 输出统计
for stream_id, data in tcp_streams.items():
    print(f"Stream {stream_id}: {len(data['packets'])} packets, {data['bytes']} bytes")

capture.close()
```

---

## 数据提取

### 提取HTTP文件

```python
import pyshark
import base64

def extract_http_files(pcap_file, output_dir='extracted'):
    """从PCAP中提取HTTP传输的文件"""
    import os
    os.makedirs(output_dir, exist_ok=True)

    capture = pyshark.FileCapture(pcap_file, display_filter='http')

    file_count = 0
    for packet in capture:
        try:
            # 检查HTTP响应中的文件数据
            if hasattr(packet.http, 'file_data'):
                file_data = packet.http.file_data

                # 获取Content-Type
                content_type = packet.http.content_type if hasattr(packet.http, 'content_type') else 'unknown'

                # 确定文件扩展名
                ext_map = {
                    'image/png': '.png',
                    'image/jpeg': '.jpg',
                    'image/gif': '.gif',
                    'text/html': '.html',
                    'application/pdf': '.pdf',
                    'application/zip': '.zip'
                }
                ext = ext_map.get(content_type.split(';')[0], '.bin')

                # 保存文件
                file_count += 1
                filename = f'{output_dir}/file_{file_count}{ext}'

                # 文件数据是十六进制字符串,需要转换
                file_bytes = bytes.fromhex(file_data.replace(':', ''))

                with open(filename, 'wb') as f:
                    f.write(file_bytes)

                print(f"提取文件: {filename} ({content_type})")

        except Exception as e:
            print(f"错误: {e}")

    capture.close()
    print(f"\n总共提取 {file_count} 个文件")

# 使用示例
extract_http_files('web_traffic.pcap')
```

### 提取FTP文件

```python
import pyshark

def extract_ftp_files(pcap_file):
    """提取FTP传输的文件"""
    capture = pyshark.FileCapture(pcap_file, display_filter='ftp-data')

    for packet in capture:
        if hasattr(packet, 'ftp_data'):
            # FTP数据通道内容
            data = packet.ftp_data.command_data
            print(f"FTP Data: {data[:100]}...")  # 前100字节

    capture.close()

extract_ftp_files('ftp_traffic.pcap')
```

### 提取凭据信息

```python
import pyshark

def extract_credentials(pcap_file):
    """提取明文传输的凭据"""
    capture = pyshark.FileCapture(pcap_file)

    print("正在搜索凭据...")

    for packet in capture:
        try:
            # HTTP基本认证
            if hasattr(packet, 'http') and hasattr(packet.http, 'authorization'):
                auth = packet.http.authorization
                if 'Basic' in auth:
                    print(f"[HTTP Auth] {auth}")

            # FTP登录
            if hasattr(packet, 'ftp'):
                if hasattr(packet.ftp, 'request_command'):
                    cmd = packet.ftp.request_command
                    if cmd in ['USER', 'PASS']:
                        arg = packet.ftp.request_arg
                        print(f"[FTP] {cmd}: {arg}")

            # SMTP认证
            if hasattr(packet, 'smtp'):
                if hasattr(packet.smtp, 'req_parameter'):
                    param = packet.smtp.req_parameter
                    if 'AUTH' in str(param):
                        print(f"[SMTP Auth] {param}")

        except AttributeError:
            pass

    capture.close()

extract_credentials('network.pcap')
```

---

## CTF常见场景

### 场景1: USB键盘流量分析

```python
import pyshark

# USB键盘扫描码映射表
usb_codes = {
    0x04: 'a', 0x05: 'b', 0x06: 'c', 0x07: 'd', 0x08: 'e',
    0x09: 'f', 0x0a: 'g', 0x0b: 'h', 0x0c: 'i', 0x0d: 'j',
    0x0e: 'k', 0x0f: 'l', 0x10: 'm', 0x11: 'n', 0x12: 'o',
    0x13: 'p', 0x14: 'q', 0x15: 'r', 0x16: 's', 0x17: 't',
    0x18: 'u', 0x19: 'v', 0x1a: 'w', 0x1b: 'x', 0x1c: 'y',
    0x1d: 'z', 0x1e: '1', 0x1f: '2', 0x20: '3', 0x21: '4',
    0x22: '5', 0x23: '6', 0x24: '7', 0x25: '8', 0x26: '9',
    0x27: '0', 0x28: '\n', 0x2c: ' '
}

def parse_usb_keyboard(pcap_file):
    """解析USB键盘流量"""
    capture = pyshark.FileCapture(pcap_file)

    keystrokes = []

    for packet in capture:
        try:
            if hasattr(packet, 'usb') and hasattr(packet.usb, 'capdata'):
                data = packet.usb.capdata.replace(':', '')

                # 键盘数据格式: [修饰键][保留][按键码]...
                if len(data) >= 6:
                    key_code = int(data[4:6], 16)

                    if key_code in usb_codes:
                        keystrokes.append(usb_codes[key_code])

        except:
            pass

    capture.close()

    typed_text = ''.join(keystrokes)
    print(f"键盘输入: {typed_text}")
    return typed_text

# 使用示例
parse_usb_keyboard('usb_keyboard.pcap')
```

### 场景2: TCP流重组

```python
import pyshark

def follow_tcp_stream(pcap_file, stream_id):
    """重组TCP流"""
    capture = pyshark.FileCapture(
        pcap_file,
        display_filter=f'tcp.stream eq {stream_id}'
    )

    stream_data = []

    for packet in capture:
        if hasattr(packet, 'tcp') and hasattr(packet.tcp, 'payload'):
            # 提取TCP载荷
            payload = packet.tcp.payload.replace(':', '')
            payload_bytes = bytes.fromhex(payload)

            stream_data.append({
                'time': packet.sniff_time,
                'src': f"{packet.ip.src}:{packet.tcp.srcport}",
                'dst': f"{packet.ip.dst}:{packet.tcp.dstport}",
                'data': payload_bytes
            })

    capture.close()

    # 输出重组数据
    print(f"TCP流 {stream_id} 重组:")
    for segment in stream_data:
        print(f"\n{segment['time']} {segment['src']} -> {segment['dst']}")
        print(f"Data: {segment['data'][:100]}")

    # 合并所有数据
    full_data = b''.join([s['data'] for s in stream_data])
    return full_data

# 使用示例
data = follow_tcp_stream('traffic.pcap', stream_id=0)
with open('stream_0.bin', 'wb') as f:
    f.write(data)
```

### 场景3: 无线流量分析

```python
import pyshark

def analyze_wifi_traffic(pcap_file):
    """分析WiFi流量"""
    capture = pyshark.FileCapture(pcap_file)

    ssids = set()
    clients = set()

    for packet in capture:
        try:
            # 802.11协议
            if hasattr(packet, 'wlan'):
                # SSID
                if hasattr(packet.wlan, 'ssid'):
                    ssid = packet.wlan.ssid
                    if ssid:
                        ssids.add(ssid)

                # 客户端MAC地址
                if hasattr(packet.wlan, 'sa'):
                    clients.add(packet.wlan.sa)

        except:
            pass

    capture.close()

    print("发现的SSID:")
    for ssid in ssids:
        print(f"  - {ssid}")

    print(f"\n客户端数量: {len(clients)}")

analyze_wifi_traffic('wifi.pcap')
```

### 场景4: ICMP隐蔽通道

```python
import pyshark

def detect_icmp_covert_channel(pcap_file):
    """检测ICMP隐蔽通道"""
    capture = pyshark.FileCapture(pcap_file, display_filter='icmp')

    data_extracted = []

    for packet in capture:
        try:
            if hasattr(packet.icmp, 'data'):
                # ICMP数据部分
                data = packet.icmp.data.replace(':', '')
                data_bytes = bytes.fromhex(data)

                # 检查是否为可打印字符
                try:
                    text = data_bytes.decode('ascii')
                    if text.isprintable():
                        data_extracted.append(text)
                        print(f"发现可疑数据: {text}")
                except:
                    pass

        except:
            pass

    capture.close()

    full_message = ''.join(data_extracted)
    print(f"\n提取的消息: {full_message}")
    return full_message

detect_icmp_covert_channel('icmp_covert.pcap')
```

---

## 实战案例

### 案例1: 提取图片

```python
import pyshark
import re

def extract_images_from_pcap(pcap_file, output_dir='images'):
    """从HTTP流量中提取图片"""
    import os
    os.makedirs(output_dir, exist_ok=True)

    capture = pyshark.FileCapture(pcap_file, display_filter='http')

    image_count = 0

    for packet in capture:
        try:
            if hasattr(packet.http, 'content_type'):
                content_type = packet.http.content_type

                # 检查是否为图片
                if 'image' in content_type:
                    if hasattr(packet.http, 'file_data'):
                        file_data = packet.http.file_data.replace(':', '')
                        image_bytes = bytes.fromhex(file_data)

                        # 确定扩展名
                        if 'png' in content_type:
                            ext = '.png'
                        elif 'jpeg' in content_type or 'jpg' in content_type:
                            ext = '.jpg'
                        elif 'gif' in content_type:
                            ext = '.gif'
                        else:
                            ext = '.bin'

                        image_count += 1
                        filename = f'{output_dir}/image_{image_count}{ext}'

                        with open(filename, 'wb') as f:
                            f.write(image_bytes)

                        print(f"提取图片: {filename}")

        except Exception as e:
            pass

    capture.close()
    print(f"总共提取 {image_count} 张图片")

extract_images_from_pcap('web.pcap')
```

### 案例2: 密码爆破检测

```python
import pyshark
from collections import defaultdict

def detect_brute_force(pcap_file):
    """检测密码爆破攻击"""
    capture = pyshark.FileCapture(pcap_file)

    # 统计每个IP的登录尝试
    login_attempts = defaultdict(list)

    for packet in capture:
        try:
            # SSH登录尝试
            if hasattr(packet, 'ssh'):
                src_ip = packet.ip.src
                login_attempts[src_ip].append(packet.sniff_time)

            # HTTP POST (可能是登录)
            if hasattr(packet, 'http'):
                if hasattr(packet.http, 'request_method'):
                    if packet.http.request_method == 'POST':
                        if 'login' in packet.http.request_uri.lower():
                            src_ip = packet.ip.src
                            login_attempts[src_ip].append(packet.sniff_time)

        except:
            pass

    capture.close()

    # 分析结果
    print("可疑登录活动:")
    for ip, attempts in login_attempts.items():
        if len(attempts) > 10:  # 阈值
            print(f"\n[!] {ip}: {len(attempts)} 次尝试")
            print(f"    时间跨度: {attempts[0]} 到 {attempts[-1]}")

detect_brute_force('auth.pcap')
```

### 案例3: DNS隧道检测

```python
import pyshark

def detect_dns_tunneling(pcap_file):
    """检测DNS隧道"""
    capture = pyshark.FileCapture(pcap_file, display_filter='dns')

    suspicious_domains = []

    for packet in capture:
        try:
            if hasattr(packet.dns, 'qry_name'):
                domain = packet.dns.qry_name

                # 检测特征
                # 1. 子域名长度异常
                subdomains = domain.split('.')
                for subdomain in subdomains:
                    if len(subdomain) > 30:
                        suspicious_domains.append(domain)
                        print(f"[!] 异常长子域名: {domain}")
                        break

                # 2. 熵值异常(随机字符串)
                import math
                from collections import Counter

                def entropy(s):
                    p, lns = Counter(s), float(len(s))
                    return -sum(count / lns * math.log(count / lns, 2) for count in p.values())

                if entropy(domain) > 3.5:  # 高熵值
                    suspicious_domains.append(domain)
                    print(f"[!] 高熵值域名: {domain} (熵={entropy(domain):.2f})")

        except:
            pass

    capture.close()

    print(f"\n发现 {len(set(suspicious_domains))} 个可疑域名")

detect_dns_tunneling('dns.pcap')
```

---

## 高级技巧

### 自定义解析器

```python
import pyshark

class CustomProtocolAnalyzer:
    def __init__(self, pcap_file):
        self.capture = pyshark.FileCapture(pcap_file)

    def analyze(self):
        for packet in self.capture:
            # 自定义分析逻辑
            self.process_packet(packet)

    def process_packet(self, packet):
        # 实现自定义协议解析
        pass

analyzer = CustomProtocolAnalyzer('custom.pcap')
analyzer.analyze()
```

### 并发处理

```python
import pyshark
from concurrent.futures import ThreadPoolExecutor

def process_packet(packet):
    """处理单个数据包"""
    # 自定义处理逻辑
    return packet.highest_layer

def analyze_pcap_concurrent(pcap_file, max_workers=4):
    capture = pyshark.FileCapture(pcap_file)

    with ThreadPoolExecutor(max_workers=max_workers) as executor:
        results = executor.map(process_packet, capture)

    return list(results)

# 使用示例
results = analyze_pcap_concurrent('large.pcap')
```

---

## 常见问题解决

### 问题1: tshark未找到

```bash
# 确认tshark路径
which tshark

# 设置tshark路径
import pyshark
pyshark.config.tshark_path = '/usr/local/bin/tshark'
```

### 问题2: 权限不足

```bash
# Linux: 添加用户到wireshark组
sudo usermod -aG wireshark $USER

# 设置dumpcap权限
sudo setcap cap_net_raw,cap_net_admin=eip /usr/bin/dumpcap
```

### 问题3: 内存不足

```python
# 使用eventloop方式处理大文件
capture = pyshark.FileCapture('large.pcap', use_json=True, include_raw=True)

# 或使用keep_packets=False
capture = pyshark.FileCapture('large.pcap', keep_packets=False)

for packet in capture:
    # 处理数据包
    pass  # 处理完立即释放
```

---

## 参考资源

### 官方资源
- **GitHub**: https://github.com/KimiNewt/pyshark
- **文档**: https://pyshark.readthedocs.io/
- **Wireshark**: https://www.wireshark.org/

### 学习资源
- **Wireshark用户指南**: https://www.wireshark.org/docs/wsug_html_chunked/
- **Display过滤器参考**: https://www.wireshark.org/docs/dfref/
- **CTF流量分析**: https://ctf-wiki.org/misc/traffic/

### 相关工具
- **Scapy**: Python数据包操作库
- **Tshark**: Wireshark命令行版本
- **NetworkMiner**: 网络取证工具

---

**文档版本**: v1.0
**更新日期**: 2025-01
**适用版本**: Pyshark v0.6+
