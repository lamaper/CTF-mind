1. 网页与基础数据

```
text/html                 # HTML 文档
text/plain                # 纯文本
text/css                  # CSS 样式表
application/javascript    # JavaScript 脚本
application/json          # JSON 数据（API 接口常用）
application/xml           # XML 数据
```

2. 表单提交

在构造 POST 请求或利用文件上传漏洞时，必须区分这两种类型。

```
application/x-www-form-urlencoded    # 默认表单提交（key=val&key2=val2）
multipart/form-data                  # 文件上传表单（需配合 boundary 分隔符）
```

3. 图片文件格式

常用于绕过文件类型检查（MIME-Type 校验）或伪造文件头。

```
image/jpeg                # JPG/JPEG 图片
image/png                 # PNG 图片
image/gif                 # GIF 动画（常用于简单的一句话木马隐藏）
image/x-icon              # Icon 图标
image/svg+xml             # SVG 矢量图（可能涉及 XSS 攻击）
```
4. 应用程序与二进制流

处理文件下载、附件上传或特定格式漏洞（如 PDF 溢出）时使用。
Plaintext

```
application/octet-stream  # 二进制流（未知文件类型，常用于文件下载）
application/pdf           # PDF 文档
application/msword        # Word 文档 (.doc)
application/vnd.openxmlformats-officedocument.wordprocessingml.document # .docx
application/zip           # ZIP 压缩包
application/x-7z-compressed # 7-Zip 压缩包
application/x-rar-compressed # RAR 压缩包
```

5. 音视频格式

```
audio/mpeg                # MP3 音频
audio/wav                 # WAV 音频
video/mp4                 # MP4 视频
video/x-msvideo           # AVI 视频
```