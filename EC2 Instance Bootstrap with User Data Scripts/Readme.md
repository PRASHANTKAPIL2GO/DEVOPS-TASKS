# 🏛️ AWS 3-Tier Architecture — Flask + RDS on EC2

A classic 3-tier web application deployed on AWS, where a **Flask web server** (running on EC2) connects to a **MySQL RDS database**, with the EC2 instance using an **IAM role** for secure access to AWS services like S3.

**Live proof:** `http://51.21.193.56:5000` → **"3-Tier Architecture Running Successfully!"**

---

## 📌 What This Project Does

- Deploys a **Python Flask web application** on an EC2 instance
- Connects Flask to a **MySQL database via RDS** using `pymysql`
- Attaches an **IAM role** (`ec2-s3-role`) to the EC2 instance for S3 access without hardcoded credentials
- Demonstrates the full **3-tier architecture** — Presentation → Application → Data — on AWS
- App is accessible publicly over the internet on port 5000

---

## 🏗️ Architecture Overview

```
                        Internet
                            │
                            │ HTTP:5000
                            ▼
              ┌─────────────────────────────┐
              │   Tier 1: Presentation       │
              │   Browser / Client           │
              │   51.21.193.56:5000          │
              └──────────────┬──────────────┘
                             │
                             ▼
              ┌─────────────────────────────┐
              │   Tier 2: Application        │
              │   flask-app-server (EC2)     │
              │   i-0f715dfd7e0e5c032        │
              │   t3.micro                   │
              │   Public IP: 13.60.92.123    │
              │   Private IP: 172.31.47.50   │
              │   IAM Role: ec2-s3-role      │
              │                             │
              │   Flask app (port 5000)      │
              │   → python3 app.py           │
              └──────────────┬──────────────┘
                             │ pymysql / MySQL
                             ▼
              ┌─────────────────────────────┐
              │   Tier 3: Data               │
              │   Amazon RDS (MySQL)         │
              │   Private subnet             │
              │   Security group: ec2-rds-1  │
              └─────────────────────────────┘
```

---

## 🧩 EC2 Instance — flask-app-server

| Property | Value |
|---|---|
| **Instance Name** | `flask-app-server` |
| **Instance ID** | `i-0f715dfd7e0e5c032` |
| **Instance Type** | `t3.micro` |
| **State** | ✅ Running |
| **Public IP** | `13.60.92.123` |
| **Private IP** | `172.31.47.50` |
| **Public DNS** | `ec2-13-60-92-123.eu-north-1.compute.amazonaws.com` |
| **VPC** | `vpc-03434cafac89b43b1` (`new-vpc`) |
| **Subnet** | `subnet-01b7afc53ad4ad84f` |
| **IAM Role** | `ec2-s3-role` |
| **IMDSv2** | Required |
| **Security Groups** | `ec2-rds-2`, `ec2-rds-1` |
| **Launched** | April 11, 2026 at 15:30:32 IST |
| **Region** | Europe — Stockholm (`eu-north-1`) |

---

## 🐍 Flask Application

### Dependencies Installed

```bash
pip3 install flask pymysql
```

Packages installed automatically as dependencies:

| Package | Version | Purpose |
|---|---|---|
| `flask` | 3.1.3 | Web framework |
| `pymysql` | 1.1.2 | MySQL database connector |
| `jinja2` | 3.1.6 | HTML templating |
| `werkzeug` | 3.1.6 | WSGI utility library |
| `click` | 8.1.8 | CLI framework |
| `blinker` | 1.9.0 | Signal support |
| `itsdangerous` | 2.2.0 | Secure cookie signing |
| `markupsafe` | 3.0.3 | Safe string handling |

### Sample `app.py`

```python
from flask import Flask
import pymysql

app = Flask(__name__)

# Database connection
def get_db_connection():
    connection = pymysql.connect(
        host='YOUR_RDS_ENDPOINT',
        user='admin',
        password='YOUR_PASSWORD',
        database='mydb'
    )
    return connection

@app.route('/')
def index():
    return '3-Tier Architecture Running Successfully!'

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000, debug=False)
```

### Running the App

```bash
python3 app.py
```

Output:
```
 * Serving Flask app 'app'
 * Debug mode: off
WARNING: This is a development server. Do not use it in a production deployment.
Use a production WSGI server instead.
 * Running on all addresses (0.0.0.0)
 * Running on http://127.0.0.1:5000
 * Running on http://172.31.47.50:5000
Press CTRL+C to quit
152.59.121.158 - - [01/Mar/2026 09:29:22] "GET / HTTP/1.1" 200 -
152.59.121.158 - - [01/Mar/2026 09:29:23] "GET /favicon.ico HTTP/1.1" 404 -
```

> ⚠️ The `WARNING` about development server is expected — for production, use **Gunicorn** or **uWSGI** behind an Nginx reverse proxy.

---

## 🔐 IAM Role — ec2-s3-role

The EC2 instance has the `ec2-s3-role` IAM role attached, which allows it to access AWS services (like S3) without needing hardcoded credentials in the application code.

This follows AWS best practice: **never put AWS access keys in your application code**. Instead, the app inherits permissions from the instance profile.

```python
# No credentials needed in code — boto3 picks them up from the instance role
import boto3
s3 = boto3.client('s3')  # Works automatically via ec2-s3-role
```

---

## ⚙️ Setup Guide

### Step 1 — Launch the EC2 Instance

1. Go to **EC2 → Launch Instance**
2. Name: `flask-app-server`
3. AMI: Amazon Linux 2023
4. Type: `t3.micro`
5. VPC: `new-vpc`, select a public subnet
6. Enable auto-assign public IP
7. Security group: allow inbound **port 5000** (custom TCP) and **port 22** (SSH)
8. IAM instance profile: `ec2-s3-role`
9. Launch

---

### Step 2 — Connect and Install Dependencies

SSH into your instance:

```bash
ssh -i your-key.pem ec2-user@13.60.92.123
```

Install Python and pip:

```bash
sudo yum update -y
sudo yum install python3-pip -y
```

Install Flask and PyMySQL:

```bash
pip3 install flask pymysql
```

---

### Step 3 — Create the Flask App

```bash
nano app.py
# Paste your app code, save with Ctrl+X → Y → Enter
```

---

### Step 4 — Set Up RDS (MySQL)

1. Go to **RDS → Create database**
2. Engine: MySQL
3. Template: Free tier
4. DB identifier: `mydb`
5. Credentials: set username/password
6. VPC: same as EC2 (`new-vpc`)
7. **Do NOT** make it publicly accessible
8. Security group: create one that allows MySQL (port 3306) only from the EC2 security group

Once created, copy the **RDS endpoint** and update `app.py`:

```python
host='your-rds-instance.xxxxxx.eu-north-1.rds.amazonaws.com'
```

---

### Step 5 — Run the App

```bash
python3 app.py
```

Access it in your browser:
```
http://13.60.92.123:5000
```

You should see: **"3-Tier Architecture Running Successfully!"**

---

## 🔒 Security Group Configuration

### EC2 Security Groups (`ec2-rds-2`, `ec2-rds-1`)

| Type | Protocol | Port | Source | Purpose |
|---|---|---|---|---|
| Custom TCP | TCP | 5000 | `0.0.0.0/0` | Flask app access from internet |
| SSH | TCP | 22 | Your IP | Management access |

### RDS Security Group

| Type | Protocol | Port | Source | Purpose |
|---|---|---|---|---|
| MySQL/Aurora | TCP | 3306 | EC2 security group | Allow only EC2 to talk to RDS |

> Never open RDS port 3306 to `0.0.0.0/0` — always restrict to the EC2 security group.

---

## 🚀 Production Upgrade Path

The current setup runs Flask's built-in dev server, which is fine for learning but not for production. Here's how to upgrade:

```bash
# Install Gunicorn (production WSGI server)
pip3 install gunicorn

# Run with 4 workers
gunicorn -w 4 -b 0.0.0.0:5000 app:app
```

For a proper production setup, add **Nginx** as a reverse proxy in front of Gunicorn:

```
Internet → Nginx (port 80/443) → Gunicorn (port 5000) → Flask app → RDS
```

And consider adding:
- **HTTPS** via AWS Certificate Manager + Load Balancer
- **Auto Scaling** for the EC2 tier
- **RDS Multi-AZ** for database high availability
- **ElastiCache** (Redis/Memcached) for session/caching layer

---

## 🧹 Cleanup

```bash
# 1. Terminate EC2 instance
aws ec2 terminate-instances --instance-ids i-0f715dfd7e0e5c032

# 2. Delete RDS instance (uncheck final snapshot if not needed)
aws rds delete-db-instance \
  --db-instance-identifier mydb \
  --skip-final-snapshot

# 3. Delete security groups (after instances are terminated)
# 4. Remove IAM role if no longer needed
```

---

## 👤 Author

**Prashant** (`prashant_iam`)  
AWS Account: `1140-1518-7882`  
Region: Europe — Stockholm (`eu-north-1`)