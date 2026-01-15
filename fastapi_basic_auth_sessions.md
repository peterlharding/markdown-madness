
# FastAPI Session Management with BasicAuth and SQLAlchemy

This guide explains how to:

1. Extract a username from a BasicAuth header.
2. Check if a session already exists for that username.
3. Create a new session in the database if none exists.
4. Update `last_seen` for an existing session.
5. Use a cookie to track session IDs between requests.

---

## 1. SQLAlchemy Model for Login Sessions

Define a `login_session` table to store session data.

```python
# models.py
import uuid
from sqlalchemy import Column, Integer, String, DateTime, Boolean, func
from sqlalchemy.dialects.postgresql import UUID
from sqlalchemy.orm import declarative_base

Base = declarative_base()

class LoginSession(Base):
    __tablename__ = "login_session"

    id = Column(Integer, primary_key=True)
    session_id = Column(UUID(as_uuid=True), unique=True, nullable=False, default=uuid.uuid4)
    username = Column(String, index=True, nullable=False)
    user_agent = Column(String)
    ip = Column(String)
    created_at = Column(DateTime(timezone=True), server_default=func.now(), nullable=False)
    last_seen = Column(DateTime(timezone=True), server_default=func.now(), onupdate=func.now(), nullable=False)
    is_active = Column(Boolean, nullable=False, default=True)
```

---

## 2. Extract Username from BasicAuth

This helper function decodes the `Authorization` header.

```python
# auth.py
import base64, binascii
from fastapi import HTTPException
from starlette.requests import Request

def get_current_username_from_header(request: Request) -> str:
    authorization = request.headers.get("Authorization")
    if not authorization or not authorization.startswith("Basic "):
        raise HTTPException(status_code=401, detail="Missing or invalid authorization header")
    try:
        encoded = authorization.split(" ")[1]
        decoded = base64.b64decode(encoded).decode("utf-8")
        username, _password = decoded.split(":", 1)
        return username
    except (binascii.Error, ValueError, UnicodeDecodeError):
        raise HTTPException(status_code=401, detail="Invalid authorization header format")
```

---

## 3. Get or Create Session Dependency

This dependency checks for an existing session and creates one if necessary.

```python
# deps.py
import uuid
from datetime import datetime, timezone
from fastapi import Depends, Request
from sqlalchemy.exc import IntegrityError
from sqlalchemy.orm import Session
from starlette.responses import Response

from .auth import get_current_username_from_header
from .database import SessionLocal
from .models import LoginSession

def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()

class SessionContext:
    def __init__(self, username: str, session: LoginSession, created: bool):
        self.username = username
        self.session = session
        self.created = created

async def get_or_create_login_session(
    request: Request,
    db: Session = Depends(get_db),
) -> SessionContext:
    username = get_current_username_from_header(request)

    # Try to find an existing session using a cookie (sid)
    sid = request.cookies.get("sid")
    session = None
    if sid:
        session = (
            db.query(LoginSession)
            .filter(
                LoginSession.session_id == sid,
                LoginSession.username == username,
                LoginSession.is_active.is_(True),
            )
            .one_or_none()
        )

    created = False
    if session is None:
        # No session found, create a new one
        session = LoginSession(
            username=username,
            user_agent=request.headers.get("user-agent"),
            ip=(request.client.host if request.client else None),
        )
        db.add(session)
        try:
            db.commit()
        except IntegrityError:
            db.rollback()
            # If another request created the session concurrently
            session = (
                db.query(LoginSession)
                .filter(
                    LoginSession.username == username,
                    LoginSession.is_active.is_(True),
                )
                .order_by(LoginSession.created_at.desc())
                .first()
            )
        else:
            created = True
        db.refresh(session)

    # Update last_seen timestamp
    session.last_seen = datetime.now(timezone.utc)
    db.commit()
    db.refresh(session)

    return SessionContext(username=username, session=session, created=created)
```

---

## 4. Using the Dependency in Routes

Example of using the session dependency in FastAPI routes.

```python
# main.py
from fastapi import FastAPI, Depends
from starlette.responses import RedirectResponse, Response

from .deps import get_or_create_login_session, SessionContext

app = FastAPI()

@app.get("/profile")
async def profile(response: Response, ctx: SessionContext = Depends(get_or_create_login_session)):
    # If a new session was created, set a cookie
    if ctx.created:
        response.set_cookie(
            key="sid",
            value=str(ctx.session.session_id),
            httponly=True,
            samesite="Lax",
            secure=False,  # True if HTTPS
            max_age=60 * 60 * 24 * 7,  # 7 days
        )

    return {
        "username": ctx.username,
        "session_id": str(ctx.session.session_id),
        "last_seen": ctx.session.last_seen.isoformat(),
    }

@app.post("/logout")
async def logout(response: Response, ctx: SessionContext = Depends(get_or_create_login_session)):
    # Mark the session as inactive
    ctx.session.is_active = False
    response.delete_cookie("sid")
    return RedirectResponse(url="/", status_code=303)
```

---

## 5. Additional Considerations

### Why Use a Cookie With BasicAuth?
BasicAuth is stateless, meaning each request must include credentials. A cookie allows you to:

- Track sessions over time.
- Update `last_seen` without creating a new row on each request.
- Simplify client tracking for analytics.

### Security Recommendations
- Always use HTTPS and set `secure=True` on cookies.
- Rotate or expire sessions regularly.
- Optionally enforce a single active session per user by adding a partial unique index:

```sql
CREATE UNIQUE INDEX uniq_active_session_per_user
ON login_session (username)
WHERE is_active = TRUE;
```

### Built-in HTTPBasic Alternative
You can simplify BasicAuth parsing using FastAPI's built-in security dependency:

```python
from fastapi.security import HTTPBasic, HTTPBasicCredentials

security = HTTPBasic()

@app.get("/whoami")
def whoami(credentials: HTTPBasicCredentials = Depends(security)):
    return {"username": credentials.username}
```

---

## Summary of Flow

| Step                     | Action |
|--------------------------|--------|
| 1. Extract username      | Decode BasicAuth header |
| 2. Check cookie `sid`    | Look for active session in DB |
| 3. No session found      | Create new `login_session` row |
| 4. Update `last_seen`    | Touch timestamp every request |
| 5. Send cookie if created| `response.set_cookie` with session UUID |
| 6. Logout                | Mark session inactive, clear cookie |
