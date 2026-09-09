# 🛡️ CloudSentinel — Cloud Security Detection & Auto-Response Platform

CloudSentinel is a highly-available, self-defending web platform built end-to-end on AWS. Beyond serving traffic reliably, it actively watches for attackers: it exposes convincing decoy endpoints, logs every probe, and pushes a real-time e-mail alert to the security team the moment something suspicious happens.

Built as part of the **Digital Egypt Pioneers Initiative (DEPI) — AWS Cloud Architecting Track**.

---

## 📌 Table of Contents

- [Architecture](#-architecture)
- [Core Features](#-core-features)
- [Tech Stack](#-tech-stack)
- [Implementation Walkthrough](#-implementation-walkthrough)
  1. [VPC & Networking](#1-vpc--networking)
  2. [Security Groups](#2-security-groups)
  3. [S3 Bucket](#3-s3-bucket)
  4. [SNS Alerting](#4-sns-alerting)
  5. [RDS Database](#5-rds-database)
  6. [EC2 Instance](#6-ec2-instance)
  7. [SSM Session Manager](#7-ssm-session-manager)
  8. [Deploying & Testing the App](#8-deploying--testing-the-app)
  9. [Application Load Balancer](#9-application-load-balancer)
  10. [Auto Scaling Group](#10-auto-scaling-group)
- [Application Code](#-application-code)
- [Testing](#-testing)
- [Team](#-team)

---

## 🏗 Architecture

The detection-and-response concept implemented across the project — activity is captured, filtered, and pushed to the security team as an alert.

> 📎 Add `architecture-concept.png` to the image folder below to display the diagram here.

**Traffic flow:** Internet → ALB → Auto Scaling Group (EC2, private subnets) → RDS (private subnet). Alerts flow: decoy hit → application log → SNS topic → e-mail.

---

## ✨ Core Features

| Feature | Description |
|---|---|
| 🌐 Isolated Multi-AZ Network | Custom VPC with public/private subnets across two Availability Zones |
| 🔒 Least-Privilege Security Groups | Dedicated ALB, EC2, and RDS security groups, each scoped to only what it needs |
| 🎭 Deception-Based Detection | Fake `/admin`, `/debug`, `/internal`, `/config`, `/backup`, `/.env` routes that bait and log attacker behaviour |
| 📣 Real-Time Alerting | Amazon SNS pushes an e-mail the instant a decoy path is triggered |
| 🔑 Keyless Server Access | AWS Systems Manager (SSM) Session Manager — no SSH keys, no open port 22 |
| ⚖️ High Availability | Application Load Balancer + Auto Scaling Group (2–4 instances) |
| 🗄 Managed Data Layer | RDS (MySQL) for application data, S3 for evidence/log storage |

---

## 🧰 Tech Stack

**AWS:** VPC, EC2, RDS (MySQL), S3, SNS, Systems Manager (SSM), Application Load Balancer, Auto Scaling
**Application:** Python 3, Flask
**Access:** IAM-authenticated Session Manager (no SSH)

---

## 🪜 Implementation Walkthrough

### 1. VPC & Networking

A custom VPC spans two Availability Zones with public/private subnets, route tables, and an Internet Gateway.

| Subnets | Internet Gateway |
|---|---|
| ![Subnets](<WhatsApp Unknown 2026-09-01 at 7.45.07 AM/WhatsApp Image 2026-09-01 at 12.37.31 AM (1).jpeg>) | ![Internet Gateway](<WhatsApp Unknown 2026-09-01 at 7.45.07 AM/WhatsApp Image 2026-09-01 at 12.37.31 AM.jpeg>) |

| Public Route Table | VPC Resource Map |
|---|---|
| ![Route table](<WhatsApp Unknown 2026-09-01 at 7.45.07 AM/WhatsApp Image 2026-09-01 at 12.37.32 AM.jpeg>) | ![Resource map](<WhatsApp Unknown 2026-09-01 at 7.45.07 AM/WhatsApp Image 2026-09-01 at 12.37.33 AM.jpeg>) |

### 2. Security Groups

Three dedicated security groups enforce least-privilege access between the ALB, EC2 web tier, and RDS database.

| ALB Security Group | EC2 Security Group |
|---|---|
| ![ALB SG](<WhatsApp Unknown 2026-09-01 at 7.45.07 AM/WhatsApp Image 2026-09-01 at 12.37.32 AM (1).jpeg>) | ![EC2 SG](<WhatsApp Unknown 2026-09-01 at 7.45.07 AM/WhatsApp Image 2026-09-01 at 12.37.34 AM.jpeg>) |

| RDS Security Group | All Security Groups |
|---|---|
| ![RDS SG](<WhatsApp Unknown 2026-09-01 at 7.45.07 AM/WhatsApp Image 2026-09-01 at 12.37.36 AM.jpeg>) | ![SG overview](<WhatsApp Unknown 2026-09-01 at 7.45.07 AM/WhatsApp Image 2026-09-01 at 12.37.35 AM.jpeg>) |

### 3. S3 Bucket

A dedicated S3 bucket stores security artifacts and logs collected by the platform.

![S3 bucket](<WhatsApp Unknown 2026-09-01 at 7.45.07 AM/WhatsApp Image 2026-09-01 at 12.37.37 AM.jpeg>)

### 4. SNS Alerting

An SNS topic pushes an e-mail notification to the security team the instant a suspicious event is logged.

| SNS Topic & Subscription | Confirmation E-mail |
|---|---|
| ![SNS topic](<WhatsApp Unknown 2026-09-01 at 7.45.07 AM/WhatsApp Image 2026-09-01 at 12.37.38 AM.jpeg>) | ![Confirmation email](<WhatsApp Unknown 2026-09-01 at 7.45.07 AM/WhatsApp Image 2026-09-01 at 12.37.39 AM.jpeg>) |

**Alert flow:** Decoy path hit → Event logged (`security.log`) → SNS topic notified → E-mail sent to security team

### 5. RDS Database

A private MySQL instance on Amazon RDS backs the application, reachable only from the EC2 security group.

![RDS instance](<WhatsApp Unknown 2026-09-01 at 7.45.07 AM/WhatsApp Image 2026-09-01 at 12.37.40 AM.jpeg>)

### 6. EC2 Instance

An Amazon Linux 2023 instance is launched into the private subnet with a scoped security group and key pair.

| AMI & Instance Type | Key Pair & Network Settings |
|---|---|
| ![EC2 launch AMI](<WhatsApp Unknown 2026-09-01 at 7.45.07 AM/WhatsApp Image 2026-09-01 at 12.37.41 AM.jpeg>) | ![EC2 network settings](<WhatsApp Unknown 2026-09-01 at 7.45.07 AM/WhatsApp Image 2026-09-01 at 12.37.43 AM.jpeg>) |

### 7. SSM Session Manager

No SSH keys, no open port 22 — the instance is reached through an IAM-authenticated, browser-based shell.

![SSM Session Manager](<WhatsApp Unknown 2026-09-01 at 7.45.07 AM/WhatsApp Image 2026-09-01 at 12.37.44 AM.jpeg>)

### 8. Deploying & Testing the App

Inside the session, the Flask app is deployed and quietly logs any visit to a decoy route.

```bash
sudo dnf update -y
sudo dnf install -y python3
python3 --version

mkdir -p ~/cloudsentinel
cd ~/cloudsentinel
python3 -m venv venv
source venv/bin/activate
pip install flask

nano app.py   # see app.py below
sudo python3 app.py
```

![Deploying and testing in Session Manager](<WhatsApp Unknown 2026-09-01 at 7.45.07 AM/WhatsApp Image 2026-09-01 at 12.37.47 AM.jpeg>)

### 9. Application Load Balancer

The ALB distributes traffic across instances via a health-checked target group, enabling zero-downtime scaling.

| Target Group | Load Balancer |
|---|---|
| ![Target group](<WhatsApp Unknown 2026-09-01 at 7.45.07 AM/WhatsApp Image 2026-09-01 at 12.37.45 AM.jpeg>) | ![Load balancer](<WhatsApp Unknown 2026-09-01 at 7.45.07 AM/WhatsApp Image 2026-09-01 at 12.37.46 AM.jpeg>) |

### 10. Auto Scaling Group

An ASG keeps 2–4 instances running behind the load balancer, replacing unhealthy nodes automatically.

![Auto Scaling Group](<WhatsApp Unknown 2026-09-01 at 7.45.07 AM/WhatsApp Image 2026-09-01 at 12.37.48 AM.jpeg>)

---

## 💻 Application Code

`app.py` — a Flask app that serves the CloudSentinel status page and silently logs any request to a decoy path.

``python
from flask import Flask, request, render_template_string
from datetime import datetime
import logging

app = Flask(__name__)

logging.basicConfig(
    filename="security.log",
    level=logging.INFO,
    format="%(asctime)s %(message)s"
)

DECOY_PATHS = {
    "/admin": 20,
    "/debug": 20,
    "/internal": 30,
    "/internal/admin": 40,
    "/config": 30,
    "/backup": 30,
    "/.env": 40
}

HTML = """
<!DOCTYPE html>
<html>
<head>
    <title>CloudSentinel</title>
    <style>
        body { font-family: Arial; background: #111827; color: white; padding: 50px; }
        .card { background: #1f2937; padding: 30px; border-radius: 12px; max-width: 800px; margin: auto; }
        h1 { color: #60a5fa; }
        .status { padding: 15px; background: #064e3b; border-radius: 8px; }
    </style>
</head>
<body>
<div class="card">
<h1>CloudSentinel</h1>
<p>Cloud Security Monitoring Platform</p>
<div class="status">Application Status: ONLINE</div>
<h2>Security Operations</h2>
<p>Monitoring: ACTIVE</p>
<p>Threat Detection: ACTIVE</p>
<p>Deception Layer: ACTIVE</p>
</div>
</body>
</html>
"""

@app.route("/")
def home():
    return render_template_string(HTML)

@app.before_request
def security_monitor():
    path = request.path
    if path in DECOY_PATHS:
        score = DECOY_PATHS[path]
        event = {
            "event_type": "DECEPTION_TRIGGER",
            "path": path,
            "risk": score,
            "source_ip": request.remote_addr,
            "user_agent": request.headers.get("User-Agent"),
            "timestamp": datetime.utcnow().isoformat()
        }
        logging.warning("SECURITY_EVENT %s", event)

@app.route("/health")
def health():
    return {"status": "healthy"}

@app.route("/admin")
def admin():
    return "Access denied", 403

@app.route("/debug")
def debug():
    return "Access denied", 403

@app.route("/internal")
def internal():
    return "Access denied", 403

@app.route("/internal/admin")
def internal_admin():
    return "Access denied", 403

@app.route("/config")
def config():
    return "Access denied", 403

@app.route("/backup")
def backup():
    return "Access denied", 403

@app.route("/.env")
def env():
    return "Access denied", 403

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=80)
`



 🧪 Testing

| Test | Result |
|---|---|
| `curl http://localhost/` | Returns the CloudSentinel HTML page |
| `curl http://localhost/health` | `{"status":"healthy"}` |
| Visit a decoy path (e.g. `/admin`) | `403 Access denied` — silently returned to the visitor |
| `cat security.log` | `SECURITY_EVENT` logged with `event_type=DECEPTION_TRIGGER`, `path=/admin`, `risk=20`, source IP & user agent |

---

## 👥 Team

- Khaled Wasef Helil
- Eyad Mohammed Shaltoot
- Samaa Esam Mostafa
- Ali Mahmoud Ali
- Mohamed Fathy Mohamed
- Youssef Saber Salama
