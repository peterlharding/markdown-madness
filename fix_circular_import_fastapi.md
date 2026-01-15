
# Fixing Circular Import Errors in FastAPI with SQLAlchemy

This guide explains how to resolve the following error:

```
ImportError: cannot import name 'app' from partially initialized module 'app'
(most likely due to a circular import)
```

This happens when two modules **import each other** at the top level, causing Python to load one before it’s ready.

---

## **The Problem**
In the current setup:

- `app/__init__.py` imports models:
  ```python
  from .models import LoginSession
  ```
- `app/models/__init__.py` (or another model file) imports something back from `app`.

This creates a circular dependency: `app` → `models` → `app`.

---

## **Solution: Break the Cycle**

The fix is to **separate concerns** and ensure imports flow in one direction only.

### 1. Keep `app/__init__.py` Empty or Minimal
```python
# app/__init__.py
# Leave empty or minimal to avoid recursive imports.
```

Move your FastAPI instance into a separate file like `app/main.py`:

```python
# app/main.py
from fastapi import FastAPI
from .routers import api  # example of including routers

app = FastAPI()
app.include_router(api)

# Optional: Import models here if needed at startup
# from . import models
```

Run with:
```
uvicorn app.main:app
```

---

### 2. Put Database Logic in `database.py`
Create a dedicated module for SQLAlchemy setup.

```python
# app/database.py
from sqlalchemy import create_engine
from sqlalchemy.orm import declarative_base, sessionmaker

DATABASE_URL = "postgresql+psycopg2://user:pass@host/db"

engine = create_engine(DATABASE_URL, future=True)
SessionLocal = sessionmaker(bind=engine, autoflush=False, autocommit=False)
Base = declarative_base()
```

---

### 3. Models Import Only from `database.py`
Models should **not import `app`** or anything that depends on the FastAPI instance.

```python
# app/models/login_session.py
from sqlalchemy import Column, Integer, String, Boolean, DateTime, func
from ..database import Base

class LoginSession(Base):
    __tablename__ = "login_session"
    id = Column(Integer, primary_key=True)
    username = Column(String, nullable=False)
    # ... other fields ...
```

Then re-export the model cleanly:

```python
# app/models/__init__.py
from .login_session import LoginSession

__all__ = ["LoginSession"]
```

> **Important:** Do **not** do `from .. import app` in this file.

---

### 4. Create Tables Without Circular Imports
When creating tables with `Base.metadata.create_all`, ensure models are imported first:

```python
# app/startup.py
from .database import Base, engine
from . import models  # ensures LoginSession is registered

Base.metadata.create_all(bind=engine)
```

You can run this at app startup:

```python
# app/main.py
from fastapi import FastAPI

app = FastAPI()

@app.on_event("startup")
def on_startup():
    from . import models
    from .database import Base, engine
    Base.metadata.create_all(bind=engine)
```

---

### 5. If You Must Import Lazily
If restructuring is hard, **move imports inside functions** so they execute later:

```python
# app/models/__init__.py
def get_app():
    from ..main import app  # imported only when function is called
    return app
```

This prevents the import from happening during module load time.

---

## **Recommended Directory Structure**
```
app/
├── __init__.py
├── main.py          # FastAPI instance here
├── database.py      # SQLAlchemy setup here
├── models/
│   ├── __init__.py  # Re-exports model classes
│   └── login_session.py
└── routers/
    └── api.py
```

---

## **Checklist**
- [ ] `app/__init__.py` is empty or minimal.
- [ ] FastAPI instance lives in `app/main.py`.
- [ ] `database.py` holds `engine`, `SessionLocal`, and `Base`.
- [ ] Models only import `Base` from `database.py`.
- [ ] `models/__init__.py` **never** imports `app`.
- [ ] Tables created after models are imported.

---

## **Summary**
Circular imports happen when two modules depend on each other.  
By separating the FastAPI app, database, and models into distinct layers:

- Models depend only on the database layer.
- The app depends on models, **but not vice versa**.
- Startup events or helper modules ensure tables are created without circular imports.

This clean separation ensures reliable startup and keeps your code modular.
