# 🧭 AWS Route 53 Simple Routing Project — With SSL nginx

## For more projects, check out  
[https://harishnshetty.github.io/projects.html](https://harishnshetty.github.io/projects.html)

## 1️⃣ Install and Configure Nginx with Certbot

```bash
sudo apt update
sudo apt install nginx certbot python3-certbot-nginx -y

sudo systemctl enable nginx
sudo systemctl start nginx
sudo nginx -t
sudo systemctl status nginx
```

---

## 2️⃣ Obtain Wildcard SSL Certificate Using Certbot (Manual DNS Challenge)

```bash
sudo certbot certonly --manual --preferred-challenges=dns \
-d "*.harishshetty.xyz"
```

Expected Output:

```
_acme-challenge.harishshetty.xyz → some-long-random-text
```

Certificates will be stored at:

```
/etc/letsencrypt/live/harishshetty.xyz/fullchain.pem
/etc/letsencrypt/live/harishshetty.xyz/privkey.pem
```
---

### ✅ Verify DNS Record

```bash
nslookup -type=TXT _acme-challenge.harishshetty.xyz
```

Add TXT in the Route53
and Confirm  via the google dns and console
---

## 3️⃣ Configure Nginx for SSL Redirection in the AP-South-1

```bash
#!/bin/bash
# === Update and install dependencies ===
apt update -y
apt install nginx certbot python3-certbot-nginx -y

# === Enable and start Nginx ===
systemctl enable nginx
systemctl start nginx

# === Set up Nginx configuration for harishshetty.xyz ===
cat <<EOF >/etc/nginx/sites-available/harishshetty.xyz
server {
    listen 80;
    server_name *.harishshetty.xyz harishshetty.xyz;
    return 301 https://\$host\$request_uri;
}

server {
    listen 443 ssl;
    server_name *.harishshetty.xyz harishshetty.xyz;

    ssl_certificate /etc/letsencrypt/live/harishshetty.xyz/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/harishshetty.xyz/privkey.pem;

    location / {
        root /var/www/html;
        index index.html index.htm;
    }
}
EOF

# === Remove default site and enable custom site ===
rm -f /etc/nginx/sites-enabled/default
ln -s /etc/nginx/sites-available/harishshetty.xyz /etc/nginx/sites-enabled/

# === Create website content ===
mkdir -p /var/www/html
cat <<EOF >/var/www/html/index.html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>AWS Route 53 Region Test</title>
</head>
<body>
  <h1>AP-South-1-server</h1>
</body>
</html>
EOF

# === Restart Nginx to apply HTTP configuration ===
systemctl restart nginx
```

---

## 6️⃣ Copy SSL Certificates Between Instances

From **Source Instance**:

```bash
scp -i "new-keypair.pem" -r /etc/letsencrypt/live/harishshetty.xyz ubuntu@ec2-13-201-186-36.ap-south-1.compute.amazonaws.com:/tmp/
```

On **Destination Instance**:

```bash
sudo su -
sudo mkdir -p /etc/letsencrypt/live/harishshetty.xyz
sudo cp -r /tmp/harishshetty.xyz/* /etc/letsencrypt/live/harishshetty.xyz/
systemctl restart nginx
```

Connect via SSH:

```bash
ssh -i "new-keypair.pem" ubuntu@ec2-13-201-186-36.ap-south-1.compute.amazonaws.com
```
## 🟢 1️⃣ Simple Routing Policy

> Sends all traffic to **a single region or endpoint**.
> Ideal for a simple website or one-region deployment.

```
 🌍  Internet Users
         |
         ▼
   +----------------+
   |   Route 53     |
   |  Simple Record |
   +----------------+
         |
         ▼
 ┌────────────────────┐
 | EC2 (ap-south-1)   |
 | Nginx + SSL + HTML |
 | <h1>AP-SOUTH-1</h1>|
 └────────────────────┘
```

✅ **Use Case:** Single-region static or dynamic site.
💡 Example DNS record:
`harishshetty.xyz → EC2 (ap-south-1)`

---
[![Video Tutorial](https://github.com/harishnshetty/image-data-project/blob/b0cd7327a0d04ac52c0cfd8067d8622c2ee775f6/1Simple.jpg)](https://youtu.be/KwKtMHBQXk4)
---

```bash
I=1
while true
do
  curl -sL https://app.harishshetty.xyz | grep -i '\-server'
  echo "The Count value is $I"
  I=$((I+1))
  sleep 1
done

```