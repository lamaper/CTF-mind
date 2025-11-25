# Nginx 使用手册

## 目录
1. [Nginx 简介](#nginx-简介)
2. [安装 Nginx](#安装-nginx)
3. [基本配置](#基本配置)
4. [常用指令](#常用指令)
5. [配置文件结构](#配置文件结构)
6. [常用配置示例](#常用配置示例)
7. [反向代理配置](#反向代理配置)
8. [负载均衡配置](#负载均衡配置)
9. [SSL/HTTPS 配置](#sslhttps-配置)
10. [性能优化](#性能优化)
11. [安全配置](#安全配置)
12. [日志管理](#日志管理)
13. [故障排除](#故障排除)

## Nginx 简介

Nginx（发音为 "engine-x"）是一个高性能的HTTP和反向代理服务器，也是一个IMAP/POP3/SMTP代理服务器。Nginx以其高并发、高性能和低资源消耗而闻名。

### 主要特点
- 高并发处理能力
- 低内存消耗
- 反向代理功能
- 负载均衡
- 静态文件服务
- SSL/TLS支持
- 压缩和缓存

## 安装 Nginx

### Ubuntu/Debian
```bash
sudo apt update
sudo apt install nginx
```

### CentOS/RHEL
```bash
sudo yum install nginx
# 或者使用 dnf (CentOS 8+)
sudo dnf install nginx
```

### macOS (使用 Homebrew)
```bash
brew install nginx
```

### Windows
从官网下载安装包：http://nginx.org/en/download.html

## 基本配置

### 启动 Nginx
```bash
sudo systemctl start nginx
# 或者
sudo service nginx start
```

### 停止 Nginx
```bash
sudo systemctl stop nginx
# 或者
sudo service nginx stop
```

### 重启 Nginx
```bash
sudo systemctl restart nginx
# 或者
sudo service nginx restart
```

### 重新加载配置（不中断服务）
```bash
sudo nginx -s reload
# 或者
sudo systemctl reload nginx
```

### 检查配置文件语法
```bash
sudo nginx -t
```

### 设置开机自启
```bash
sudo systemctl enable nginx
```

## 常用指令

```bash
nginx -h            # 显示帮助信息
nginx -v            # 显示版本信息
nginx -V            # 显示版本和配置信息
nginx -t            # 测试配置文件语法
nginx -T            # 测试配置文件并显示
nginx -s stop       # 快速停止
nginx -s quit       # 优雅停止
nginx -s reload     # 重新加载配置
nginx -s reopen     # 重新打开日志文件
```

## 配置文件结构

Nginx 的主配置文件通常位于：
- `/etc/nginx/nginx.conf` (Linux)
- `/usr/local/nginx/conf/nginx.conf` (源码安装)
- `/usr/local/etc/nginx/nginx.conf` (macOS Homebrew)

### 配置文件基本结构
```nginx
# 全局块
worker_processes  auto;

# events块
events {
    worker_connections  1024;
}

# http块
http {
    # http全局块
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    # 日志格式
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    # 虚拟服务器配置
    server {
        listen       80;
        server_name  localhost;

        # 服务器全局块
        location / {
            root   /usr/share/nginx/html;
            index  index.html index.htm;
        }

        # 错误页面
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   /usr/share/nginx/html;
        }
    }
}
```

## 常用配置示例

### 基本静态网站配置
```nginx
server {
    listen 80;
    server_name example.com www.example.com;
    root /var/www/html;
    index index.html index.htm;

    location / {
        try_files $uri $uri/ =404;
    }

    # 静态文件缓存
    location ~* \.(jpg|jpeg|png|gif|ico|css|js)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }
}
```

### PHP 网站配置
```nginx
server {
    listen 80;
    server_name example.com;
    root /var/www/html;
    index index.php index.html;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;
    }

    location ~ /\.ht {
        deny all;
    }
}
```

## 反向代理配置

### 基本反向代理
```nginx
server {
    listen 80;
    server_name proxy.example.com;

    location / {
        proxy_pass http://backend_server;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### WebSocket 反向代理
```nginx
server {
    listen 80;
    server_name ws.example.com;

    location / {
        proxy_pass http://websocket_backend;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
    }
}
```

## 负载均衡配置

### 轮询（默认）
```nginx
upstream backend {
    server backend1.example.com;
    server backend2.example.com;
    server backend3.example.com;
}

server {
    listen 80;
    server_name example.com;

    location / {
        proxy_pass http://backend;
    }
}
```

### 加权轮询
```nginx
upstream backend {
    server backend1.example.com weight=3;
    server backend2.example.com weight=2;
    server backend3.example.com weight=1;
}
```

### IP 哈希
```nginx
upstream backend {
    ip_hash;
    server backend1.example.com;
    server backend2.example.com;
}
```

### 最少连接
```nginx
upstream backend {
    least_conn;
    server backend1.example.com;
    server backend2.example.com;
}
```

## SSL/HTTPS 配置

### 基本 HTTPS 配置
```nginx
server {
    listen 443 ssl;
    server_name example.com;

    ssl_certificate /path/to/certificate.crt;
    ssl_certificate_key /path/to/private.key;

    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-RSA-AES256-GCM-SHA512:DHE-RSA-AES256-GCM-SHA512:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES256-GCM-SHA384;
    ssl_prefer_server_ciphers off;

    location / {
        root /var/www/html;
        index index.html;
    }
}

# HTTP 重定向到 HTTPS
server {
    listen 80;
    server_name example.com;
    return 301 https://$server_name$request_uri;
}
```

### Let's Encrypt SSL 配置
```nginx
server {
    listen 80;
    server_name example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name example.com;

    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;

    # 其他配置...
}
```

## 性能优化

### 基本优化配置
```nginx
# 工作进程数
worker_processes auto;

# 工作连接数
events {
    worker_connections 1024;
    use epoll;
    multi_accept on;
}

http {
    # 基本优化
    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 65;

    # Gzip 压缩
    gzip on;
    gzip_vary on;
    gzip_min_length 1024;
    gzip_proxied any;
    gzip_comp_level 6;
    gzip_types
        text/plain
        text/css
        text/xml
        text/javascript
        application/json
        application/javascript
        application/xml+rss
        application/atom+xml
        image/svg+xml;

    # 缓冲区优化
    client_body_buffer_size 128k;
    client_max_body_size 10m;
    client_header_buffer_size 1k;
    large_client_header_buffers 4 4k;
    output_buffers 1 32k;
    postpone_output 1460;
}
```

### 静态文件缓存
```nginx
location ~* \.(jpg|jpeg|png|gif|ico|css|js)$ {
    expires 1y;
    add_header Cache-Control "public, immutable";
    access_log off;
}
```

## 安全配置

### 基本安全设置
```nginx
server {
    # 隐藏版本号
    server_tokens off;

    # 防止点击劫持
    add_header X-Frame-Options "SAMEORIGIN" always;

    # 防止 XSS
    add_header X-XSS-Protection "1; mode=block" always;

    # 防止 MIME 类型嗅探
    add_header X-Content-Type-Options "nosniff" always;

    # Content Security Policy
    add_header Content-Security-Policy "default-src 'self' http: https: data: blob: 'unsafe-inline'" always;

    # 限制请求方法
    if ($request_method !~ ^(GET|HEAD|POST)$ ) {
        return 405;
    }

    # 防止访问隐藏文件
    location ~ /\. {
        deny all;
        access_log off;
        log_not_found off;
    }
}
```

### 限制请求频率
```nginx
# 限制每个 IP 的请求频率
limit_req_zone $binary_remote_addr zone=one:10m rate=1r/s;

server {
    location /login {
        limit_req zone=one burst=5;
    }
}
```

### 限制连接数
```nginx
# 限制每个 IP 的连接数
limit_conn_zone $binary_remote_addr zone=addr:10m;

server {
    location /download/ {
        limit_conn addr 1;
    }
}
```

## 日志管理

### 自定义日志格式
```nginx
http {
    log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                    '$status $body_bytes_sent "$http_referer" '
                    '"$http_user_agent" "$http_x_forwarded_for"';

    log_format detailed '$remote_addr - $remote_user [$time_local] "$request" '
                        '$status $body_bytes_sent "$http_referer" '
                        '"$http_user_agent" "$http_x_forwarded_for" '
                        '$request_time $upstream_response_time';

    access_log /var/log/nginx/access.log main;
}
```

### 分割日志
```nginx
# 按天分割日志
map $time_iso8601 $logdate {
    '~^(?<ymd>\d{4}-\d{2}-\d{2})' $ymd;
    default 'nodate';
}

access_log /var/log/nginx/access-$logdate.log main;
```

## 故障排除

### 常见错误及解决方案

#### 502 Bad Gateway
```nginx
# 检查后端服务是否运行
# 增加超时时间
proxy_connect_timeout 300s;
proxy_send_timeout 300s;
proxy_read_timeout 300s;
```

#### 413 Request Entity Too Large
```nginx
# 增加客户端最大请求体大小
client_max_body_size 100M;
```

#### 403 Forbidden
```nginx
# 检查文件权限
# 检查 root 路径是否正确
# 检查 index 文件是否存在
```

#### 404 Not Found
```nginx
# 检查文件路径
# 检查 location 配置
# 检查 try_files 配置
```

### 调试技巧

#### 启用调试日志
```nginx
error_log /var/log/nginx/error.log debug;
```

#### 查看实时日志
```bash
tail -f /var/log/nginx/access.log
tail -f /var/log/nginx/error.log
```

#### 测试配置
```bash
nginx -t
nginx -T
```

## 附录

### 常用变量
- `$remote_addr` - 客户端地址
- `$remote_user` - 客户端用户
- `$time_local` - 本地时间
- `$request` - 请求行
- `$status` - 状态码
- `$http_user_agent` - 用户代理
- `$http_referer` - 引用页
- `$request_time` - 请求处理时间

### 常用模块
- `ngx_http_core_module` - 核心模块
- `ngx_http_access_module` - 访问控制
- `ngx_http_proxy_module` - 反向代理
- `ngx_http_upstream_module` - 负载均衡
- `ngx_http_ssl_module` - SSL 支持
- `ngx_http_gzip_module` - Gzip 压缩

### 参考资源
- [Nginx 官方文档](http://nginx.org/en/docs/)
- [Nginx Wiki](https://www.nginx.com/resources/wiki/)
- [Mozilla SSL 配置生成器](https://ssl-config.mozilla.org/)