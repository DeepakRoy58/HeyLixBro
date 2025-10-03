# SecureComm Defence - Backend Setup Guide

## Overview
This document provides comprehensive instructions for setting up the Django backend for the SecureComm Defence application.

## Architecture

### System Components
1. **Django REST API** - Core backend service
2. **PostgreSQL Database** - Primary data store
3. **Redis** - Session management and caching
4. **Socket.IO Server** - Real-time messaging
5. **VPN Gateway** - Secure tunnel management
6. **Nginx** - Reverse proxy and load balancer

### Security Features
- Military-grade AES-256-GCM encryption
- VPN tunnel authentication
- JWT-based access control
- Role-based permissions (RBAC)
- HQ approval workflow
- Audit logging
- Screenshot prevention
- Export restrictions

## Backend Configuration

### Environment Variables

Create a `.env` file in your backend directory:

```env
# Django Settings
SECRET_KEY=your-secret-key-here
DEBUG=False
ALLOWED_HOSTS=your-domain.com,localhost

# Database
DB_NAME=securecomm_db
DB_USER=securecomm_user
DB_PASSWORD=your-secure-password
DB_HOST=localhost
DB_PORT=5432

# Redis
REDIS_HOST=localhost
REDIS_PORT=6379
REDIS_PASSWORD=your-redis-password

# VPN Configuration
VPN_SERVER=vpn.your-domain.com
VPN_PORT=8443
VPN_ENCRYPTION_KEY=your-vpn-encryption-key

# JWT Settings
JWT_SECRET_KEY=your-jwt-secret
JWT_ACCESS_TOKEN_LIFETIME=3600
JWT_REFRESH_TOKEN_LIFETIME=86400

# Socket.IO
SOCKET_IO_PORT=8000
SOCKET_IO_CORS_ORIGINS=https://your-domain.com

# Security
MAX_LOGIN_ATTEMPTS=3
LOCKOUT_DURATION=900
SESSION_TIMEOUT=3600
```

### API Endpoints

#### Authentication Endpoints

**POST /api/auth/login**
```json
Request:
{
  "serviceNumber": "string",
  "password": "string",
  "deviceId": "string",
  "vpnToken": "string (optional)"
}

Response:
{
  "success": true,
  "data": {
    "user": {
      "id": "string",
      "serviceNumber": "string",
      "name": "string",
      "email": "string",
      "role": "personnel|veteran|family|admin",
      "isApproved": boolean,
      "isActive": boolean
    },
    "tokens": {
      "accessToken": "string",
      "refreshToken": "string",
      "expiresIn": number
    },
    "vpn": {
      "vpnToken": "string",
      "vpnServer": "string",
      "vpnPort": "string",
      "encryptionKey": "string",
      "expiresAt": "string"
    }
  }
}
```

**POST /api/auth/register**
```json
Request:
{
  "serviceNumber": "string",
  "name": "string",
  "email": "string",
  "phone": "string",
  "password": "string",
  "role": "personnel|veteran|family",
  "unit": "string (optional)",
  "rank": "string (optional)",
  "approvalCode": "string"
}

Response:
{
  "success": true,
  "message": "Registration submitted for HQ approval"
}
```

**POST /api/auth/logout**
```json
Request: (Requires Authorization header)

Response:
{
  "success": true,
  "message": "Logged out successfully"
}
```

**GET /api/auth/verify**
```json
Request: (Requires Authorization header)

Response:
{
  "success": true,
  "data": {
    "user": { /* user object */ },
    "valid": true
  }
}
```

**POST /api/auth/refresh**
```json
Request:
{
  "refreshToken": "string"
}

Response:
{
  "success": true,
  "data": {
    "accessToken": "string",
    "refreshToken": "string",
    "expiresIn": number
  }
}
```

**POST /api/auth/vpn/refresh**
```json
Request: (Requires Authorization header)

Response:
{
  "success": true,
  "data": {
    "vpnToken": "string",
    "vpnServer": "string",
    "vpnPort": "string",
    "encryptionKey": "string",
    "expiresAt": "string"
  }
}
```

#### Group Management Endpoints

**GET /api/groups**
```json
Response:
{
  "success": true,
  "data": [
    {
      "id": "string",
      "name": "string",
      "description": "string",
      "createdBy": "string",
      "members": ["userId1", "userId2"],
      "isHQApproved": boolean,
      "createdAt": "string"
    }
  ]
}
```

**POST /api/groups**
```json
Request:
{
  "name": "string",
  "description": "string",
  "members": ["userId1", "userId2"]
}

Response:
{
  "success": true,
  "data": { /* group object */ }
}
```

**GET /api/groups/:groupId**
**PUT /api/groups/:groupId**
**DELETE /api/groups/:groupId**

#### Messaging Endpoints

**GET /api/messages/:groupId**
```json
Query Parameters:
- limit: number (default: 50)
- offset: number (default: 0)
- before: timestamp (optional)

Response:
{
  "success": true,
  "data": {
    "messages": [
      {
        "id": "string",
        "groupId": "string",
        "senderId": "string",
        "senderName": "string",
        "content": "string (encrypted)",
        "type": "text|image|video|audio|file",
        "timestamp": "string",
        "encrypted": true,
        "metadata": {}
      }
    ],
    "hasMore": boolean
  }
}
```

**POST /api/messages/:groupId**
```json
Request:
{
  "content": "string (encrypted)",
  "type": "text|image|video|audio|file",
  "metadata": {}
}

Response:
{
  "success": true,
  "data": { /* message object */ }
}
```

#### Admin Endpoints

**GET /api/admin/pending-approvals**
**POST /api/admin/approve-user/:userId**
**POST /api/admin/approve-group/:groupId**
**GET /api/admin/audit-logs**
**GET /api/admin/active-sessions**

### Socket.IO Events

#### Client → Server Events

**connect**
```javascript
// Automatically handled with auth token
```

**join_group**
```json
{
  "groupId": "string"
}
```

**leave_group**
```json
{
  "groupId": "string"
}
```

**message**
```json
{
  "id": "string",
  "groupId": "string",
  "senderId": "string",
  "senderName": "string",
  "content": "string (encrypted)",
  "type": "text|image|video|audio|file",
  "timestamp": "string",
  "encrypted": true,
  "metadata": {}
}
```

**typing**
```json
{
  "groupId": "string",
  "isTyping": boolean
}
```

#### Server → Client Events

**message**
```json
{
  "id": "string",
  "groupId": "string",
  "senderId": "string",
  "senderName": "string",
  "content": "string (encrypted)",
  "type": "text|image|video|audio|file",
  "timestamp": "string",
  "encrypted": true,
  "metadata": {}
}
```

**user_joined**
**user_left**
**typing**
**error**

## Database Schema

### Users Table
```sql
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    service_number VARCHAR(50) UNIQUE NOT NULL,
    name VARCHAR(255) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    phone VARCHAR(20),
    password_hash VARCHAR(255) NOT NULL,
    role VARCHAR(20) NOT NULL CHECK (role IN ('personnel', 'veteran', 'family', 'admin')),
    unit VARCHAR(100),
    rank VARCHAR(50),
    is_approved BOOLEAN DEFAULT FALSE,
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    last_login TIMESTAMP,
    approval_code VARCHAR(100),
    approved_by UUID REFERENCES users(id),
    approved_at TIMESTAMP
);
```

### Groups Table
```sql
CREATE TABLE groups (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(255) NOT NULL,
    description TEXT,
    created_by UUID REFERENCES users(id),
    is_hq_approved BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### Group Members Table
```sql
CREATE TABLE group_members (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    group_id UUID REFERENCES groups(id) ON DELETE CASCADE,
    user_id UUID REFERENCES users(id) ON DELETE CASCADE,
    role VARCHAR(20) DEFAULT 'member' CHECK (role IN ('admin', 'moderator', 'member')),
    joined_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(group_id, user_id)
);
```

### Messages Table
```sql
CREATE TABLE messages (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    group_id UUID REFERENCES groups(id) ON DELETE CASCADE,
    sender_id UUID REFERENCES users(id),
    content TEXT NOT NULL,
    type VARCHAR(20) NOT NULL CHECK (type IN ('text', 'image', 'video', 'audio', 'file')),
    encrypted BOOLEAN DEFAULT TRUE,
    metadata JSONB,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    deleted_at TIMESTAMP
);
```

### VPN Sessions Table
```sql
CREATE TABLE vpn_sessions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id),
    vpn_token VARCHAR(255) UNIQUE NOT NULL,
    encryption_key VARCHAR(255) NOT NULL,
    device_id VARCHAR(255),
    ip_address VARCHAR(45),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    expires_at TIMESTAMP NOT NULL,
    revoked_at TIMESTAMP
);
```

### Audit Logs Table
```sql
CREATE TABLE audit_logs (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id),
    action VARCHAR(100) NOT NULL,
    resource_type VARCHAR(50),
    resource_id UUID,
    ip_address VARCHAR(45),
    user_agent TEXT,
    metadata JSONB,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

## Security Implementation

### Password Hashing
```python
from django.contrib.auth.hashers import make_password, check_password

# Hash password
hashed = make_password(password, salt=None, hasher='pbkdf2_sha256')

# Verify password
is_valid = check_password(password, hashed)
```

### JWT Token Generation
```python
import jwt
from datetime import datetime, timedelta

def generate_access_token(user_id):
    payload = {
        'user_id': str(user_id),
        'exp': datetime.utcnow() + timedelta(seconds=3600),
        'iat': datetime.utcnow(),
        'type': 'access'
    }
    return jwt.encode(payload, settings.JWT_SECRET_KEY, algorithm='HS256')
```

### VPN Token Generation
```python
import secrets

def generate_vpn_token():
    return secrets.token_urlsafe(32)

def generate_encryption_key():
    return secrets.token_hex(32)
```

### Message Encryption
```python
from cryptography.hazmat.primitives.ciphers.aead import AESGCM
import os

def encrypt_message(message, key):
    aesgcm = AESGCM(bytes.fromhex(key))
    nonce = os.urandom(12)
    ciphertext = aesgcm.encrypt(nonce, message.encode(), None)
    return nonce.hex() + ciphertext.hex()

def decrypt_message(encrypted_message, key):
    aesgcm = AESGCM(bytes.fromhex(key))
    nonce = bytes.fromhex(encrypted_message[:24])
    ciphertext = bytes.fromhex(encrypted_message[24:])
    plaintext = aesgcm.decrypt(nonce, ciphertext, None)
    return plaintext.decode()
```

## Deployment

### Docker Compose Setup
```yaml
version: '3.8'

services:
  db:
    image: postgres:15
    environment:
      POSTGRES_DB: securecomm_db
      POSTGRES_USER: securecomm_user
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    volumes:
      - postgres_data:/var/lib/postgresql/data

  redis:
    image: redis:7-alpine
    command: redis-server --requirepass ${REDIS_PASSWORD}

  backend:
    build: .
    command: gunicorn config.wsgi:application --bind 0.0.0.0:8000
    environment:
      - DATABASE_URL=postgresql://securecomm_user:${DB_PASSWORD}@db:5432/securecomm_db
      - REDIS_URL=redis://:${REDIS_PASSWORD}@redis:6379/0
    depends_on:
      - db
      - redis

  socketio:
    build: .
    command: python manage.py run_socketio
    depends_on:
      - redis
      - backend

  nginx:
    image: nginx:alpine
    ports:
      - "443:443"
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
      - ./ssl:/etc/nginx/ssl
    depends_on:
      - backend

volumes:
  postgres_data:
```

## Testing

### API Testing with cURL

```bash
# Login
curl -X POST http://localhost:8000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "serviceNumber": "DEF12345",
    "password": "SecurePass123!",
    "deviceId": "device-123"
  }'

# Get Groups
curl -X GET http://localhost:8000/api/groups \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN"
```

## Monitoring & Logging

### Log Levels
- **INFO**: Normal operations
- **WARN**: Potential issues
- **ERROR**: Errors that need attention
- **CRITICAL**: System failures

### Metrics to Monitor
- Active VPN connections
- Message throughput
- Failed login attempts
- API response times
- Database query performance

## Support

For backend setup issues, contact: backend-support@securecomm-defence.mil

## License

Classified - Defence Use Only
