# FastAPI + Auth0 Integration Guide

## Overview

Auth0 + FastAPI provides a powerful combination for building secure APIs with enterprise-grade authentication. Auth0 handles all authentication complexity as a managed service, while FastAPI provides high-performance API development.

## Key Benefits

- **Identity-as-a-Service**: Offload user management to Auth0
- **Enterprise Features**: SSO, MFA, compliance (SOC2, GDPR) out of the box
- **30+ OAuth Providers**: Google, GitHub, Microsoft, etc.
- **Scalability**: Infinite scale without infrastructure management
- **Security**: Industry-standard JWT tokens with RS256 encryption

## Installation

```bash
pip install fastapi python-jose[cryptography] python-multipart requests uvicorn
```

## Environment Configuration

```bash
# .env file
AUTH0_DOMAIN=your-tenant.auth0.com
AUTH0_CLIENT_ID=your-client-id
AUTH0_CLIENT_SECRET=your-client-secret
AUTH0_AUDIENCE=https://your-api.com
```

## Configuration Setup

```python
# config.py
import os

AUTH0_DOMAIN = os.getenv("AUTH0_DOMAIN", "your-tenant.auth0.com")
AUTH0_CLIENT_ID = os.getenv("AUTH0_CLIENT_ID")
AUTH0_CLIENT_SECRET = os.getenv("AUTH0_CLIENT_SECRET")
AUTH0_AUDIENCE = os.getenv("AUTH0_AUDIENCE", "https://your-api.com")
AUTH0_ALGORITHMS = ["RS256"]
```

## JWT Token Verification

```python
# auth.py
from fastapi import Depends, HTTPException, status
from fastapi.security import HTTPBearer
from jose import JWTError, jwt
import requests
from functools import lru_cache

security = HTTPBearer()

@lru_cache()
def get_auth0_public_key():
    """Get Auth0's public key for JWT verification"""
    response = requests.get(f"https://{AUTH0_DOMAIN}/.well-known/jwks.json")
    jwks = response.json()
    return jwks

def verify_token(token: str = Depends(security)):
    """Verify Auth0 JWT token"""
    try:
        # Get the key
        jwks = get_auth0_public_key()
        unverified_header = jwt.get_unverified_header(token.credentials)
        
        # Find the correct key
        rsa_key = {}
        for key in jwks["keys"]:
            if key["kid"] == unverified_header["kid"]:
                rsa_key = {
                    "kty": key["kty"],
                    "kid": key["kid"],
                    "use": key["use"],
                    "n": key["n"],
                    "e": key["e"]
                }
        
        if rsa_key:
            # Verify the token
            payload = jwt.decode(
                token.credentials,
                rsa_key,
                algorithms=AUTH0_ALGORITHMS,
                audience=AUTH0_AUDIENCE,
                issuer=f"https://{AUTH0_DOMAIN}/"
            )
            return payload
        
    except JWTError:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Could not validate credentials",
            headers={"WWW-Authenticate": "Bearer"},
        )
    
    raise HTTPException(
        status_code=status.HTTP_401_UNAUTHORIZED,
        detail="Could not validate credentials",
        headers={"WWW-Authenticate": "Bearer"},
    )
```

## Role-Based Access Control (RBAC)

```python
# rbac.py
from fastapi import Depends, HTTPException
from typing import List

def require_permissions(required_permissions: List[str]):
    """Decorator for permission-based access control"""
    def permission_checker(token_payload = Depends(verify_token)):
        user_permissions = token_payload.get("permissions", [])
        
        for permission in required_permissions:
            if permission not in user_permissions:
                raise HTTPException(
                    status_code=403,
                    detail=f"Missing required permission: {permission}"
                )
        
        return token_payload
    
    return permission_checker

def require_role(required_role: str):
    """Decorator for role-based access control"""
    def role_checker(token_payload = Depends(verify_token)):
        user_roles = token_payload.get(f"{AUTH0_AUDIENCE}/roles", [])
        
        if required_role not in user_roles:
            raise HTTPException(
                status_code=403,
                detail=f"Missing required role: {required_role}"
            )
        
        return token_payload
    
    return role_checker
```

## FastAPI Application Example

```python
# main.py
from fastapi import FastAPI, Depends
from fastapi.middleware.cors import CORSMiddleware

app = FastAPI(title="Auth0 + FastAPI App")

# CORS for frontend integration
app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:3000"],  # React app
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# Public route
@app.get("/")
async def public_route():
    return {"message": "This is a public endpoint"}

# Protected route - requires valid JWT
@app.get("/protected")
async def protected_route(token_payload = Depends(verify_token)):
    return {
        "message": "This is a protected endpoint",
        "user": token_payload.get("sub"),
        "email": token_payload.get("email")
    }

# Admin-only route - requires admin role
@app.get("/admin")
async def admin_route(token_payload = Depends(require_role("admin"))):
    return {
        "message": "Admin access granted",
        "user": token_payload
    }

# Permission-based route - requires specific permission
@app.get("/dashboard")
async def dashboard_route(
    token_payload = Depends(require_permissions(["read:dashboard"]))
):
    return {"message": "Dashboard access granted"}

# User profile endpoint
@app.get("/profile")
async def get_profile(token_payload = Depends(verify_token)):
    return {
        "user_id": token_payload.get("sub"),
        "email": token_payload.get("email"),
        "name": token_payload.get("name"),
        "roles": token_payload.get(f"{AUTH0_AUDIENCE}/roles", []),
        "permissions": token_payload.get("permissions", [])
    }
```

## Frontend Integration (React)

```javascript
// Frontend integration with Auth0 React SDK
import { useAuth0 } from "@auth0/auth0-react";

function ApiCall() {
    const { getAccessTokenSilently } = useAuth0();
    
    const callProtectedApi = async () => {
        try {
            const token = await getAccessTokenSilently({
                authorizationParams: {
                    audience: "https://your-api.com",
                }
            });
            
            const response = await fetch("http://localhost:8000/protected", {
                headers: {
                    Authorization: `Bearer ${token}`,
                },
            });
            
            const data = await response.json();
            console.log(data);
        } catch (error) {
            console.error(error);
        }
    };
    
    return <button onClick={callProtectedApi}>Call Protected API</button>;
}
```

## Machine-to-Machine Authentication

For API-to-API communication:

```python
import requests

def get_m2m_token():
    """Get machine-to-machine token for backend services"""
    response = requests.post(
        f"https://{AUTH0_DOMAIN}/oauth/token",
        json={
            "client_id": AUTH0_M2M_CLIENT_ID,
            "client_secret": AUTH0_M2M_CLIENT_SECRET,
            "audience": AUTH0_AUDIENCE,
            "grant_type": "client_credentials"
        }
    )
    return response.json()["access_token"]

# Use the token for service-to-service calls
async def call_protected_service():
    token = get_m2m_token()
    headers = {"Authorization": f"Bearer {token}"}
    response = requests.get("https://api.example.com/data", headers=headers)
    return response.json()
```

## Custom Claims and Metadata

```python
def add_custom_claims(token_payload = Depends(verify_token)):
    """Extract custom claims from Auth0 token"""
    return {
        "user_id": token_payload.get("sub"),
        "email": token_payload.get("email"),
        "organization": token_payload.get(f"{AUTH0_AUDIENCE}/organization"),
        "department": token_payload.get(f"{AUTH0_AUDIENCE}/department"),
        "is_admin": "admin" in token_payload.get(f"{AUTH0_AUDIENCE}/roles", [])
    }
```

## Testing Protected Endpoints

```bash
# Get token from Auth0 (or frontend)
export TOKEN="eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiJ9..."

# Test protected endpoint
curl -H "Authorization: Bearer $TOKEN" \
     http://localhost:8000/protected

# Test with HTTPie
http GET localhost:8000/protected \
     Authorization:"Bearer $TOKEN"
```

## Error Handling

```python
from fastapi import HTTPException
from jose import JWTError

def handle_auth_errors():
    """Common authentication error patterns"""
    try:
        # Token verification logic
        pass
    except JWTError as e:
        if "expired" in str(e):
            raise HTTPException(401, "Token has expired")
        elif "invalid" in str(e):
            raise HTTPException(401, "Invalid token")
        else:
            raise HTTPException(401, "Authentication failed")
    except Exception as e:
        raise HTTPException(500, "Authentication service unavailable")
```

## Deployment Considerations

### Environment Variables
```bash
# Production environment
AUTH0_DOMAIN=production.auth0.com
AUTH0_AUDIENCE=https://api.production.com
AUTH0_ALGORITHMS=["RS256"]

# Development environment  
AUTH0_DOMAIN=dev.auth0.com
AUTH0_AUDIENCE=https://api.dev.com
```

### Security Best Practices
- Always use HTTPS in production
- Set appropriate CORS origins
- Use RS256 algorithm (not HS256)
- Implement proper error handling
- Cache Auth0 public keys
- Set reasonable token expiration times

## Comparison with Alternatives

| Feature | Auth0 + FastAPI | FastAPI Users | Flask-AppBuilder |
|---------|-----------------|---------------|------------------|
| **User Management** | Auth0 Dashboard | Code/Database | Built-in UI |
| **OAuth Providers** | 30+ providers | Manual setup | Manual setup |
| **Scalability** | Infinite | Your infrastructure | Your infrastructure |
| **Setup Complexity** | Medium | Low | Low |
| **Cost** | Pay per user | Free | Free |
| **Customization** | Limited | Full control | Full control |
| **Compliance** | SOC2, GDPR included | Your responsibility | Your responsibility |
| **Multi-tenancy** | Built-in | Manual implementation | Manual implementation |

## When to Choose Auth0 + FastAPI

### ✅ Choose Auth0 when:
- Building production applications requiring enterprise features
- Need multiple OAuth providers out of the box
- Want to focus on business logic, not authentication
- Building multi-tenant applications
- Need global scale and reliability
- Compliance requirements (SOC2, GDPR, HIPAA)

### ❌ Avoid Auth0 when:
- Budget constraints (pay-per-user model)
- Need heavy customization of authentication flows
- Want complete control over user data storage
- Building simple internal tools
- Have specific regulatory requirements for data locality

## Getting Started

1. **Create Auth0 Account**: Sign up at [auth0.com](https://auth0.com)
2. **Create Application**: Set up a Single Page Application
3. **Configure API**: Create an API in Auth0 dashboard
4. **Set Environment Variables**: Configure your FastAPI app
5. **Implement Authentication**: Add the verification middleware
6. **Test**: Use Auth0's test token or frontend integration

## Resources

- [Auth0 FastAPI Quickstart](https://auth0.com/docs/quickstart/backend/python)
- [FastAPI Security Documentation](https://fastapi.tiangolo.com/tutorial/security/)
- [Auth0 Python SDK](https://github.com/auth0/auth0-python)
- [JWT.io Debugger](https://jwt.io/) - for token debugging

Auth0 + FastAPI provides enterprise-grade authentication with minimal development overhead, making it ideal for production applications requiring robust security and scalability.