# Course 5: Authentication & Authorization

## 📖 What You Will Learn

- Authentication vs. Authorization concepts
- OAuth 2.0, JWT, API Keys, Session-based auth
- Testing authentication and authorization flows
- Common security testing scenarios

---

## 1. Authentication vs. Authorization

```
Authentication (AuthN): WHO are you?
  → Verifying identity (login, credentials)

Authorization (AuthZ): WHAT can you do?
  → Verifying permissions (access rights)

  ┌──────────────┬────────────────────────────────────────┐
  │              │ Authentication      │ Authorization     │
  ├──────────────┼─────────────────────┼───────────────────┤
  │ Purpose      │ Verify identity     │ Verify permission │
  │ Question     │ "Who are you?"      │ "What can you do?"│
  │ Happens      │ First               │ After AuthN       │
  │ Example      │ Login with password │ Admin vs. User    │
  │ HTTP Code    │ 401 Unauthorized    │ 403 Forbidden     │
  │ Visible to   │ User (login form)   │ Often invisible   │
  └──────────────┴─────────────────────┴───────────────────┘

  Flow:
    User → [Login] → AuthN: Is this user valid? → YES
                  → AuthZ: Can this user access /admin? → NO → 403
```

---

## 2. Authentication Methods

### 2.1 Session-Based Authentication

```
  Client                         Server
    │                               │
    │── POST /login ───────────────→│
    │   {user, password}            │
    │                               │── Validate credentials
    │                               │── Create session in memory/DB
    │←── Set-Cookie: sessionId=abc ─│
    │                               │
    │── GET /profile ──────────────→│
    │   Cookie: sessionId=abc       │
    │                               │── Look up session "abc"
    │←── 200 OK {user data} ───────│

  Characteristics:
    ✓ Server stores session state (stateful)
    ✓ Cookie sent automatically by browser
    ✓ Good for traditional web apps
    ✗ Hard to scale (session storage)
    ✗ Vulnerable to CSRF attacks
```

### 2.2 JWT (JSON Web Token)

```
  Structure: header.payload.signature

  Header:    {"alg": "HS256", "typ": "JWT"}
  Payload:   {"sub": "user123", "role": "admin", "exp": 1700000000}
  Signature: HMACSHA256(base64(header) + "." + base64(payload), secret)

  Flow:
    Client                         Server
      │── POST /login ────────────→│
      │   {user, password}          │
      │                             │── Validate credentials
      │                             │── Generate JWT
      │←── {"token": "eyJhb..."} ──│
      │                             │
      │── GET /profile ────────────→│
      │   Authorization: Bearer eyJhb...
      │                             │── Verify JWT signature
      │                             │── Extract claims from payload
      │←── 200 OK {user data} ─────│

  Characteristics:
    ✓ Stateless (server doesn't store sessions)
    ✓ Self-contained (has user info in payload)
    ✓ Scalable (no server-side storage)
    ✓ Works across services (microservices)
    ✗ Cannot be revoked until expiry
    ✗ Payload is base64, NOT encrypted (readable!)
```

### 2.3 OAuth 2.0

```
  OAuth 2.0 = Authorization framework for third-party access

  Roles:
    Resource Owner:  The user
    Client:          The app requesting access
    Auth Server:     Issues tokens (e.g., Google, GitHub)
    Resource Server: Holds protected resources (API)

  Authorization Code Flow:
    User → App → "Login with Google" →
      → Redirect to Google Auth Server →
      → User logs in & consents →
      → Google redirects back with auth code →
      → App exchanges code for access token →
      → App uses token to call API

  ┌──────────────────────────────────────────────────────┐
  │ 1. User clicks "Login with Google"                   │
  │ 2. App redirects to Google /authorize                │
  │ 3. User authenticates with Google                    │
  │ 4. Google redirects to app with ?code=abc123         │
  │ 5. App sends code to Google /token endpoint          │
  │ 6. Google returns access_token + refresh_token       │
  │ 7. App uses access_token to call Google API          │
  └──────────────────────────────────────────────────────┘
```

### 2.4 API Keys

```
  Simplest authentication method:

  GET /api/weather?city=London
  X-API-Key: abc123def456

  OR in query parameter:
  GET /api/weather?city=London&api_key=abc123def456

  Characteristics:
    ✓ Simple to implement
    ✓ Good for server-to-server communication
    ✗ No user context (identifies app, not user)
    ✗ Hard to rotate/revoke
    ✗ Easy to leak if in URLs or logs
```

---

## 3. Testing Authentication

```
┌──────────────────────────────────────────────────────────┐
│  Test Scenario                   │ Expected Result        │
├──────────────────────────────────┼────────────────────────┤
│ Valid credentials                │ 200 OK + token/session │
│ Invalid password                 │ 401 Unauthorized       │
│ Non-existent user                │ 401 Unauthorized       │
│ Empty username/password          │ 400 Bad Request        │
│ SQL injection in login           │ 401 (not SQL error)    │
│ Expired token/session            │ 401 Unauthorized       │
│ Missing Authorization header     │ 401 Unauthorized       │
│ Malformed JWT token              │ 401 Unauthorized       │
│ Brute force (many attempts)      │ 429 Too Many Requests  │
│ Password with special chars      │ 200 OK (should work)   │
│ Case sensitivity of username     │ Depends on system      │
│ Concurrent sessions              │ Per policy             │
│ Logout invalidates token         │ 401 on reuse           │
└──────────────────────────────────┴────────────────────────┘
```

---

## 4. Testing Authorization

```
┌──────────────────────────────────────────────────────────────┐
│  Test Scenario                      │ Expected Result         │
├─────────────────────────────────────┼─────────────────────────┤
│ Admin accesses admin panel          │ 200 OK                  │
│ Regular user accesses admin panel   │ 403 Forbidden           │
│ User accesses own profile           │ 200 OK                  │
│ User accesses another's profile     │ 403 Forbidden           │
│ Deleted user's token still works?   │ 401 or 403              │
│ Role escalation (change role claim) │ 403 (server validates)  │
│ IDOR: /users/123 vs /users/456     │ 403 for wrong user      │
│ Horizontal access (same role, diff  │ 403 for other's data    │
│   user's data)                      │                         │
│ Vertical access (lower role, higher │ 403 for admin resources │
│   privilege endpoint)               │                         │
└─────────────────────────────────────┴─────────────────────────┘

  IDOR (Insecure Direct Object Reference):
    User A is logged in → tries GET /api/orders/999
    Order 999 belongs to User B
    Expected: 403 Forbidden (not User A's order)
    Vulnerability: 200 OK with User B's data = IDOR bug!
```

---

## 5. Common Security Considerations

```
  Password Security:
    ✓ Minimum length (8+ chars)
    ✓ Complexity requirements
    ✓ Passwords stored as hashes (bcrypt, argon2)
    ✓ Account lockout after N failed attempts
    ✓ Secure password reset flow

  Token Security:
    ✓ Short-lived access tokens (15 min – 1 hour)
    ✓ Refresh tokens for renewal
    ✓ HTTPS only (never HTTP)
    ✓ Tokens not stored in localStorage (use httpOnly cookies)
    ✓ Token rotation on refresh

  API Security:
    ✓ Rate limiting
    ✓ Input validation
    ✓ CORS properly configured
    ✓ No sensitive data in URLs
    ✓ Proper error messages (no stack traces)
```

---

## ✅ Self-Check Questions

1. What is the difference between 401 and 403 HTTP status codes?

<details>
<summary>Answer</summary>

**401 Unauthorized** means the client is not authenticated — identity is unknown (missing or invalid credentials). **403 Forbidden** means the client is authenticated but not authorized — identity is known, but lacks permission for the resource. 401 = "Who are you?" (authentication failure). 403 = "I know who you are, but you can't do this" (authorization failure).

</details>

2. How does JWT authentication work? What are its advantages?

<details>
<summary>Answer</summary>

JWT flow: (1) User sends credentials to login endpoint, (2) Server validates and generates a JWT token (header.payload.signature), (3) Client stores token and sends it in `Authorization: Bearer <token>` header, (4) Server verifies the signature and extracts user info from the payload. **Advantages:** stateless (no server-side session storage), self-contained (carries user claims), scalable (works across microservices), cross-domain friendly. **Limitations:** cannot be revoked before expiry, payload is readable (base64, not encrypted).

</details>

3. What is IDOR and how do you test for it?

<details>
<summary>Answer</summary>

**IDOR (Insecure Direct Object Reference)** is a vulnerability where a user can access another user's resources by manipulating identifiers (e.g., changing `/api/orders/123` to `/api/orders/456`). Testing: (1) Log in as User A, (2) Note resource IDs belonging to User A, (3) Try accessing resources belonging to User B using User A's token, (4) Expected: 403 Forbidden. If you get 200 OK with User B's data, it's an IDOR vulnerability. Test both horizontal (same role, different user) and vertical (different privilege levels).

</details>

4. Compare session-based and JWT-based authentication.

<details>
<summary>Answer</summary>

**Session-based:** Server stores session in memory/DB, sends session ID via cookie, stateful, good for traditional web apps, hard to scale. **JWT-based:** Server generates signed token, client stores and sends it, stateless, self-contained, easy to scale, works across microservices. Session-based requires server storage and is vulnerable to CSRF; JWT doesn't need server storage but can't be easily revoked and the payload is readable.

</details>

5. List five important authentication test scenarios.

<details>
<summary>Answer</summary>

(1) **Valid credentials** — returns 200 + token, (2) **Invalid password** — returns 401, with no hint whether username or password is wrong, (3) **Expired token** — returns 401, must re-authenticate, (4) **Brute force protection** — returns 429 after N failed attempts, (5) **SQL injection in login** — returns 401 (not a database error), (6) **Logout invalidation** — using a token after logout returns 401.

</details>