
# PostgreSQL: TEXT vs VARCHAR

In PostgreSQL, you can **use `TEXT` and `VARCHAR` almost interchangeably**, but there are a few subtle differences to be aware of.

---

## 1. Similarities

- Both `TEXT` and `VARCHAR` are **variable-length string types**.
- They **store the same type of data** (strings) and have **identical performance**.
- In PostgreSQL, **`VARCHAR` without a length limit (`VARCHAR` or `VARCHAR(n)` with no `n`) behaves exactly like `TEXT`**.

For example, all of these are equivalent:

```sql
CREATE TABLE example (
    name TEXT,
    email VARCHAR,
    description VARCHAR
);
```

The above three columns behave identically because `VARCHAR` without a limit is treated like `TEXT`.

---

## 2. Key Difference: Length Constraints

The only real difference comes when you **define a maximum length** with `VARCHAR(n)`.

Example:

```sql
CREATE TABLE users (
    username VARCHAR(20), -- max 20 characters
    bio TEXT               -- unlimited length
);
```

- If you try to insert a string longer than 20 characters into `username`, PostgreSQL **throws an error**:

  ```sql
  INSERT INTO users (username, bio)
  VALUES ('thisusernameiswaytoolong', 'Hello there!');
  ```

  ```
  ERROR:  value too long for type character varying(20)
  ```

- The `bio` column, being `TEXT`, accepts any length.

---

## 3. Performance Considerations

- **No performance difference** between `TEXT` and `VARCHAR(n)` in PostgreSQL.  
  Unlike some older databases (like Oracle or MySQL), PostgreSQL treats them the same internally.
- Adding a `VARCHAR(n)` limit does **not improve performance**; it's purely for validation or documentation.

---

## 4. Best Practice

- Use `TEXT` when you **don't care about enforcing a length limit**.
- Use `VARCHAR(n)` when you **need to enforce a maximum length**, like for usernames, codes, etc.

A common pattern is:

```sql
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    sku VARCHAR(32) NOT NULL,    -- Enforced length
    name TEXT NOT NULL           -- No enforced limit
);
```

---

## 5. Special Note: CHAR(n)

- `CHAR(n)` is **fixed-length**, and PostgreSQL will **pad the value with spaces**.
- Generally, you should **avoid `CHAR(n)`** unless you're dealing with legacy systems or strict formatting requirements (e.g., ISO codes).

---

## Summary Table

| Type        | Enforced Length? | Performance | Notes |
|-------------|------------------|-------------|-------|
| `TEXT`      | ❌ No             | Same as `VARCHAR` | Unlimited length |
| `VARCHAR`   | ❌ No             | Same as `TEXT` | Behaves like `TEXT` |
| `VARCHAR(n)`| ✅ Yes            | Same as `TEXT` | Good for validation |
| `CHAR(n)`   | ✅ Yes (fixed)    | Slightly worse | Pads with spaces |

---

## Conclusion

- `TEXT` and `VARCHAR` are **interchangeable unless you need to enforce a maximum length**.
- If you don't need strict validation, use `TEXT` for simplicity.  
- PostgreSQL doesn't penalize you for using `TEXT` everywhere.
