# SSL Setup Guide for Project

This guide explains how to set up SSL certificates for the development environment, securing three services (Frontend, Backend, and Redis) on a single VM with custom domain names.

## Services Overview

| Service | HTTP URL | HTTPS URL |
|---------|----------|-----------|
| Frontend | http://0.0.0.0/ | https://frontend.sauravkumar.cloud/ |
| Backend | http://0.0.0.0:8000/ | https://backend.sauravkumar.cloud/ |
| Redis | http://0.0.0.0:8001/ | https://redis.sauravkumar.cloud/ |

## Prerequisites

- A virtual machine with Ubuntu/Debian
- Root or sudo access to the VM
- Domain names properly configured in DNS
- Basic knowledge of Linux, Nginx, and SSL

## Step 1: DNS Configuration

First, configure your DNS records to point to your VM's IP address:

1. Log in to your domain registrar or DNS management console
2. Add the following A records:

```
frontend.sauravkumar.cloud  →  Your VM's public IP address
backend.sauravkumar.cloud   →  Your VM's public IP address
redis.sauravkumar.cloud     →  Your VM's public IP address
```

3. Wait for DNS propagation (can take from minutes to 48 hours)
4. Verify DNS configuration using dig or nslookup:

```bash
dig frontend.sauravkumar.cloud
dig backend.sauravkumar.cloud
dig redis.sauravkumar.cloud
```

## Step 2: Install Required Software

Connect to your VM via SSH and install the required software:

```bash
# Update package lists
sudo apt update

# Install Nginx and Certbot with Nginx plugin
sudo apt install -y nginx certbot python3-certbot-nginx
```

## Step 3: Configure Nginx

Create separate configuration files for each service:

### Create Directory Structure

```bash
sudo mkdir -p /etc/nginx/sites-available
sudo mkdir -p /etc/nginx/sites-enabled
```

### Frontend Configuration

```bash
sudo nano /etc/nginx/sites-available/frontend.sauravkumar.cloud.conf
```

Add the following content:

```nginx
server {
    listen 80;
    server_name frontend.sauravkumar.cloud;
    
    location / {
        proxy_pass http://0.0.0.0:80;  # Frontend service port
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### Backend Configuration

```bash
sudo nano /etc/nginx/sites-available/backend.sauravkumar.cloud.conf
```

Add the following content:

```nginx
server {
    listen 80;
    server_name backend.sauravkumar.cloud;
    
    location / {
        proxy_pass http://0.0.0.0:8000;  # Backend service port
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### Redis Configuration

```bash
sudo nano /etc/nginx/sites-available/redis.sauravkumar.cloud.conf
```

Add the following content:

```nginx
server {
    listen 80;
    server_name redis.sauravkumar.cloud;
    
    location / {
        proxy_pass http://0.0.0.0:8001;  # Redis service port
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### Enable Sites

Create symbolic links to enable the sites:

```bash
sudo ln -s /etc/nginx/sites-available/frontend.sauravkumar.cloud.conf /etc/nginx/sites-enabled/
sudo ln -s /etc/nginx/sites-available/backend.sauravkumar.cloud.conf /etc/nginx/sites-enabled/
sudo ln -s /etc/nginx/sites-available/redis.sauravkumar.cloud.conf /etc/nginx/sites-enabled/
```

### Test and Restart Nginx

```bash
# Test Nginx configuration
sudo nginx -t

# If test is successful, restart Nginx
sudo systemctl restart nginx
```

## Step 4: Obtain SSL Certificates

Use Certbot to obtain SSL certificates and automatically configure Nginx:

```bash
sudo certbot --nginx -d frontend.sauravkumar.cloud -d backend.sauravkumar.cloud -d redis.sauravkumar.cloud
```

During the Certbot process:
1. Enter your email address for renewal notifications
2. Agree to the terms of service
3. Choose whether to redirect HTTP traffic to HTTPS (recommended)

## Step 5: Verify SSL Configuration

After Certbot completes:

```bash
# Test Nginx configuration again
sudo nginx -t

# Restart Nginx to apply changes
sudo systemctl restart nginx
```

## Step 6: Set Up Auto-Renewal

Enable automatic certificate renewal:

```bash
sudo systemctl enable certbot.timer
sudo systemctl start certbot.timer
```

Verify the timer is active:

```bash
sudo systemctl status certbot.timer
```

## Step 7: Test Your Setup

Visit each of your domains with HTTPS to verify everything is working correctly:
- https://frontend.sauravkumar.cloud/
- https://backend.sauravkumar.cloud/
- https://redis.sauravkumar.cloud/

You can also check the SSL certificate using an online SSL checker tool.

## Important File Locations

- **Nginx configuration files**: 
  - `/etc/nginx/sites-available/` - Configuration files
  - `/etc/nginx/sites-enabled/` - Symbolic links to enabled sites

- **SSL certificates**: 
  - `/etc/letsencrypt/live/[domain-name]/fullchain.pem` - Certificate
  - `/etc/letsencrypt/live/[domain-name]/privkey.pem` - Private key

- **Logs**:
  - `/var/log/nginx/access.log` - Nginx access logs
  - `/var/log/nginx/error.log` - Nginx error logs

## Troubleshooting

### Certificate Renewal Issues

If automatic renewal fails, you can manually renew:

```bash
sudo certbot renew
```

### Testing Certificate Renewal

To test the renewal process without actually renewing certificates:

```bash
sudo certbot renew --dry-run
```

### Firewall Configuration

Ensure ports 80 and 443 are open:

```bash
sudo ufw allow 80
sudo ufw allow 443
```

## Maintenance Tasks

### Updating Certbot

```bash
sudo apt update
sudo apt upgrade certbot python3-certbot-nginx
```

### Checking Certificate Expiry

```bash
sudo certbot certificates
```

## Security Recommendations

1. **Setup fail2ban** to protect against brute force attacks
2. **Configure CSP headers** for enhanced security
3. **Implement rate limiting** in Nginx
4. **Regular security audits** of your setup

## Additional Resources

- [Let's Encrypt Documentation](https://letsencrypt.org/docs/)
- [Nginx Documentation](https://nginx.org/en/docs/)
- [Certbot Documentation](https://certbot.eff.org/docs/)
