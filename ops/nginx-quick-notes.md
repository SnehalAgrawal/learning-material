
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
---
### ⚡ Advanced Caching Strategies in NGINX

#### 1. **Static File Caching (Client-side)**

Tell browsers to cache assets:

```nginx
location ~* \.(jpg|jpeg|png|gif|ico|css|js)$ {
    expires 30d;
    access_log off;
}
```

> ✅ Reduces repeated requests for static files.

---

#### 2. **Proxy Caching (Server-side)**

Enable caching for responses from an upstream (like Node.js API):

```nginx
proxy_cache_path /tmp/nginx_cache levels=1:2 keys_zone=my_cache:10m inactive=60m;

server {
    location /api/ {
        proxy_cache my_cache;
        proxy_pass http://localhost:3000;
        proxy_cache_valid 200 302 10m;
        proxy_cache_valid 404 1m;
        add_header X-Cache-Status $upstream_cache_status;
    }
}
```
`proxy_cache_path /tmp/nginx_cache`
- This defines the **directory on disk** where cached files will be stored.
- NGINX will write cached responses here.
- Make sure this path exists and is writable by the NGINX process (`www-data` in most Linux distros).    

 `levels=1:2`
- Controls the **directory structure** to avoid too many files in a single folder.
- In this case:
    - First-level directory: 1 character
    - Second-level directory: 2 characters
- Example path: `/tmp/nginx_cache/a/bc/hashed_file.cache`
> ✅ This helps improve file system performance by distributing cache files across folders.
 
 `keys_zone=my_cache:10m`
- Defines a **shared memory zone** for cache keys named `my_cache`, with **10 MB** allocated. 
- This memory is used to store metadata (keys, timestamps, headers) — **not** the actual response bodies.
- Required to use the cache in `proxy_cache` later.
> ✅ Caches successful and 404 responses; saves upstream load.



---

#### 3. **Microcaching (Good for APIs)**

Cache responses for just a few seconds to absorb sudden traffic spikes:

```nginx
proxy_cache_valid 200 1s;
```

> ⚠️ Make sure your APIs are **safe to cache** (no user-specific data unless using cache keys carefully).

---

### 🌐 Load Balancing Between Multiple Backends

NGINX can distribute traffic between multiple app servers.

#### 1. **Basic Round-Robin Load Balancing**

```nginx
upstream myapp {
    server 127.0.0.1:3000;
    server 127.0.0.1:3001;
}

server {
    location /api/ {
        proxy_pass http://myapp;
    }
}
```

#### 2. **Least Connections (Better for longer-lived requests)**

```nginx
upstream myapp {
    least_conn;
    server 127.0.0.1:3000;
    server 127.0.0.1:3001;
}
```

#### 3. **Health Checks (via `nginx-plus` or 3rd party modules)**

Open source NGINX doesn't support active health checks out of the box. You can:

* Use passive health checks (mark server down on error)
* Or use `NGINX Plus` for more advanced health checking

---

### 📈 Monitoring and Logging Best Practices

#### 1. **Access and Error Logs**

Default locations:

```bash
/var/log/nginx/access.log
/var/log/nginx/error.log
```

You can split logs per site:

```nginx
access_log /var/log/nginx/myapp_access.log;
error_log /var/log/nginx/myapp_error.log;
```

---

#### 2. **Enable JSON Logging (for structured logging + monitoring tools)**

```nginx
log_format json_combined escape=json '{ "time_local": "$time_local", '
                                     '"remote_addr": "$remote_addr", '
                                     '"request": "$request", '
                                     '"status": "$status", '
                                     '"body_bytes_sent": "$body_bytes_sent", '
                                     '"http_referer": "$http_referer", '
                                     '"http_user_agent": "$http_user_agent" }';

access_log /var/log/nginx/access.json json_combined;
```

> Great for ingesting logs into ELK, Loki, or Datadog.

---

#### 3. **Real-Time Monitoring Tools**

* **Nginx Amplify**: Official tool with free tier.
* **Netdata**: Beautiful real-time graphs.
* **Grafana + Prometheus**: If you integrate NGINX exporter or custom logs.
* **ELK Stack**: Ingest access logs for full analysis.
