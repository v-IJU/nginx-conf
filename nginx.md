# ✅ Replace Default NGINX Config with Custom Configuration

This guide helps you replace the default NGINX configuration with your own custom config, including SSL setup for a Laravel site.

---

## 🔄 Step 1: Remove Default NGINX Site

Unlink the default site configuration:

```bash
sudo unlink /etc/nginx/sites-enabled/default
```

---

## ✏️ Step 2: Create Your Own Config File

Create a new NGINX config file:

```bash
sudo nano /etc/nginx/sites-available/mydomain.conf
```

Paste your custom configuration. Here's an example for a basic Laravel setup with SSL:

```nginx
server {
    listen 80;
    server_name dev.example.com;

    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name dev.example.com;

    ssl_certificate /etc/nginx/ssl/dev.{domain}.com.chained.crt;
    ssl_certificate_key /etc/nginx/ssl/generated-private-key.txt;

    root /var/www/{folder_name}/public;
    index index.html index.htm index.php;

    location / {
        try_files $uri $uri/ /index.php$is_args$args;
    }

    client_max_body_size 100M;
    error_page 404 /index.php;

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php8.1-fpm.sock;
    }
}
```

> Replace `{domain}` and `{folder_name}` with your actual domain and Laravel folder name.

---

## 🔗 Step 3: Link to Sites-Enabled

Create a symbolic link to enable your site:

```bash
sudo ln -s /etc/nginx/sites-available/mydomain.conf /etc/nginx/sites-enabled/
```

---

## ✅ Step 4: Test and Reload NGINX

Test your configuration:

```bash
sudo nginx -t
```

If the test is successful, reload NGINX:

```bash
sudo systemctl reload nginx
```

---

✅ Your custom NGINX configuration with SSL is now live!
