# 🚀 AWS Auto Scaling Web App

A production-ready web application deployment on AWS using EC2, AMIs, Launch Templates, and Auto Scaling Groups — designed to automatically scale based on demand and stay highly available.

---

## 📌 What This Project Does

This project sets up a **scalable web server** on AWS that:

- Serves a PHP-based web app via Apache (httpd) on EC2
- Uses a **custom AMI** (`web-app-ami`) so every new instance comes pre-configured
- Spins instances up or down automatically using an **Auto Scaling Group (ASG)**
- Uses a **Launch Template** to standardize how new EC2 instances are created

---

## 🏗️ Architecture Overview

```
User Request
     │
     ▼
Load Balancer (optional)
     │
     ▼
Auto Scaling Group (web-app-ASG)
     │
     ├── EC2 Instance 1  ──┐
     ├── EC2 Instance 2    ├── All launched from: web-app-template
     └── EC2 Instance N  ──┘
              │
              ▼
     Apache + PHP (port 80)
              │
              ▼
     /var/www/html/index.php
```

---

## 🧩 AWS Components Used

| Component | Name | Details |
|---|---|---|
| **AMI** | `web-app-ami` | ID: `ami-04ef49aa3751becee` — Private, owned by account `179698342028` |
| **Launch Template** | `web-app-template` | ID: `lt-059c67bbdcfd9c41b` — Default v1, Latest v3 |
| **Auto Scaling Group** | `web-app-ASG` | Min: 1 / Desired: 1 / Max: 3 |
| **Region** | Asia Pacific (Mumbai) | `ap-south-1` |

---

## ⚙️ Setup Walkthrough

### Step 1 — Prepare Your EC2 Instance

SSH into your base EC2 instance and install Apache + PHP:

```bash
sudo yum update -y
sudo yum install -y httpd php

# Create your web app
sudo bash -c 'cat > /var/www/html/index.php << EOF
<h1>Welcome to My Auto Scaling Web App</h1>
<p>Server hostname:</p>
<p><b><?php echo gethostname(); ?></b></p>
EOF'

# Start Apache and enable it on boot
sudo systemctl start httpd
sudo systemctl enable httpd
```

Verify it's working:

```bash
sudo systemctl status httpd
curl http://localhost
```

You should see something like:
```
<h1>Welcome to My Auto Scaling Web App</h1>
<p>Server hostname:</p>
<p><b>ip-172-31-25-95.ec2.internal</b></p>
```

---

### Step 2 — Create a Custom AMI

Once your instance is configured and working:

1. Go to **EC2 → Instances**, select your instance
2. Click **Actions → Image and templates → Create image**
3. Name it `web-app-ami`
4. Wait for the AMI status to become **Available**

> **Why?** This "bakes" your setup into a reusable image so every Auto Scaling instance starts ready to serve traffic — no manual setup required.

---

### Step 3 — Create a Launch Template

1. Go to **EC2 → Launch Templates → Create launch template**
2. Name it `web-app-template`
3. Set your AMI to `web-app-ami`
4. Choose instance type (e.g., `t2.micro` or `t3.micro`)
5. Attach your key pair and security group (allow inbound port 80 and 22)
6. Click **Create launch template**

---

### Step 4 — Create an Auto Scaling Group

1. Go to **EC2 → Auto Scaling Groups → Create Auto Scaling group**
2. Name it `web-app-ASG`
3. Select your launch template `web-app-template`
4. Choose your VPC and availability zones
5. Set scaling limits:
   - **Minimum**: 1
   - **Desired**: 1
   - **Maximum**: 3
6. (Optional) Attach a Load Balancer for traffic distribution
7. Click **Create Auto Scaling group**

---

## 📊 Current State

```
Auto Scaling Groups:
  ├── web-app-ASG   → 1 instance running (Min: 1 | Max: 3)
  └── web-app-asg   → 1 instance running (Min: 1 | Max: 3)

Launch Template:
  └── web-app-template (v1 default, v3 latest)

AMI:
  └── web-app-ami (Private | ami-04ef49aa3751becee)
```

---

## 🔁 How Auto Scaling Works

- When CPU or traffic **spikes**, ASG launches new instances (up to max: 3)
- When load **drops**, ASG terminates excess instances (down to min: 1)
- All new instances are launched using the `web-app-template`, which uses `web-app-ami` — so they're pre-configured and immediately ready

---

## 🛡️ Security Notes

- The AMI is set to **Private** visibility — only your AWS account can use it
- Ensure your Security Group allows:
  - **Port 80** (HTTP) — for web traffic
  - **Port 22** (SSH) — for management (restrict to your IP)
- Avoid hardcoding credentials in `index.php` or user data scripts

---

## 🧹 Cleanup (Avoid Surprise Bills)

When you're done, clean up in this order:

```
1. Delete Auto Scaling Groups   (EC2 → Auto Scaling Groups)
2. Terminate any running EC2 instances
3. Delete the Launch Template
4. Deregister the AMI           (EC2 → AMIs → Actions → Deregister)
5. Delete the associated EBS Snapshot
```

---

## 📁 Project Structure

```
/var/www/html/
└── index.php          # Main web app served by Apache

AWS Resources:
├── AMI: web-app-ami
├── Launch Template: web-app-template
└── Auto Scaling Group: web-app-ASG
```

---

## 👤 Author

**Prashant** (`prashant_iam`)  
AWS Account: `1160-3630-7538`  
Region: Asia Pacific — Mumbai (`ap-south-1`)