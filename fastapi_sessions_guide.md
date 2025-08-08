# FastAPI Sessions Implementation Guide

This guide covers various methods to implement user sessions in FastAPI applications.

## 1. Starlette's Built-in SessionMiddleware

FastAPI inherits from Starlette, providing built-in session support for simple applications.

```python
from fastapi import FastAPI, Request
from starlette.middleware.sessions import SessionMiddleware

app = FastAPI()

# Add session middleware with a secret key
app.add_middleware(SessionMiddleware, secret_key="your-secret-key-here")

@app.post("/login")
async def login(request: Request, username: str):
    # Set session data
    request.session["user_id"] = 123
    request.session["username"] = username
    return {"message": "Logged in successfully"}

@app.get("/profile")
async def get_profile(request: Request):
    # Get session data
    user_id = request.session.get("user_id")
    if not user_id:
        return {"error": "Not logged in"}
    return {"user_id": user_id, "username": request.session.get("username")}

@app.post("/logout")
async def logout(request: Request):
    # Clear session
    request.session.clear()
    return {"message": "Logged out successfully"}
```

## 2. Redis for Server-Side Sessions

For production applications requiring scalability and persistence.

```python
from fastapi import FastAPI, Request, Depends
import redis
import json
import uuid

app = FastAPI()
redis_client = redis.Redis(host='localhost', port=6379, db=0)

async def get_session(request: Request):
    session_id = request.cookies.get("session_id")
    if not session_id:
        return {}
    
    session_data = redis_client.get(f"session:{session_id}")
    if session_data:
        return json.loads(session_data)
    return {}

@app.post("/login")
async def login(request: Request, username: str):
    session_id = str(uuid.uuid4())
    session_data = {"user_id": 123, "username": username}
    
    # Store in Redis with expiration (1 hour)
    redis_client.setex(f"session:{session_id}", 3600, json.dumps(session_data))
    
    response = {"message": "Logged in successfully"}
    # Set cookie (use secure=True, httponly=True in production)
    response = JSONResponse(content=response)
    response.set_cookie("session_id", session_id, httponly=True)
    return response

@app.get("/profile")
async def get_profile(session_data: dict = Depends(get_session)):
    if not session_data.get("user_id"):
        return {"error": "Not logged in"}
    return session_data
```

## 3. FastAPI Sessions Library

Install the dedicated library: `pip install fastapi-sessions`

```python
from fastapi import FastAPI, Depends
from fastapi_sessions.frontends.implementations import SessionCookie, CookieParameters
from fastapi_sessions.backends.implementations import InMemoryBackend
from fastapi_sessions.session_verifier import SessionVerifier
from pydantic import BaseModel
import uuid

app = FastAPI()

# Session configuration
cookie_params = CookieParameters()
frontend = SessionCookie(
    cookie_name="session",
    identifier="general_verifier",
    auto_error=True,
    secret_key="your-secret-key",
    cookie_params=cookie_params,
)
backend = InMemoryBackend[uuid.UUID, dict]()

class SessionData(BaseModel):
    user_id: int
    username: str

verifier = SessionVerifier[SessionCookie, dict](
    identifier="general_verifier",
    auto_error=True,
    backend=backend,
    auth_http_exception=HTTPException(status_code=403, detail="Invalid session"),
)

@app.post("/login")
async def login(username: str):
    session = uuid.uuid4()
    data = {"user_id": 123, "username": username}
    
    await backend.create(session, data)
    cookie = frontend.create_session_cookie(session)
    
    response = RedirectResponse(url="/profile", status_code=302)
    response.headers.update(cookie)
    return response

@app.get("/profile")
async def get_profile(session_data: dict = Depends(verifier)):
    return session_data
```

## 4. Custom Database Session Implementation

For applications requiring persistent session storage with full control.

```python
from fastapi import FastAPI, Request, Response, Depends, HTTPException
from sqlalchemy.orm import Session
import secrets
from datetime import datetime, timedelta

app = FastAPI()

# Database session manager
class SessionDB:
    def __init__(self, db: Session):
        self.db = db
    
    def create_session(self, user_id: int) -> str:
        session_token = secrets.token_urlsafe(32)
        expires_at = datetime.utcnow() + timedelta(days=7)
        
        # Save to database (pseudo-code)
        # session = SessionModel(token=session_token, user_id=user_id, expires_at=expires_at)
        # self.db.add(session)
        # self.db.commit()
        
        return session_token
    
    def get_session(self, token: str) -> dict:
        # Get from database (pseudo-code)
        # session = self.db.query(SessionModel).filter(SessionModel.token == token).first()
        # if session and session.expires_at > datetime.utcnow():
        #     return {"user_id": session.user_id}
        return None

def get_current_user(request: Request, db: Session = Depends(get_db)):
    session_token = request.cookies.get("session_token")
    if not session_token:
        raise HTTPException(status_code=401, detail="Not authenticated")
    
    session_db = SessionDB(db)
    session_data = session_db.get_session(session_token)
    if not session_data:
        raise HTTPException(status_code=401, detail="Invalid session")
    
    return session_data

@app.post("/login")
async def login(response: Response, username: str, db: Session = Depends(get_db)):
    # Validate user credentials here
    user_id = 123  # Get from your user authentication
    
    session_db = SessionDB(db)
    session_token = session_db.create_session(user_id)
    
    response.set_cookie(
        "session_token", 
        session_token, 
        httponly=True, 
        secure=True,  # Use in production with HTTPS
        samesite="lax"
    )
    return {"message": "Logged in successfully"}
```

## Security Best Practices

### Secure Cookie Configuration

```python
# Production-ready cookie settings
response.set_cookie(
    "session_id",
    session_token,
    httponly=True,     # Prevents XSS attacks
    secure=True,       # HTTPS only
    samesite="strict", # CSRF protection
    max_age=3600,      # 1 hour expiration
    domain="yourdomain.com",  # Limit to your domain
)
```

### Security Checklist

- **Strong Secret Keys**: Use cryptographically secure random keys
- **Key Rotation**: Regularly rotate session secret keys
- **Session Expiration**: Implement both absolute and idle timeouts
- **HTTPS Only**: Always use `secure=True` in production
- **XSS Protection**: Use `httponly=True` to prevent JavaScript access
- **CSRF Protection**: Set `samesite="strict"` or `samesite="lax"`
- **Session Invalidation**: Provide logout functionality that clears sessions

## Method Comparison

| Method | Use Case | Pros | Cons |
|--------|----------|------|------|
| **Starlette Middleware** | Simple apps, prototyping | Easy setup, built-in | Client-side storage, limited scalability |
| **Redis Sessions** | Production apps, microservices | Fast, scalable, persistent | Requires Redis infrastructure |
| **FastAPI Sessions** | Feature-rich applications | Type safety, flexible backends | Additional dependency |
| **Database Sessions** | Enterprise applications | Full control, audit trails | More complex implementation |

## Recommendations

- **Development**: Start with Starlette's SessionMiddleware
- **Production**: Use Redis or database-backed sessions
- **High Traffic**: Redis with clustering for horizontal scaling
- **Compliance**: Database sessions for audit requirements

Choose the implementation that best fits your application's security requirements, scalability needs, and infrastructure constraints.