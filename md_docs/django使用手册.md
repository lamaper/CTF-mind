# Django 使用手册（Python Web框架）

> Django 是Python最流行的全栈Web框架,在CTF中用于快速搭建Web应用、漏洞环境和在线平台。

---

## 目录
1. [概述](#概述)
2. [安装与配置](#安装与配置)
3. [项目结构](#项目结构)
4. [URL路由](#url路由)
5. [视图与模板](#视图与模板)
6. [模型与数据库](#模型与数据库)
7. [表单处理](#表单处理)
8. [用户认证](#用户认证)
9. [安全机制](#安全机制)
10. [CTF应用场景](#ctf应用场景)
11. [常见漏洞与防御](#常见漏洞与防御)
12. [实战案例](#实战案例)
13. [参考资源](#参考资源)

---

## 概述

Django 是遵循MVT(Model-View-Template)模式的Web框架,提供:
- **ORM系统**: 对象关系映射,简化数据库操作
- **模板引擎**: 动态生成HTML页面
- **URL路由**: 优雅的URL设计
- **Admin后台**: 自动生成管理界面
- **安全特性**: 内置CSRF、XSS、SQL注入防护

**CTF应用场景**:
- 搭建CTF平台
- 创建漏洞靶场
- Web题目环境
- 在线工具开发
- 安全测试环境

---

## 安装与配置

### 基础安装

```bash
# 安装Django
pip install django

# 验证安装
django-admin --version

# 创建新项目
django-admin startproject myproject

# 项目结构
myproject/
├── manage.py           # 管理命令
└── myproject/
    ├── __init__.py
    ├── settings.py     # 配置文件
    ├── urls.py         # URL路由
    ├── asgi.py        # ASGI配置
    └── wsgi.py        # WSGI配置
```

### 创建应用

```bash
# 进入项目目录
cd myproject

# 创建应用
python manage.py startapp myapp

# 应用结构
myapp/
├── __init__.py
├── admin.py          # 管理后台
├── apps.py          # 应用配置
├── models.py        # 数据模型
├── tests.py         # 测试
├── views.py         # 视图函数
└── migrations/      # 数据库迁移
```

### 基本配置

```python
# myproject/settings.py

# 应用注册
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'myapp',  # 添加自己的应用
]

# 数据库配置
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': BASE_DIR / 'db.sqlite3',
    }
}

# 语言和时区
LANGUAGE_CODE = 'zh-hans'
TIME_ZONE = 'Asia/Shanghai'

# 静态文件
STATIC_URL = '/static/'
STATIC_ROOT = BASE_DIR / 'staticfiles'

# 媒体文件
MEDIA_URL = '/media/'
MEDIA_ROOT = BASE_DIR / 'media'
```

---

## 项目结构

### 标准项目布局

```
myproject/
├── manage.py
├── myproject/
│   ├── __init__.py
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
├── myapp/
│   ├── migrations/
│   ├── templates/
│   │   └── myapp/
│   ├── static/
│   │   └── myapp/
│   ├── __init__.py
│   ├── admin.py
│   ├── apps.py
│   ├── models.py
│   ├── views.py
│   └── urls.py
├── templates/          # 全局模板
├── static/            # 全局静态文件
├── media/             # 用户上传文件
└── requirements.txt   # 依赖列表
```

### 常用管理命令

```bash
# 启动开发服务器
python manage.py runserver
python manage.py runserver 0.0.0.0:8000  # 指定地址和端口

# 数据库操作
python manage.py makemigrations  # 生成迁移文件
python manage.py migrate         # 应用迁移

# 创建超级用户
python manage.py createsuperuser

# 收集静态文件
python manage.py collectstatic

# 进入Shell
python manage.py shell

# 运行测试
python manage.py test
```

---

## URL路由

### 基础路由

```python
# myproject/urls.py

from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('myapp.urls')),
]
```

```python
# myapp/urls.py

from django.urls import path
from . import views

app_name = 'myapp'

urlpatterns = [
    path('', views.index, name='index'),
    path('about/', views.about, name='about'),
    path('post/<int:pk>/', views.post_detail, name='post_detail'),
    path('search/', views.search, name='search'),
]
```

### 路由参数

```python
from django.urls import path
from . import views

urlpatterns = [
    # 整数参数
    path('post/<int:id>/', views.post_detail),

    # 字符串参数
    path('user/<str:username>/', views.user_profile),

    # Slug参数
    path('article/<slug:slug>/', views.article_detail),

    # UUID参数
    path('item/<uuid:uuid>/', views.item_detail),

    # 路径参数
    path('file/<path:filepath>/', views.download),
]
```

### 正则路由

```python
from django.urls import re_path
from . import views

urlpatterns = [
    re_path(r'^post/(?P<year>[0-9]{4})/$', views.year_archive),
    re_path(r'^post/(?P<year>[0-9]{4})/(?P<month>[0-9]{2})/$', views.month_archive),
]
```

---

## 视图与模板

### 函数视图

```python
# myapp/views.py

from django.shortcuts import render, redirect, get_object_or_404
from django.http import HttpResponse, JsonResponse
from .models import Post

# 简单视图
def index(request):
    return HttpResponse("Hello, Django!")

# 渲染模板
def post_list(request):
    posts = Post.objects.all()
    return render(request, 'myapp/post_list.html', {'posts': posts})

# JSON响应
def api_data(request):
    data = {'status': 'success', 'message': 'Hello'}
    return JsonResponse(data)

# 重定向
def redirect_view(request):
    return redirect('myapp:index')

# 404处理
def post_detail(request, pk):
    post = get_object_or_404(Post, pk=pk)
    return render(request, 'myapp/post_detail.html', {'post': post})
```

### 类视图

```python
from django.views import View
from django.views.generic import ListView, DetailView, CreateView

# 基础类视图
class PostListView(View):
    def get(self, request):
        posts = Post.objects.all()
        return render(request, 'myapp/post_list.html', {'posts': posts})

# 通用视图
class PostListView(ListView):
    model = Post
    template_name = 'myapp/post_list.html'
    context_object_name = 'posts'
    paginate_by = 10

class PostDetailView(DetailView):
    model = Post
    template_name = 'myapp/post_detail.html'
```

### 模板语法

```django
{# myapp/templates/myapp/post_list.html #}

{% extends 'base.html' %}

{% block title %}文章列表{% endblock %}

{% block content %}
<h1>所有文章</h1>

{% for post in posts %}
    <article>
        <h2><a href="{% url 'myapp:post_detail' post.pk %}">{{ post.title }}</a></h2>
        <p>{{ post.content|truncatewords:30 }}</p>
        <small>发布于 {{ post.created_at|date:"Y-m-d" }}</small>
    </article>
{% empty %}
    <p>暂无文章</p>
{% endfor %}

{# 分页 #}
{% if is_paginated %}
    <div class="pagination">
        {% if page_obj.has_previous %}
            <a href="?page={{ page_obj.previous_page_number }}">上一页</a>
        {% endif %}
        <span>第 {{ page_obj.number }} / {{ page_obj.paginator.num_pages }} 页</span>
        {% if page_obj.has_next %}
            <a href="?page={{ page_obj.next_page_number }}">下一页</a>
        {% endif %}
    </div>
{% endif %}
{% endblock %}
```

---

## 模型与数据库

### 定义模型

```python
# myapp/models.py

from django.db import models
from django.contrib.auth.models import User

class Post(models.Model):
    title = models.CharField(max_length=200, verbose_name='标题')
    content = models.TextField(verbose_name='内容')
    author = models.ForeignKey(User, on_delete=models.CASCADE, verbose_name='作者')
    created_at = models.DateTimeField(auto_now_add=True, verbose_name='创建时间')
    updated_at = models.DateTimeField(auto_now=True, verbose_name='更新时间')
    published = models.BooleanField(default=False, verbose_name='是否发布')

    class Meta:
        ordering = ['-created_at']
        verbose_name = '文章'
        verbose_name_plural = '文章'

    def __str__(self):
        return self.title

class Comment(models.Model):
    post = models.ForeignKey(Post, on_delete=models.CASCADE, related_name='comments')
    author = models.CharField(max_length=100)
    content = models.TextField()
    created_at = models.DateTimeField(auto_now_add=True)

    def __str__(self):
        return f'{self.author} on {self.post.title}'
```

### ORM查询

```python
from myapp.models import Post, Comment

# 创建
post = Post.objects.create(title='标题', content='内容', author=user)

# 查询所有
posts = Post.objects.all()

# 过滤
published_posts = Post.objects.filter(published=True)
recent_posts = Post.objects.filter(created_at__gte='2025-01-01')

# 排除
unpublished = Post.objects.exclude(published=True)

# 获取单个对象
post = Post.objects.get(pk=1)
post = Post.objects.first()  # 第一个
post = Post.objects.last()   # 最后一个

# 更新
Post.objects.filter(pk=1).update(title='新标题')
post = Post.objects.get(pk=1)
post.title = '新标题'
post.save()

# 删除
Post.objects.filter(pk=1).delete()

# 聚合
from django.db.models import Count, Avg
post_count = Post.objects.count()
avg_comments = Post.objects.aggregate(Avg('comments__count'))

# 关联查询
posts_with_comments = Post.objects.prefetch_related('comments')
for post in posts_with_comments:
    for comment in post.comments.all():
        print(comment.content)
```

---

## 表单处理

### 表单定义

```python
# myapp/forms.py

from django import forms
from .models import Post

class PostForm(forms.ModelForm):
    class Meta:
        model = Post
        fields = ['title', 'content', 'published']
        widgets = {
            'title': forms.TextInput(attrs={'class': 'form-control'}),
            'content': forms.Textarea(attrs={'class': 'form-control', 'rows': 5}),
        }

    def clean_title(self):
        title = self.cleaned_data['title']
        if len(title) < 5:
            raise forms.ValidationError('标题至少5个字符')
        return title

class LoginForm(forms.Form):
    username = forms.CharField(max_length=100)
    password = forms.CharField(widget=forms.PasswordInput)

    def clean(self):
        cleaned_data = super().clean()
        # 自定义验证逻辑
        return cleaned_data
```

### 表单处理

```python
# myapp/views.py

from django.shortcuts import render, redirect
from .forms import PostForm

def create_post(request):
    if request.method == 'POST':
        form = PostForm(request.POST)
        if form.is_valid():
            post = form.save(commit=False)
            post.author = request.user
            post.save()
            return redirect('myapp:post_detail', pk=post.pk)
    else:
        form = PostForm()

    return render(request, 'myapp/post_form.html', {'form': form})

def edit_post(request, pk):
    post = get_object_or_404(Post, pk=pk)

    if request.method == 'POST':
        form = PostForm(request.POST, instance=post)
        if form.is_valid():
            form.save()
            return redirect('myapp:post_detail', pk=post.pk)
    else:
        form = PostForm(instance=post)

    return render(request, 'myapp/post_form.html', {'form': form})
```

---

## 用户认证

### 用户注册

```python
# myapp/views.py

from django.contrib.auth.forms import UserCreationForm
from django.contrib.auth import login

def register(request):
    if request.method == 'POST':
        form = UserCreationForm(request.POST)
        if form.is_valid():
            user = form.save()
            login(request, user)
            return redirect('myapp:index')
    else:
        form = UserCreationForm()

    return render(request, 'registration/register.html', {'form': form})
```

### 登录登出

```python
from django.contrib.auth import authenticate, login, logout
from django.contrib.auth.decorators import login_required

def user_login(request):
    if request.method == 'POST':
        username = request.POST['username']
        password = request.POST['password']
        user = authenticate(request, username=username, password=password)

        if user is not None:
            login(request, user)
            return redirect('myapp:index')
        else:
            return render(request, 'registration/login.html', {'error': '用户名或密码错误'})

    return render(request, 'registration/login.html')

def user_logout(request):
    logout(request)
    return redirect('myapp:index')

# 需要登录的视图
@login_required
def profile(request):
    return render(request, 'myapp/profile.html')
```

### 权限控制

```python
from django.contrib.auth.decorators import login_required, permission_required
from django.contrib.auth.mixins import LoginRequiredMixin, PermissionRequiredMixin

# 函数视图权限
@login_required
@permission_required('myapp.add_post', raise_exception=True)
def create_post(request):
    # ...
    pass

# 类视图权限
class PostCreateView(LoginRequiredMixin, PermissionRequiredMixin, CreateView):
    model = Post
    permission_required = 'myapp.add_post'
    # ...
```

---

## 安全机制

### CSRF保护

```python
# 设置中启用(默认已启用)
# settings.py
MIDDLEWARE = [
    'django.middleware.csrf.CsrfViewMiddleware',
    # ...
]

# 模板中使用
# template.html
<form method="post">
    {% csrf_token %}
    {{ form.as_p }}
    <button type="submit">提交</button>
</form>

# AJAX请求
# JavaScript
function getCookie(name) {
    let cookieValue = null;
    if (document.cookie && document.cookie !== '') {
        const cookies = document.cookie.split(';');
        for (let i = 0; i < cookies.length; i++) {
            const cookie = cookies[i].trim();
            if (cookie.substring(0, name.length + 1) === (name + '=')) {
                cookieValue = decodeURIComponent(cookie.substring(name.length + 1));
                break;
            }
        }
    }
    return cookieValue;
}

const csrftoken = getCookie('csrftoken');

fetch('/api/endpoint/', {
    method: 'POST',
    headers: {
        'X-CSRFToken': csrftoken,
        'Content-Type': 'application/json'
    },
    body: JSON.stringify(data)
})
```

### SQL注入防护

```python
# 使用ORM(自动防护)
posts = Post.objects.filter(title=user_input)  # 安全

# 原始SQL(需要参数化)
from django.db import connection

# 不安全
cursor.execute(f"SELECT * FROM posts WHERE title = '{user_input}'")  # 危险!

# 安全
cursor.execute("SELECT * FROM posts WHERE title = %s", [user_input])  # 安全
```

### XSS防护

```django
{# 模板自动转义 #}
{{ user_input }}  {# 自动转义 #}

{# 禁用转义(谨慎使用) #}
{{ user_input|safe }}
{% autoescape off %}
    {{ user_input }}
{% endautoescape %}

{# Python代码 #}
from django.utils.html import escape
safe_text = escape(user_input)
```

---

## CTF应用场景

### 场景1: 简单Flag提交平台

```python
# models.py
from django.db import models
from django.contrib.auth.models import User

class Challenge(models.Model):
    name = models.CharField(max_length=100)
    description = models.TextField()
    flag = models.CharField(max_length=100)
    points = models.IntegerField()

class Submission(models.Model):
    user = models.ForeignKey(User, on_delete=models.CASCADE)
    challenge = models.ForeignKey(Challenge, on_delete=models.CASCADE)
    submitted_flag = models.CharField(max_length=100)
    is_correct = models.BooleanField(default=False)
    submitted_at = models.DateTimeField(auto_now_add=True)

# views.py
from django.shortcuts import render, redirect
from django.contrib.auth.decorators import login_required
from .models import Challenge, Submission

@login_required
def submit_flag(request, challenge_id):
    challenge = Challenge.objects.get(pk=challenge_id)

    if request.method == 'POST':
        submitted_flag = request.POST.get('flag')

        submission = Submission.objects.create(
            user=request.user,
            challenge=challenge,
            submitted_flag=submitted_flag,
            is_correct=(submitted_flag == challenge.flag)
        )

        if submission.is_correct:
            return render(request, 'success.html', {'points': challenge.points})
        else:
            return render(request, 'error.html', {'message': 'Flag错误'})

    return render(request, 'submit.html', {'challenge': challenge})
```

### 场景2: SSTI漏洞环境

```python
# 不安全的模板渲染(用于CTF题目)
from django.shortcuts import render
from django.template import Template, Context

def vulnerable_view(request):
    user_input = request.GET.get('name', 'Guest')

    # 危险: 直接使用用户输入构造模板
    template_string = f"Hello, {user_input}!"
    template = Template(template_string)
    result = template.render(Context())

    return render(request, 'result.html', {'result': result})

# 利用: ?name={{7*7}}  # 输出: Hello, 49!
# 利用: ?name={{''.join.__globals__.__builtins__.open('/etc/passwd').read()}}
```

### 场景3: 文件上传漏洞

```python
# models.py
class UploadedFile(models.Model):
    file = models.FileField(upload_to='uploads/')
    uploaded_at = models.DateTimeField(auto_now_add=True)

# views.py (不安全版本 - CTF题目)
def upload_file(request):
    if request.method == 'POST' and request.FILES.get('file'):
        uploaded_file = request.FILES['file']

        # 危险: 未验证文件类型
        file_obj = UploadedFile.objects.create(file=uploaded_file)

        return render(request, 'success.html', {'file_url': file_obj.file.url})

    return render(request, 'upload.html')

# 安全版本
import os

ALLOWED_EXTENSIONS = {'.jpg', '.png', '.gif'}

def upload_file_secure(request):
    if request.method == 'POST' and request.FILES.get('file'):
        uploaded_file = request.FILES['file']
        file_ext = os.path.splitext(uploaded_file.name)[1].lower()

        # 验证文件扩展名
        if file_ext not in ALLOWED_EXTENSIONS:
            return render(request, 'error.html', {'message': '不允许的文件类型'})

        # 验证文件内容(魔术字节)
        file_content = uploaded_file.read(8)
        uploaded_file.seek(0)

        # 限制文件大小
        if uploaded_file.size > 5 * 1024 * 1024:  # 5MB
            return render(request, 'error.html', {'message': '文件过大'})

        file_obj = UploadedFile.objects.create(file=uploaded_file)
        return render(request, 'success.html', {'file_url': file_obj.file.url})

    return render(request, 'upload.html')
```

---

## 常见漏洞与防御

### SQL注入

```python
# 漏洞代码
def search_posts(request):
    keyword = request.GET.get('q', '')
    # 危险: 字符串拼接
    query = f"SELECT * FROM posts WHERE title LIKE '%{keyword}%'"
    posts = Post.objects.raw(query)
    return render(request, 'results.html', {'posts': posts})

# 修复方法
def search_posts_secure(request):
    keyword = request.GET.get('q', '')
    # 安全: 使用ORM或参数化查询
    posts = Post.objects.filter(title__icontains=keyword)
    return render(request, 'results.html', {'posts': posts})
```

### XSS(跨站脚本)

```python
# 漏洞代码
from django.utils.safestring import mark_safe

def display_comment(request):
    comment = request.GET.get('comment', '')
    # 危险: 标记为安全
    safe_comment = mark_safe(comment)
    return render(request, 'comment.html', {'comment': safe_comment})

# 修复方法
from django.utils.html import escape

def display_comment_secure(request):
    comment = request.GET.get('comment', '')
    # 安全: 自动转义或手动转义
    return render(request, 'comment.html', {'comment': comment})
```

### CSRF绕过

```python
# 漏洞代码
from django.views.decorators.csrf import csrf_exempt

@csrf_exempt  # 危险: 禁用CSRF保护
def api_endpoint(request):
    # 处理请求
    pass

# 修复方法: 移除csrf_exempt,使用CSRF令牌
def api_endpoint_secure(request):
    if request.method == 'POST':
        # CSRF会自动验证
        pass
    return JsonResponse({'status': 'success'})
```

### 任意文件读取

```python
# 漏洞代码
import os

def download_file(request):
    filename = request.GET.get('file')
    # 危险: 未验证路径
    file_path = os.path.join('/var/www/files/', filename)
    with open(file_path, 'rb') as f:
        response = HttpResponse(f.read())
        response['Content-Disposition'] = f'attachment; filename="{filename}"'
        return response

# 修复方法
import os
from django.http import Http404

ALLOWED_DIR = '/var/www/files/'

def download_file_secure(request):
    filename = request.GET.get('file', '')

    # 验证文件名
    if '..' in filename or filename.startswith('/'):
        raise Http404("非法文件名")

    file_path = os.path.join(ALLOWED_DIR, filename)

    # 验证路径在允许目录内
    if not os.path.abspath(file_path).startswith(os.path.abspath(ALLOWED_DIR)):
        raise Http404("非法路径")

    # 验证文件存在
    if not os.path.exists(file_path):
        raise Http404("文件不存在")

    with open(file_path, 'rb') as f:
        response = HttpResponse(f.read())
        response['Content-Disposition'] = f'attachment; filename="{os.path.basename(filename)}"'
        return response
```

---

## 实战案例

### 案例1: CTF记分板

```python
# models.py
from django.db import models
from django.contrib.auth.models import User

class Team(models.Model):
    name = models.CharField(max_length=100, unique=True)
    members = models.ManyToManyField(User)
    score = models.IntegerField(default=0)

    def __str__(self):
        return self.name

class Challenge(models.Model):
    CATEGORY_CHOICES = [
        ('web', 'Web'),
        ('pwn', 'Pwn'),
        ('crypto', 'Crypto'),
        ('misc', 'Misc'),
    ]

    name = models.CharField(max_length=100)
    category = models.CharField(max_length=20, choices=CATEGORY_CHOICES)
    description = models.TextField()
    flag = models.CharField(max_length=100)
    points = models.IntegerField()

class Solve(models.Model):
    team = models.ForeignKey(Team, on_delete=models.CASCADE)
    challenge = models.ForeignKey(Challenge, on_delete=models.CASCADE)
    solved_at = models.DateTimeField(auto_now_add=True)

    class Meta:
        unique_together = ('team', 'challenge')

# views.py
from django.shortcuts import render
from django.db.models import Sum

def scoreboard(request):
    teams = Team.objects.annotate(
        total_score=Sum('solve__challenge__points')
    ).order_by('-total_score')

    return render(request, 'scoreboard.html', {'teams': teams})

def submit_flag(request, challenge_id):
    if request.method == 'POST':
        flag = request.POST.get('flag')
        challenge = Challenge.objects.get(pk=challenge_id)
        team = request.user.team_set.first()

        if flag == challenge.flag:
            Solve.objects.get_or_create(team=team, challenge=challenge)
            return JsonResponse({'status': 'correct'})
        else:
            return JsonResponse({'status': 'incorrect'})

    return render(request, 'submit.html')
```

### 案例2: 在线代码执行沙箱

```python
# views.py
import subprocess
import tempfile
import os

def code_executor(request):
    if request.method == 'POST':
        code = request.POST.get('code', '')
        language = request.POST.get('language', 'python')

        # 创建临时文件
        with tempfile.NamedTemporaryFile(mode='w', suffix='.py', delete=False) as f:
            f.write(code)
            temp_file = f.name

        try:
            # 执行代码(需要配置安全沙箱)
            result = subprocess.run(
                ['python3', temp_file],
                capture_output=True,
                text=True,
                timeout=5,
                cwd='/tmp'
            )

            output = result.stdout
            error = result.stderr

            return JsonResponse({
                'output': output,
                'error': error,
                'returncode': result.returncode
            })

        except subprocess.TimeoutExpired:
            return JsonResponse({'error': '执行超时'})

        finally:
            os.unlink(temp_file)

    return render(request, 'executor.html')
```

---

## 常见问题解决

### 问题1: 静态文件404

```python
# settings.py
STATIC_URL = '/static/'
STATICFILES_DIRS = [BASE_DIR / 'static']
STATIC_ROOT = BASE_DIR / 'staticfiles'

# 开发环境(urls.py)
from django.conf import settings
from django.conf.urls.static import static

urlpatterns = [
    # ...
] + static(settings.STATIC_URL, document_root=settings.STATIC_ROOT)
```

### 问题2: 数据库迁移错误

```bash
# 重置迁移
python manage.py migrate --fake app_name zero
python manage.py migrate app_name

# 或删除迁移文件重新生成
rm myapp/migrations/0*.py
python manage.py makemigrations
python manage.py migrate
```

### 问题3: CSRF验证失败

```python
# 确认中间件启用
# settings.py
MIDDLEWARE = [
    'django.middleware.csrf.CsrfViewMiddleware',
    # ...
]

# 模板包含token
{% csrf_token %}

# AJAX请求包含token
headers: {
    'X-CSRFToken': csrftoken
}
```

---

## 参考资源

### 官方资源
- **Django官网**: https://www.djangoproject.com/
- **文档**: https://docs.djangoproject.com/
- **教程**: https://docs.djangoproject.com/en/stable/intro/tutorial01/

### 安全资源
- **Django安全**: https://docs.djangoproject.com/en/stable/topics/security/
- **OWASP**: https://owasp.org/
- **CTF Wiki**: https://ctf-wiki.org/web/

### 学习资源
- **Django Girls教程**: https://tutorial.djangogirls.org/
- **Real Python**: https://realpython.com/tutorials/django/
- **CTFtime**: https://ctftime.org/

---

**文档版本**: v1.0
**更新日期**: 2025-01
**适用版本**: Django v5.0+
