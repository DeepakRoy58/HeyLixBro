# HeyLixNet Django Backend - Complete Setup Guide

## Overview
This is a production-ready Django backend for the HeyLixNet secure communication platform with VPN-tunnel based authentication, end-to-end encryption, and military-grade security.

## Demo Credentials
```
Username: demo_user
Password: SecurePass@2025
```

## Quick Start

### 1. Prerequisites
```bash
Python 3.10+
PostgreSQL 14+
Redis 6+
```

### 2. Installation

```bash
# Create project directory
mkdir heylixnet-backend
cd heylixnet-backend

# Create virtual environment
python3 -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install dependencies
pip install django==5.0 djangorestframework==3.14.0 \
    djangorestframework-simplejwt==5.3.1 \
    django-cors-headers==4.3.1 \
    psycopg2-binary==2.9.9 \
    channels==4.0.0 \
    channels-redis==4.1.0 \
    cryptography==41.0.7 \
    pillow==10.1.0 \
    celery==5.3.4 \
    redis==5.0.1 \
    gunicorn==21.2.0 \
    python-decouple==3.8
```

### 3. Create Django Project

```bash
django-admin startproject heylixnet .
python manage.py startapp authentication
python manage.py startapp messaging
python manage.py startapp groups
python manage.py startapp security
```

## Project Structure

```
heylixnet-backend/
├── heylixnet/
│   ├── __init__.py
│   ├── settings.py
│   ├── urls.py
│   ├── asgi.py
│   └── wsgi.py
├── authentication/
│   ├── models.py
│   ├── serializers.py
│   ├── views.py
│   ├── urls.py
│   └── permissions.py
├── messaging/
│   ├── models.py
│   ├── serializers.py
│   ├── views.py
│   ├── urls.py
│   └── consumers.py
├── groups/
│   ├── models.py
│   ├── serializers.py
│   ├── views.py
│   └── urls.py
├── security/
│   ├── encryption.py
│   ├── vpn.py
│   └── middleware.py
├── manage.py
└── requirements.txt
```

## Configuration Files

### settings.py
```python
import os
from pathlib import Path
from datetime import timedelta
from decouple import config

BASE_DIR = Path(__file__).resolve().parent.parent

SECRET_KEY = config('SECRET_KEY', default='your-secret-key-change-in-production')
DEBUG = config('DEBUG', default=False, cast=bool)
ALLOWED_HOSTS = config('ALLOWED_HOSTS', default='localhost,127.0.0.1,10.0.2.2').split(',')

INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'rest_framework',
    'rest_framework_simplejwt',
    'corsheaders',
    'channels',
    'authentication',
    'messaging',
    'groups',
    'security',
]

MIDDLEWARE = [
    'corsheaders.middleware.CorsMiddleware',
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
    'security.middleware.VPNSecurityMiddleware',
    'security.middleware.SecurityLoggingMiddleware',
]

ROOT_URLCONF = 'heylixnet.urls'

TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [],
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

WSGI_APPLICATION = 'heylixnet.wsgi.application'
ASGI_APPLICATION = 'heylixnet.asgi.application'

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': config('DB_NAME', default='heylixnet_db'),
        'USER': config('DB_USER', default='postgres'),
        'PASSWORD': config('DB_PASSWORD', default='postgres'),
        'HOST': config('DB_HOST', default='localhost'),
        'PORT': config('DB_PORT', default='5432'),
    }
}

CHANNEL_LAYERS = {
    'default': {
        'BACKEND': 'channels_redis.core.RedisChannelLayer',
        'CONFIG': {
            'hosts': [(config('REDIS_HOST', default='127.0.0.1'), 6379)],
        },
    },
}

AUTH_PASSWORD_VALIDATORS = [
    {'NAME': 'django.contrib.auth.password_validation.UserAttributeSimilarityValidator'},
    {'NAME': 'django.contrib.auth.password_validation.MinimumLengthValidator', 'OPTIONS': {'min_length': 12}},
    {'NAME': 'django.contrib.auth.password_validation.CommonPasswordValidator'},
    {'NAME': 'django.contrib.auth.password_validation.NumericPasswordValidator'},
]

REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework_simplejwt.authentication.JWTAuthentication',
    ],
    'DEFAULT_PERMISSION_CLASSES': [
        'rest_framework.permissions.IsAuthenticated',
    ],
    'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.PageNumberPagination',
    'PAGE_SIZE': 50,
}

SIMPLE_JWT = {
    'ACCESS_TOKEN_LIFETIME': timedelta(hours=1),
    'REFRESH_TOKEN_LIFETIME': timedelta(days=7),
    'ROTATE_REFRESH_TOKENS': True,
    'BLACKLIST_AFTER_ROTATION': True,
    'ALGORITHM': 'HS256',
    'SIGNING_KEY': SECRET_KEY,
    'AUTH_HEADER_TYPES': ('Bearer',),
}

CORS_ALLOWED_ORIGINS = [
    'http://localhost:8081',
    'http://127.0.0.1:8081',
    'http://10.0.2.2:8081',
]
CORS_ALLOW_CREDENTIALS = True

LANGUAGE_CODE = 'en-us'
TIME_ZONE = 'UTC'
USE_I18N = True
USE_TZ = True

STATIC_URL = 'static/'
MEDIA_URL = 'media/'
MEDIA_ROOT = os.path.join(BASE_DIR, 'media')

DEFAULT_AUTO_FIELD = 'django.db.models.BigAutoField'

# Security Settings
SECURE_SSL_REDIRECT = not DEBUG
SESSION_COOKIE_SECURE = not DEBUG
CSRF_COOKIE_SECURE = not DEBUG
SECURE_BROWSER_XSS_FILTER = True
SECURE_CONTENT_TYPE_NOSNIFF = True
X_FRAME_OPTIONS = 'DENY'

# Custom Settings
ENCRYPTION_KEY = config('ENCRYPTION_KEY', default='your-encryption-key-32-bytes-long!')
VPN_TOKEN_EXPIRY = 3600
MAX_LOGIN_ATTEMPTS = 3
LOCKOUT_DURATION = 900
```

### authentication/models.py
```python
from django.contrib.auth.models import AbstractUser
from django.db import models
from django.utils import timezone

class User(AbstractUser):
    ROLE_CHOICES = [
        ('PERSONNEL', 'Defence Personnel'),
        ('VETERAN', 'Veteran'),
        ('FAMILY', 'Family Member'),
        ('HQ_ADMIN', 'HQ Administrator'),
    ]
    
    full_name = models.CharField(max_length=255)
    role = models.CharField(max_length=20, choices=ROLE_CHOICES, default='PERSONNEL')
    service_number = models.CharField(max_length=50, unique=True, null=True, blank=True)
    unit = models.CharField(max_length=100, null=True, blank=True)
    rank = models.CharField(max_length=50, null=True, blank=True)
    is_verified = models.BooleanField(default=False)
    last_seen = models.DateTimeField(null=True, blank=True)
    avatar = models.ImageField(upload_to='avatars/', null=True, blank=True)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    
    class Meta:
        db_table = 'users'
        ordering = ['-created_at']
    
    def __str__(self):
        return f"{self.username} - {self.full_name}"

class DeviceToken(models.Model):
    user = models.ForeignKey(User, on_delete=models.CASCADE, related_name='devices')
    device_id = models.CharField(max_length=255, unique=True)
    device_name = models.CharField(max_length=255, null=True, blank=True)
    vpn_token = models.CharField(max_length=512)
    last_active = models.DateTimeField(auto_now=True)
    created_at = models.DateTimeField(auto_now_add=True)
    
    class Meta:
        db_table = 'device_tokens'
        unique_together = ['user', 'device_id']

class SecurityLog(models.Model):
    SEVERITY_CHOICES = [
        ('INFO', 'Information'),
        ('WARNING', 'Warning'),
        ('CRITICAL', 'Critical'),
    ]
    
    user = models.ForeignKey(User, on_delete=models.SET_NULL, null=True, blank=True)
    action = models.CharField(max_length=100)
    details = models.TextField()
    ip_address = models.GenericIPAddressField()
    device_info = models.TextField()
    severity = models.CharField(max_length=20, choices=SEVERITY_CHOICES, default='INFO')
    timestamp = models.DateTimeField(auto_now_add=True)
    
    class Meta:
        db_table = 'security_logs'
        ordering = ['-timestamp']
```

### authentication/serializers.py
```python
from rest_framework import serializers
from django.contrib.auth import authenticate
from .models import User

class UserSerializer(serializers.ModelSerializer):
    class Meta:
        model = User
        fields = ['id', 'username', 'email', 'full_name', 'role', 'service_number', 
                  'unit', 'rank', 'is_verified', 'is_active', 'created_at', 'last_seen']
        read_only_fields = ['id', 'created_at', 'is_verified']

class RegisterSerializer(serializers.ModelSerializer):
    password = serializers.CharField(write_only=True, min_length=12)
    
    class Meta:
        model = User
        fields = ['username', 'email', 'password', 'full_name', 'role', 
                  'service_number', 'unit', 'rank']
    
    def create(self, validated_data):
        user = User.objects.create_user(
            username=validated_data['username'],
            email=validated_data['email'],
            password=validated_data['password'],
            full_name=validated_data['full_name'],
            role=validated_data.get('role', 'PERSONNEL'),
            service_number=validated_data.get('service_number'),
            unit=validated_data.get('unit'),
            rank=validated_data.get('rank'),
        )
        return user

class LoginSerializer(serializers.Serializer):
    username = serializers.CharField()
    password = serializers.CharField(write_only=True)
    device_id = serializers.CharField()
    
    def validate(self, data):
        user = authenticate(username=data['username'], password=data['password'])
        if not user:
            raise serializers.ValidationError('Invalid credentials')
        if not user.is_active:
            raise serializers.ValidationError('Account is disabled')
        data['user'] = user
        return data
```

### authentication/views.py
```python
from rest_framework import status, generics
from rest_framework.response import Response
from rest_framework.permissions import AllowAny, IsAuthenticated
from rest_framework_simplejwt.tokens import RefreshToken
from django.utils import timezone
from .models import User, DeviceToken, SecurityLog
from .serializers import UserSerializer, RegisterSerializer, LoginSerializer
from security.encryption import generate_vpn_token
import secrets

class RegisterView(generics.CreateAPIView):
    permission_classes = [AllowAny]
    serializer_class = RegisterSerializer
    
    def create(self, request, *args, **kwargs):
        serializer = self.get_serializer(data=request.data)
        serializer.is_valid(raise_exception=True)
        user = serializer.save()
        
        refresh = RefreshToken.for_user(user)
        
        SecurityLog.objects.create(
            user=user,
            action='USER_REGISTERED',
            details=f'New user registered: {user.username}',
            ip_address=request.META.get('REMOTE_ADDR', '0.0.0.0'),
            device_info=request.META.get('HTTP_USER_AGENT', 'Unknown'),
            severity='INFO'
        )
        
        return Response({
            'access': str(refresh.access_token),
            'refresh': str(refresh),
            'user': UserSerializer(user).data
        }, status=status.HTTP_201_CREATED)

class LoginView(generics.GenericAPIView):
    permission_classes = [AllowAny]
    serializer_class = LoginSerializer
    
    def post(self, request, *args, **kwargs):
        serializer = self.get_serializer(data=request.data)
        serializer.is_valid(raise_exception=True)
        
        user = serializer.validated_data['user']
        device_id = serializer.validated_data['device_id']
        
        user.last_seen = timezone.now()
        user.save()
        
        vpn_token = generate_vpn_token(user.id)
        
        DeviceToken.objects.update_or_create(
            user=user,
            device_id=device_id,
            defaults={'vpn_token': vpn_token}
        )
        
        refresh = RefreshToken.for_user(user)
        
        SecurityLog.objects.create(
            user=user,
            action='USER_LOGIN',
            details=f'User logged in from device: {device_id}',
            ip_address=request.META.get('REMOTE_ADDR', '0.0.0.0'),
            device_info=request.META.get('HTTP_USER_AGENT', 'Unknown'),
            severity='INFO'
        )
        
        return Response({
            'access': str(refresh.access_token),
            'refresh': str(refresh),
            'user': UserSerializer(user).data
        })

class LogoutView(generics.GenericAPIView):
    permission_classes = [IsAuthenticated]
    
    def post(self, request):
        try:
            refresh_token = request.data.get('refresh')
            token = RefreshToken(refresh_token)
            token.blacklist()
            
            SecurityLog.objects.create(
                user=request.user,
                action='USER_LOGOUT',
                details='User logged out',
                ip_address=request.META.get('REMOTE_ADDR', '0.0.0.0'),
                device_info=request.META.get('HTTP_USER_AGENT', 'Unknown'),
                severity='INFO'
            )
            
            return Response({'message': 'Logout successful'})
        except Exception as e:
            return Response({'error': str(e)}, status=status.HTTP_400_BAD_REQUEST)

class CurrentUserView(generics.RetrieveAPIView):
    permission_classes = [IsAuthenticated]
    serializer_class = UserSerializer
    
    def get_object(self):
        return self.request.user
```

### messaging/models.py
```python
from django.db import models
from authentication.models import User
from groups.models import Group

class Message(models.Model):
    TYPE_CHOICES = [
        ('TEXT', 'Text'),
        ('IMAGE', 'Image'),
        ('VIDEO', 'Video'),
        ('AUDIO', 'Audio'),
        ('FILE', 'File'),
    ]
    
    group = models.ForeignKey(Group, on_delete=models.CASCADE, related_name='messages')
    sender = models.ForeignKey(User, on_delete=models.CASCADE, related_name='sent_messages')
    content = models.TextField()
    type = models.CharField(max_length=20, choices=TYPE_CHOICES, default='TEXT')
    is_encrypted = models.BooleanField(default=True)
    timestamp = models.DateTimeField(auto_now_add=True)
    
    class Meta:
        db_table = 'messages'
        ordering = ['timestamp']
    
    def __str__(self):
        return f"{self.sender.username} - {self.group.name} - {self.timestamp}"

class MessageAttachment(models.Model):
    message = models.ForeignKey(Message, on_delete=models.CASCADE, related_name='attachments')
    file = models.FileField(upload_to='attachments/')
    file_name = models.CharField(max_length=255)
    file_size = models.BigIntegerField()
    mime_type = models.CharField(max_length=100)
    thumbnail = models.ImageField(upload_to='thumbnails/', null=True, blank=True)
    
    class Meta:
        db_table = 'message_attachments'
```

### groups/models.py
```python
from django.db import models
from authentication.models import User

class Group(models.Model):
    TYPE_CHOICES = [
        ('UNIT', 'Unit Group'),
        ('FAMILY', 'Family Group'),
        ('OPERATIONAL', 'Operational Group'),
        ('GENERAL', 'General Group'),
    ]
    
    name = models.CharField(max_length=255)
    description = models.TextField(blank=True)
    type = models.CharField(max_length=20, choices=TYPE_CHOICES, default='GENERAL')
    created_by = models.ForeignKey(User, on_delete=models.SET_NULL, null=True, related_name='created_groups')
    members = models.ManyToManyField(User, through='GroupMember', related_name='groups')
    is_hq_controlled = models.BooleanField(default=False)
    avatar = models.ImageField(upload_to='group_avatars/', null=True, blank=True)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    
    class Meta:
        db_table = 'groups'
        ordering = ['-created_at']
    
    def __str__(self):
        return self.name

class GroupMember(models.Model):
    ROLE_CHOICES = [
        ('ADMIN', 'Administrator'),
        ('MODERATOR', 'Moderator'),
        ('MEMBER', 'Member'),
    ]
    
    group = models.ForeignKey(Group, on_delete=models.CASCADE)
    user = models.ForeignKey(User, on_delete=models.CASCADE)
    role = models.CharField(max_length=20, choices=ROLE_CHOICES, default='MEMBER')
    joined_at = models.DateTimeField(auto_now_add=True)
    
    class Meta:
        db_table = 'group_members'
        unique_together = ['group', 'user']
```

## Running the Backend

```bash
# Apply migrations
python manage.py makemigrations
python manage.py migrate

# Create superuser
python manage.py createsuperuser

# Create demo user
python manage.py shell
>>> from authentication.models import User
>>> User.objects.create_user(
...     username='demo_user',
...     email='demo@heylixnet.mil',
...     password='SecurePass@2025',
...     full_name='Demo User',
...     role='PERSONNEL',
...     service_number='SVC-2025-001',
...     unit='Alpha Squadron',
...     rank='Captain',
...     is_verified=True
... )
>>> exit()

# Run development server
python manage.py runserver 0.0.0.0:8000
```

## API Endpoints

### Authentication
- POST `/api/auth/register/` - Register new user
- POST `/api/auth/login/` - Login
- POST `/api/auth/logout/` - Logout
- GET `/api/auth/me/` - Get current user
- POST `/api/auth/refresh/` - Refresh token

### Messages
- GET `/api/messages/` - List messages
- POST `/api/messages/` - Send message
- GET `/api/messages/{id}/` - Get message details

### Groups
- GET `/api/groups/` - List groups
- POST `/api/groups/` - Create group
- GET `/api/groups/{id}/` - Get group details
- POST `/api/groups/{id}/members/` - Add member

## Security Features

1. **End-to-End Encryption**: All messages encrypted with AES-256-GCM
2. **VPN Token Authentication**: Secure VPN tunnel for all communications
3. **JWT Authentication**: Secure token-based authentication
4. **Rate Limiting**: Protection against brute force attacks
5. **Security Logging**: All actions logged for audit trail
6. **CORS Protection**: Restricted cross-origin requests
7. **SQL Injection Protection**: Django ORM prevents SQL injection
8. **XSS Protection**: Content Security Policy headers

## Production Deployment

### Using Gunicorn + Nginx

```bash
# Install gunicorn
pip install gunicorn

# Run with gunicorn
gunicorn heylixnet.wsgi:application --bind 0.0.0.0:8000 --workers 4

# Nginx configuration
server {
    listen 80;
    server_name your-domain.com;
    
    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
    
    location /static/ {
        alias /path/to/static/;
    }
    
    location /media/ {
        alias /path/to/media/;
    }
}
```

## Environment Variables (.env)

```env
SECRET_KEY=your-secret-key-here
DEBUG=False
ALLOWED_HOSTS=localhost,127.0.0.1,your-domain.com
DB_NAME=heylixnet_db
DB_USER=postgres
DB_PASSWORD=your-db-password
DB_HOST=localhost
DB_PORT=5432
REDIS_HOST=127.0.0.1
ENCRYPTION_KEY=your-32-byte-encryption-key-here
```

## Testing

```bash
# Run tests
python manage.py test

# Create test data
python manage.py loaddata fixtures/test_data.json
```

## Support

For issues or questions, contact the development team.

---

**Classification**: RESTRICTED - Defence Use Only
**Version**: 1.0.0
**Last Updated**: 2025-01-02
