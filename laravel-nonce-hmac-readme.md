# 🔐 Laravel API Security: Nonce + HMAC Signature

This guide explains how to protect your **Laravel API** against **data tampering** and **replay attacks** using **Nonce + HMAC signatures**, with Redis as a fast cache store.

---

## 📌 Overview

Each API request from the client must include:

- **X-Nonce** → Random unique value (used once per request)
- **X-Timestamp** → Current Unix timestamp (seconds)
- **X-Signature** → HMAC hash of (request body + nonce + timestamp) using a shared secret key

Laravel will:

1. Verify that the timestamp is within a short window (e.g., 2 minutes)
2. Verify that the nonce has not been used before (store in Redis)
3. Recompute HMAC and compare with the signature

If all checks pass → request is accepted ✅  
Otherwise → request is rejected ❌

---

## 🖥 Install & Configure Redis (Ubuntu)

### 1. Install Redis
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install redis-server -y
```

### 2. Configure Redis
Edit config file:
```bash
sudo nano /etc/redis/redis.conf
```
Find:
```
supervised no
```
Change to:
```
supervised systemd
```

Restart Redis:
```bash
sudo systemctl restart redis
sudo systemctl enable redis
```

Verify:
```bash
redis-cli ping
# Expected Output: PONG
```

---

## ⚙ Install PHP Redis Extension

```bash
sudo apt install php-redis -y
sudo systemctl restart php8.2-fpm # Adjust PHP version if needed
php -m | grep redis # Verify extension is loaded
```

---

## 🛠 Configure Laravel to Use Redis

Update `.env`:
```env
CACHE_DRIVER=redis
SESSION_DRIVER=redis

REDIS_HOST=127.0.0.1
REDIS_PORT=6379
REDIS_PASSWORD=null
```

Run:
```bash
php artisan config:cache
```

---

## 🔧 Create Middleware: `VerifyHmacSignature`

Generate middleware:
```bash
php artisan make:middleware VerifyHmacSignature
```

Edit `app/Http/Middleware/VerifyHmacSignature.php`:

```php
namespace App\Http\Middleware;

use Closure;
use Illuminate\Support\Facades\Cache;

class VerifyHmacSignature
{
    public function handle($request, Closure $next)
    {
        $secret = config('app.hmac_secret'); // Store in .env securely

        $nonce = $request->header('X-Nonce');
        $timestamp = $request->header('X-Timestamp');
        $signature = $request->header('X-Signature');

        if (!$nonce || !$timestamp || !$signature) {
            return response()->json(['error' => 'Missing security headers'], 400);
        }

        // 1. Verify timestamp freshness (2-minute window)
        if (abs(time() - (int) $timestamp) > 120) {
            return response()->json(['error' => 'Request expired'], 400);
        }

        // 2. Prevent replay attack
        if (Cache::has("nonce:$nonce")) {
            return response()->json(['error' => 'Replay attack detected'], 400);
        }
        Cache::put("nonce:$nonce", true, now()->addSeconds(120));

        // 3. Verify HMAC signature
        $data = $request->getContent();
        $computedSignature = hash_hmac('sha256', $data.$nonce.$timestamp, $secret);

        if (!hash_equals($computedSignature, $signature)) {
            return response()->json(['error' => 'Invalid signature'], 400);
        }

        return $next($request);
    }
}
```

Register middleware in `app/Http/Kernel.php` under API group:

```php
protected $middlewareGroups = [
    'api' => [
        // ...
        \App\Http\Middleware\VerifyHmacSignature::class,
    ],
];
```

Add secret to `.env`:
```env
HMAC_SECRET=my_super_secret_key
```

In `config/app.php`:
```php
'hmac_secret' => env('HMAC_SECRET', 'fallback-secret'),
```

---

## 📱 React Native Client Example

Install crypto library:
```bash
npm install crypto-js
```

Example request:

```javascript
import * as Crypto from "crypto-js";

const API_SECRET = "my_super_secret_key";

async function sendSecureRequest(url, payload) {
  const nonce = Math.random().toString(36).substring(2);
  const timestamp = Math.floor(Date.now() / 1000);
  const body = JSON.stringify(payload);

  const signature = Crypto.HmacSHA256(body + nonce + timestamp, API_SECRET).toString();

  const res = await fetch(url, {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      "X-Nonce": nonce,
      "X-Timestamp": timestamp.toString(),
      "X-Signature": signature,
    },
    body,
  });

  return res.json();
}
```

---

## ✅ Testing

1. Start Redis:  
   ```bash
   sudo systemctl status redis
   ```
2. Send request from React Native or Postman with headers.
3. Replay the same request → should fail with `Replay attack detected`.

---

## 🔐 Best Practices

- ✅ **Use HTTPS** to encrypt traffic
- ✅ **Short timestamp window** (e.g., 1–2 mins)
- ✅ **Rotate HMAC secrets** periodically
- ✅ **Store nonce in Redis** with TTL (auto-expires)
- ✅ **Rate-limit API** to prevent brute-force attacks

---

## 🧪 Quick Debug Commands

Check stored keys in Redis:
```bash
redis-cli keys '*'
```

Flush all (for development):
```bash
redis-cli FLUSHALL
```

---

## 🎯 Result

After implementing this setup:

- Requests cannot be tampered with (invalid HMAC = rejected)
- Replay attacks are blocked (nonce can be used only once)
- Performance remains fast (Redis cache is in-memory)

This is a **production-grade solution** for securing Laravel APIs.
