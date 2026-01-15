
# Using `hybrid_property` in SQLAlchemy

In SQLAlchemy, `hybrid_property` lets you define attributes on your models that behave like regular Python properties **in Python code**, but can also be **translated into SQL expressions** when used in queries. This makes them very powerful for things like computed columns, filters, and sorting.

---

## 1. Import and Setup

You’ll need the `hybrid_property` from `sqlalchemy.ext.hybrid`:

```python
from sqlalchemy.ext.hybrid import hybrid_property
from sqlalchemy import Column, Integer, String
from sqlalchemy.orm import declarative_base

Base = declarative_base()
```

---

## 2. Basic Example

Suppose you have a `User` model with `first_name` and `last_name`, and you want to add a `full_name` property.

```python
class User(Base):
    __tablename__ = "users"

    id = Column(Integer, primary_key=True)
    first_name = Column(String)
    last_name = Column(String)

    @hybrid_property
    def full_name(self):
        return f"{self.first_name} {self.last_name}"
```

### Usage in Python
```python
user = User(first_name="John", last_name="Doe")
print(user.full_name)  # "John Doe"
```

However, if you try to query like:
```python
session.query(User).filter(User.full_name == "John Doe")
```
It **won’t work yet** — because SQLAlchemy doesn’t know how to turn the Python function into SQL.

---

## 3. Adding a `.expression` Method

To make it queryable, you define an **expression** version:

```python
from sqlalchemy import func

class User(Base):
    __tablename__ = "users"

    id = Column(Integer, primary_key=True)
    first_name = Column(String)
    last_name = Column(String)

    @hybrid_property
    def full_name(self):
        return f"{self.first_name} {self.last_name}"

    @full_name.expression
    def full_name(cls):
        return func.concat(cls.first_name, " ", cls.last_name)
```

### Now you can query using SQL
```python
# Filter
users = session.query(User).filter(User.full_name == "John Doe").all()

# Sort
users = session.query(User).order_by(User.full_name).all()
```

---

## 4. Read/Write Example

You can also make a hybrid property **settable**, similar to a Python `@property.setter`:

```python
class Product(Base):
    __tablename__ = "products"

    id = Column(Integer, primary_key=True)
    _price_cents = Column("price_cents", Integer)

    @hybrid_property
    def price(self):
        return self._price_cents / 100.0

    @price.setter
    def price(self, value):
        self._price_cents = int(value * 100)

    @price.expression
    def price(cls):
        return cls._price_cents / 100.0
```

### Usage
```python
p = Product()
p.price = 19.99
print(p._price_cents)  # 1999

session.query(Product).filter(Product.price > 10.0).all()
```

---

## Summary Table

| Feature                     | Usage                                              |
|-----------------------------|---------------------------------------------------|
| `@hybrid_property`           | Behaves like a normal Python property             |
| `@hybrid_property.expression`| Adds SQL expression for querying                  |
| `@hybrid_property.setter`    | Makes it writable like a Python property          |

---

## When to Use `hybrid_property`
- Computed columns like `full_name`, `price`, or `age`.
- When you need the same logic in both **Python code** and **database queries**.
- For cleaner, DRY code instead of repeating `func.concat` or other SQL functions everywhere.


