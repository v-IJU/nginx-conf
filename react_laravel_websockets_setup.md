# React + Laravel WebSockets with Nginx & Cloudflare

This document explains the complete setup for connecting a **React frontend** to a **Laravel WebSockets backend**.  
The architecture uses **Nginx** as a reverse proxy to handle WebSocket connections securely, with **SSL provided by Cloudflare**.

---

## 🔹 Architecture Overview

The core challenge is to get a **secure WebSocket connection (WSS)** from the user's browser to your backend WebSocket server, which runs on a non-standard, insecure port.  

A direct connection is **not ideal** for security or configuration.

### ✅ Correct Flow of a Connection:
1. **React App (Browser)**  
   - Initiates a secure WebSocket connection to your public domain  
   - Example: `wss://multibranch.raregroup.info:443`

2. **Cloudflare**  
   - Receives the `wss://` request  
   - Handles SSL Termination  
   - Forwards request as plain `ws://` to your server on **port 80**

3. **Nginx (Reverse Proxy)**  
   - Listens on port 80  
   - Detects if request is normal HTTP or WebSocket upgrade  
   - Routes to Laravel (HTTP) or Laravel WebSockets server (WS)

4. **Laravel WebSockets Server**  
   - Runs internally on port `6001` (managed by Supervisor)  
   - Handles the persistent WebSocket connection

---

## 🔹 Why Use Nginx as Reverse Proxy?

- **Security & SSL**: Terminates `wss://` into local `ws://`
- **Single Point of Entry**: Keeps everything under port 80/443
- **Simplicity**: Frontend only needs to connect to your domain
- **Scalability**: Supports load balancing across multiple servers

---

## Install Required Packages

```

# Install the websockets package
composer require beyondcode/laravel-websockets

# Install the Pusher PHP SDK (used as the driver)
composer require pusher/pusher-php-server

```

## Publish and Configure

```
php artisan vendor:publish --provider="BeyondCode\LaravelWebSockets\WebSocketsServiceProvider" --tag="migrations"
php artisan vendor:publish --provider="BeyondCode\LaravelWebSockets\WebSocketsServiceProvider" --tag="config"
php artisan migrate
```   

## 🔹 Configuration Breakdown

### 1. React Client (`echo.js`)

Frontend must connect to **your public domain**, not the WebSocket server port.

#### `.env`
```env
VITE_APP_PUSHER_APP_KEY="your-app-key"
VITE_APP_PUSHER_HOST=multibranch.raregroup.info
VITE_APP_PUSHER_PORT=443
VITE_APP_PUSHER_SCHEME=https
VITE_APP_PUSHER_APP_CLUSTER=mt1
```

#### `src/echo.js`
```javascript
import Echo from "laravel-echo";
import Pusher from "pusher-js";

window.Pusher = Pusher;

const echo = new Echo({
  broadcaster: "pusher",
  key: import.meta.env.VITE_APP_PUSHER_APP_KEY,
  wsHost: import.meta.env.VITE_APP_PUSHER_HOST,
  wsPort: import.meta.env.VITE_APP_PUSHER_PORT,
  wssPort: import.meta.env.VITE_APP_PUSHER_PORT,
  forceTLS: true,
  disableStats: true,
  enabledTransports: ["ws", "wss"],
  cluster: import.meta.env.VITE_APP_PUSHER_APP_CLUSTER,
});

export default echo;
```

---

### 2. Nginx Configuration

#### `/etc/nginx/sites-available/your-site.conf`
```nginx
map $http_upgrade $type {
  default "web";
  websocket "ws";
}

server {
    listen 80;
    listen [::]:80;
    client_max_body_size 20M;

    root /var/www/florist_multibranch/public;
    index index.php index.html;
    charset utf-8;

    server_name multibranch.raregroup.info;

    location / {
        try_files /nonexistent @$type;
    }

    location @web {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location @ws {
        proxy_pass             http://127.0.0.1:6001;
        proxy_set_header Host  $host;
        proxy_read_timeout     60;
        proxy_connect_timeout  60;
        proxy_redirect         off;

        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/run/php/php8.1-fpm.sock;
    }
}
```

---

### 3. Supervisor Configuration

Supervisor keeps the WebSocket server alive.  

#### `/etc/supervisor/conf.d/websockets.conf`
```ini
[program:websockets]
command=php /var/www/florist_multibranch/artisan websockets:serve --port=6001
directory=/var/www/florist_multibranch
user=www-data
autostart=true
autorestart=true
stdout_logfile=/var/log/supervisor/websockets-stdout.log
stderr_logfile=/var/log/supervisor/websockets-stderr.log
```
- sudo supervisorctl reread
- sudo supervisorctl update
- sudo supervisorctl start websockets
---

### 4. How to Configure CORS for Laravel WebSockets

```
// config/websockets.php

'apps' => [
    // ... your app configuration
],

/*
 * This array holds the allowed origins that are allowed to connect to the websockets server.
 * Keep this empty to allow all origins. Or specify the origins you want to allow.
 */
'allowed_origins' => [
    // Add your React app's local development server URL
    'http://localhost:5173',  // Common for Vite
    'http://localhost:3000',  // Common for Create React App

    // Add your production React app's domain
    // Replace this with your actual production domain
    'https://your-production-react-app.com', 
],

// ... rest of the file
```

### 5. Restart the WebSocket Server

- The WebSocket server process itself loads the configuration when it starts. Since it's a long-running process managed by Supervisor, it won't see the changes until it's restarted.

Run this command on your server to restart the process: sudo supervisorctl restart websockets

### 6. Let the Package Generate Them create Secret and App key For pusher 

```
php artisan websockets:secret
```

## ✅ Final Notes
- React connects via **wss://multibranch.raregroup.info:443**
- Cloudflare handles SSL termination
- Nginx proxies WebSocket requests to `127.0.0.1:6001`
- Supervisor ensures the WebSocket server is always running

This setup ensures a **secure, reliable, and production-ready WebSocket architecture** with React + Laravel.
