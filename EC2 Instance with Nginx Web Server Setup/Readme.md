# 🌐 Nginx Web Server on AWS EC2

A straightforward AWS project that sets up **Nginx** as a web server on an EC2 instance (Ubuntu), enables it as a system service so it starts automatically on reboot, and verifies it's serving content — confirmed with a `curl http://localhost` returning **"HELLO AWS"**.

---

## 📌 What This Project Does

- Launches EC2 instances in the Asia Pacific (Mumbai) region
- Installs and configures **Nginx** on an Ubuntu EC2 instance
- Enables Nginx as a **systemd service** (auto-starts on reboot)
- Serves a custom HTML page responding with `HELLO AWS`
- Verifies the server is live using `curl`

---

## 🧩 EC2 Instances

| Name | Instance ID | Type | State | AZ | Status |
|---|---|---|---|---|---|
| `MyTerraformI...` | `i-026312e5294f99469` | t3.micro | ✅ Running | ap-south-1a | 3/3 checks passed |
| `ngnix` | `i-0638074601898df3f` | t2.micro | ✅ Running | ap-south-1a | 2/2 checks passed |
| `ngnix-server` | `i-0a86afeeddc7f4607` | t2.micro | ✅ Running | ap-south-1b | 2/2 checks passed |

> The `ngnix` instance (`i-0638074601898df3f`) is the one configured and verified in this project. It's running on **Ubuntu** in `ap-south-1a`.

---

## ⚙️ Setup Guide

### Step 1 — Launch an EC2 Instance

1. Go to **EC2 → Launch Instance**
2. Name: `ngnix`
3. AMI: **Ubuntu Server 22.04 LTS**
4. Instance type: `t2.micro`
5. Key pair: select or create one
6. Security group: allow **port 80** (HTTP) and **port 22** (SSH) inbound
7. Launch

---

### Step 2 — Connect and Update the System

SSH into your instance:

```bash
ssh -i your-key.pem ubuntu@<your-public-ip>
```

Update packages:

```bash
sudo apt update
sudo apt upgrade -y
```

Output you'll see:
```
340 packages can be upgraded. Run 'apt list --upgradable' to see them.
```

---

### Step 3 — Install Nginx

```bash
sudo apt install nginx -y
```

If already installed, you'll see:
```
nginx is already the newest version (1.24.0-2ubuntu7.6).
0 upgraded, 0 newly installed, 0 to remove and 340 not upgraded.
```

---

### Step 4 — Start and Enable Nginx

Start Nginx immediately:

```bash
sudo systemctl start nginx
```

Enable it to start automatically on every reboot:

```bash
sudo systemctl enable nginx
```

Output:
```
Synchronizing state of nginx.service with SysV service script...
Executing: /usr/lib/systemd/systemd-sysv-install enable nginx
```

---

### Step 5 — Verify Nginx is Running

```bash
sudo systemctl status nginx
```

Expected output:
```
● nginx.service - A high performance web server and a reverse proxy server
     Loaded: loaded (/usr/lib/systemd/system/nginx.service; enabled; preset: enabled)
     Active: active (running) since Fri 2026-04-03 16:28:29 UTC; 1h 17min ago
    Main PID: 577 (nginx)
      Tasks: 2 (limit: 1130)
     Memory: 3.1M (peak: 3.4M)
        CPU: 91ms
     CGroup: /system.slice/nginx.service
             ├─577 "nginx: master process /usr/sbin/nginx -g daemon on; master_process on;"
             └─578 "nginx: worker process"
```

Key things to confirm:
- `Active: active (running)` ✅
- `enabled` in the Loaded line — means it'll survive a reboot ✅
- Master + worker process visible in CGroup ✅

---

### Step 6 — Set Custom Content

Replace the default Nginx page with your own:

```bash
echo "HELLO AWS" | sudo tee /var/www/html/index.html
```

---

### Step 7 — Test It

Test locally on the instance:

```bash
curl http://localhost
```

Output:
```
HELLO AWS
```

Test from the internet using your public IP:

```
http://<your-ec2-public-ip>
```

---

## 📊 Nginx Service Status — Quick Reference

| Check | Command | What to Look For |
|---|---|---|
| Is Nginx running? | `sudo systemctl status nginx` | `active (running)` |
| Is it enabled on boot? | `sudo systemctl is-enabled nginx` | `enabled` |
| Check access logs | `sudo tail -f /var/log/nginx/access.log` | Incoming requests |
| Check error logs | `sudo tail -f /var/log/nginx/error.log` | Any errors |
| Reload config (no downtime) | `sudo systemctl reload nginx` | No output = success |
| Restart Nginx | `sudo systemctl restart nginx` | Brief downtime |

---

## 📁 Important Nginx File Locations

```
/etc/nginx/                    ← Main config directory
├── nginx.conf                 ← Global Nginx config
└── sites-available/
    └── default                ← Default site config

/var/www/html/
└── index.html                 ← Your web content (serve files here)

/var/log/nginx/
├── access.log                 ← All HTTP requests
└── error.log                  ← Errors and warnings
```

---

## 🔒 Security Group — Required Rules

| Type | Protocol | Port | Source | Purpose |
|---|---|---|---|---|
| HTTP | TCP | 80 | `0.0.0.0/0` | Web traffic from internet |
| SSH | TCP | 22 | Your IP | Management access |

> If you plan to add HTTPS later, also open port **443**.

---

## 🚀 What's Next — Nginx as a Reverse Proxy

Nginx's real power is as a **reverse proxy** — sitting in front of an application server like Flask or Node.js:

```nginx
# /etc/nginx/sites-available/myapp
server {
    listen 80;
    server_name your-domain.com;

    location / {
        proxy_pass http://127.0.0.1:5000;  # Forward to Flask app
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

Enable the config:
```bash
sudo ln -s /etc/nginx/sites-available/myapp /etc/nginx/sites-enabled/
sudo nginx -t          # Test config syntax
sudo systemctl reload nginx
```

---

## 🧹 Cleanup

```bash
# Stop and disable Nginx
sudo systemctl stop nginx
sudo systemctl disable nginx

# Terminate EC2 instances from the console or:
aws ec2 terminate-instances \
  --instance-ids i-0638074601898df3f i-0a86afeeddc7f4607 \
  --region ap-south-1
```

---

## 👤 Author

**Prashant** (`prashant_iam`)  
AWS Account: `1160-3630-7538`  
Region: Asia Pacific — Mumbai (`ap-south-1`)