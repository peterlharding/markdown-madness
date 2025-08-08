# Complete FastAPI Authentication Stack Guide

## Overview

This guide covers self-hosted authentication solutions for FastAPI, providing alternatives to managed services like Auth0. These solutions give you complete control over user data, authentication flows, and customization while maintaining the performance benefits of FastAPI.

## Authentication Framework Options

### 1. FastAPI Users (Recommended - Flask-AppBuilder Equivalent)

**FastAPI Users** is the most comprehensive authentication solution for FastAPI, providing Flask-AppBuilder-like functionality.

#### Features:
- ✅ User registration/authentication
- ✅ Role-based access control (RBAC)
- ✅ JWT & Cookie authentication
- ✅ OAuth providers (Google, GitHub, etc.)
- ✅ Password reset & email verification
- ✅ Database integration (SQLAlchemy/SQLModel)
- ✅ Async support

#### Installation:
```bash
pip install fastapi-users[sqlalchemy,oauth]
```

### 2. FastAPI Admin

**FastAPI Admin** provides Django-style admin interface for CRUD operations.

#### Features:
- ✅ Auto-generated admin interface
- ✅ CRUD operations via web UI
- ✅ Role-based permissions
- ✅ Dashboard and reporting

#### Installation:
```bash
pip install fastapi-admin
```

### 3. Authx (Lightweight)

**Authx** is a simpler authentication library for basic JWT functionality.

#### Installation:
```bash
pip install authx
```

### 4. Starlette Admin (Modern UI)

**Starlette Admin** provides a modern admin interface similar to Django Admin.

#### Installation:
```bash
pip install starlette-admin
```

## Complete FastAPI Users Implementation

### Project Structure
```
fastapi-auth-app/
├── app/
│   ├── __init__.py
│   ├── main.py
│   ├── models.py
│   ├── schemas.py
│   ├── database.py
│   ├── auth.py
│   └── config.py
├── requirements.txt
└── alembic/
```

### Dependencies (requirements.txt)
```txt
fastapi
fastapi-users[sqlalchemy,oauth]
sqlmodel
alembic
uvicorn[standard]
python-multipart
python-jose[cryptography]
passlib[bcrypt]
redis
celery
```

### Configuration
```python
# config.py
import os
from typing import Optional

DATABASE_URL = os.getenv("DATABASE_URL", "postgresql://user:pass@localhost/db")
SECRET_KEY = os.getenv("SECRET_KEY", "your-secret-key-change-in-production")
REDIS_URL = os.getenv("REDIS_URL", "redis://localhost:6379")

# OAuth Configuration (optional)
GOOGLE_CLIENT_ID: Optional[str] = os.getenv("GOOGLE_CLIENT_ID")
GOOGLE_CLIENT_SECRET: Optional[str] = os.getenv("GOOGLE_CLIENT_SECRET")

# JWT Configuration
JWT_LIFETIME_SECONDS = 3600
COOKIE_MAX_AGE = 3600
```

### Database Models
```python
# models.py
import uuid
from typing import Optional
from fastapi_users.db import SQLAlchemyBaseUserTableUUID
from fastapi_users_db_sqlalchemy import SQLAlchemyUserDatabase
from sqlalchemy import Boolean, Column, String, DateTime, ForeignKey
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import relationship
from sqlalchemy.dialects.postgresql import UUID

Base = declarative_base()

class User(SQLAlchemyBaseUserTableUUID, Base):
    """User model with additional fields"""
    __tablename__ = "users"
    
    first_name: Optional[str] = Column(String, nullable=True)
    last_name: Optional[str] = Column(String, nullable=True)
    organization: Optional[str] = Column(String, nullable=True)
    department: Optional[str] = Column(String, nullable=True)
    phone: Optional[str] = Column(String, nullable=True)
    
    # Relationships
    roles = relationship("UserRole", back_populates="user")

class Role(Base):
    """Role model for RBAC"""
    __tablename__ = "roles"
    
    id = Column(UUID(as_uuid=True), primary_key=True, default=uuid.uuid4)
    name = Column(String(50), unique=True, nullable=False)
    description = Column(String(200))
    
    # Relationships
    users = relationship("UserRole", back_populates="role")
    permissions = relationship("RolePermission", back_populates="role")

class Permission(Base):
    """Permission model for fine-grained access control"""
    __tablename__ = "permissions"
    
    id = Column(UUID(as_uuid=True), primary_key=True, default=uuid.uuid4)
    name = Column(String(50), unique=True, nullable=False)
    resource = Column(String(50), nullable=False)  # e.g., 'dashboard', 'users'
    action = Column(String(20), nullable=False)    # e.g., 'read', 'write', 'delete'
    
    # Relationships
    roles = relationship("RolePermission", back_populates="permission")

class UserRole(Base):
    """Many-to-many relationship between users and roles"""
    __tablename__ = "user_roles"
    
    user_id = Column(UUID(as_uuid=True), ForeignKey("users.id"), primary_key=True)
    role_id = Column(UUID(as_uuid=True), ForeignKey("roles.id"), primary_key=True)
    
    # Relationships
    user = relationship("User", back_populates="roles")
    role = relationship("Role", back_populates="users")

class RolePermission(Base):
    """Many-to-many relationship between roles and permissions"""
    __tablename__ = "role_permissions"
    
    role_id = Column(UUID(as_uuid=True), ForeignKey("roles.id"), primary_key=True)
    permission_id = Column(UUID(as_uuid=True), ForeignKey("permissions.id"), primary_key=True)
    
    # Relationships
    role = relationship("Role", back_populates="permissions")
    permission = relationship("Permission", back_populates="roles")
```

### Database Setup
```python
# database.py
from sqlalchemy import create_engine
from sqlalchemy.ext.asyncio import AsyncSession, create_async_engine
from sqlalchemy.orm import sessionmaker
from config import DATABASE_URL

# Async database setup
engine = create_async_engine(DATABASE_URL)
async_session_maker = sessionmaker(engine, class_=AsyncSession, expire_on_commit=False)

async def get_async_session() -> AsyncSession:
    async with async_session_maker() as session:
        yield session

# User database for FastAPI Users
async def get_user_db(session: AsyncSession = Depends(get_async_session)):
    yield SQLAlchemyUserDatabase(session, User)
```

### Pydantic Schemas
```python
# schemas.py
import uuid
from typing import Optional, List
from fastapi_users import schemas
from pydantic import BaseModel

class UserRead(schemas.BaseUser[uuid.UUID]):
    """Schema for reading user data"""
    first_name: Optional[str]
    last_name: Optional[str]
    organization: Optional[str]
    department: Optional[str]
    phone: Optional[str]

class UserCreate(schemas.BaseUserCreate):
    """Schema for creating users"""
    first_name: Optional[str]
    last_name: Optional[str]
    organization: Optional[str]
    department: Optional[str]
    phone: Optional[str]

class UserUpdate(schemas.BaseUserUpdate):
    """Schema for updating users"""
    first_name: Optional[str]
    last_name: Optional[str]
    organization: Optional[str]
    department: Optional[str]
    phone: Optional[str]

class RoleBase(BaseModel):
    name: str
    description: Optional[str]

class RoleCreate(RoleBase):
    pass

class RoleRead(RoleBase):
    id: uuid.UUID
    
    class Config:
        from_attributes = True

class PermissionBase(BaseModel):
    name: str
    resource: str
    action: str

class PermissionCreate(PermissionBase):
    pass

class PermissionRead(PermissionBase):
    id: uuid.UUID
    
    class Config:
        from_attributes = True
```

### Authentication Setup
```python
# auth.py
import uuid
from typing import Optional
from fastapi import Depends, Request
from fastapi_users import BaseUserManager, FastAPIUsers, UUIDIDMixin
from fastapi_users.authentication import (
    AuthenticationBackend,
    BearerTransport,
    CookieTransport,
    JWTStrategy,
)
from fastapi_users.db import SQLAlchemyUserDatabase
from models import User
from config import SECRET_KEY, JWT_LIFETIME_SECONDS, COOKIE_MAX_AGE

class UserManager(UUIDIDMixin, BaseUserManager[User, uuid.UUID]):
    """Custom user manager with additional functionality"""
    reset_password_token_secret = SECRET_KEY
    verification_token_secret = SECRET_KEY

    async def on_after_register(self, user: User, request: Optional[Request] = None):
        print(f"User {user.id} has registered.")

    async def on_after_forgot_password(
        self, user: User, token: str, request: Optional[Request] = None
    ):
        print(f"User {user.id} has forgot their password. Reset token: {token}")

    async def on_after_request_verify(
        self, user: User, token: str, request: Optional[Request] = None
    ):
        print(f"Verification requested for user {user.id}. Verification token: {token}")

async def get_user_manager(user_db: SQLAlchemyUserDatabase = Depends(get_user_db)):
    yield UserManager(user_db)

# Authentication backends
bearer_transport = BearerTransport(tokenUrl="auth/jwt/login")
cookie_transport = CookieTransport(cookie_max_age=COOKIE_MAX_AGE)

def get_jwt_strategy() -> JWTStrategy:
    return JWTStrategy(secret=SECRET_KEY, lifetime_seconds=JWT_LIFETIME_SECONDS)

# JWT backend
jwt_backend = AuthenticationBackend(
    name="jwt",
    transport=bearer_transport,
    get_strategy=get_jwt_strategy,
)

# Cookie backend
cookie_backend = AuthenticationBackend(
    name="cookie",
    transport=cookie_transport,
    get_strategy=get_jwt_strategy,
)

# FastAPI Users instance
fastapi_users = FastAPIUsers[User, uuid.UUID](
    get_user_manager,
    [jwt_backend, cookie_backend],
)

# Dependencies for route protection
current_active_user = fastapi_users.current_user(active=True)
current_superuser = fastapi_users.current_user(active=True, superuser=True)
```

### Role-Based Access Control
```python
# rbac.py
from typing import List
from fastapi import Depends, HTTPException
from sqlalchemy.ext.asyncio import AsyncSession
from sqlalchemy.orm import selectinload
from sqlalchemy import select
from models import User, UserRole, Role, RolePermission, Permission
from database import get_async_session

async def get_user_roles(
    user: User = Depends(current_active_user),
    session: AsyncSession = Depends(get_async_session)
) -> List[str]:
    """Get user's roles"""
    stmt = select(Role.name).join(UserRole).where(UserRole.user_id == user.id)
    result = await session.execute(stmt)
    return [role[0] for role in result.fetchall()]

async def get_user_permissions(
    user: User = Depends(current_active_user),
    session: AsyncSession = Depends(get_async_session)
) -> List[str]:
    """Get user's permissions through roles"""
    stmt = (
        select(Permission.name)
        .join(RolePermission)
        .join(Role)
        .join(UserRole)
        .where(UserRole.user_id == user.id)
    )
    result = await session.execute(stmt)
    return [perm[0] for perm in result.fetchall()]

def require_roles(required_roles: List[str]):
    """Decorator to require specific roles"""
    async def role_checker(
        user_roles: List[str] = Depends(get_user_roles)
    ):
        if not any(role in user_roles for role in required_roles):
            raise HTTPException(
                status_code=403,
                detail=f"Requires one of these roles: {required_roles}"
            )
        return True
    return role_checker

def require_permissions(required_permissions: List[str]):
    """Decorator to require specific permissions"""
    async def permission_checker(
        user_permissions: List[str] = Depends(get_user_permissions)
    ):
        missing_permissions = set(required_permissions) - set(user_permissions)
        if missing_permissions:
            raise HTTPException(
                status_code=403,
                detail=f"Missing permissions: {list(missing_permissions)}"
            )
        return True
    return permission_checker
```

### Main Application
```python
# main.py
from fastapi import FastAPI, Depends
from fastapi.middleware.cors import CORSMiddleware
from auth import fastapi_users, jwt_backend, cookie_backend
from schemas import UserRead, UserCreate, UserUpdate
from rbac import require_roles, require_permissions

app = FastAPI(title="FastAPI Authentication App")

# CORS
app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:3000"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# Authentication routes
app.include_router(
    fastapi_users.get_auth_router(jwt_backend),
    prefix="/auth/jwt",
    tags=["auth"],
)

app.include_router(
    fastapi_users.get_auth_router(cookie_backend),
    prefix="/auth/cookie",
    tags=["auth"],
)

app.include_router(
    fastapi_users.get_register_router(UserRead, UserCreate),
    prefix="/auth",
    tags=["auth"],
)

app.include_router(
    fastapi_users.get_users_router(UserRead, UserUpdate),
    prefix="/users",
    tags=["users"],
)

app.include_router(
    fastapi_users.get_reset_password_router(),
    prefix="/auth",
    tags=["auth"],
)

app.include_router(
    fastapi_users.get_verify_router(UserRead),
    prefix="/auth",
    tags=["auth"],
)

# Protected routes
@app.get("/")
async def public_route():
    return {"message": "This is a public endpoint"}

@app.get("/protected")
async def protected_route(user: User = Depends(current_active_user)):
    return {
        "message": "This is a protected endpoint",
        "user_id": str(user.id),
        "email": user.email
    }

@app.get("/admin")
async def admin_route(
    user: User = Depends(current_active_user),
    _: bool = Depends(require_roles(["admin"]))
):
    return {"message": "Admin access granted", "user": user.email}

@app.get("/dashboard")
async def dashboard_route(
    user: User = Depends(current_active_user),
    _: bool = Depends(require_permissions(["dashboard:read"]))
):
    return {"message": "Dashboard access granted"}

@app.get("/users/manage")
async def manage_users(
    user: User = Depends(current_active_user),
    _: bool = Depends(require_permissions(["users:write"]))
):
    return {"message": "User management access granted"}
```

### OAuth Integration (Optional)
```python
# oauth.py - Add to auth.py
from httpx_oauth.clients.google import GoogleOAuth2

google_oauth_client = GoogleOAuth2(
    GOOGLE_CLIENT_ID,
    GOOGLE_CLIENT_SECRET
)

# Add to main.py
from fastapi_users.authentication import AuthenticationBackend
from auth import google_oauth_client

app.include_router(
    fastapi_users.get_oauth_router(
        google_oauth_client,
        jwt_backend,
        SECRET_KEY,
        associate_by_email=True,
    ),
    prefix="/auth/google",
    tags=["auth"],
)
```

## Admin Interface Integration

### FastAPI Admin Setup
```python
# admin.py
from fastapi_admin.app import app as admin_app
from fastapi_admin.providers.login import UsernamePasswordProvider
from fastapi_admin.resources import Model, Field

# Admin resources
class UserResource(Model):
    label = "User"
    model = User
    icon = "fas fa-users"
    fields = [
        Field(name="id", label="ID"),
        Field(name="email", label="Email"),
        Field(name="first_name", label="First Name"),
        Field(name="last_name", label="Last Name"),
        Field(name="is_active", label="Active"),
        Field(name="is_superuser", label="Superuser"),
    ]

# Mount admin
app.mount('/admin', admin_app)
```

## Database Migration with Alembic

### Alembic Setup
```bash
# Initialize Alembic
alembic init alembic

# Create migration
alembic revision --autogenerate -m "Create user and role tables"

# Apply migration
alembic upgrade head
```

### Migration Script Example
```python
# alembic/versions/xxx_create_user_tables.py
from alembic import op
import sqlalchemy as sa
import fastapi_users_db_sqlalchemy

def upgrade():
    # Create users table
    op.create_table('users',
        sa.Column('id', fastapi_users_db_sqlalchemy.generics.GUID(), nullable=False),
        sa.Column('email', sa.String(length=320), nullable=False),
        sa.Column('hashed_password', sa.String(length=1024), nullable=False),
        sa.Column('is_active', sa.Boolean(), nullable=False, default=True),
        sa.Column('is_superuser', sa.Boolean(), nullable=False, default=False),
        sa.Column('is_verified', sa.Boolean(), nullable=False, default=False),
        sa.Column('first_name', sa.String(), nullable=True),
        sa.Column('last_name', sa.String(), nullable=True),
        sa.PrimaryKeyConstraint('id')
    )
    
    # Create roles and permissions tables
    # ... additional table creation code
```

## CLI Management Commands

### Create Admin User Script
```python
# cli.py
import asyncio
import uuid
from fastapi_users.exceptions import UserAlreadyExists
from auth import get_user_manager
from database import get_user_db, get_async_session
from schemas import UserCreate

async def create_admin_user():
    """Create admin user - equivalent to 'superset fab create-admin'"""
    try:
        async for session in get_async_session():
            async for user_db in get_user_db(session):
                async for user_manager in get_user_manager(user_db):
                    user = await user_manager.create(
                        UserCreate(
                            email="admin@example.com",
                            password="admin",
                            is_superuser=True,
                            is_verified=True,
                            first_name="Admin",
                            last_name="User"
                        )
                    )
                    print(f"Admin user created: {user.email}")
                    break
                break
            break
    except UserAlreadyExists:
        print("Admin user already exists")

if __name__ == "__main__":
    asyncio.run(create_admin_user())
```

### Usage Commands
```bash
# Create admin user
python cli.py

# Run the application
uvicorn main:app --reload

# Run with Gunicorn (production)
gunicorn main:app -w 4 -k uvicorn.workers.UvicornWorker
```

## Testing

### Test Setup
```python
# test_auth.py
import pytest
from httpx import AsyncClient
from fastapi.testclient import TestClient
from main import app

@pytest.mark.asyncio
async def test_register_user():
    async with AsyncClient(app=app, base_url="http://test") as ac:
        response = await ac.post("/auth/register", json={
            "email": "test@example.com",
            "password": "testpass123",
            "first_name": "Test",
            "last_name": "User"
        })
    assert response.status_code == 201

@pytest.mark.asyncio
async def test_login():
    async with AsyncClient(app=app, base_url="http://test") as ac:
        response = await ac.post("/auth/jwt/login", data={
            "username": "test@example.com",
            "password": "testpass123"
        })
    assert response.status_code == 200
    assert "access_token" in response.json()

@pytest.mark.asyncio
async def test_protected_route():
    # Login first to get token
    async with AsyncClient(app=app, base_url="http://test") as ac:
        login_response = await ac.post("/auth/jwt/login", data={
            "username": "admin@example.com",
            "password": "admin"
        })
        token = login_response.json()["access_token"]
        
        # Access protected route
        response = await ac.get("/protected", headers={
            "Authorization": f"Bearer {token}"
        })
    assert response.status_code == 200
```

## Deployment

### Docker Setup
```dockerfile
# Dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

### Docker Compose
```yaml
# docker-compose.yml
version: '3.8'

services:
  postgres:
    image: postgres:15
    environment:
      POSTGRES_DB: fastapi_auth
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: password
    volumes:
      - postgres_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"

  app:
    build: .
    ports:
      - "8000:8000"
    environment:
      DATABASE_URL: postgresql://postgres:password@postgres:5432/fastapi_auth
      REDIS_URL: redis://redis:6379
      SECRET_KEY: your-secret-key-change-in-production
    depends_on:
      - postgres
      - redis

volumes:
  postgres_data:
```

## Production Considerations

### Security Best Practices
- Use strong SECRET_KEY in production
- Enable HTTPS/TLS
- Set appropriate CORS origins
- Use environment variables for sensitive config
- Implement rate limiting
- Add request validation
- Use secure session configuration

### Performance Optimization
- Use connection pooling
- Implement caching (Redis)
- Add database indexing
- Use async database drivers
- Implement background tasks with Celery

### Monitoring
- Add logging configuration
- Implement health checks
- Monitor authentication failures
- Track user activity
- Set up error tracking (Sentry)

## Framework Comparison

| Feature | FastAPI Users | Auth0 + FastAPI | Flask-AppBuilder |
|---------|---------------|-----------------|------------------|
| **User Management** | Full control | Auth0 Dashboard | Built-in UI |
| **OAuth Providers** | Manual setup | 30+ providers | Manual setup |
| **Customization** | Full control | Limited | Full control |
| **Database Control** | Complete | None | Complete |
| **Cost** | Free | Pay per user | Free |
| **Complexity** | Medium | Medium | Low |
| **Async Support** | Native | Native | Limited |
| **Modern Stack** | FastAPI | FastAPI | Flask |

## Getting Started Checklist

- [ ] Install dependencies: `pip install fastapi-users[sqlalchemy,oauth]`
- [ ] Set up database models and migrations
- [ ] Configure authentication backends (JWT/Cookie)
- [ ] Implement RBAC system
- [ ] Create admin user: `python cli.py`
- [ ] Test authentication endpoints
- [ ] Add admin interface (optional)
- [ ] Configure OAuth providers (optional)
- [ ] Set up production deployment
- [ ] Implement monitoring and logging

This stack provides enterprise-grade authentication with complete control over your data and authentication flows, making it ideal for applications requiring customization and compliance.