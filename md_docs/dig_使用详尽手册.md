# dig 使用详尽手册

面向 CTF / 渗透测试 与 日常排错的 `dig` 参考手册：基本用法、常用选项、输出字段解释、进阶技巧和大量示例。

---

## 目录
1. 基本命令结构
2. 常用记录类型
3. 重要选项与 `+` 开关
4. 输出结构说明
5. 常见实战命令（CTF / 渗透场景）
6. 进阶技巧与子域收集
7. 常见坑与调试提示
8. 快速参考表（Cheat sheet）
9. 示例（实战案例）
10. 推荐练习

---

## 1) 基本命令结构

```
dig [@server] [name] [type] [options]
```

- `@server`：查询的 DNS 服务器（可选），例如 `@8.8.8.8`。
- `name`：要查询的域名或 IP（反向查询见 `-x`）。
- `type`：记录类型（A, AAAA, MX, TXT, NS, SOA, CNAME, PTR, SRV, ANY, AXFR 等）。
- `options`：dig 的 `+` 开头的特殊选项（如 `+short`、`+trace` 等）。

示例：
```bash
# 查询 A 记录（默认）
dig example.com A
# 查询 TXT 记录
dig example.com TXT
# 使用特定 DNS 服务器查询
dig @1.1.1.1 example.com A
```

---

## 2) 常用记录类型（`-t` 或直接写类型）

- `A`：IPv4 地址。
- `AAAA`：IPv6 地址。
- `MX`：邮件交换记录。
- `TXT`：文本记录（常包含 SPF、验证字符串等）。
- `NS`：权威域名服务器。
- `SOA`：Start of Authority（区域起始记录）。
- `CNAME`：别名记录。
- `PTR`：反向查找（IP -> 域名）。
- `SRV`：服务记录（例如 XMPP、SIP）。
- `DNSKEY` / `RRSIG` / `DS`：DNSSEC 相关记录。
- `ANY`：请求所有可返回类型（现代 DNS 对 ANY 常有限制）。
- `AXFR`：区域传输（zone transfer）。

示例：
```bash
dig -t TXT example.com
```

---

## 3) 重要选项与 `+` 开关（实战常用）

- `+short`：只显示关键结果，便于脚本化。
  - 例：`dig +short example.com A`。
- `+noall +answer`：只显示 ANSWER 部分，清爽可读。
  - 例：`dig example.com +noall +answer`。
- `+trace`：递归追踪（从根服务器开始），用于排查委派/传播问题。
  - 例：`dig +trace example.com`。
- `+tcp`：强制使用 TCP（默认 UDP），用于较大响应或调试。
- `+dnssec`：请求并显示 DNSSEC 相关（RRSIG、DNSKEY）。
- `+time=SECONDS`：设置超时（秒）。
- `+tries=N`：重试次数。
- `+multiline`：把长记录分行显示（便于阅读 TXT/SPF）。
- `+qr`：显示 query/response header（调试低层头信息）。
- `+nocomments`：去掉输出中注释行。
- `+noedns` / `+edns=0`：禁用/设置 EDNS。
- `+bufsize=NNN`：设置 EDNS buffer size。
- `-x`：反向 PTR 查询（IP -> 名称）。
  - 例：`dig -x 8.8.8.8 +short`。

---

## 4) 输出结构（典型 `dig example.com`）

```
; <<>> DiG 9.x.x <<>> example.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 12345
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 2, ADDITIONAL: 3

;; QUESTION SECTION:
;example.com.                   IN      A

;; ANSWER SECTION:
example.com.            3600    IN      A       93.184.216.34

;; AUTHORITY SECTION:
...

;; ADDITIONAL SECTION:
...

;; Query time: 15 msec
;; SERVER: 8.8.8.8#53(8.8.8.8)
;; WHEN: Sat Sep 20 15:00:00 UTC 2025
;; MSG SIZE  rcvd: 123
```

- `HEADER`：状态码（NOERROR、NXDOMAIN 等）与标志（qr, aa, rd, ra, ad）。
- `QUESTION`：查询的问题。
- `ANSWER`：返回的记录（最重要）。
- `AUTHORITY`：权威 NS 信息（委派信息）。
- `ADDITIONAL`：附加记录（如 NS 的 A/AAAA）。
- `Query time` / `SERVER` / `WHEN`：性能与所用服务器信息。

提示：想要最简洁的结果，用 `+short` 或 `+noall +answer`。

---

## 5) 常见实战命令（CTF / 渗透场景）

- 快速只看 IP：
```bash
dig +short example.com A
```

- 查看 MX：
```bash
dig +short example.com MX
```

- 查看 TXT（SPF 等）并按行展示：
```bash
dig example.com TXT +multiline +noall +answer
# 或
dig +short TXT example.com
```

- 反向查 IP：
```bash
dig -x 203.0.113.5 +short
```

- 指定 DNS 服务器（例如 8.8.8.8）：
```bash
dig @8.8.8.8 example.com A
```

- 使用 TCP（防止 UDP 截断）：
```bash
dig +tcp example.com ANY
```

- 追踪解析路径（从根开始）：
```bash
dig +trace example.com
```

- 检查 DNSSEC 签名（显示 RRSIG）：
```bash
dig example.com DNSKEY +dnssec
dig example.com A +dnssec
```

- 尝试 zone transfer（AXFR）：
```bash
dig @ns1.example.com example.com AXFR
```
> 注意：大多数权威 NS 已禁用 AXFR 给任意客户端；若成功则可直接获得完整 zone，常见 CTF 漏洞点。

- 列出域名服务器（NS）以及对应 IP（用于后续 AXFR 测试）：
```bash
dig example.com NS +short
dig @<ns> example.com AXFR
```

- 查询 SOA（序列号、refresh 等）：
```bash
dig example.com SOA +noall +answer
```

- 查询某子域名：
```bash
dig spf.example.com TXT +short
```

- 查询所有记录（注意 ANY 在现代 DNS 中被限制）：
```bash
dig example.com ANY
```

- 把 TXT 输出写到文件（脚本场景）：
```bash
dig example.com TXT +short > txt_records.txt
```

- 批量查询（Bash 单行示例）：
```bash
for d in $(cat domains.txt); do echo "---- $d ----"; dig +short $d TXT; done
```

---

## 6) 进阶：解析子域发现 / 被动信息搜集的组合命令

- 把 `dig` 与 `crt.sh`、`amass`、`subfinder`、`massdns` 等工具配合使用，用于子域发现与验证。
- 在 CTF 中的实战流程常是：
  1. 用 `crt.sh` / `SecurityTrails` / `VirusTotal` 找子域候选列表。
  2. 用 `dig +short` 批量验证哪些子域存在以及它们的 A/TXT/MX 记录。
  3. 对可疑 NS 做 AXFR 测试、对 IP 做反向/证书分析。

示例：
```bash
# crt.sh 结果保存为 domains.txt 后批量验证 TXT
for d in $(cat domains.txt); do dig +short $d TXT; done
```

---

## 7) 常见 Pitfall（踩坑与调试提示）

- `dig +short` 不显示状态或头信息——仅用于脚本或快速确认。
- `ANY` 在现代 DNS 中经常被过滤或返回部分记录（不可靠）。
- 出现 `status: NXDOMAIN` 表示域名不存在或未被委派。
- `dig @ns example.com AXFR` 若成功说明 NS 配置错误—这是重要漏洞。
- 使用 `+trace` 可以定位是哪一级的委派配置不正确。
- 对于很长的 TXT/SPF 记录，使用 `+multiline` 更易阅读和复制。

---

## 8) 快速参考表（Cheat sheet）

| 命令 | 说明 |
|---|---|
| `dig +short example.com A` | 只显示 A 记录 IP |
| `dig example.com MX +short` | 只显示 MX 主机（简洁） |
| `dig example.com TXT +multiline +noall +answer` | 可读的 TXT（SPF）输出 |
| `dig -x 203.0.113.5 +short` | 反向 PTR |
| `dig @8.8.8.8 example.com A` | 指定 DNS 服务器查询 |
| `dig +trace example.com` | 从根开始的解析追踪 |
| `dig example.com DNSKEY +dnssec` | 查询 DNSSEC 公钥/签名 |
| `dig @ns1.example.com example.com AXFR` | 尝试 zone transfer（AXFR） |
| `dig example.com SOA +noall +answer` | 查看 SOA 记录（serial 等） |
| `dig example.com ANY +tcp` | 用 TCP 请求 ANY（注意被限制） |

---

## 9) 示例（实战案例）

1. **查看 SPF（分行显示，便于复制）**：
```bash
dig example.com TXT +multiline +noall +answer
```

2. **检查某个 NS 是否允许 zone transfer**：
```bash
for ns in $(dig +short example.com NS); do echo "TRY $ns"; dig @$ns example.com AXFR; done
```

3. **用 8.8.8.8 强制 TCP 并只看 ANSWER**：
```bash
dig @8.8.8.8 example.com A +tcp +noall +answer
```

4. **追踪子域委派问题（为什么 `sub.example.com` 解析不到）**：
```bash
dig +trace sub.example.com
```

---

## 10) 推荐练习（快速上手）

- 用 `dig +short` 检查 `google.com`、`github.com` 的 A/AAAA/MX/TXT。
- 找一个 CTF 目标域名，跑 `dig <domain> NS`，再对 NS 尝试 `AXFR`（注意合法性）。
- 用 `dig +trace` 观察某个新注册域名的解析链路。
- 在本地脚本里把 `dig +short` 的输出作为后续脚本的输入（自动化批量核验）。

---

如果你需要：
- 我可以把这个 Markdown 导出为 PDF；
- 或生成一个一键 bash 脚本：输入域名自动跑常用查询并把结果归档。

告诉我你想要哪一种，我来直接生成。

