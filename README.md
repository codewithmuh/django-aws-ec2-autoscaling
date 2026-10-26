# High-Availability Django Deployment on AWS with EC2 Auto Scaling, ALB & RDS

An enterprise architecture and deployment blueprint for running scalable, fault-tolerant **Django** applications on **AWS** using **EC2 Auto Scaling Groups (ASG)**, **Application Load Balancers (ALB)**, **Amazon RDS PostgreSQL**, **Route 53 DNS**, and automated **ACM SSL/TLS**.

[![Release](https://img.shields.io/github/v/release/codewithmuh/django-aws-ec2-autoscaling?style=flat-square&color=blue)](https://github.com/codewithmuh/django-aws-ec2-autoscaling/releases)
[![Stars](https://img.shields.io/github/stars/codewithmuh/django-aws-ec2-autoscaling?style=flat-square&color=gold)](https://github.com/codewithmuh/django-aws-ec2-autoscaling/stargazers)
[![Forks](https://img.shields.io/github/forks/codewithmuh/django-aws-ec2-autoscaling?style=flat-square&color=blueviolet)](https://github.com/codewithmuh/django-aws-ec2-autoscaling/network/members)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg?style=flat-square)](LICENSE)
[![Python](https://img.shields.io/badge/Python-3.10+-3776AB.svg?style=flat-square&logo=python&logoColor=white)](https://python.org)
[![Django](https://img.shields.io/badge/Django-5.0-092E20.svg?style=flat-square&logo=django&logoColor=white)](https://djangoproject.com)
[![AWS](https://img.shields.io/badge/AWS-ALB%20%7C%20ASG%20%7C%20RDS-FF9900.svg?style=flat-square&logo=amazon-aws&logoColor=white)](https://aws.amazon.com)
[![YouTube](https://img.shields.io/badge/YouTube-codewithmuh-red?style=flat-square&logo=youtube)](https://youtube.com/@codewithmuh)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-Muhammad%20Rashid-blue?style=flat-square&logo=linkedin)](https://www.linkedin.com/in/muhammadrashid-daha/)

![Django AWS Auto Scaling Cover](https://github.com/codewithmuh/django-aws-ec2-autoscaling/assets/51082957/8042ab6b-e1fb-46b6-b9cb-fdd22d05291e)

---

## 📑 Table of Contents
- [Architecture & High Availability Topology](#architecture--topology)
- [Repository File Layout](#repository-file-layout)
- [Prerequisites & AWS Security Groups](#prerequisites--security-groups)
- [Step-by-Step Production Setup](#step-by-step-setup)
  - [1. Provision Amazon RDS PostgreSQL Database](#1-provision-amazon-rds-postgresql-database)
  - [2. Configure EC2 Launch Template with UserData](#2-configure-ec2-launch-template-with-userdata)
  - [3. Setup Application Load Balancer & Target Group](#3-setup-application-load-balancer--target-group)
  - [4. Create Auto Scaling Group (ASG)](#4-create-auto-scaling-group-asg)
  - [5. Setup Custom Domain (Route 53) & ACM SSL](#5-custom-domain-route-53--acm-ssl)
- [Scaling Policies & CloudWatch Alarms](#scaling-policies--cloudwatch-alarms)
- [Author & Community](#author)
- [License](#license)

---

## 🏛️ Architecture & Topology

```mermaid
flowchart TD
    Users([Internet Clients]) -->|HTTPS :443| Route53[AWS Route 53 DNS]
    Route53 --> ALB[Application Load Balancer]
    subgraph MultiAZ [AWS Multi-AZ VPC]
        ALB -->|HTTP :80| TG[Target Group: Health Checks]
        subgraph ASG [Auto Scaling Group (Min: 2, Max: 6)]
            TG --> EC2_A[EC2 Instance - AZ 1a<br/>Django + Gunicorn + Nginx]
            TG --> EC2_B[EC2 Instance - AZ 1b<br/>Django + Gunicorn + Nginx]
        end
        EC2_A --> RDS[(Amazon RDS PostgreSQL Multi-AZ)]
        EC2_B --> RDS
    end
```

---

## 📂 Repository File Layout

| Path | Purpose |
| :--- | :--- |
| `userdata.sh` | Automated EC2 Launch Template bootstrap script cloning code, configuring Python virtualenv, Gunicorn, and Nginx. |
| `core/` | Django web application configured with database connection settings and WhiteNoise static asset serving. |
| `gunicorn/` | Systemd service unit definitions for Gunicorn process management. |
| `scripts/` | Shell automation scripts for OS packages, Python dependencies, and service startups. |

---

## 🛡️ Prerequisites & Security Groups

Set up three distinct Security Groups:

| Security Group | Inbound Rules | Outbound | Purpose |
| :--- | :--- | :--- | :--- |
| **ALB-SG** | HTTP `80` & HTTPS `443` from `0.0.0.0/0` | All traffic | Public entrypoint for web clients |
| **EC2-SG** | Port `80` from `ALB-SG` only, SSH `22` from `My IP` | All traffic | Private application instances |
| **RDS-SG** | PostgreSQL `5432` from `EC2-SG` only | None | Private database layer |

---

## 🚀 Step-by-Step Setup

### 1. Provision Amazon RDS PostgreSQL Database
1. Go to AWS RDS Console → Create Database → **PostgreSQL**.
2. Select **Free Tier** or **Standard Create** (Multi-AZ for high availability).
3. Set master username and strong password.
4. Attach `RDS-SG` to allow incoming traffic only from EC2 instances.

### 2. Configure EC2 Launch Template with UserData
1. Open EC2 Dashboard → **Launch Templates** → **Create launch template**.
2. Select **Ubuntu 22.04 LTS / 24.04 LTS** on `t2.micro` or `t3.micro`.
3. In Network settings, assign `EC2-SG`.
4. In **Advanced details → User data**, paste `userdata.sh`. Ensure you replace database credentials and git repository URL.

### 3. Setup Application Load Balancer & Target Group
1. Go to **Target Groups** → Create target group (Instances, Port 80, HTTP, Health check path `/`).
2. Go to **Load Balancers** → Create **Application Load Balancer** (Internet-facing, select all public subnets, attach `ALB-SG`).
3. Add a listener forwarding port 80 to the created Target Group.

### 4. Create Auto Scaling Group (ASG)
1. In EC2 Dashboard → **Auto Scaling Groups** → Create ASG.
2. Select the Launch Template created in Step 2.
3. Select your VPC subnets (across at least 2 Availability Zones for fault tolerance).
4. Attach to existing load balancer → select Target Group.
5. Set Group size: Desired = `2`, Minimum = `2`, Maximum = `6`.

### 5. Custom Domain (Route 53) & ACM SSL
1. In **AWS Certificate Manager (ACM)**, request a public SSL certificate for your domain.
2. In **Route 53**, point your domain's `A` Alias record to the ALB DNS name.
3. On the ALB, add an HTTPS `443` listener with the ACM certificate attached, and set HTTP `80` to redirect to HTTPS `443`.

---

## 📈 Scaling Policies & CloudWatch Alarms

Configure a **Target Tracking Scaling Policy**:
- **Metric Type:** Average CPU Utilization
- **Target Value:** `60%`
- Instances will automatically scale out when traffic surges and scale in during low-traffic periods to minimize AWS hosting costs.

---

## 👨‍💻 Author

**Muhammad Rashid** — Backend & Cloud DevOps Engineer, AWS Community Builder.  
- 🎥 **YouTube:** [@codewithmuh](https://youtube.com/@codewithmuh)  
- 💼 **LinkedIn:** [Muhammad Rashid](https://www.linkedin.com/in/muhammadrashid-daha/)  
- 🌐 **Website:** [codewithmuh.com](https://codewithmuh.com) · [devopsbymuh.com](https://devopsbymuh.com)

⭐ **If you find this high-availability deployment guide helpful, please star the repository!**

---

## 📄 License

This repository is distributed under the [MIT License](LICENSE).
