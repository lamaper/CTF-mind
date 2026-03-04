# Pycryptodome 使用手册（Python加密库）

> Pycryptodome 是一个独立的 Python 加密库，提供了各种加密算法和协议的实现。

---

## 目录
1. [概述](#概述)
2. [安装方法](#安装方法)
3. [对称加密](#对称加密)
4. [非对称加密](#非对称加密)
5. [哈希函数](#哈希函数)
6. [CTF常见场景](#ctf常见场景)
7. [实战案例](#实战案例)
8. [参考资源](#参考资源)

---

## 概述

Pycryptodome 是 PyCrypto 的分支，提供：
- **对称加密**：AES、DES、3DES、ChaCha20 等
- **非对称加密**：RSA、DSA、ECC 等
- **哈希函数**：SHA-1、SHA-2、SHA-3、MD5 等
- **消息认证码**：HMAC、CMAC 等
- **密钥派生**：PBKDF2、scrypt 等

**适用场景**：
- CTF 密码学题目
- 数据加密/解密
- 密码破解
- 数字签名验证

---

## 安装方法

```bash
# 安装 Pycryptodome
pip install pycryptodome

# 或安装 Pycryptodomex（独立版本）
pip install pycryptodomex

# 验证安装
python -c "from Crypto.Cipher import AES; print('OK')"
```

---

## 对称加密

### AES 加密/解密

**AES-ECB 模式**：
```python
from Crypto.Cipher import AES
from Crypto.Util.Padding import pad, unpad

# 密钥必须是 16/24/32 字节
key = b'Sixteen byte key'
plaintext = b'Hello, World!'

# 加密
cipher = AES.new(key, AES.MODE_ECB)
ciphertext = cipher.encrypt(pad(plaintext, AES.block_size))
print(f"Ciphertext: {ciphertext.hex()}")

# 解密
cipher = AES.new(key, AES.MODE_ECB)
decrypted = unpad(cipher.decrypt(ciphertext), AES.block_size)
print(f"Decrypted: {decrypted.decode()}")
```

**AES-CBC 模式**：
```python
from Crypto.Cipher import AES
from Crypto.Random import get_random_bytes
from Crypto.Util.Padding import pad, unpad

key = b'Sixteen byte key'
plaintext = b'Hello, World!'

# 加密
iv = get_random_bytes(16)  # 初始化向量
cipher = AES.new(key, AES.MODE_CBC, iv)
ciphertext = cipher.encrypt(pad(plaintext, AES.block_size))

print(f"IV: {iv.hex()}")
print(f"Ciphertext: {ciphertext.hex()}")

# 解密
cipher = AES.new(key, AES.MODE_CBC, iv)
decrypted = unpad(cipher.decrypt(ciphertext), AES.block_size)
print(f"Decrypted: {decrypted.decode()}")
```

**AES-CTR 模式**：
```python
from Crypto.Cipher import AES
from Crypto.Util import Counter

key = b'Sixteen byte key'
plaintext = b'Hello, World! This is CTR mode.'

# 加密
counter = Counter.new(128)
cipher = AES.new(key, AES.MODE_CTR, counter=counter)
ciphertext = cipher.encrypt(plaintext)

# 解密
counter = Counter.new(128)
cipher = AES.new(key, AES.MODE_CTR, counter=counter)
decrypted = cipher.decrypt(ciphertext)
print(f"Decrypted: {decrypted.decode()}")
```

**AES-GCM 模式（带认证）**：
```python
from Crypto.Cipher import AES
from Crypto.Random import get_random_bytes

key = get_random_bytes(16)
plaintext = b'Hello, World!'

# 加密
cipher = AES.new(key, AES.MODE_GCM)
ciphertext, tag = cipher.encrypt_and_digest(plaintext)
nonce = cipher.nonce

print(f"Nonce: {nonce.hex()}")
print(f"Ciphertext: {ciphertext.hex()}")
print(f"Tag: {tag.hex()}")

# 解密
cipher = AES.new(key, AES.MODE_GCM, nonce=nonce)
decrypted = cipher.decrypt_and_verify(ciphertext, tag)
print(f"Decrypted: {decrypted.decode()}")
```

### DES/3DES 加密

```python
from Crypto.Cipher import DES, DES3
from Crypto.Util.Padding import pad, unpad

# DES (密钥 8 字节)
key = b'8bytekey'
plaintext = b'Hello!'

cipher = DES.new(key, DES.MODE_ECB)
ciphertext = cipher.encrypt(pad(plaintext, DES.block_size))

# 3DES (密钥 16/24 字节)
key3 = b'Sixteen byte key'
cipher = DES3.new(key3, DES3.MODE_ECB)
ciphertext = cipher.encrypt(pad(plaintext, DES3.block_size))
```

---

## 非对称加密

### RSA 加密/解密

**生成密钥对**：
```python
from Crypto.PublicKey import RSA

# 生成 RSA 密钥对
key = RSA.generate(2048)

# 导出密钥
private_key = key.export_key()
public_key = key.publickey().export_key()

print(private_key.decode())
print(public_key.decode())

# 保存到文件
with open('private.pem', 'wb') as f:
    f.write(private_key)

with open('public.pem', 'wb') as f:
    f.write(public_key)
```

**加密/解密**：
```python
from Crypto.PublicKey import RSA
from Crypto.Cipher import PKCS1_OAEP

# 加载密钥
public_key = RSA.import_key(open('public.pem').read())
private_key = RSA.import_key(open('private.pem').read())

# 加密
plaintext = b'Hello, RSA!'
cipher = PKCS1_OAEP.new(public_key)
ciphertext = cipher.encrypt(plaintext)
print(f"Ciphertext: {ciphertext.hex()}")

# 解密
cipher = PKCS1_OAEP.new(private_key)
decrypted = cipher.decrypt(ciphertext)
print(f"Decrypted: {decrypted.decode()}")
```

**数字签名**：
```python
from Crypto.PublicKey import RSA
from Crypto.Signature import pkcs1_15
from Crypto.Hash import SHA256

# 生成密钥
key = RSA.generate(2048)
private_key = key
public_key = key.publickey()

# 签名
message = b'This is the message to sign'
h = SHA256.new(message)
signature = pkcs1_15.new(private_key).sign(h)
print(f"Signature: {signature.hex()}")

# 验证
h = SHA256.new(message)
try:
    pkcs1_15.new(public_key).verify(h, signature)
    print("Signature is valid")
except (ValueError, TypeError):
    print("Signature is invalid")
```

### RSA 常见攻击（CTF）

**小指数攻击（e=3）**：
```python
import gmpy2
from Crypto.PublicKey import RSA

# 假设 e=3, c = m^3 mod n, 且 m^3 < n
c = 12345678901234567890
n = 99999999999999999999

# 直接开三次方
m, exact = gmpy2.iroot(c, 3)
if exact:
    print(f"Message: {m}")
    print(f"Text: {bytes.fromhex(hex(m)[2:]).decode()}")
```

**共模攻击**：
```python
import gmpy2

def gcd(a, b):
    while b:
        a, b = b, a % b
    return a

def ext_gcd(a, b):
    if b == 0:
        return 1, 0
    else:
        x, y = ext_gcd(b, a % b)
        return y, x - (a // b) * y

# 相同 n, 不同 e
n = 12345678901234567890
e1 = 3
e2 = 5
c1 = 11111111111111111111
c2 = 22222222222222222222

# 计算扩展欧几里得
s, t = ext_gcd(e1, e2)

# 恢复明文
if s < 0:
    c1 = gmpy2.invert(c1, n)
    s = -s
if t < 0:
    c2 = gmpy2.invert(c2, n)
    t = -t

m = (pow(c1, s, n) * pow(c2, t, n)) % n
print(f"Message: {m}")
```

---

## 哈希函数

### 常用哈希算法

```python
from Crypto.Hash import MD5, SHA1, SHA256, SHA512

message = b'Hello, World!'

# MD5
h = MD5.new(message)
print(f"MD5: {h.hexdigest()}")

# SHA-1
h = SHA1.new(message)
print(f"SHA1: {h.hexdigest()}")

# SHA-256
h = SHA256.new(message)
print(f"SHA256: {h.hexdigest()}")

# SHA-512
h = SHA512.new(message)
print(f"SHA512: {h.hexdigest()}")
```

### HMAC 消息认证码

```python
from Crypto.Hash import HMAC, SHA256

key = b'secret_key'
message = b'Hello, World!'

# 生成 HMAC
h = HMAC.new(key, message, digestmod=SHA256)
mac = h.hexdigest()
print(f"HMAC: {mac}")

# 验证 HMAC
h = HMAC.new(key, message, digestmod=SHA256)
try:
    h.hexverify(mac)
    print("HMAC is valid")
except ValueError:
    print("HMAC is invalid")
```

### PBKDF2 密钥派生

```python
from Crypto.Protocol.KDF import PBKDF2
from Crypto.Hash import SHA256

password = b'password123'
salt = b'random_salt'

# 派生密钥（32字节）
key = PBKDF2(password, salt, dkLen=32, count=1000000, hmac_hash_module=SHA256)
print(f"Derived key: {key.hex()}")
```

---

## CTF常见场景

### 场景1：AES ECB 模式选择明文攻击

```python
from Crypto.Cipher import AES
from Crypto.Util.Padding import pad

# 服务器使用固定密钥加密
def oracle(plaintext):
    key = b'Sixteen byte key'  # 未知密钥
    cipher = AES.new(key, AES.MODE_ECB)
    return cipher.encrypt(pad(plaintext, AES.block_size))

# 攻击：检测 ECB 模式
plaintext = b'A' * 64
ciphertext = oracle(plaintext)

# ECB 模式下，相同的明文块加密为相同的密文块
blocks = [ciphertext[i:i+16] for i in range(0, len(ciphertext), 16)]
if len(blocks) != len(set(blocks)):
    print("[+] ECB mode detected!")
```

### 场景2：RSA Wiener 攻击（小d）

```python
import owiener

# 已知 n 和 e
n = 12345678901234567890123456789012345678901234567890
e = 98765432109876543210987654321098765432109876543210

# Wiener 攻击（当 d < n^0.25 时有效）
d = owiener.attack(e, n)

if d:
    print(f"Found d: {d}")
    # 解密
    c = 11111111111111111111111111111111111111111111111111
    m = pow(c, d, n)
    print(f"Message: {m}")
```

### 场景3：CBC 字节翻转攻击

```python
from Crypto.Cipher import AES
from Crypto.Util.Padding import pad, unpad

key = b'Sixteen byte key'

# 加密
iv = b'\x00' * 16
plaintext = b'user=guest;role=user'
cipher = AES.new(key, AES.MODE_CBC, iv)
ciphertext = cipher.encrypt(pad(plaintext, AES.block_size))

# 翻转 IV 以修改第一个块的明文
# 目标：user=admin;role=admin
# 原始：user=guest;role=user
# 需要翻转的位置：索引 5-9 (guest -> admin)

modified_iv = bytearray(iv)
modified_iv[5] ^= ord('g') ^ ord('a')
modified_iv[6] ^= ord('u') ^ ord('d')
modified_iv[7] ^= ord('e') ^ ord('m')
modified_iv[8] ^= ord('s') ^ ord('i')
modified_iv[9] ^= ord('t') ^ ord('n')

# 解密
cipher = AES.new(key, AES.MODE_CBC, bytes(modified_iv))
decrypted = cipher.decrypt(ciphertext)
print(f"Modified: {decrypted[:20]}")  # user=admin;role=...
```

---

## 实战案例

### 案例1：AES 密钥爆破

```python
from Crypto.Cipher import AES
from Crypto.Util.Padding import unpad
import itertools
import string

ciphertext = bytes.fromhex('...')  # 已知密文
known_plaintext = b'flag{'  # 已知明文前缀

# 假设密钥是 4 位数字
for combo in itertools.product(string.digits, repeat=4):
    key = (''.join(combo) * 4)[:16].encode()  # 扩展到 16 字节

    try:
        cipher = AES.new(key, AES.MODE_ECB)
        plaintext = unpad(cipher.decrypt(ciphertext), AES.block_size)

        if plaintext.startswith(known_plaintext):
            print(f"[+] Key found: {key.decode()}")
            print(f"[+] Plaintext: {plaintext.decode()}")
            break
    except:
        continue
```

### 案例2：RSA 分解 n

```python
from Crypto.PublicKey import RSA
from Crypto.Util.number import long_to_bytes
import requests

# 已知 n, e, c
n = 12345678901234567890
e = 65537
c = 11111111111111111111

# 使用 factordb.com 分解 n
response = requests.get(f'http://factordb.com/api?query={n}')
data = response.json()

if data['status'] == 'FF':  # 完全分解
    factors = data['factors']
    p = int(factors[0][0])
    q = int(factors[1][0])

    # 计算私钥
    phi = (p - 1) * (q - 1)
    d = pow(e, -1, phi)

    # 解密
    m = pow(c, d, n)
    print(f"Message: {long_to_bytes(m).decode()}")
```

---

## 参考资源

### 官方文档
- **Pycryptodome 官网**：https://www.pycryptodome.org/
- **GitHub**：https://github.com/Legrandin/pycryptodome
- **文档**：https://pycryptodome.readthedocs.io/

### CTF 工具
- **RsaCtfTool**：https://github.com/Ganapati/RsaCtfTool
- **FeatherDuster**：https://github.com/nccgroup/featherduster
- **CyberChef**：https://gchq.github.io/CyberChef/

---

**文档版本**：v1.0
**更新日期**：2025-01
**适用版本**：Pycryptodome v3.15+
