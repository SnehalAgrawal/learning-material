
# NGINX Quick Notes

Concise reference for working with **NGINX** on Linux-based systems.

---

## 📁 Common Paths

| Path                       | Description                               |
|---------------------------|-------------------------------------------|
| `/var/www/html`           | Default HTML document root.               |
| `/etc/nginx/sites-enabled`| All active (enabled) site configurations. |

---

## ⚙️ Useful Commands

### ✅ Test NGINX Configuration
```bash
sudo nginx -t
```
Checks the syntax and validity of your NGINX configuration files.
> ⚠️ Tip: Always run `nginx -t` before reloading or restarting to avoid downtime due to config errors.

---

### 🔁 Restart NGINX Server
```bash
sudo systemctl reload nginx
sudo service nginx restart
```
- `reload`: Reloads configuration without stopping the service.
- `restart`: Stops and starts the service again.

---

### 📊 Check NGINX Status
```bash
sudo systemctl status nginx.service
```
Displays whether NGINX is running and gives detailed service info.

---

### 🔗 `sites-available` and `sites-enabled`

- Store your site configs in `/etc/nginx/sites-available/`.
- Use symbolic links to enable them:

```bash
sudo ln -s /etc/nginx/sites-available/your-site /etc/nginx/sites-enabled/
```

- To disable a site, remove the symlink from `sites-enabled`.

```bash
sudo rm /etc/nginx/sites-enabled/your-site
```

---

### 🔁 Reverse Proxy Example: Node.js API + Frontend

Assuming:

- Node.js backend runs on `localhost:3000` and should be accessible at `/api`.
- Frontend build files are in `/var/www/html` and should be served at `/`.

#### Sample NGINX Config (`/etc/nginx/sites-available/myapp`)
```nginx
server {
    listen 80;
    server_name yourdomain.com;

    root /var/www/html;
    index index.html;

    location / {
        try_files $uri /index.html;
    }

    location /api/ {
        proxy_pass http://localhost:3000/;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

Then enable it:
```bash
sudo ln -s /etc/nginx/sites-available/myapp /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

---

### 🔒 SSL with Let's Encrypt

#### ✅ Prerequisites

* A domain name (e.g., yourdomain.com)
* DNS A record pointing to your server’s public IP
* NGINX installed and serving your site
* Port 80 (HTTP) and 443 (HTTPS) open on your firewall

#### 1. Install Certbot and NGINX Plugin
```bash
sudo apt update
sudo apt install certbot python3-certbot-nginx
sudo certbot --nginx -d yourdomain.com
```

#### 2. Issue SSL Certificate with Certbot
Certbot automatically edits your NGINX config and sets up HTTPS:
```bash
sudo certbot --nginx -d yourdomain.com -d www.yourdomain.com
```
* Certbot will:
  * Verify your domain via HTTP challenge
  * Create/renew certs stored in /etc/letsencrypt/live/yourdomain.com/
  * Update your NGINX config with ssl_certificate and ssl_certificate_key
  * Reload NGINX

#### 3. Example: Updated NGINX Config with SSL
After Certbot runs, your NGINX config (/etc/nginx/sites-available/myapp) will look like this:
```bash
server {
    listen 80;
    server_name yourdomain.com www.yourdomain.com;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name yourdomain.com www.yourdomain.com;

    root /var/www/html;
    index index.html;

    ssl_certificate /etc/letsencrypt/live/yourdomain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/yourdomain.com/privkey.pem;
    include /etc/letsencrypt/options-ssl-nginx.conf;
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;

    location / {
        try_files $uri /index.html;
    }

    location /api/ {
        proxy_pass http://localhost:3000/;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```
#### 4. Auto-Renewal (Handled by Certbot)
Check if Certbot’s auto-renew cron is already set up:
```bash
sudo systemctl list-timers | grep certbot
```
Manual test renewal:
```bash
sudo certbot renew --dry-run
```
