# Django 完全指南

> [!NOTE]
> 本文档最后更新于 **2026年4月**，涵盖 Django 5 新特性、MVT 架构、ORM 进阶、REST API 构建及与 Flask/FastAPI 的深度对比。

---

## 目录

1. [[#Django 概述与市场定位]]
2. [[#Django 5 新特性]]
3. [[#MVT 架构详解]]
4. [[#Django ORM 进阶]]
5. [[#Admin 自动后台]]
6. [[#表单处理]]
7. [[#Django REST Framework]]
8. [[#认证系统]]
9. [[#Django Channels（异步）]]
10. [[#Django vs Flask vs FastAPI 对比]]
11. [[#实战场景与选型建议]]

---

## Django 概述与市场定位

### Django 的核心理念

Django 是 Python 生态中最成熟的「**全功能**」Web 框架，其设计哲学是「**包含电池**」（Batteries Included）：

- **全功能**：内置 ORM、Admin、表单、认证、缓存等
- **安全优先**：防止 SQL 注入、XSS、CSRF 等
- **可扩展**：松耦合设计，组件可替换
- **文档完善**：被誉为「文档最好的框架」
- **成熟稳定**：15+ 年生产验证

### Django 适用场景

| 场景 | 适用度 |
|------|--------|
| **企业级 Web 应用** | ⭐⭐⭐⭐⭐ |
| **内容管理系统（CMS）** | ⭐⭐⭐⭐⭐ |
| **社交网络** | ⭐⭐⭐⭐ |
| **电商平台** | ⭐⭐⭐⭐ |
| **数据驱动的仪表板** | ⭐⭐⭐⭐ |
| **REST API** | ⭐⭐⭐⭐（需 DRF） |
| **微服务** | ⭐⭐（偏重） |
| **Serverless** | ⭐⭐（需适配） |

---

## Django 5 新特性

### Django 5 主要更新

| 特性 | 说明 | Django 版本 |
|------|------|-------------|
| **生成式 AI 集成** | 内置 LLM 集成支持 | 5.0+ |
| **数据库约束** | GenAI 字段级约束 | 5.0+ |
| **简化模板配置** | 减少 boilerplate | 5.0+ |
| **异步 ORM** | 异步数据库操作 | 5.0+ |
| **分区索引** | 改进查询性能 | 5.0+ |
| **Shrink Fields** | 模型字段精简 | 5.0+ |

### GenAI 集成

```python
# Django 5 内置 GenAI 支持
from django.db import models
from django.ai import model_field

class Article(models.Model):
    title = models.CharField(max_length=200)
    # 使用 AI 字段自动生成摘要
    summary = model_field.SummaryField(source_field='content')
    # 使用 AI 生成标签
    tags = model_field.TagField(source_field='content')
    
    class Meta:
        indexes = [
            # 分区索引优化
            models.Index(fields=['created_at'], name='article_created_idx'),
        ]
```

### 异步 ORM（预览）

```python
# Django 5 异步 ORM
from django.db import models

class Product(models.Model):
    name = models.CharField(max_length=200)
    price = models.DecimalField(max_digits=10, decimal_places=2)
    
# 异步查询
async def get_products():
    async with database:
        products = await Product.objects.filter(price__gt=100).all()
    return products
```

---

## MVT 架构详解

### MVT vs MVC

Django 采用 MVT（Model-View-Template）架构，与传统 MVC 的对应关系：

| Django MVT | MVC 对应 | 说明 |
|------------|---------|------|
| **Model** | Model | 数据模型、ORM |
| **View** | Controller | 业务逻辑、请求处理 |
| **Template** | View | 模板渲染、HTML 生成 |
| **URLconf** | Router | 路由映射 |

### 项目结构

```
myproject/
├── manage.py                 # Django CLI
├── myproject/
│   ├── __init__.py
│   ├── settings.py           # 配置
│   ├── urls.py               # 根路由
│   ├── asgi.py               # ASGI 配置
│   └── wsgi.py               # WSGI 配置
├── apps/
│   ├── blog/
│   │   ├── __init__.py
│   │   ├── models.py         # 数据模型
│   │   ├── views.py          # 视图
│   │   ├── urls.py           # 路由
│   │   ├── admin.py          # Admin 配置
│   │   └── apps.py           # App 配置
│   └── accounts/
│       └── ...
├── templates/                 # 全局模板
├── static/                    # 静态文件
├── media/                     # 用户上传
└── requirements.txt
```

### settings.py 配置

```python
# myproject/settings.py
from pathlib import Path

BASE_DIR = Path(__file__).resolve().parent.parent

SECRET_KEY = 'your-secret-key-here'
DEBUG = True
ALLOWED_HOSTS = ['localhost', '127.0.0.1']

INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    # 第三方应用
    'rest_framework',
    'corsheaders',
    'django_filters',
    # 本地应用
    'apps.blog',
    'apps.accounts',
]

MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'corsheaders.middleware.CorsMiddleware',  # CORS
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'myproject',
        'USER': 'postgres',
        'PASSWORD': 'password',
        'HOST': 'localhost',
        'PORT': '5432',
    }
}

LANGUAGE_CODE = 'zh-hans'
TIME_ZONE = 'Asia/Shanghai'
USE_I18N = True
USE_TZ = True

STATIC_URL = '/static/'
STATIC_ROOT = BASE_DIR / 'staticfiles'

MEDIA_URL = '/media/'
MEDIA_ROOT = BASE_DIR / 'media'

DEFAULT_AUTO_FIELD = 'django.db.models.BigAutoField'

# REST Framework 配置
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework.authentication.SessionAuthentication',
        'rest_framework.authentication.TokenAuthentication',
    ],
    'DEFAULT_PERMISSION_CLASSES': [
        'rest_framework.permissions.IsAuthenticated',
    ],
    'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.PageNumberPagination',
    'PAGE_SIZE': 20,
    'DEFAULT_FILTER_BACKENDS': [
        'django_filters.rest_framework.DjangoFilterBackend',
        'rest_framework.filters.SearchFilter',
        'rest_framework.filters.OrderingFilter',
    ],
}

# CORS 配置
CORS_ALLOWED_ORIGINS = [
    'http://localhost:3000',
    'https://example.com',
]
```

---

## Django ORM 进阶

### 模型定义

```python
# apps/blog/models.py
from django.db import models
from django.contrib.auth import get_user_model
from django.core.validators import MinValueValidator, MaxValueValidator

User = get_user_model()


class Category(models.Model):
    name = models.CharField(max_length=100, unique=True)
    slug = models.SlugField(max_length=100, unique=True)
    description = models.TextField(blank=True)
    
    class Meta:
        verbose_name_plural = 'categories'
        ordering = ['name']
    
    def __str__(self):
        return self.name


class Tag(models.Model):
    name = models.CharField(max_length=50, unique=True)
    
    def __str__(self):
        return self.name


class Post(models.Model):
    STATUS_CHOICES = [
        ('draft', '草稿'),
        ('published', '已发布'),
        ('archived', '已归档'),
    ]
    
    title = models.CharField(max_length=200)
    slug = models.SlugField(max_length=200, unique=True)
    author = models.ForeignKey(
        User,
        on_delete=models.CASCADE,
        related_name='blog_posts'
    )
    category = models.ForeignKey(
        Category,
        on_delete=models.SET_NULL,
        null=True,
        related_name='posts'
    )
    tags = models.ManyToManyField(Tag, related_name='posts', blank=True)
    excerpt = models.TextField(max_length=500, blank=True)
    content = models.TextField()
    featured_image = models.ImageField(
        upload_to='blog/images/',
        blank=True, null=True
    )
    status = models.CharField(
        max_length=10,
        choices=STATUS_CHOICES,
        default='draft'
    )
    views = models.PositiveIntegerField(default=0)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    published_at = models.DateTimeField(null=True, blank=True)
    
    class Meta:
        ordering = ['-published_at', '-created_at']
        indexes = [
            models.Index(fields=['-published_at']),
            models.Index(fields=['slug']),
            models.Index(fields=['status', '-published_at']),
        ]
    
    def __str__(self):
        return self.title


class Comment(models.Model):
    post = models.ForeignKey(
        Post,
        on_delete=models.CASCADE,
        related_name='comments'
    )
    author = models.ForeignKey(
        User,
        on_delete=models.CASCADE,
        related_name='comments'
    )
    parent = models.ForeignKey(
        'self',
        on_delete=models.CASCADE,
        null=True,
        blank=True,
        related_name='replies'
    )
    content = models.TextField()
    is_approved = models.BooleanField(default=True)
    created_at = models.DateTimeField(auto_now_add=True)
    
    class Meta:
        ordering = ['created_at']
    
    def __str__(self):
        return f'{self.author} - {self.content[:50]}'
```

### Django ORM 常用操作表

| 操作 | 方法 | 示例 |
|------|------|------|
| **创建** | `Model.objects.create()` | `Post.objects.create(title='Hello')` |
| **查询单个** | `Model.objects.get()` | `Post.objects.get(id=1)` |
| **查询全部** | `Model.objects.all()` | `Post.objects.all()` |
| **过滤** | `Model.objects.filter()` | `Post.objects.filter(status='published')` |
| **排除** | `Model.objects.exclude()` | `Post.objects.exclude(status='draft')` |
| **排序** | `order_by()` | `Post.objects.order_by('-created_at')` |
| **分页** | `[start:end]` | `Post.objects.all()[0:10]` |
| **计数** | `count()` | `Post.objects.count()` |
| **存在** | `exists()` | `Post.objects.filter(...).exists()` |
| **去重** | `distinct()` | `Post.objects.distinct()` |
| **聚合** | `aggregate()` | `Post.objects.aggregate(Count('id'))` |
| **分组** | `values().annotate()` | `Post.objects.values('author').annotate(Count('id'))` |
| **更新** | `update()` | `Post.objects.filter(id=1).update(views=F('views')+1)` |
| **删除** | `delete()` | `Post.objects.filter(id=1).delete()` |
| **关系查询** | `select_related()` | `Post.objects.select_related('author')` |
| **预加载** | `prefetch_related()` | `Post.objects.prefetch_related('tags')` |

### 查询示例

```python
# 基础查询
latest_posts = Post.objects.filter(status='published').order_by('-published_at')[:5]

# 复杂查询
popular_posts = (
    Post.objects
    .filter(status='published', views__gte=100)
    .select_related('author', 'category')
    .prefetch_related('tags')
    .exclude(category__name='Archived')
    .order_by('-views', '-published_at')
)

# 聚合查询
from django.db.models import Count, Avg, Q

# 每个分类的文章数
category_counts = (
    Category.objects
    .annotate(post_count=Count('posts'))
    .order_by('-post_count')
)

# 平均评分
from django.db.models import Avg
Product.objects.aggregate(Avg('rating'))

# 搜索
search_results = Post.objects.filter(
    Q(title__icontains=query) | 
    Q(content__icontains=query)
)

# F 表达式更新
from django.db.models import F
Post.objects.filter(id=1).update(views=F('views') + 1)

# 条件表达式
from django.db.models import Case, When, Value
Post.objects.annotate(
    status_display=Case(
        When(status='published', then=Value('已发布')),
        When(status='draft', then=Value('草稿')),
        default=Value('未知'),
    )
)
```

---

## Admin 自动后台

### Admin 配置

```python
# apps/blog/admin.py
from django.contrib import admin
from django.utils.html import format_html
from .models import Category, Tag, Post, Comment


@admin.register(Category)
class CategoryAdmin(admin.ModelAdmin):
    list_display = ['name', 'slug', 'post_count', 'created_at']
    prepopulated_fields = {'slug': ('name',)}
    search_fields = ['name']
    ordering = ['name']
    
    def post_count(self, obj):
        return obj.posts.count()
    post_count.short_description = '文章数'


@admin.register(Tag)
class TagAdmin(admin.ModelAdmin):
    list_display = ['name']


@admin.register(Post)
class PostAdmin(admin.ModelAdmin):
    list_display = ['title', 'author', 'category', 'status', 
                    'views', 'published_at', 'created_at']
    list_filter = ['status', 'category', 'author', 'created_at']
    search_fields = ['title', 'content']
    prepopulated_fields = {'slug': ('title',)}
    date_hierarchy = 'created_at'
    ordering = ['-created_at']
    
    fieldsets = (
        ('基本信息', {
            'fields': ('title', 'slug', 'author', 'category')
        }),
        ('内容', {
            'fields': ('excerpt', 'content', 'featured_image')
        }),
        ('标签', {
            'fields': ('tags',)
        }),
        ('SEO', {
            'fields': ('meta_title', 'meta_description'),
            'classes': ('collapse',)
        }),
        ('发布', {
            'fields': ('status', 'published_at')
        }),
    )
    
    actions = ['publish_posts', 'unpublish_posts']
    
    @admin.action(description='发布选中的文章')
    def publish_posts(self, request, queryset):
        queryset.update(status='published')
    
    @admin.action(description='取消发布选中的文章')
    def unpublish_posts(self, request, queryset):
        queryset.update(status='draft')


@admin.register(Comment)
class CommentAdmin(admin.ModelAdmin):
    list_display = ['author', 'post', 'is_approved', 'created_at']
    list_filter = ['is_approved', 'created_at']
    search_fields = ['content', 'author__username']
    actions = ['approve_comments', 'disapprove_comments']
    
    def has_add_permission(self, request):
        return False  # 禁止手动添加评论
```

### Admin 定制化

```python
# 使用装饰器
@admin.register(Post)
class PostAdmin(admin.ModelAdmin):
    # ...
    list_display = ['title', 'status', 'created_at']
    
    def get_queryset(self, request):
        qs = super().get_queryset(request)
        if request.user.is_superuser:
            return qs
        return qs.filter(author=request.user)
```

---

## 表单处理

### Django 表单

```python
# apps/blog/forms.py
from django import forms
from django.core.exceptions import ValidationError
from .models import Post, Comment


class PostForm(forms.ModelForm):
    class Meta:
        model = Post
        fields = ['title', 'slug', 'category', 'tags', 
                  'excerpt', 'content', 'featured_image']
        widgets = {
            'title': forms.TextInput(attrs={
                'class': 'form-control',
                'placeholder': '文章标题'
            }),
            'content': forms.Textarea(attrs={
                'class': 'form-control',
                'rows': 15,
            }),
            'tags': forms.CheckboxSelectMultiple(),
        }
    
    def clean_slug(self):
        slug = self.cleaned_data['slug']
        if Post.objects.filter(slug=slug).exclude(
            pk=self.instance.pk
        ).exists():
            raise ValidationError('该 slug 已存在')
        return slug


class CommentForm(forms.ModelForm):
    class Meta:
        model = Comment
        fields = ['content']
        widgets = {
            'content': forms.Textarea(attrs={
                'class': 'form-control',
                'rows': 3,
                'placeholder': '写下你的评论...'
            }),
        }
    
    def clean_content(self):
        content = self.cleaned_data['content']
        if len(content) < 10:
            raise ValidationError('评论至少需要 10 个字符')
        return content
```

### 视图处理

```python
# apps/blog/views.py
from django.shortcuts import render, get_object_or_404, redirect
from django.views.generic import ListView, DetailView
from django.contrib.auth.decorators import login_required
from django.contrib import messages
from .models import Post, Category
from .forms import PostForm, CommentForm


def post_create(request):
    if not request.user.is_authenticated:
        return redirect('login')
    
    if request.method == 'POST':
        form = PostForm(request.POST, request.FILES)
        if form.is_valid():
            post = form.save(commit=False)
            post.author = request.user
            post.save()
            form.save_m2m()  # 保存 ManyToMany 关系
            messages.success(request, '文章创建成功！')
            return redirect('blog:post_detail', slug=post.slug)
    else:
        form = PostForm()
    
    return render(request, 'blog/post_form.html', {'form': form})


def post_detail(request, slug):
    post = get_object_or_404(Post, slug=slug, status='published')
    post.views += 1
    post.save(update_fields=['views'])
    
    comments = post.comments.filter(is_approved=True).order_by('-created_at')
    
    if request.method == 'POST':
        if not request.user.is_authenticated:
            return redirect('login')
        
        comment_form = CommentForm(data=request.POST)
        if comment_form.is_valid():
            comment = comment_form.save(commit=False)
            comment.post = post
            comment.author = request.user
            comment.save()
            messages.success(request, '评论已提交，等待审核')
            return redirect('blog:post_detail', slug=post.slug)
    else:
        comment_form = CommentForm()
    
    return render(request, 'blog/post_detail.html', {
        'post': post,
        'comments': comments,
        'comment_form': comment_form,
    })
```

---

## Django REST Framework

### DRF 安装与配置

```bash
pip install djangorestframework
pip install django-filter
pip install markdown
pip install django-cors-headers
```

```python
# settings.py
INSTALLED_APPS = [
    # ...
    'rest_framework',
    'rest_framework.authtoken',
    'corsheaders',
    'django_filters',
]

REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework.authentication.SessionAuthentication',
        'rest_framework.authentication.TokenAuthentication',
        'rest_framework.authentication.BasicAuthentication',
    ],
    'DEFAULT_PERMISSION_CLASSES': [
        'rest_framework.permissions.IsAuthenticatedOrReadOnly',
    ],
    'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.PageNumberPagination',
    'PAGE_SIZE': 10,
    'DEFAULT_FILTER_BACKENDS': [
        'django_filters.rest_framework.DjangoFilterBackend',
        'rest_framework.filters.SearchFilter',
        'rest_framework.filters.OrderingFilter',
    ],
    'DEFAULT_RENDERER_CLASSES': [
        'rest_framework.renderers.JSONRenderer',
        'rest_framework.renderers.BrowsableAPIRenderer',
    ],
}
```

### Serializers

```python
# apps/blog/serializers.py
from rest_framework import serializers
from django.contrib.auth import get_user_model
from .models import Category, Tag, Post, Comment

User = get_user_model()


class UserSerializer(serializers.ModelSerializer):
    class Meta:
        model = User
        fields = ['id', 'username', 'email', 'date_joined']
        read_only_fields = ['id', 'date_joined']


class TagSerializer(serializers.ModelSerializer):
    class Meta:
        model = Tag
        fields = ['id', 'name']


class CategorySerializer(serializers.ModelSerializer):
    post_count = serializers.IntegerField(read_only=True)
    
    class Meta:
        model = Category
        fields = ['id', 'name', 'slug', 'post_count']


class CommentSerializer(serializers.ModelSerializer):
    author = UserSerializer(read_only=True)
    replies = serializers.SerializerMethodField()
    
    class Meta:
        model = Comment
        fields = ['id', 'author', 'content', 'created_at', 'replies']
        read_only_fields = ['id', 'created_at']
    
    def get_replies(self, obj):
        if obj.replies.exists():
            return CommentSerializer(obj.replies.all(), many=True).data
        return []


class PostListSerializer(serializers.ModelSerializer):
    author = UserSerializer(read_only=True)
    category = CategorySerializer(read_only=True)
    comment_count = serializers.IntegerField(read_only=True)
    
    class Meta:
        model = Post
        fields = ['id', 'title', 'slug', 'author', 'category',
                  'excerpt', 'featured_image', 'views',
                  'comment_count', 'published_at', 'created_at']


class PostDetailSerializer(serializers.ModelSerializer):
    author = UserSerializer(read_only=True)
    category = CategorySerializer(read_only=True)
    tags = TagSerializer(many=True, read_only=True)
    comments = serializers.SerializerMethodField()
    
    class Meta:
        model = Post
        fields = ['id', 'title', 'slug', 'author', 'category', 'tags',
                  'content', 'excerpt', 'featured_image', 'views',
                  'comments', 'published_at', 'created_at', 'updated_at']
    
    def get_comments(self, obj):
        approved_comments = obj.comments.filter(is_approved=True, parent__isnull=True)
        return CommentSerializer(approved_comments, many=True).data
```

### Views

```python
# apps/blog/views.py
from rest_framework import viewsets, filters, permissions
from rest_framework.decorators import action
from rest_framework.response import Response
from django_filters.rest_framework import DjangoFilterBackend
from .models import Category, Tag, Post, Comment
from .serializers import (
    CategorySerializer, TagSerializer,
    PostListSerializer, PostDetailSerializer,
    CommentSerializer
)


class CategoryViewSet(viewsets.ModelViewSet):
    queryset = Category.objects.all()
    serializer_class = CategorySerializer
    lookup_field = 'slug'
    filter_fields = ['name']
    search_fields = ['name', 'description']


class TagViewSet(viewsets.ModelViewSet):
    queryset = Tag.objects.all()
    serializer_class = TagSerializer


class PostViewSet(viewsets.ModelViewSet):
    queryset = Post.objects.filter(status='published')
    lookup_field = 'slug'
    permission_classes = [permissions.IsAuthenticatedOrReadOnly]
    
    filter_backends = [DjangoFilterBackend, filters.SearchFilter, filters.OrderingFilter]
    filterset_fields = ['category', 'author', 'status']
    search_fields = ['title', 'content', 'excerpt']
    ordering_fields = ['published_at', 'views', 'created_at']
    ordering = ['-published_at']
    
    def get_serializer_class(self):
        if self.action == 'list':
            return PostListSerializer
        return PostDetailSerializer
    
    def perform_create(self, serializer):
        serializer.save(author=self.request.user)
    
    @action(detail=True, methods=['get'])
    def related(self, request, slug=None):
        post = self.get_object()
        related_posts = Post.objects.filter(
            category=post.category
        ).exclude(id=post.id)[:5]
        serializer = PostListSerializer(related_posts, many=True)
        return Response(serializer.data)


class CommentViewSet(viewsets.ModelViewSet):
    serializer_class = CommentSerializer
    permission_classes = [permissions.IsAuthenticatedOrReadOnly]
    
    def get_queryset(self):
        return Comment.objects.filter(is_approved=True)
    
    def perform_create(self, serializer):
        serializer.save(author=self.request.user)
```

### URLs

```python
# apps/blog/urls.py
from django.urls import path, include
from rest_framework.routers import DefaultRouter
from . import views

router = DefaultRouter()
router.register(r'categories', views.CategoryViewSet)
router.register(r'tags', views.TagViewSet)
router.register(r'posts', views.PostViewSet)
router.register(r'comments', views.CommentViewSet)

urlpatterns = [
    path('', include(router.urls)),
]
```

---

## 认证系统

### Django Auth 扩展

```python
# apps/accounts/models.py
from django.contrib.auth.models import AbstractUser
from django.db import models

class User(AbstractUser):
    avatar = models.ImageField(upload_to='users/avatars/', blank=True, null=True)
    bio = models.TextField(max_length=500, blank=True)
    website = models.URLField(blank=True)
    email_verified = models.BooleanField(default=False)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    
    class Meta:
        verbose_name = '用户'
        verbose_name_plural = '用户'

# settings.py
# AUTH_USER_MODEL = 'accounts.User'
```

### JWT 认证

```python
# 安装 djangorestframework-simplejwt
pip install djangorestframework-simplejwt

# settings.py
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework_simplejwt.authentication.JWTAuthentication',
    ],
}

from datetime import timedelta
SIMPLE_JWT = {
    'ACCESS_TOKEN_LIFETIME': timedelta(hours=1),
    'REFRESH_TOKEN_LIFETIME': timedelta(days=7),
    'ROTATE_REFRESH_TOKENS': True,
    'BLACKLIST_AFTER_ROTATION': True,
    'AUTH_HEADER_TYPES': ('Bearer',),
}
```

```python
# urls.py
from rest_framework_simplejwt.views import (
    TokenObtainPairView,
    TokenRefreshView,
    TokenVerifyView,
)

urlpatterns = [
    path('api/token/', TokenObtainPairView.as_view(), name='token_obtain_pair'),
    path('api/token/refresh/', TokenRefreshView.as_view(), name='token_refresh'),
    path('api/token/verify/', TokenVerifyView.as_view(), name='token_verify'),
]
```

---

## Django Channels（异步）

### 安装与配置

```bash
pip install channels channels_redis
```

```python
# settings.py
INSTALLED_APPS = [
    # ...
    'channels',
]

ASGI_APPLICATION = 'myproject.asgi.application'

CHANNEL_LAYERS = {
    'default': {
        'BACKEND': 'channels_redis.core.RedisChannelLayer',
        'CONFIG': {
            'hosts': [('127.0.0.1', 6379)],
        },
    },
}
```

### WebSocket 消费者

```python
# apps/chat/consumers.py
import json
from channels.generic.websocket import AsyncWebsocketConsumer
from channels.db import database_sync_to_async
from django.contrib.auth import get_user_model

User = get_user_model()


class ChatConsumer(AsyncWebsocketConsumer):
    async def connect(self):
        self.room_name = self.scope['url_route']['kwargs']['room_name']
        self.room_group_name = f'chat_{self.room_name}'
        
        # 加入房间组
        await self.channel_layer.group_add(
            self.room_group_name,
            self.channel_name
        )
        
        await self.accept()
    
    async def disconnect(self, close_code):
        # 离开房间组
        await self.channel_layer.group_discard(
            self.room_group_name,
            self.channel_name
        )
    
    async def receive(self, text_data):
        data = json.loads(text_data)
        message_type = data.get('type', 'message')
        
        if message_type == 'chat_message':
            message = data['message']
            user = self.scope.get('user')
            
            # 广播消息
            await self.channel_layer.group_send(
                self.room_group_name,
                {
                    'type': 'chat_message',
                    'message': message,
                    'username': user.username if user.is_authenticated else 'Anonymous',
                    'timestamp': data.get('timestamp'),
                }
            )
    
    async def chat_message(self, event):
        # 发送消息到 WebSocket
        await self.send(text_data=json.dumps({
            'type': 'message',
            'message': event['message'],
            'username': event['username'],
            'timestamp': event.get('timestamp'),
        }))
```

### Routing

```python
# apps/chat/routing.py
from django.urls import re_path
from . import consumers

websocket_urlpatterns = [
    re_path(r'ws/chat/(?P<room_name>\w+)/$', consumers.ChatConsumer.as_asgi()),
]

# asgi.py
import os
from django.core.asgi import get_asgi_application
from channels.routing import ProtocolTypeRouter, URLRouter
from channels.auth import AuthMiddlewareStack

os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'myproject.settings')

application = ProtocolTypeRouter({
    'http': get_asgi_application(),
    'websocket': AuthMiddlewareStack(
        URLRouter(websocket_urlpatterns)
    ),
})
```

---

## Django vs Flask vs FastAPI 对比

### 核心对比表

| 维度 | Django | Flask | FastAPI |
|------|--------|-------|---------|
| **设计哲学** | 全功能、自包含 | 微框架、灵活 | 现代、性能 |
| **ORM** | 内置 Django ORM | SQLAlchemy/其他 | SQLModel/Peewee |
| **Admin** | 内置 | 无 | 无 |
| **表单** | 内置 Forms | Flask-WTF | Pydantic |
| **认证** | 内置 Auth | Flask-Login/其他 | 需手动 |
| **API** | DRF | Flask-RESTful | 原生支持 |
| **异步** | 有限 | 有限 | 原生异步 |
| **类型安全** | 需配置 | 需配置 | 原生 Pydantic |
| **学习曲线** | 陡峭 | 平缓 | 平缓 |
| **项目规模** | 大型 | 小型-中型 | 小型-中型 |
| **性能** | 一般 | 一般 | 极佳 |
| **生态** | 庞大 | 丰富 | 增长中 |

### Django vs Rails（思想层面）

| 方面 | Django | Rails |
|------|--------|-------|
| **口号** | "The Web framework for perfectionists with deadlines" | "Web development that doesn't hurt" |
| **ORM** | Django ORM（更 Pythonic） | ActiveRecord（更 Ruby） |
| **哲学** | 显式优于隐式 | 约定优于配置 |
| **灵活性** | 组件可替换 | 约定强约束 |
| **配置** | settings.py | 环境变量 + YAML |
| **迁移** | migrate 命令 | migrate 命令 |
| **Admin** | 内置强大 | ActiveAdmin 需安装 |

> [!TIP]
> Django 和 Rails 都适合快速开发企业级应用：
> - **Django**：Python 团队的首选
> - **Rails**：Ruby 团队的首选
> - 两者都能快速交付高质量产品

---

## 实战场景与选型建议

### AI 应用集成实战

```python
# apps/ai/services.py
import openai
from django.conf import settings
from typing import List, Dict

openai.api_key = settings.OPENAI_API_KEY


class AIService:
    @staticmethod
    def chat_completion(
        messages: List[Dict[str, str]],
        model: str = 'gpt-4o',
        temperature: float = 0.7,
        stream: bool = False,
    ):
        return openai.chat.completions.create(
            model=model,
            messages=messages,
            temperature=temperature,
            stream=stream,
        )
    
    @staticmethod
    def generate_summary(content: str, max_tokens: int = 200):
        response = openai.chat.completions.create(
            model='gpt-4o',
            messages=[
                {'role': 'system', 'content': '你是一个专业的摘要生成助手。'},
                {'role': 'user', 'content': f'请为以下内容生成简洁的摘要：\n\n{content}'}
            ],
            max_tokens=max_tokens,
        )
        return response.choices[0].message.content
    
    @staticmethod
    def moderate_content(content: str):
        response = openai.moderations.create(input=content)
        return response.results[0]


# apps/ai/views.py
from rest_framework.views import APIView
from rest_framework.response import Response
from rest_framework.permissions import IsAuthenticated
from rest_framework import status
from .services import AIService

class ChatView(APIView):
    permission_classes = [IsAuthenticated]
    
    def post(self, request):
        messages = request.data.get('messages', [])
        
        if not messages:
            return Response(
                {'error': 'Messages are required'},
                status=status.HTTP_400_BAD_REQUEST
            )
        
        try:
            completion = AIService.chat_completion(messages)
            return Response({
                'id': completion.id,
                'model': completion.model,
                'choices': [
                    {
                        'message': choice.message.content,
                        'finish_reason': choice.finish_reason,
                    }
                    for choice in completion.choices
                ],
                'usage': {
                    'prompt_tokens': completion.usage.prompt_tokens,
                    'completion_tokens': completion.usage.completion_tokens,
                    'total_tokens': completion.usage.total_tokens,
                }
            })
        except Exception as e:
            return Response(
                {'error': str(e)},
                status=status.HTTP_500_INTERNAL_SERVER_ERROR
            )


# urls.py
from django.urls import path
from .views import ChatView

urlpatterns = [
    path('chat/', ChatView.as_view(), name='ai-chat'),
]
```

### 成本估算

| 方案 | 月成本 | 说明 |
|------|--------|------|
| **Railway** | $5-100 | 按使用计费 |
| **Render** | $7-50 | 自动扩缩容 |
| **AWS EB** | $20-200 | 弹性 Beanstalk |
| **自托管** | $20-100 | VPS |

---

## 完整安装与环境配置

### 环境要求

```bash
# Python 版本要求
python --version  # >= 3.10 (Django 5.x)
python --version  # >= 3.8 (Django 4.x)

# pip 版本
pip --version  # >= 21.0
```

### 项目创建

```bash
# 使用 pip 安装 Django
pip install Django

# 创建项目
django-admin startproject myproject

# 创建应用
cd myproject
python manage.py startapp blog

# 创建 API 应用
python manage.py startapp api
```

### 推荐的完整依赖

```bash
# Django 核心
pip install Django>=5.0,<6.0

# REST API
pip install djangorestframework djangorestframework-simplejwt

# 数据库
pip install psycopg2-binary  # PostgreSQL
pip install mysqlclient       # MySQL
pip install djongo            # MongoDB

# Redis 和缓存
pip install django-redis django-cache-url

# 异步任务
pip install celery redis

# AI 集成
pip install openai anthropic

# 认证
pip install django-allauth social-auth-app-django

# 存储
pip install boto3 django-storages

# 管理界面
pip install django-import-export django-summernote

# 开发工具
pip install black isort mypy
pip install --save-dev pytest pytest-django
```

### 项目结构

```
myproject/
├── myproject/
│   ├── __init__.py
│   ├── settings.py          # 配置
│   ├── urls.py              # URL 配置
│   ├── wsgi.py              # WSGI 应用
│   ├── asgi.py              # ASGI 应用
│   └── celery.py            # Celery 配置
├── apps/
│   ├── __init__.py
│   ├── blog/
│   │   ├── __init__.py
│   │   ├── models.py
│   │   ├── views.py
│   │   ├── urls.py
│   │   ├── serializers.py
│   │   ├── admin.py
│   │   └── tests.py
│   └── users/
│       └── ...
├── core/
│   ├── __init__.py
│   ├── config.py
│   ├── middleware.py
│   └── exceptions.py
├── templates/
│   └── ...
├── static/
│   └── ...
├── media/
│   └── ...
├── tests/
│   ├── __init__.py
│   └── conftest.py
├── .env
├── .env.example
├── requirements.txt
├── manage.py
└── pytest.ini
```

### settings.py 配置

```python
# myproject/settings.py
import os
from pathlib import Path
from datetime import timedelta

BASE_DIR = Path(__file__).resolve().parent.parent

# 安全设置
SECRET_KEY = os.environ.get('DJANGO_SECRET_KEY')
DEBUG = os.environ.get('DEBUG', 'False') == 'True'
ALLOWED_HOSTS = os.environ.get('ALLOWED_HOSTS', '').split(',')

# 应用
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    # 第三方
    'rest_framework',
    'rest_framework_simplejwt',
    'corsheaders',
    'drf_spectacular',
    # 本地应用
    'apps.blog',
    'apps.users',
]

MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'corsheaders.middleware.CorsMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]

ROOT_URLCONF = 'myproject.urls'

TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [BASE_DIR / 'templates'],
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
]

WSGI_APPLICATION = 'myproject.wsgi.application'

# 数据库
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': os.environ.get('DB_NAME', 'mydb'),
        'USER': os.environ.get('DB_USER', 'postgres'),
        'PASSWORD': os.environ.get('DB_PASSWORD'),
        'HOST': os.environ.get('DB_HOST', 'localhost'),
        'PORT': os.environ.get('DB_PORT', '5432'),
        'CONN_MAX_AGE': 60,
        'OPTIONS': {
            'connect_timeout': 10,
        },
    }
}

# 缓存
CACHES = {
    'default': {
        'BACKEND': 'django_redis.cache.RedisCache',
        'LOCATION': os.environ.get('REDIS_URL', 'redis://localhost:6379/1'),
        'OPTIONS': {
            'CLIENT_CLASS': 'django_redis.client.DefaultClient',
        },
    }
}

# 认证
AUTH_PASSWORD_VALIDATORS = [
    {'NAME': 'django.contrib.auth.password_validation.UserAttributeSimilarityValidator'},
    {'NAME': 'django.contrib.auth.password_validation.MinimumLengthValidator'},
    {'NAME': 'django.contrib.auth.password_validation.CommonPasswordValidator'},
    {'NAME': 'django.contrib.auth.password_validation.NumericPasswordValidator'},
]

AUTH_USER_MODEL = 'users.User'

# REST Framework
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework_simplejwt.authentication.JWTAuthentication',
    ],
    'DEFAULT_PERMISSION_CLASSES': [
        'rest_framework.permissions.IsAuthenticated',
    ],
    'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.PageNumberPagination',
    'PAGE_SIZE': 20,
    'DEFAULT_SCHEMA_CLASS': 'drf_spectacular.openapi.AutoSchema',
}

# JWT
SIMPLE_JWT = {
    'ACCESS_TOKEN_LIFETIME': timedelta(minutes=15),
    'REFRESH_TOKEN_LIFETIME': timedelta(days=7),
    'ROTATE_REFRESH_TOKENS': True,
    'BLACKLIST_AFTER_ROTATION': True,
    'AUTH_HEADER_TYPES': ('Bearer',),
}

# CORS
CORS_ALLOWED_ORIGINS = os.environ.get('CORS_ORIGINS', '').split(',')
CORS_ALLOW_CREDENTIALS = True

# 国际化
LANGUAGE_CODE = 'zh-hans'
TIME_ZONE = 'Asia/Shanghai'
USE_I18N = True
USE_TZ = True

# 静态文件
STATIC_URL = '/static/'
STATIC_ROOT = BASE_DIR / 'staticfiles'
STATICFILES_DIRS = [BASE_DIR / 'static']

# 媒体文件
MEDIA_URL = '/media/'
MEDIA_ROOT = BASE_DIR / 'media'

# 日志
LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    'formatters': {
        'verbose': {
            'format': '{levelname} {asctime} {module} {message}',
            'style': '{',
        },
    },
    'handlers': {
        'console': {
            'class': 'logging.StreamHandler',
            'formatter': 'verbose',
        },
        'file': {
            'class': 'logging.FileHandler',
            'filename': BASE_DIR / 'logs' / 'django.log',
            'formatter': 'verbose',
        },
    },
    'root': {
        'handlers': ['console', 'file'],
        'level': 'INFO',
    },
}
```

---

## Django ORM 深度使用

### 模型关系设计

```python
# apps/users/models.py
from django.db import models
from django.contrib.auth.models import AbstractUser, BaseUserManager
from django.utils.translation import gettext_lazy as _


class UserManager(BaseUserManager):
    """自定义用户管理器"""

    def create_user(self, email, password=None, **extra_fields):
        if not email:
            raise ValueError('Email is required')
        email = self.normalize_email(email)
        user = self.model(email=email, **extra_fields)
        user.set_password(password)
        user.save(using=self._db)
        return user

    def create_superuser(self, email, password=None, **extra_fields):
        extra_fields.setdefault('is_staff', True)
        extra_fields.setdefault('is_superuser', True)
        return self.create_user(email, password, **extra_fields)


class User(AbstractUser):
    """自定义用户模型"""

    username = None
    email = models.EmailField(_('email address'), unique=True)

    USERNAME_FIELD = 'email'
    REQUIRED_FIELDS = []

    objects = UserManager()

    role = models.CharField(
        max_length=20,
        choices=[
            ('USER', 'User'),
            ('ADMIN', 'Admin'),
            ('MODERATOR', 'Moderator'),
        ],
        default='USER',
    )
    avatar = models.ImageField(upload_to='avatars/', null=True, blank=True)
    bio = models.TextField(max_length=500, blank=True)
    is_verified = models.BooleanField(default=False)

    class Meta:
        db_table = 'users'
        verbose_name = _('user')
        verbose_name_plural = _('users')
```

```python
# apps/blog/models.py
from django.db import models
from django.contrib.auth import get_user_model

User = get_user_model()


class Category(models.Model):
    """分类"""

    name = models.CharField(max_length=100)
    slug = models.SlugField(max_length=100, unique=True)
    description = models.TextField(blank=True)
    parent = models.ForeignKey(
        'self',
        on_delete=models.CASCADE,
        null=True,
        blank=True,
        related_name='children',
    )
    order = models.IntegerField(default=0)

    class Meta:
        verbose_name = _('category')
        verbose_name_plural = _('categories')
        ordering = ['order', 'name']

    def __str__(self):
        return self.name


class Tag(models.Model):
    """标签"""

    name = models.CharField(max_length=50, unique=True)
    slug = models.SlugField(max_length=50, unique=True)

    class Meta:
        verbose_name = _('tag')
        verbose_name_plural = _('tags')

    def __str__(self):
        return self.name


class Post(models.Model):
    """文章"""

    STATUS_CHOICES = [
        ('DRAFT', 'Draft'),
        ('PUBLISHED', 'Published'),
        ('ARCHIVED', 'Archived'),
    ]

    title = models.CharField(max_length=200)
    slug = models.SlugField(max_length=200, unique=True)
    excerpt = models.TextField(max_length=300, blank=True)
    content = models.TextField()
    featured_image = models.ImageField(upload_to='posts/', null=True, blank=True)
    status = models.CharField(max_length=20, choices=STATUS_CHOICES, default='DRAFT')
    published_at = models.DateTimeField(null=True, blank=True)

    author = models.ForeignKey(
        User,
        on_delete=models.CASCADE,
        related_name='posts',
    )
    category = models.ForeignKey(
        Category,
        on_delete=models.SET_NULL,
        null=True,
        related_name='posts',
    )
    tags = models.ManyToManyField(Tag, related_name='posts', blank=True)

    view_count = models.PositiveIntegerField(default=0)
    like_count = models.PositiveIntegerField(default=0)

    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)

    class Meta:
        verbose_name = _('post')
        verbose_name_plural = _('posts')
        ordering = ['-published_at', '-created_at']
        indexes = [
            models.Index(fields=['status', 'published_at']),
            models.Index(fields=['slug']),
            models.Index(fields=['author', 'created_at']),
        ]

    def __str__(self):
        return self.title


class Comment(models.Model):
    """评论"""

    content = models.TextField()
    is_approved = models.BooleanField(default=True)

    author = models.ForeignKey(
        User,
        on_delete=models.CASCADE,
        related_name='comments',
    )
    post = models.ForeignKey(
        Post,
        on_delete=models.CASCADE,
        related_name='comments',
    )
    parent = models.ForeignKey(
        'self',
        on_delete=models.CASCADE,
        null=True,
        blank=True,
        related_name='children',
    )

    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)

    class Meta:
        verbose_name = _('comment')
        verbose_name_plural = _('comments')
        ordering = ['created_at']

    def __str__(self):
        return f'{self.author} - {self.content[:50]}'
```

### 查询优化

```python
# apps/blog/services.py
from django.db.models import Q, Count, Prefetch
from django.core.paginator import Paginator


class PostService:
    """文章服务"""

    @staticmethod
    def get_published_posts(page=1, per_page=10, category=None, tag=None):
        """获取已发布的文章（分页）"""
        queryset = Post.objects.filter(status='PUBLISHED')

        if category:
            queryset = queryset.filter(category__slug=category)
        if tag:
            queryset = queryset.filter(tags__slug=tag)

        queryset = queryset.select_related('author', 'category')
        queryset = queryset.prefetch_related('tags')

        paginator = Paginator(queryset, per_page)
        page_obj = paginator.get_page(page)

        return page_obj

    @staticmethod
    def get_post_detail(slug):
        """获取文章详情（增加浏览数）"""
        post = Post.objects.select_related('author', 'category').prefetch_related(
            Prefetch(
                'comments',
                queryset=Comment.objects.filter(is_approved=True)
                .select_related('author')
                .order_by('-created_at')
            ),
            'tags',
        ).get(slug=slug)

        # 增加浏览数
        Post.objects.filter(pk=post.pk).update(view_count=models.F('view_count') + 1)

        return post

    @staticmethod
    def get_related_posts(post, limit=5):
        """获取相关文章"""
        return Post.objects.filter(
            status='PUBLISHED',
            category=post.category,
        ).exclude(pk=post.pk).order_by('-published_at')[:limit]

    @staticmethod
    def search_posts(query, page=1, per_page=10):
        """搜索文章"""
        queryset = Post.objects.filter(
            Q(title__icontains=query) | Q(content__icontains=query),
            status='PUBLISHED',
        ).select_related('author', 'category')

        paginator = Paginator(queryset, per_page)
        page_obj = paginator.get_page(page)

        return page_obj
```

---

## Django REST Framework 深度使用

### Serializers

```python
# apps/users/serializers.py
from rest_framework import serializers
from django.contrib.auth import get_user_model
from apps.blog.serializers import PostSerializer

User = get_user_model()


class UserSerializer(serializers.ModelSerializer):
    """用户序列化器"""

    class Meta:
        model = User
        fields = [
            'id', 'email', 'first_name', 'last_name',
            'avatar', 'role', 'bio', 'is_verified',
            'date_joined',
        ]
        read_only_fields = ['id', 'email', 'role', 'is_verified', 'date_joined']


class UserCreateSerializer(serializers.ModelSerializer):
    """用户创建序列化器"""

    password = serializers.CharField(write_only=True, min_length=8)

    class Meta:
        model = User
        fields = ['email', 'password', 'first_name', 'last_name']

    def create(self, validated_data):
        password = validated_data.pop('password')
        user = User(**validated_data)
        user.set_password(password)
        user.save()
        return user


class UserProfileSerializer(serializers.ModelSerializer):
    """用户资料序列化器"""

    posts_count = serializers.SerializerMethodField()
    recent_posts = serializers.SerializerMethodField()

    class Meta:
        model = User
        fields = [
            'id', 'email', 'first_name', 'last_name',
            'avatar', 'bio', 'role', 'is_verified',
            'date_joined', 'last_login',
            'posts_count', 'recent_posts',
        ]

    def get_posts_count(self, obj):
        return obj.posts.filter(status='PUBLISHED').count()

    def get_recent_posts(self, obj):
        posts = obj.posts.filter(status='PUBLISHED')[:5]
        return PostSerializer(posts, many=True).data
```

```python
# apps/blog/serializers.py
from rest_framework import serializers
from .models import Post, Category, Tag, Comment
from apps.users.serializers import UserSerializer


class TagSerializer(serializers.ModelSerializer):
    """标签序列化器"""

    class Meta:
        model = Tag
        fields = ['id', 'name', 'slug']


class CategorySerializer(serializers.ModelSerializer):
    """分类序列化器"""

    posts_count = serializers.SerializerMethodField()

    class Meta:
        model = Category
        fields = ['id', 'name', 'slug', 'description', 'posts_count']

    def get_posts_count(self, obj):
        return obj.posts.filter(status='PUBLISHED').count()


class CommentSerializer(serializers.ModelSerializer):
    """评论序列化器"""

    author = UserSerializer(read_only=True)
    replies = serializers.SerializerMethodField()
    replies_count = serializers.SerializerMethodField()

    class Meta:
        model = Comment
        fields = [
            'id', 'content', 'author', 'created_at',
            'replies', 'replies_count',
        ]
        read_only_fields = ['id', 'author', 'created_at']

    def get_replies(self, obj):
        if obj.children.exists():
            return CommentSerializer(obj.children.all(), many=True).data
        return []

    def get_replies_count(self, obj):
        return obj.children.count()


class PostListSerializer(serializers.ModelSerializer):
    """文章列表序列化器"""

    author = UserSerializer(read_only=True)
    category = CategorySerializer(read_only=True)
    tags = TagSerializer(many=True, read_only=True)

    class Meta:
        model = Post
        fields = [
            'id', 'title', 'slug', 'excerpt', 'featured_image',
            'status', 'published_at', 'view_count', 'like_count',
            'author', 'category', 'tags',
            'created_at', 'updated_at',
        ]


class PostDetailSerializer(PostListSerializer):
    """文章详情序列化器"""

    comments = serializers.SerializerMethodField()

    class Meta(PostListSerializer.Meta):
        fields = PostListSerializer.Meta.fields + ['content', 'comments']

    def get_comments(self, obj):
        comments = obj.comments.filter(is_approved=True, parent=None)
        return CommentSerializer(comments, many=True).data
```

### Views

```python
# apps/blog/views.py
from rest_framework import viewsets, status, permissions
from rest_framework.decorators import action
from rest_framework.response import Response
from django.shortcuts import get_object_or_404

from .models import Post, Category, Tag, Comment
from .serializers import (
    PostListSerializer,
    PostDetailSerializer,
    CategorySerializer,
    TagSerializer,
    CommentSerializer,
)
from .services import PostService


class IsOwnerOrReadOnly(permissions.BasePermission):
    """只有作者可以修改"""

    def has_object_permission(self, request, view, obj):
        if request.method in permissions.SAFE_METHODS:
            return True
        return obj.author == request.user


class PostViewSet(viewsets.ModelViewSet):
    """文章视图集"""

    serializer_class = PostListSerializer
    permission_classes = [permissions.IsAuthenticatedOrReadOnly, IsOwnerOrReadOnly]
    lookup_field = 'slug'

    def get_queryset(self):
        queryset = Post.objects.filter(status='PUBLISHED')

        # 过滤
        category = self.request.query_params.get('category')
        tag = self.request.query_params.get('tag')
        author = self.request.query_params.get('author')

        if category:
            queryset = queryset.filter(category__slug=category)
        if tag:
            queryset = queryset.filter(tags__slug=tag)
        if author:
            queryset = queryset.filter(author__username=author)

        # 优化查询
        queryset = queryset.select_related('author', 'category')
        queryset = queryset.prefetch_related('tags')

        return queryset

    def get_serializer_class(self):
        if self.action == 'retrieve':
            return PostDetailSerializer
        return PostListSerializer

    def retrieve(self, request, *args, **kwargs):
        instance = get_object_or_404(
            Post.objects.select_related('author', 'category')
            .prefetch_related('tags', 'comments'),
            slug=kwargs.get('slug'),
            status='PUBLISHED',
        )

        # 增加浏览数
        Post.objects.filter(pk=instance.pk).update(
            view_count=models.F('view_count') + 1
        )

        serializer = self.get_serializer(instance)
        return Response(serializer.data)

    @action(detail=False, methods=['get'])
    def search(self, request):
        """搜索文章"""
        query = request.query_params.get('q', '')
        page = int(request.query_params.get('page', 1))

        if not query:
            return Response({'error': 'Query parameter q is required'})

        results = PostService.search_posts(query, page)
        serializer = PostListSerializer(results.object_list, many=True)

        return Response({
            'results': serializer.data,
            'page': results.number,
            'pages': results.paginator.num_pages,
            'total': results.paginator.count,
        })


class CategoryViewSet(viewsets.ReadOnlyModelViewSet):
    """分类视图集"""

    queryset = Category.objects.all()
    serializer_class = CategorySerializer
    permission_classes = [permissions.AllowAny]


class TagViewSet(viewsets.ReadOnlyModelViewSet):
    """标签视图集"""

    queryset = Tag.objects.all()
    serializer_class = TagSerializer
    permission_classes = [permissions.AllowAny]


class CommentViewSet(viewsets.ModelViewSet):
    """评论视图集"""

    serializer_class = CommentSerializer
    permission_classes = [permissions.IsAuthenticatedOrReadOnly]

    def get_queryset(self):
        post_slug = self.request.query_params.get('post')
        if post_slug:
            return Comment.objects.filter(
                post__slug=post_slug,
                is_approved=True,
                parent=None,
            ).select_related('author', 'post')
        return Comment.objects.none()

    def perform_create(self, serializer):
        serializer.save(author=self.request.user)
```

---

## 常见陷阱与最佳实践

### 陷阱 1：N+1 查询

```python
# ❌ 错误：N+1 查询
def get_posts():
    posts = Post.objects.all()
    for post in posts:
        print(post.author.name)  # 每篇文章都会执行一次查询

# ✅ 正确：使用 select_related
def get_posts():
    posts = Post.objects.select_related('author').all()
    for post in posts:
        print(post.author.name)  # 使用预加载的数据
```

### 陷阱 2：未使用数据库索引

```python
# ❌ 错误：频繁查询的字段没有索引
class Post(models.Model):
    slug = models.SlugField()  # 缺少索引
    author = models.ForeignKey(User, on_delete=models.CASCADE)

# ✅ 正确：添加索引
class Post(models.Model):
    slug = models.SlugField(unique=True, db_index=True)  # 添加索引
    author = models.ForeignKey(User, on_delete=models.CASCADE)

    class Meta:
        indexes = [
            models.Index(fields=['status', 'published_at']),
            models.Index(fields=['author', 'created_at']),
        ]
```

### 陷阱 3：在视图中直接处理业务逻辑

```python
# ❌ 错误：业务逻辑放在视图中
class PostViewSet(viewsets.ModelViewSet):
    def create(self, request):
        # 复杂的业务逻辑
        if request.data['status'] == 'PUBLISHED':
            request.data['published_at'] = timezone.now()
        # 更多逻辑...

# ✅ 正确：使用 Service 层
class PostViewSet(viewsets.ModelViewSet):
    def create(self, request):
        serializer = self.get_serializer(data=request.data)
        serializer.is_valid(raise_exception=True)
        post = PostService.create_post(
            serializer.validated_data,
            request.user
        )
        return Response(PostSerializer(post).data, status=status.HTTP_201_CREATED)
```

### 最佳实践清单

1. **使用虚拟环境**：每个项目使用独立的虚拟环境。
2. **环境变量**：使用 `python-decouple` 或 `django-environ` 管理配置。
3. **数据库迁移**：始终使用 `makemigrations` 和 `migrate`。
4. **查询优化**：使用 `select_related` 和 `prefetch_related`。
5. **索引**：为经常查询的字段添加索引。
6. **缓存**：使用 Django 缓存框架减少数据库查询。
7. **异步任务**：使用 Celery 处理耗时任务。
8. **测试**：使用 pytest-django 编写测试。
9. **API 文档**：使用 drf-spectacular 生成 OpenAPI 文档。
10. **安全**：使用 HTTPS、CORS、CSRF 保护。

---

## 与其他框架对比

### Django vs Flask

| 特性 | Django | Flask |
|------|--------|-------|
| **架构** | 全功能 MTV | 微框架 |
| **ORM** | Django ORM | SQLAlchemy |
| **Admin** | 内置 | 需要扩展 |
| **认证** | 内置 | 需要扩展 |
| **表单** | Django Forms | WTForms |
| **适用场景** | 企业级应用 | 轻量应用 |

### Django vs FastAPI

| 特性 | Django | FastAPI |
|------|--------|---------|
| **架构** | 全功能 MTV | 轻量 ASGI |
| **ORM** | Django ORM | SQLAlchemy |
| **性能** | 中等 | 极高 |
| **异步** | Django 4.1+ | 原生异步 |
| **API** | DRF | 原生支持 |
| **适用场景** | 全栈应用 | API-first |

### Django vs Spring Boot

| 特性 | Django | Spring Boot |
|------|--------|-------------|
| **语言** | Python | Java/Kotlin |
| **ORM** | Django ORM | JPA/Hibernate |
| **Admin** | 内置 | 需要开发 |
| **认证** | 内置 | Spring Security |
| **性能** | 中等 | 高 |
| **生态** | 成熟 | 庞大企业级 |

---

> [!SUCCESS]
> Django 是 Python 生态中最成熟的全功能 Web 框架，其内置的 ORM、Admin、认证和表单系统使其成为企业级应用的首选。Django REST Framework 提供了强大的 API 构建能力，结合 Django Channels 可以实现 WebSocket 等实时功能。虽然 Django 在性能上不如 FastAPI，但对于需要快速开发、功能完善、维护成本低的团队，Django 仍然是最佳选择。
