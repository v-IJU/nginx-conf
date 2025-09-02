# Laravel Sanctum Authentication Guide

This document explains the different ways to use **Laravel Sanctum** for authentication, covering both **SPA cookie/session-based auth** and **token-based auth**. It also includes environment setup, middleware, and CORS configuration.

---

## 🔑 Authentication Methods in Sanctum

### 1. **Cookie / Session-based (Stateful) Authentication**
- Best for **first-party SPAs** (React, Vue, etc.) hosted on the **same domain or subdomain** as Laravel.
- Uses **Laravel sessions & CSRF tokens**.
- Sanctum identifies requests as *stateful* if the domain is in `SANCTUM_STATEFUL_DOMAINS`.

**How it works:**
1. SPA calls `sanctum/csrf-cookie` → backend returns `XSRF-TOKEN` cookie.
2. SPA sends login request → session is created.
3. SPA makes API requests with `withCredentials: true` → session cookie is sent automatically.
4. Sanctum checks `SANCTUM_STATEFUL_DOMAINS` and `SESSION_DOMAIN` to validate cookies.

---

### 2. **Token-based (Stateless) Authentication**
- Best for **mobile apps**, **third-party clients**, or when frontend and backend are on **different domains**.
- No cookies are required.
- You generate a **Personal Access Token** and send it with each request.

**How it works:**
1. User logs in → backend generates a token via `$user->createToken('app')->plainTextToken`.
2. SPA/Mobile stores token (e.g., `localStorage`).
3. All API requests include:

```http
Authorization: Bearer <token>
```

4. Sanctum authenticates using the token.

---

## ⚙️ Configuration

### 1. Middleware (`app/Http/Kernel.php`)
```php
"api" => [
    \Laravel\Sanctum\Http\Middleware\EnsureFrontendRequestsAreStateful::class,
    "throttle:api",
    \Illuminate\Routing\Middleware\SubstituteBindings::class,
],
```

- `EnsureFrontendRequestsAreStateful` → required for cookie/session-based auth.
- For token-based auth, it’s ignored.

---

### 2. `.env` Setup

#### For Cookie/Session-based Auth (SPA)
```env
SESSION_DRIVER=cookie
SESSION_DOMAIN=.cordeliakare.com   # Notice the leading dot for subdomains
SANCTUM_STATEFUL_DOMAINS=cordeliakare.com,dev.cordeliakare.com,localhost:3000,127.0.0.1:8000
```

#### For Token-based Auth
```env
# No need for SESSION_DOMAIN or SANCTUM_STATEFUL_DOMAINS
# Works across any domain
```

---

### 3. `config/cors.php`
```php
return [
    "paths" => ["api/*", "login", "logout", "sanctum/csrf-cookie"],
    "allowed_methods" => ["*"],
    "allowed_origins" => [
        "http://localhost:3000",
        "https://cordeliakare.com",
        "https://dev.cordeliakare.com"
    ],
    "allowed_headers" => ["*"],
    "supports_credentials" => true, // must be true for cookie-based auth
];
```

---

## 🚀 Usage Examples

### Cookie-based Auth (SPA + Laravel same domain)
```js
// Fetch CSRF cookie first
await axios.get("/sanctum/csrf-cookie", { withCredentials: true });

// Then login
await axios.post("/login", { email, password }, { withCredentials: true });

// Authenticated request
await axios.get("/api/user", { withCredentials: true });
```

### Token-based Auth (SPA / Mobile / Different domain)
```js
// After login, backend gives a token
localStorage.setItem("auth_token", token);

// Use it in every request
await axios.get("/api/user", {
    headers: {
        Authorization: `Bearer ${localStorage.getItem("auth_token")}`
    }
});
```

---

## ✅ Which One Should You Use?

- **Cookie/Session-based (stateful)**
  - If frontend and backend are on the **same domain/subdomain**.
  - Example: `app.cordeliakare.com` (React) + `api.cordeliakare.com` (Laravel).

- **Token-based (stateless)**
  - If frontend and backend are on **different domains**.
  - For **mobile apps** or **third-party APIs**.
  - Example: `cordeliakare.com` (React) + `api.cordeliakare.com` (Laravel).

---

## 🔍 Common Issues

1. **Getting `null` for `Auth::user()`**
   - `withCredentials: true` missing in axios request.
   - Wrong `SESSION_DOMAIN` or missing `SANCTUM_STATEFUL_DOMAINS`.

2. **Token works but cookies don’t**
   - You’re mixing **stateful** and **stateless** auth. Decide one strategy.

3. **Localhost setup**
   - Use:  
     ```env
     SANCTUM_STATEFUL_DOMAINS=localhost:3000,127.0.0.1:8000
     SESSION_DOMAIN=localhost
     ```

---

## 📝 Summary

- **Token-based auth** = works everywhere, stateless, simple for SPA & mobile.  
- **Cookie-based auth** = more secure for first-party apps, needs correct domain setup.  
- `SANCTUM_STATEFUL_DOMAINS` → required **only for cookie/session auth**.  
- `SESSION_DOMAIN` → defines cookie scope (e.g., `.cordeliakare.com` for subdomains).  

