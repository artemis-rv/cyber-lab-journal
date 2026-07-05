# 🛡️ SQL Injection (CWE-89)

> **OWASP:** A03 – Injection  
> **CWE:** CWE-89 – Improper Neutralization of Special Elements used in an SQL Command ('SQL Injection')

---

# 📌 What is SQL Injection?

SQL Injection (SQLi) is a vulnerability where user-controlled input is interpreted as part of an SQL query due to improper input handling.

Instead of treating the input as data, the application treats it as SQL code, allowing an attacker to manipulate the intended query.

If successful, an attacker may:

- Read sensitive information
- Modify existing records
- Delete data
- Bypass authentication
- Execute administrative database operations

---

# 🧠 Basic SQL Query Structure

Example:

```sql
SELECT username, password
FROM users
WHERE username = 'admin'
AND password = 'password';
```

Suppose the backend builds the query like this:

```sql
SELECT * FROM users
WHERE username='$username'
AND password='$password';
```

User input:

```text
' OR 1=1 --
```

Generated query:

```sql
SELECT * FROM users
WHERE username='' OR 1=1 -- '
AND password='';
```

Since:

```sql
1 = 1
```

is always true,

the database returns every matching row, often resulting in authentication bypass.

---

# 🎯 Root Cause

SQL Injection occurs because applications directly concatenate user input into SQL queries.

Common causes include:

- Lack of input validation
- Dynamic SQL query construction
- Failure to use parameterized queries
- Poor input sanitization
- Excessive database privileges

---

# 💥 Consequences

## Confidentiality

- Database disclosure
- Credential leakage
- Sensitive business information exposure

---

## Integrity

- Record modification
- Unauthorized updates
- Data deletion

---

## Availability

- Database crashes
- Expensive queries causing DoS
- Table deletion

---

# 🚨 Real-World Impact

- Credential theft
- Authentication bypass
- Financial loss
- Reputational damage
- Regulatory violations
- Full database compromise

---

# 📂 Types of SQL Injection

## 1. In-band SQL Injection

The attacker receives results through the same communication channel.

Includes:

- Error-based SQLi
- UNION-based SQLi

---

## 2. Blind SQL Injection

The application does not reveal database errors.

Attacker infers results using:

- Boolean-based logic
- Time-based delays

---

## 3. Second-Order SQL Injection

Malicious payloads are stored in the database and executed later by another query.

---

# ⚔️ Common Payloads

## Authentication Bypass

```sql
' OR 1=1 --
```

---

## UNION-based Extraction

```sql
' UNION SELECT username,password FROM users --
```

---

## Comments

```sql
--
```

```sql
#
```

```sql
/**/
```

---

## Time-based

```sql
SLEEP(5)
```

```sql
WAITFOR DELAY '0:0:5'
```

---

## Boolean-based

```sql
' AND 1=1 --
```

```sql
' AND 1=2 --
```

---

## WAF Bypass Techniques

- URL Encoding
- Double Encoding
- Mixed Case Keywords
- Inline Comments
- HTTP Parameter Pollution (HPP)
- HTTP Parameter Fragmentation (HPF)

---

# 🔍 Detection

Typical indicators:

- SQL syntax errors after entering `'`
- Unexpected login bypass
- Database error messages
- Time delays after payloads
- Different page behavior between TRUE and FALSE conditions

Useful payload:

```sql
'
```

Then observe:

- Error message
- HTTP response
- Timing
- Status code

---

# 🛡️ Prevention

## Parameterized Queries (Prepared Statements)

✅ Best solution

---

## Stored Procedures

Use carefully.

---

## Allow-list Validation

Accept only expected values.

---

## Least Privilege

Application database accounts should have minimal permissions.

---

## Escape User Input

Last line of defense.

---

# ⚠️ Precautions While Testing

Never test destructive payloads against production systems.

Be cautious with payloads like:

```sql
' OR 1=1 --
```

If the backend query is:

```sql
UPDATE
```

or

```sql
DELETE
```

you may unintentionally modify or delete production data.

Always test in authorized environments.

---

# 🧠 Notes

- `--` begins a comment in many SQL dialects.
- `#` is supported in MySQL.
- `/* ... */` can be used as inline comments.
- SQL Injection is not limited to login pages.
- Any database interaction point can potentially be vulnerable.

---

# 📚 Related CWEs

- CWE-89 — SQL Injection
- CWE-564 — SQL Injection: Hibernate
- CWE-943 — Improper Neutralization in Data Query Logic

---

# 🔗 MITRE ATT&CK

- T1190 – Exploit Public-Facing Application

---

# 🧪 Practice Labs

## PortSwigger Web Security Academy

- SQL Injection Basics
- UNION Attacks
- Blind SQL Injection
- Error-based SQLi
- Time-based SQLi

---

## Additional Practice

- OWASP Juice Shop
- DVWA
- bWAPP

---

# 📖 References

- OWASP SQL Injection Prevention Cheat Sheet
- PortSwigger Web Security Academy
- CWE-89 Documentation
- OWASP: SQL Injection Bypassing WAF [https://owasp.org/www-community/attacks/SQL_Injection_Bypassing_WAF]
