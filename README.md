# Nginx Configuration for Hurry Dev Services

## Overview
This Nginx configuration is set up to serve three services:
- **Backend Service** (Port 8000)
- **Frontend Service** (Port 8002)
- **Redis Service** (Port 8001)

Each service is configured with SSL certificates using Let's Encrypt and supports automatic redirection from HTTP to HTTPS.

---

## Configuration Details

### General Settings
- **User**: `www-data`
- **Worker Processes**: Auto
- **Error Log**: `/var/log/nginx/error.log`
- **PID File**: `/run/nginx.pid`
- **Worker Connections**: 1024

### SSL Settings
- **Session Cache**: Shared 10m
- **Session Timeout**: 10m
- **Protocols**: TLSv1.2, TLSv1.3
- **Ciphers**: Secure cipher list with preference for AES-GCM and strong encryption methods

### Services Configuration
Each service runs on its respective subdomain with SSL termination:

#### Backend Service
- **Domain**: `hurry-backend-dev.sauravkumar.cloud`
- **Proxy to**: `http://localhost:8000`
- **SSL Certificates**:
  - `fullchain.pem`
  - `privkey.pem`

#### Frontend Service
- **Domain**: `hurry-frontend-dev.sauravkumar.cloud`
- **Proxy to**: `http://localhost:8002`
- **SSL Certificates**:
  - `fullchain.pem`
  - `privkey.pem`

#### Redis Service
- **Domain**: `hurry-redis-dev.sauravkumar.cloud`
- **Proxy to**: `http://localhost:8001`
- **SSL Certificates**:
  - `fullchain.pem`
  - `privkey.pem`

### HTTP to HTTPS Redirection
A dedicated server block listens on port 80 and redirects all HTTP requests to HTTPS using a 301 redirect.

---

## Installation & Setup
### 1. Install Nginx
```bash
sudo apt update
sudo apt install nginx -y
```

### 2. Obtain SSL Certificates (Let's Encrypt)
Ensure `certbot` is installed and run:
```bash
sudo certbot certonly --nginx -d hurry-backend-dev.sauravkumar.cloud \
-d hurry-frontend-dev.sauravkumar.cloud \
-d hurry-redis-dev.sauravkumar.cloud
```

### 3. Deploy Configuration
- Replace the default Nginx config file with the provided `nginx.conf`.
- Restart Nginx:
```bash
sudo systemctl restart nginx
```

### 4. Enable Nginx on Boot
```bash
sudo systemctl enable nginx
```

---

## Troubleshooting
### Check Configuration
```bash
sudo nginx -t
```
### View Logs
```bash
sudo tail -f /var/log/nginx/error.log
```
### Restart Nginx
```bash
sudo systemctl restart nginx
```

---

## Author
**Saurav Kumar**

---

## License
This project is licensed under the MIT License.

