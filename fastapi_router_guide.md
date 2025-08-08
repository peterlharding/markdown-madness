# FastAPI Router Guide

FastAPI's `APIRouter` allows you to organize your routes into modular, reusable components.

## Basic Router Setup

Create a router in a separate file (e.g., `users.py`):

```python
from fastapi import APIRouter

router = APIRouter()

@router.get("/users/")
async def get_users():
    return [{"username": "alice"}, {"username": "bob"}]

@router.post("/users/")
async def create_user(user_data: dict):
    return {"message": "User created", "user": user_data}

@router.get("/users/{user_id}")
async def get_user(user_id: int):
    return {"user_id": user_id, "username": "alice"}
```

## Including Router in Main App

In your main application file (`main.py`):

```python
from fastapi import FastAPI
from . import users

app = FastAPI()

# Include the router
app.include_router(users.router)
```

## Router with Prefix and Tags

Add organization with prefixes and tags:

```python
app.include_router(
    users.router,
    prefix="/api/v1",
    tags=["users"],
    responses={404: {"description": "Not found"}}
)
```

Routes become accessible at `/api/v1/users/` instead of `/users/`.

## Router with Dependencies

Apply shared dependencies to all routes:

```python
from fastapi import APIRouter, Depends

def common_dependency():
    return {"shared_data": "value"}

router = APIRouter(dependencies=[Depends(common_dependency)])

@router.get("/protected/")
async def protected_route():
    return {"message": "This route has the common dependency"}
```

## Multiple Routers

Organize different API sections:

```python
from fastapi import FastAPI
from .routers import users, items, auth

app = FastAPI()

app.include_router(users.router, prefix="/api/v1", tags=["users"])
app.include_router(items.router, prefix="/api/v1", tags=["items"])
app.include_router(auth.router, prefix="/auth", tags=["authentication"])
```

## Router with Response Models

Use Pydantic models for type safety:

```python
from fastapi import APIRouter
from pydantic import BaseModel

class User(BaseModel):
    id: int
    username: str

router = APIRouter()

@router.get("/users/{user_id}", response_model=User)
async def get_user(user_id: int):
    return User(id=user_id, username="alice")
```

## Key Benefits

- **Organization**: Group related endpoints together
- **Modularity**: Separate concerns into different files
- **Reusability**: Use routers across different applications
- **Prefixing**: Automatically add prefixes to route groups
- **Shared Dependencies**: Apply common logic to multiple endpoints

Using routers keeps your code clean and maintainable as your API scales.