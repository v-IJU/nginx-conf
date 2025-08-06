
# 🚀 Laravel Deployment on Shared Hosting (With Permissions & .htaccess Setup)

This guide covers how to deploy a Laravel application on shared hosting, including permissions and `.htaccess` configuration.

---

## 📁 Folder Structure

Typical shared hosting setup:
```
/home/username/
├── laravel-app/            ← Laravel application directory (outside web root)
├── public_html/            ← Public web root
```

### ✅ Steps:
1. Move entire Laravel app to `/home/username/laravel-app/`
2. Move contents of `/laravel-app/public/` to `/public_html/`
3. Update paths in `public_html/index.php`:

```
require __DIR__.'/../laravel-app/vendor/autoload.php';
$app = require_once __DIR__.'/../laravel-app/bootstrap/app.php';
```

---

## 🔐 Permissions Setup

Set correct permissions:

```bash
# File permissions
find . -type f -exec chmod 644 {} \;

# Folder permissions
find . -type d -exec chmod 755 {} \;

# Writable folders
chmod -R 755 storage
chmod -R 755 bootstrap/cache

# .env file
chmod 644 .env
```

Ensure ownership is correct (if SSH access is available):

```bash
chown -R your_cpanel_user:your_cpanel_user *
```

---

## 📜 Required .htaccess File (in `public_html/`)

```apache
<IfModule mod_rewrite.c>
    RewriteEngine On

    RewriteCond %{REQUEST_FILENAME} !-f
    RewriteCond %{REQUEST_FILENAME} !-d
    RewriteRule ^ index.php [L]
</IfModule>

<FilesMatch "\.(env|git|env\.example|gitattributes|gitignore|htaccess|composer\.json|composer\.lock|artisan)$">
    Order allow,deny
    Deny from all
</FilesMatch>

Options -Indexes
```

---

## ⚙️ Check PHP Version

Ensure you're running PHP **7.4 or higher** with these extensions enabled:

- mbstring
- openssl
- pdo, pdo_mysql
- tokenizer
- xml
- fileinfo
- bcmath
- ctype
- curl
- json

Use **Select PHP Version** in your cPanel to manage this.

---

## 🛠 Final Checklist

| Task                         | Done |
|------------------------------|------|
| `.env` file present          | ✅   |
| `vendor/` folder uploaded    | ✅   |
| `storage/` writable          | ✅   |
| `index.php` path updated     | ✅   |
| `.htaccess` added            | ✅   |
| PHP version >= 7.4           | ✅   |

---

## 🧪 Troubleshooting

- Check Laravel logs: `storage/logs/laravel-*.log`
- Check file/folder ownership
- Double check `.htaccess` file and syntax
- Ensure `.env` file exists and has correct values

---

For questions, contact your hosting provider or paste error logs for debugging.
