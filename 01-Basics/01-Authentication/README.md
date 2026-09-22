# 01 — Authentication

For a **4-year .NET professional**, you should not answer Authentication as simply:

> “Authentication is the process of checking username and password.”

That is correct, but it is too basic. A stronger interview answer explains **identity, credentials, verification, authentication context, and what happens after authentication**.

---

# 1. What is Authentication?

### Interview definition

> **Authentication is the process of verifying the identity of a user, application, or system before allowing it to access a protected resource.**

In simple terms:

```text
Authentication = "Who are you?"
```

For example, when a user logs into an application:

```text
User
 ↓
Username + Password
 ↓
Server verifies credentials
 ↓
Identity is established
 ↓
User is authenticated
```

The important point is:

**Authentication establishes identity.**

It does **not** determine what the user is allowed to do.

That is **Authorization**.

---

# 2. Authentication vs Authorization

This is one of the most common interview questions.

| Concept        | Question         | Example                                 |
| -------------- | ---------------- | --------------------------------------- |
| Authentication | Who are you?     | `john@example.com` successfully logs in |
| Authorization  | What can you do? | John can access `/admin/users`          |

Example:

```text
Login
  ↓
Username + Password
  ↓
Authentication
  ↓
User = John
  ↓
Authorization
  ↓
Role = Admin
  ↓
Can access Admin APIs
```

A 4-year candidate should be able to explain that these are **separate security concerns**, even though they usually work together.

---

# 3. What can be authenticated?

Authentication isn't limited to users.

It can authenticate:

```text
1. Human user
2. Application
3. Service
4. Device
5. Machine-to-machine client
```

For example:

### User authentication

```text
Username + Password
```

### Application authentication

```text
Client ID + Client Secret
```

### Token-based authentication

```text
Bearer Access Token
```

### Certificate-based authentication

```text
Client Certificate
```

---

# 4. Credentials

Authentication requires some mechanism to prove identity.

Common authentication factors include:

```text
Something you know
    ↓
Password / PIN

Something you have
    ↓
Authenticator / Security Key / OTP

Something you are
    ↓
Fingerprint / Face
```

This becomes important later when we implement:

```text
2FA
MFA
```

---

# 5. Authentication flow in a real application

Suppose we have:

```text
Angular Frontend
       ↓
ASP.NET Core Web API
       ↓
SQL Server
```

A user logs in:

```text
Angular
   │
   │ username + password
   ▼
ASP.NET Core API
   │
   │ Validate credentials
   ▼
SQL Server
   │
   │ User exists?
   ▼
Password verification
   │
   ▼
Authenticated User
```

At this point, the system knows:

```text
UserId = 101
Username = John
Email = john@example.com
```

Then another mechanism is generally used to maintain that authenticated state, such as:

```text
Cookie
Session
JWT Access Token
```

That's why Authentication itself is broader than JWT.

---

# 6. Very important distinction

This is something I want you to remember for interviews:

```text
Authentication
        │
        ├── Cookie Authentication
        ├── Session-based Authentication
        ├── Basic Authentication
        ├── JWT Bearer Authentication
        ├── OAuth 2.0
        ├── OpenID Connect
        ├── Certificate Authentication
        └── API Key-based schemes
```

However, there is an important protocol distinction:

**OAuth 2.0 is primarily an authorization framework**, while **OpenID Connect adds an identity/authentication layer on top of OAuth 2.0**.

We will cover that separately.

---

# 7. Our `BasicAuthenticationDemo`

Now let's move to your folder:

```text
01-Authentication/
│
├── README.md
│
└── BasicAuthenticationDemo/
```

Here, **Basic Authentication** means **HTTP Basic Authentication**.

It is an actual HTTP authentication scheme.

The client sends credentials in the HTTP `Authorization` header.

Example:

```http
Authorization: Basic am9objpQYXNzQDEyMw==
```

The value after `Basic` is a **Base64 encoding** of:

```text
username:password
```

For example:

```text
john:Pass@123
```

becomes something like:

```text
am9objpQYXNzQDEyMw==
```

### Critical interview point

**Base64 is encoding, not encryption.**

So this:

```text
Base64
```

does **not** protect the credentials by itself.

Therefore:

```text
Basic Authentication
        +
HTTPS/TLS
```

is essential.

Without HTTPS, credentials can potentially be exposed in transit.

---

# 8. How Basic Authentication works

The complete flow is:

```text
Client
   │
   │ HTTP Request
   ▼
API
   │
   │ No credentials
   ▼
401 Unauthorized
   │
   │ WWW-Authenticate: Basic
   ▼
Client sends credentials
   │
   │ Authorization: Basic <base64>
   ▼
API
   │
   │ Decode Base64
   ▼
username + password
   │
   ▼
Validate credentials
   │
   ├── Invalid → 401
   │
   └── Valid
          ↓
      Authenticated
          ↓
      Execute API
```

---

# 9. Example request

Suppose we have:

```http
GET /api/orders
```

Without authentication:

```http
GET /api/orders HTTP/1.1
Host: api.example.com
```

The API can respond:

```http
HTTP/1.1 401 Unauthorized
WWW-Authenticate: Basic
```

The client then sends:

```http
GET /api/orders HTTP/1.1
Host: api.example.com
Authorization: Basic am9objpQYXNzQDEyMw==
```

Server decodes:

```text
john:Pass@123
```

Then validates the credentials.

---

# 10. What does `401 Unauthorized` actually mean?

This is another interview favorite.

`401 Unauthorized` generally means:

> The request has not been successfully authenticated.

For example:

```text
Missing credentials
Invalid credentials
Expired/invalid authentication credential
```

Whereas:

```text
403 Forbidden
```

usually means:

> The caller is authenticated, but does not have sufficient permission for the requested resource.

Example:

```text
John successfully authenticated
        ↓
Role = Employee
        ↓
GET /api/admin/users
        ↓
403 Forbidden
```

So:

```text
401 → Authentication problem
403 → Authorization problem
```

That's an important distinction.

---

# 11. Basic Authentication is stateless

One useful characteristic of Basic Authentication is that the server does not necessarily need to maintain a login session.

The client sends credentials with requests:

```text
Request 1
Authorization: Basic ...

Request 2
Authorization: Basic ...

Request 3
Authorization: Basic ...
```

So the server can independently authenticate each request.

Conceptually:

```text
Request
   ↓
Credentials
   ↓
Validate
   ↓
Authenticated
```

There isn't a JWT-style access-token lifecycle here.

---

# 12. Basic Authentication vs JWT

For a 4-year interview, you should be able to explain this clearly.

| Feature                      | Basic Authentication | JWT Bearer                      |
| ---------------------------- | -------------------- | ------------------------------- |
| Credential sent              | Username + Password  | Access Token                    |
| Header                       | `Basic ...`          | `Bearer ...`                    |
| Typical credential lifetime  | Every request        | Token lifetime                  |
| Server-side session required | Not necessarily      | Not necessarily                 |
| Token format                 | No token structure   | JWT                             |
| Stateless                    | Yes, generally       | Yes, commonly                   |
| Suitable for modern APIs     | Limited use cases    | Common                          |
| Supports claims              | Not inherently       | Yes                             |
| Refresh token mechanism      | No                   | Commonly used with JWT systems  |
| Revocation model             | Credential-based     | Token lifecycle can be designed |
| Requires HTTPS               | Yes                  | Yes                             |

The key architectural difference is:

### Basic Authentication

```text
Client
 ↓
Username + Password
 ↓
Every request
```

### JWT

```text
Login
 ↓
Username + Password
 ↓
Access Token
 ↓
Subsequent requests
 ↓
Bearer Access Token
```

---

# 13. Why don't we normally use Basic Authentication for modern applications?

The main issue is architectural and security-related.

With Basic Authentication, the client repeatedly sends the user's credentials.

Conceptually:

```text
Request 1 → username/password
Request 2 → username/password
Request 3 → username/password
```

With a token-based model:

```text
Login
 ↓
Access Token
 ↓
Request 1 → token
Request 2 → token
Request 3 → token
```

This allows a much richer token lifecycle:

```text
Access Token
Refresh Token
Expiration
Rotation
Revocation
Scopes
Claims
```

which is why your later JWT/OAuth sections are important.

---

# 14. How ASP.NET Core handles Authentication

In ASP.NET Core, authentication is normally built around:

```text
Authentication Scheme
        ↓
Authentication Handler
        ↓
Authenticate Request
        ↓
ClaimsPrincipal
        ↓
HttpContext.User
```

This is the important .NET architecture to understand.

For example:

```text
Basic Authentication
        ↓
BasicAuthenticationHandler
        ↓
Validate credentials
        ↓
Create ClaimsPrincipal
        ↓
HttpContext.User
```

After authentication succeeds, your controller can access:

```csharp
HttpContext.User
```

and obtain identity information.

---

# 15. What is `ClaimsPrincipal`?

This is an important ASP.NET Core concept.

After successful authentication, ASP.NET Core represents the authenticated identity using a principal.

Conceptually:

```text
Authentication
      ↓
Claims
      ↓
ClaimsIdentity
      ↓
ClaimsPrincipal
      ↓
HttpContext.User
```

For example:

```text
User
 ├── UserId = 101
 ├── Name = John
 ├── Email = john@example.com
 └── Role = Admin
```

Later, authorization can use those claims.

For example:

```csharp
[Authorize(Roles = "Admin")]
```

So the security pipeline becomes:

```text
Authentication
      ↓
Build ClaimsPrincipal
      ↓
Authorization
      ↓
Allow / Deny endpoint
```

This is a much more professional way to explain the architecture.

---

# 16. BasicAuthenticationDemo — recommended project structure

For your repository, I'd make this demo small but production-style in architecture:

```text
BasicAuthenticationDemo/
│
├── Controllers/
│   └── ProductsController.cs
│
├── Authentication/
│   ├── BasicAuthenticationHandler.cs
│   └── BasicAuthenticationOptions.cs
│
├── Models/
│   ├── LoginUser.cs
│   └── UserCredential.cs
│
├── Services/
│   └── UserAuthenticationService.cs
│
├── Program.cs
├── appsettings.json
└── BasicAuthenticationDemo.csproj
```

For the first version, we can keep the credentials in configuration purely for learning.

For an actual application:

```text
Controller
   ↓
Authentication Handler
   ↓
Authentication Service
   ↓
User Repository
   ↓
Database
```

And passwords should **never** be stored as plaintext.

They should be stored using an appropriate password hashing mechanism.

---

# 17. Basic Authentication handler concept

The handler receives the request and reads:

```http
Authorization: Basic <credentials>
```

Then:

```text
1. Read Authorization header
2. Verify scheme = Basic
3. Decode Base64
4. Extract username/password
5. Validate credentials
6. Create ClaimsIdentity
7. Create ClaimsPrincipal
8. Return AuthenticateResult.Success()
```

Conceptually:

```text
HTTP Request
     ↓
Authorization Header
     ↓
BasicAuthenticationHandler
     ↓
Decode
     ↓
Credentials
     ↓
UserAuthenticationService
     ↓
Valid?
 ┌───┴────┐
No       Yes
 ↓        ↓
Fail   ClaimsPrincipal
          ↓
    HttpContext.User
```

---

# 18. What happens inside the handler?

A simplified version looks like this:

```csharp
protected override async Task<AuthenticateResult> HandleAuthenticateAsync()
{
    // 1. Check Authorization header

    // 2. Check whether scheme is Basic

    // 3. Decode Base64 value

    // 4. Extract username and password

    // 5. Validate credentials

    // 6. Create claims

    // 7. Create identity

    // 8. Create principal

    // 9. Return successful authentication result
}
```

The important thing is not memorizing the code.

Understand the **pipeline**.

---

# 19. Example claims

Suppose authentication succeeds.

We can create:

```csharp
var claims = new[]
{
    new Claim(ClaimTypes.NameIdentifier, "101"),
    new Claim(ClaimTypes.Name, "John"),
    new Claim(ClaimTypes.Email, "john@example.com"),
    new Claim(ClaimTypes.Role, "Admin")
};
```

Then:

```text
Claims
   ↓
ClaimsIdentity
   ↓
ClaimsPrincipal
```

Once assigned to the request:

```text
HttpContext.User
```

contains the authenticated identity.

---

# 20. Controller example

Then our protected controller could look like:

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    [HttpGet]
    [Authorize]
    public IActionResult GetProducts()
    {
        return Ok("Authenticated user can access products.");
    }
}
```

The important part is:

```csharp
[Authorize]
```

This tells ASP.NET Core:

> The request must have a successfully authenticated identity.

---

# 21. Authentication pipeline

Think of ASP.NET Core processing the request like this:

```text
HTTP Request
      ↓
Authentication Middleware
      ↓
Authentication Handler
      ↓
Is credential valid?
      │
      ├── No
      │    ↓
      │   401
      │
      └── Yes
           ↓
       ClaimsPrincipal
           ↓
       HttpContext.User
           ↓
       Authorization Middleware
           ↓
       [Authorize]
           ↓
       Controller
```

That's the architecture I would expect a **4-year .NET developer** to understand.

---

# 22. A real project-style example

Let's say you're working on an enterprise **Insurance/BFSI API**.

You have:

```text
GET /api/policies
```

The requirement says:

> Only authenticated users can access policies.

The architecture could be:

```text
Angular
   ↓
HTTPS
   ↓
ASP.NET Core API
   ↓
Authentication
   ↓
Basic Authentication
   ↓
Validate employee credentials
   ↓
ClaimsPrincipal
   ↓
Authorization
   ↓
Policy API
   ↓
SQL Server
```

And later we can change the authentication mechanism to:

```text
JWT Bearer
```

without redesigning the business logic itself.

That's one of the advantages of separating:

```text
Authentication
```

from:

```text
Business Logic
```

and:

```text
Authorization
```

---

# 23. What I would say in a 4-year interview

Suppose the interviewer asks:

### "What is authentication?"

A strong answer would be:

> **Authentication is the process of verifying the identity of a user, application, or service before granting access to protected resources. In an ASP.NET Core application, the authentication mechanism validates the supplied credentials or token and creates a `ClaimsPrincipal`, which is assigned to `HttpContext.User`. Authorization then uses that authenticated identity, along with claims, roles, or policies, to determine whether the user can access a particular resource.**
>
> **For example, in a Web API, a user may authenticate using a username and password, while subsequent requests use a JWT bearer access token. Authentication answers "who are you?", whereas authorization answers "what are you allowed to do?"**

That is much stronger than:

> "Authentication means login."

---

# 24. If the interviewer asks: "Explain Basic Authentication"

A strong 4-year answer:

> **HTTP Basic Authentication is an authentication scheme where the client sends a username and password in the `Authorization` HTTP header using the `Basic` scheme. The credentials are Base64 encoded, not encrypted, so HTTPS is mandatory to protect them in transit. On the server side, the authentication handler decodes and validates the credentials and creates the authenticated identity. Basic Authentication is simple and stateless, but for modern applications we generally prefer token-based mechanisms such as JWT or OAuth 2.0/OIDC depending on the use case.**

That's an interview-level answer.

---
