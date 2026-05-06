---
name: django-security
description: Django security best practices, authentication, authorization, CSRF protection, SQL injection prevention, XSS prevention, and secure deployment configurations.
---

# Django 安全最佳实践

保护 Django 应用免受常见漏洞攻击的综合安全指南。

## 何时启用

- 配置 Django authentication 和 authorization 时
- 实现用户权限和角色时
- 配置生产环境安全设置时
- 审查 Django 应用程序的安全问题时
- 将 Django 应用程序部署到生产环境时

## 核心安全配置

### 生产环境配置

```python
# settings/production.py
import os

DEBUG = False  # 重要: 生产环境中绝对不要设为 True

ALLOWED_HOSTS = os.environ.get('ALLOWED_HOSTS', '').split(',')

# 安全头
SECURE_SSL_REDIRECT = True
SESSION_COOKIE_SECURE = True
CSRF_COOKIE_SECURE = True
SECURE_HSTS_SECONDS = 31536000  # 1年
SECURE_HSTS_INCLUDE_SUBDOMAINS = True
SECURE_HSTS_PRELOAD = True
SECURE_CONTENT_TYPE_NOSNIFF = True
SECURE_BROWSER_XSS_FILTER = True
X_FRAME_OPTIONS = 'DENY'

# HTTPS 和 Cookie
SESSION_COOKIE_HTTPONLY = True
CSRF_COOKIE_HTTPONLY = True
SESSION_COOKIE_SAMESITE = 'Lax'
CSRF_COOKIE_SAMESITE = 'Lax'

# Secret Key（必须通过环境变量设置）
SECRET_KEY = os.environ.get('DJANGO_SECRET_KEY')
if not SECRET_KEY:
    raise ImproperlyConfigured('DJANGO_SECRET_KEY environment variable is required')

# 密码验证
AUTH_PASSWORD_VALIDATORS = [
    {
        'NAME': 'django.contrib.auth.password_validation.UserAttributeSimilarityValidator',
    },
    {
        'NAME': 'django.contrib.auth.password_validation.MinimumLengthValidator',
        'OPTIONS': {
            'min_length': 12,
        }
    },
    {
        'NAME': 'django.contrib.auth.password_validation.CommonPasswordValidator',
    },
    {
        'NAME': 'django.contrib.auth.password_validation.NumericPasswordValidator',
    },
]
```

## Authentication

### 自定义用户模型

```python
# apps/users/models.py
from django.contrib.auth.models import AbstractUser
from django.db import models

class User(AbstractUser):
    """为更好的安全性设计的自定义用户模型。"""

    email = models.EmailField(unique=True)
    phone = models.CharField(max_length=20, blank=True)

    USERNAME_FIELD = 'email'  # 使用邮箱作为用户名
    REQUIRED_FIELDS = ['username']

    class Meta:
        db_table = 'users'
        verbose_name = 'User'
        verbose_name_plural = 'Users'

    def __str__(self):
        return self.email

# settings/base.py
AUTH_USER_MODEL = 'users.User'
```

### 密码哈希

```python
# Django 默认使用 PBKDF2。为更强的安全性：
PASSWORD_HASHERS = [
    'django.contrib.auth.hashers.Argon2PasswordHasher',
    'django.contrib.auth.hashers.PBKDF2PasswordHasher',
    'django.contrib.auth.hashers.PBKDF2SHA1PasswordHasher',
    'django.contrib.auth.hashers.BCryptSHA256PasswordHasher',
]
```

### Session 管理

```python
# Session 配置
SESSION_ENGINE = 'django.contrib.sessions.backends.cache'  # 或 'db'
SESSION_CACHE_ALIAS = 'default'
SESSION_COOKIE_AGE = 3600 * 24 * 7  # 1周
SESSION_SAVE_EVERY_REQUEST = False
SESSION_EXPIRE_AT_BROWSER_CLOSE = False  # 用户体验更好，但安全性较低
```

## Authorization

### 权限

```python
# models.py
from django.db import models
from django.contrib.auth.models import Permission

class Post(models.Model):
    title = models.CharField(max_length=200)
    content = models.TextField()
    author = models.ForeignKey(User, on_delete=models.CASCADE)

    class Meta:
        permissions = [
            ('can_publish', 'Can publish posts'),
            ('can_edit_others', 'Can edit posts of others'),
        ]

    def user_can_edit(self, user):
        """检查用户是否可以编辑此帖子。"""
        return self.author == user or user.has_perm('app.can_edit_others')

# views.py
from django.contrib.auth.mixins import LoginRequiredMixin, PermissionRequiredMixin
from django.views.generic import UpdateView

class PostUpdateView(LoginRequiredMixin, PermissionRequiredMixin, UpdateView):
    model = Post
    permission_required = 'app.can_edit_others'
    raise_exception = True  # 返回 403 而不是重定向

    def get_queryset(self):
        """确保用户只能编辑自己的帖子。"""
        return Post.objects.filter(author=self.request.user)
```

### 自定义权限

```python
# permissions.py
from rest_framework import permissions

class IsOwnerOrReadOnly(permissions.BasePermission):
    """只允许对象所有者进行编辑。"""

    def has_object_permission(self, request, view, obj):
        # 读取权限允许任何请求
        if request.method in permissions.SAFE_METHODS:
            return True

        # 写入权限只限于所有者
        return obj.author == request.user

class IsAdminOrReadOnly(permissions.BasePermission):
    """管理员可以做任何事，其他人只能阅读。"""

    def has_permission(self, request, view):
        if request.method in permissions.SAFE_METHODS:
            return True
        return request.user and request.user.is_staff

class IsVerifiedUser(permissions.BasePermission):
    """只允许已验证的用户。"""

    def has_permission(self, request, view):
        return request.user and request.user.is_authenticated and request.user.is_verified
```

### 基于角色的访问控制 (RBAC)

```python
# models.py
from django.contrib.auth.models import AbstractUser, Group

class User(AbstractUser):
    ROLE_CHOICES = [
        ('admin', 'Administrator'),
        ('moderator', 'Moderator'),
        ('user', 'Regular User'),
    ]
    role = models.CharField(max_length=20, choices=ROLE_CHOICES, default='user')

    def is_admin(self):
        return self.role == 'admin' or self.is_superuser

    def is_moderator(self):
        return self.role in ['admin', 'moderator']

# Mixin
class AdminRequiredMixin:
    """要求管理员角色的 Mixin。"""

    def dispatch(self, request, *args, **kwargs):
        if not request.user.is_authenticated or not request.user.is_admin():
            from django.core.exceptions import PermissionDenied
            raise PermissionDenied
        return super().dispatch(request, *args, **kwargs)
```

## SQL Injection 预防

### Django ORM 保护

```python
# GOOD: Django ORM 自动转义参数
def get_user(username):
    return User.objects.get(username=username)  # 安全

# GOOD: raw() 使用参数
def search_users(query):
    return User.objects.raw('SELECT * FROM users WHERE username = %s', [query])

# BAD: 不要直接插值用户输入
def get_user_bad(username):
    return User.objects.raw(f'SELECT * FROM users WHERE username = {username}')  # 有漏洞！

# GOOD: 使用正确转义的 filter
def get_users_by_email(email):
    return User.objects.filter(email__iexact=email)  # 安全

# GOOD: 复杂查询使用 Q 对象
from django.db.models import Q
def search_users_complex(query):
    return User.objects.filter(
        Q(username__icontains=query) |
        Q(email__icontains=query)
    )  # 安全
```

### raw() 的额外安全措施

```python
# 如果必须使用原始 SQL，始终使用参数
User.objects.raw(
    'SELECT * FROM users WHERE email = %s AND status = %s',
    [user_input_email, status]
)
```

## XSS 预防

### 模板转义

```django
{# Django 默认自动转义变量 - 安全 #}
{{ user_input }}  {# HTML 已转义 #}

{# 只将可信内容显式标记为安全 #}
{{ trusted_html|safe }}  {# 不转义 #}

{# 使用模板过滤器确保 HTML 安全 #}
{{ user_input|escape }}  {# 与默认相同 #}
{{ user_input|striptags }}  {# 移除所有 HTML 标签 #}

{# JavaScript 转义 #}
<script>
    var username = {{ username|escapejs }};
</script>
```

### 安全字符串处理

```python
from django.utils.safestring import mark_safe
from django.utils.html import escape

# BAD: 不要不转义就将用户输入标记为安全
def render_bad(user_input):
    return mark_safe(user_input)  # 有漏洞！

# GOOD: 先转义，再标记为安全
def render_good(user_input):
    return mark_safe(escape(user_input))

# GOOD: 对带变量的 HTML 使用 format_html
from django.utils.html import format_html

def greet_user(username):
    return format_html('<span class="user">{}</span>', escape(username))
```

### HTTP 头

```python
# settings.py
SECURE_CONTENT_TYPE_NOSNIFF = True  # 防止 MIME 嗅探
SECURE_BROWSER_XSS_FILTER = True  # 启用 XSS 过滤器
X_FRAME_OPTIONS = 'DENY'  # 防止点击劫持

# 自定义中间件
from django.conf import settings

class SecurityHeaderMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response

    def __call__(self, request):
        response = self.get_response(request)
        response['X-Content-Type-Options'] = 'nosniff'
        response['X-Frame-Options'] = 'DENY'
        response['X-XSS-Protection'] = '1; mode=block'
        response['Content-Security-Policy'] = "default-src 'self'"
        return response
```

## CSRF 保护

### 默认 CSRF 保护

```python
# settings.py - CSRF 默认启用
CSRF_COOKIE_SECURE = True  # 只通过 HTTPS 发送
CSRF_COOKIE_HTTPONLY = True  # 防止 JavaScript 访问
CSRF_COOKIE_SAMESITE = 'Lax'  # 在某些情况下防止 CSRF
CSRF_TRUSTED_ORIGINS = ['https://example.com']  # 可信域名

# 模板使用
<form method="post">
    {% csrf_token %}
    {{ form.as_p }}
    <button type="submit">Submit</button>
</form>

# AJAX 请求
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

fetch('/api/endpoint/', {
    method: 'POST',
    headers: {
        'X-CSRFToken': getCookie('csrftoken'),
        'Content-Type': 'application/json',
    },
    body: JSON.stringify(data)
});
```

### 视图排除（谨慎使用）

```python
from django.views.decorators.csrf import csrf_exempt

@csrf_exempt  # 只在绝对必要时使用！
def webhook_view(request):
    # 来自外部服务的 Webhook
    pass
```

## 文件上传安全

### 文件验证

```python
import os
from django.core.exceptions import ValidationError

def validate_file_extension(value):
    """验证文件扩展名。"""
    ext = os.path.splitext(value.name)[1]
    valid_extensions = ['.jpg', '.jpeg', '.png', '.gif', '.pdf']
    if not ext.lower() in valid_extensions:
        raise ValidationError('Unsupported file extension.')

def validate_file_size(value):
    """验证文件大小（最大 5MB）。"""
    filesize = value.size
    if filesize > 5 * 1024 * 1024:
        raise ValidationError('File too large. Max size is 5MB.')

# models.py
class Document(models.Model):
    file = models.FileField(
        upload_to='documents/',
        validators=[validate_file_extension, validate_file_size]
    )
```

### 安全文件存储

```python
# settings.py
MEDIA_ROOT = '/var/www/media/'
MEDIA_URL = '/media/'

# 生产环境中对媒体文件使用单独的域名
MEDIA_DOMAIN = 'https://media.example.com'

# 不要直接提供用户上传的文件
# 静态文件使用 whitenoise 或 CDN
# 媒体文件使用单独的服务器或 S3
```

## API 安全

### 速率限制

```python
# settings.py
REST_FRAMEWORK = {
    'DEFAULT_THROTTLE_CLASSES': [
        'rest_framework.throttling.AnonRateThrottle',
        'rest_framework.throttling.UserRateThrottle'
    ],
    'DEFAULT_THROTTLE_RATES': {
        'anon': '100/day',
        'user': '1000/day',
        'upload': '10/hour',
    }
}

# 自定义节流
from rest_framework.throttling import UserRateThrottle

class BurstRateThrottle(UserRateThrottle):
    scope = 'burst'
    rate = '60/min'

class SustainedRateThrottle(UserRateThrottle):
    scope = 'sustained'
    rate = '1000/day'
```

### API 认证

```python
# settings.py
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework.authentication.TokenAuthentication',
        'rest_framework.authentication.SessionAuthentication',
        'rest_framework_simplejwt.authentication.JWTAuthentication',
    ],
    'DEFAULT_PERMISSION_CLASSES': [
        'rest_framework.permissions.IsAuthenticated',
    ],
}

# views.py
from rest_framework.decorators import api_view, permission_classes
from rest_framework.permissions import IsAuthenticated

@api_view(['GET', 'POST'])
@permission_classes([IsAuthenticated])
def protected_view(request):
    return Response({'message': 'You are authenticated'})
```

## 安全头

### Content Security Policy

```python
# settings.py
CSP_DEFAULT_SRC = "'self'"
CSP_SCRIPT_SRC = "'self' https://cdn.example.com"
CSP_STYLE_SRC = "'self' 'unsafe-inline'"
CSP_IMG_SRC = "'self' data: https:"
CSP_CONNECT_SRC = "'self' https://api.example.com"

# Middleware
class CSPMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response

    def __call__(self, request):
        response = self.get_response(request)
        response['Content-Security-Policy'] = (
            f"default-src {CSP_DEFAULT_SRC}; "
            f"script-src {CSP_SCRIPT_SRC}; "
            f"style-src {CSP_STYLE_SRC}; "
            f"img-src {CSP_IMG_SRC}; "
            f"connect-src {CSP_CONNECT_SRC}"
        )
        return response
```

## 环境变量

### 密钥管理

```python
# 使用 python-decouple 或 django-environ
import environ

env = environ.Env(
    # 设置类型转换、默认值
    DEBUG=(bool, False)
)

# 读取 .env 文件
environ.Env.read_env()

SECRET_KEY = env('DJANGO_SECRET_KEY')
DATABASE_URL = env('DATABASE_URL')
ALLOWED_HOSTS = env.list('ALLOWED_HOSTS')

# .env 文件（不要提交到版本控制）
DEBUG=False
SECRET_KEY=your-secret-key-here
DATABASE_URL=postgresql://user:password@localhost:5432/dbname
ALLOWED_HOSTS=example.com,www.example.com
```

## 安全事件日志

```python
# settings.py
LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    'handlers': {
        'file': {
            'level': 'WARNING',
            'class': 'logging.FileHandler',
            'filename': '/var/log/django/security.log',
        },
        'console': {
            'level': 'INFO',
            'class': 'logging.StreamHandler',
        },
    },
    'loggers': {
        'django.security': {
            'handlers': ['file', 'console'],
            'level': 'WARNING',
            'propagate': True,
        },
        'django.request': {
            'handlers': ['file'],
            'level': 'ERROR',
            'propagate': False,
        },
    },
}
```

## 快速安全检查清单

| 检查 | 说明 |
|-------|-------------|
| `DEBUG = False` | 生产环境中永远不要启用 DEBUG |
| HTTPS only | 强制 SSL，安全 Cookie |
| 强密钥 | SECRET_KEY 使用环境变量 |
| 密码验证 | 启用所有密码验证器 |
| CSRF 保护 | 默认启用，不要禁用 |
| XSS 预防 | Django 自动转义，不要对用户输入使用 <code>\|safe</code> |
| SQL Injection | 使用 ORM，不要在查询中连接字符串 |
| 文件上传 | 验证文件类型和大小 |
| 速率限制 | 对 API 端点进行节流 |
| 安全头 | CSP、X-Frame-Options、HSTS |
| 日志记录 | 记录安全事件 |
| 更新 | 保持 Django 和依赖项最新 |

**记住**: 安全是一个过程，而不是产品。定期审查并更新您的安全实践。
