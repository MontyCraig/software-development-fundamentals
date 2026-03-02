# Security Fundamentals

## Overview

Security is not a feature to be added at the end of development -- it is a fundamental
property that must be woven into every layer of a system. A single vulnerability can
compromise user data, destroy trust, and result in legal and financial consequences. Every
developer, not just security specialists, needs a working knowledge of security principles.

The core challenges in software security are authentication (proving identity), authorization
(enforcing access control), data protection (encryption at rest and in transit), and input
validation (preventing injection attacks). The principle of defense in depth means that no
single layer of security is sufficient; multiple overlapping protections ensure that a
failure in one layer does not compromise the system.

This chapter covers the essential security concepts, common vulnerabilities from the OWASP
Top 10, cryptographic fundamentals, and practical patterns for building secure systems.

## Key Concepts

### Authentication vs Authorization

```
  AUTHENTICATION                    AUTHORIZATION
  "Who are you?"                    "What can you do?"

  +----------+                      +----------+
  | Username | --> Verify           | User     | --> Check
  | Password |     Identity         | Role     |    Permissions
  +----------+                      +----------+

  Example:                          Example:
  Login with credentials            Admin can delete users
  Verify API key                    Viewer can only read
  Validate session token            Owner can modify own data
```

| Aspect         | Authentication              | Authorization               |
|----------------|-----------------------------|-----------------------------|
| Question       | Who is this user?           | What is this user allowed?  |
| Happens        | Before authorization        | After authentication        |
| Mechanism      | Passwords, tokens, biometrics| Roles, permissions, policies|
| Failure result | 401 Unauthorized            | 403 Forbidden               |

### Password Hashing

Never store passwords in plain text. Use a one-way hash function with a unique salt per user.

```
FUNCTION register_user(username: String, password: String):
    SET salt = GENERATE_RANDOM_BYTES(16)
    SET hash = HASH(password + salt, algorithm: "bcrypt", cost: 12)
    STORE username, salt, hash IN users_table
END FUNCTION

FUNCTION verify_login(username: String, password: String) -> Boolean:
    SET user = LOOKUP username IN users_table
    IF user IS NULL THEN
        // Perform dummy hash to prevent timing attacks
        HASH("dummy" + GENERATE_RANDOM_BYTES(16), algorithm: "bcrypt", cost: 12)
        RETURN false
    END IF

    SET computed_hash = HASH(password + user.salt, algorithm: "bcrypt", cost: 12)
    RETURN constant_time_compare(computed_hash, user.hash)
END FUNCTION
```

```
  Password Storage:
  +-----------------------------------------------------+
  | username | salt (unique per user) | hash(pwd + salt) |
  +----------+-----------------------+-------------------+
  | alice    | a3f8c2e1...           | $2b$12$kR9...    |
  | bob      | 7b1d4e9f...           | $2b$12$xP4...    |
  +----------+-----------------------+-------------------+

  Even if two users have the same password, their hashes differ!
```

### Symmetric vs Asymmetric Encryption

```
  SYMMETRIC ENCRYPTION
  (Same key for encrypt and decrypt)

  Plaintext --[Key A]--> Ciphertext --[Key A]--> Plaintext
  "Hello"   --[encrypt]-> "x8f2a..." --[decrypt]-> "Hello"

  +--------+  Key A   +----------+  Key A   +--------+
  | Sender | -------> | Network  | -------> | Receiver|
  +--------+          +----------+          +--------+
  Problem: How do you securely share Key A?


  ASYMMETRIC ENCRYPTION
  (Public key encrypts, private key decrypts)

  Plaintext --[Public Key]--> Ciphertext --[Private Key]--> Plaintext

  +--------+  Bob's Public Key  +----------+  Bob's Private Key  +-----+
  | Alice  | -----------------> | Network  | ------------------> | Bob |
  +--------+                    +----------+                     +-----+

  Anyone can encrypt with public key.
  Only the holder of the private key can decrypt.
```

| Aspect        | Symmetric               | Asymmetric                  |
|---------------|--------------------------|------------------------------|
| Keys          | One shared key           | Public + private key pair    |
| Speed         | Fast                     | Slow                         |
| Use case      | Bulk data encryption     | Key exchange, digital sigs   |
| Key exchange  | Challenging              | Public key can be shared     |
| Examples      | AES, ChaCha20            | RSA, ECC                     |

### OWASP Top 10 Concepts

#### 1. Injection Attacks

Occur when untrusted input is sent to an interpreter as part of a command or query.

```
// VULNERABLE: String concatenation with user input
FUNCTION unsafe_login(username: String, password: String) -> User:
    SET query = "SELECT * FROM users WHERE name='" + username + "'"
    RETURN EXECUTE_QUERY(query)
END FUNCTION

// Input: username = "admin' OR '1'='1"
// Resulting query: SELECT * FROM users WHERE name='admin' OR '1'='1'
// Returns ALL users!

// SAFE: Parameterized queries
FUNCTION safe_login(username: String, password: String) -> User:
    SET query = "SELECT * FROM users WHERE name = ?"
    RETURN EXECUTE_QUERY(query, parameters: [username])
END FUNCTION
```

#### 2. Cross-Site Scripting (XSS)

Attacker injects malicious scripts into content viewed by other users.

```
// VULNERABLE: Rendering user input without escaping
FUNCTION display_comment(comment: String):
    SET html = "<div>" + comment + "</div>"
    RENDER html
END FUNCTION

// Input: comment = "<script>steal_cookies()</script>"
// The script executes in every viewer's browser!

// SAFE: Escape all user-supplied content
FUNCTION safe_display_comment(comment: String):
    SET escaped = HTML_ESCAPE(comment)    // < becomes &lt; etc.
    SET html = "<div>" + escaped + "</div>"
    RENDER html
END FUNCTION
```

#### 3. Cross-Site Request Forgery (CSRF)

Attacker tricks a user's browser into making authenticated requests to another site.

```
  Victim (logged into bank.com)
       |
       | visits attacker.com which contains:
       | <img src="bank.com/transfer?to=attacker&amount=1000">
       |
       v
  Browser sends request to bank.com WITH the victim's session cookie!

  PREVENTION:
  - Include a unique CSRF token in every form
  - Verify the token on every state-changing request
```

```
FUNCTION generate_csrf_protection() -> String:
    SET token = GENERATE_RANDOM_BYTES(32)
    STORE token IN session.csrf_token
    RETURN token    // Include in form as hidden field
END FUNCTION

FUNCTION verify_csrf(request: Request) -> Boolean:
    SET submitted_token = request.body.csrf_token
    SET expected_token = request.session.csrf_token
    RETURN constant_time_compare(submitted_token, expected_token)
END FUNCTION
```

#### 4. Broken Authentication

Weak session management, credential stuffing, missing multi-factor authentication.

### Input Validation

```
FUNCTION validate_user_input(input: Map) -> ValidationResult:
    SET errors = EMPTY LIST

    // Type checking
    IF NOT IS_STRING(input.name) THEN
        APPEND "name must be a string" TO errors
    END IF

    // Length constraints
    IF LENGTH(input.name) < 1 OR LENGTH(input.name) > 100 THEN
        APPEND "name must be 1-100 characters" TO errors
    END IF

    // Format validation
    IF NOT MATCHES_PATTERN(input.email, EMAIL_REGEX) THEN
        APPEND "email format is invalid" TO errors
    END IF

    // Range validation
    IF input.age < 0 OR input.age > 150 THEN
        APPEND "age must be between 0 and 150" TO errors
    END IF

    // Whitelist validation
    IF input.role NOT IN ["user", "admin", "moderator"] THEN
        APPEND "invalid role" TO errors
    END IF

    IF errors IS NOT EMPTY THEN
        RETURN ValidationResult(valid: false, errors: errors)
    END IF
    RETURN ValidationResult(valid: true)
END FUNCTION
```

### Principle of Least Privilege

Every component should have only the minimum permissions required to perform its function.

```
  WRONG (over-privileged):
  +-------------------+
  | Web Application   |
  | DB: ALL PRIVILEGES|  <-- Can DROP tables, read all data
  +-------------------+

  CORRECT (least privilege):
  +-------------------+
  | Web Application   |
  | DB: SELECT, INSERT|  <-- Can only read and create records
  |     on specific   |      in the tables it needs
  |     tables        |
  +-------------------+
```

### Defense in Depth

```
  +----------------------------------------------------------+
  |  Layer 1: NETWORK SECURITY                                |
  |  Firewalls, DDoS protection, network segmentation        |
  |  +----------------------------------------------------+  |
  |  |  Layer 2: APPLICATION SECURITY                      |  |
  |  |  Input validation, output encoding, CSRF tokens     |  |
  |  |  +----------------------------------------------+  |  |
  |  |  |  Layer 3: AUTHENTICATION & AUTHORIZATION      |  |  |
  |  |  |  MFA, role-based access, session management   |  |  |
  |  |  |  +--------------------------------------+    |  |  |
  |  |  |  |  Layer 4: DATA SECURITY               |    |  |  |
  |  |  |  |  Encryption at rest and in transit    |    |  |  |
  |  |  |  |  +------------------------------+   |    |  |  |
  |  |  |  |  |  Layer 5: MONITORING          |   |    |  |  |
  |  |  |  |  |  Logging, alerting, auditing  |   |    |  |  |
  |  |  |  |  +------------------------------+   |    |  |  |
  |  |  |  +--------------------------------------+    |  |  |
  |  |  +----------------------------------------------+  |  |
  |  +----------------------------------------------------+  |
  +----------------------------------------------------------+
```

### Secure Session Management

```
FUNCTION create_session(user: User) -> Session:
    SET session_id = GENERATE_RANDOM_BYTES(32)
    SET session = {
        id: session_id,
        user_id: user.id,
        created_at: CURRENT_TIME(),
        expires_at: CURRENT_TIME() + 30 MINUTES,
        ip_address: request.ip
    }
    STORE session IN session_store
    SET cookie = CREATE Cookie(
        name: "session_id",
        value: session_id,
        httpOnly: true,         // Not accessible via script
        secure: true,           // HTTPS only
        sameSite: "strict",     // CSRF protection
        maxAge: 30 MINUTES
    )
    RETURN session
END FUNCTION

FUNCTION validate_session(session_id: String) -> User:
    SET session = LOOKUP session_id IN session_store
    IF session IS NULL THEN
        RAISE AuthenticationError("Invalid session")
    END IF
    IF CURRENT_TIME() > session.expires_at THEN
        DELETE session FROM session_store
        RAISE AuthenticationError("Session expired")
    END IF
    // Extend session on activity (sliding expiration)
    SET session.expires_at = CURRENT_TIME() + 30 MINUTES
    RETURN LOOKUP session.user_id IN users
END FUNCTION
```

### CORS (Cross-Origin Resource Sharing)

```
  Browser (origin: app.example.com)
       |
       | Request to api.example.com
       |
       v
  +-------------------+
  | api.example.com   |
  | CORS Headers:     |
  | Access-Control-   |
  |  Allow-Origin:    |
  |  app.example.com  |
  +-------------------+
       |
       | Response with CORS headers
       v
  Browser allows the response (origins match)

  Without proper CORS headers:
  Browser BLOCKS the response (same-origin policy)
```

### Content Security Policy and Security Headers

```
FUNCTION set_security_headers(response: Response):
    // Prevent XSS by controlling allowed content sources
    SET response.headers["Content-Security-Policy"] =
        "default-src 'self'; script-src 'self'; style-src 'self'"

    // Prevent clickjacking
    SET response.headers["X-Frame-Options"] = "DENY"

    // Force HTTPS
    SET response.headers["Strict-Transport-Security"] =
        "max-age=31536000; includeSubDomains"

    // Prevent MIME sniffing
    SET response.headers["X-Content-Type-Options"] = "nosniff"

    // Control referrer information
    SET response.headers["Referrer-Policy"] = "strict-origin-when-cross-origin"
END FUNCTION
```

## Common Pitfalls and Best Practices

| Pitfall                            | Best Practice                                  |
|------------------------------------|------------------------------------------------|
| Storing passwords in plain text    | Always hash with salt (bcrypt, scrypt, argon2)  |
| SQL injection via concatenation    | Use parameterized queries exclusively            |
| Missing CSRF protection            | Include CSRF tokens in all state-changing forms  |
| Overly permissive CORS             | Whitelist specific origins, never use wildcard   |
| Secrets in source code             | Use environment variables or secret managers     |
| Missing rate limiting on login     | Limit failed attempts; implement account lockout |
| No input validation                | Validate all input on the server side            |
| Session IDs in URLs                | Use secure, httpOnly cookies                     |
| Error messages revealing internals | Return generic errors to users; log details      |

## Real-World Applications

- **Banking applications**: Multi-factor authentication, transaction signing, fraud detection
- **Healthcare systems**: HIPAA compliance, data encryption, audit logging
- **E-commerce**: PCI DSS compliance, secure payment processing, session security
- **Social platforms**: XSS prevention, content moderation, privacy controls
- **Cloud infrastructure**: IAM policies, network security groups, encryption at rest

## Related Topics

- [API Design Principles](api_design_principles.md) -- Authentication and authorization in APIs
- [Networking Fundamentals](networking_fundamentals.md) -- TLS/SSL, HTTPS
- [Database Fundamentals](database_fundamentals.md) -- SQL injection prevention

## Practice Problems

1. Write pseudocode for a complete user registration and login system with proper password
   hashing, salting, and timing-attack-resistant comparison.

2. Given a form that transfers money between accounts, implement CSRF protection including
   token generation, form embedding, and server-side validation.

3. Audit the following pseudocode for security vulnerabilities and fix each one:
   ```
   FUNCTION search(query: String) -> Results:
       SET sql = "SELECT * FROM products WHERE name LIKE '%" + query + "%'"
       SET results = EXECUTE(sql)
       SET html = "<h1>Results for: " + query + "</h1>"
       RETURN html + format_results(results)
   END FUNCTION
   ```

4. Design a role-based access control system with three roles (admin, editor, viewer) and
   five resources. Implement the authorization check function.

5. Implement a secure session management system that handles creation, validation, renewal,
   and revocation, including protection against session fixation.

6. Design a defense-in-depth strategy for a web application that handles financial data.
   Identify at least five layers of protection and explain each.
