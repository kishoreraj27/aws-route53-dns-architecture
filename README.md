# 🌐 Production-Grade DNS Architecture on AWS Route 53

> A fully production-ready, multi-region DNS infrastructure built as a capstone project — featuring latency-based routing, active-passive failover, private hosted zones, real-time monitoring, and cost optimization.

---

## 📌 Project Overview

This project demonstrates how to architect and implement a **global web application's DNS layer** using AWS Route 53. It covers everything from public hosted zones to private internal service resolution, health-check-driven failover, and CloudWatch observability.

| Component | Technology |
|---|---|
| DNS Management | AWS Route 53 |
| Static Hosting | Amazon S3 + CloudFront |
| Dynamic Backend | Application Load Balancer (ALB) |
| SSL/TLS | AWS Certificate Manager (ACM) |
| Monitoring | Amazon CloudWatch + SNS |
| Internal DNS | Route 53 Private Hosted Zone |

---

## 🏗️ Architecture

```
                        ┌─────────────────────┐
                        │      Route 53        │
                        │  acme-example.com    │
                        └────────┬────────────┘
               ┌─────────────────┼─────────────────┐
               ▼                 ▼                  ▼
     ┌─────────────────┐ ┌────────────────┐ ┌──────────────────┐
     │  Static Content │ │  Dynamic API   │ │ Internal Services│
     │  cdn alias →    │ │  Latency-based │ │  prod.internal   │
     │  CloudFront+S3  │ │  ALB routing   │ │  Private Zone    │
     └─────────────────┘ └───────┬────────┘ └──────────────────┘
                          ┌──────┴──────┐
                          ▼             ▼
                     us-east-1     ap-south-1
                     (Primary)     (Failover)
```

---

## 📁 Project Steps

### Step 1 — Public Hosted Zone Setup
- Created hosted zone: `acme-example.com` (Public)
- Added NS records at domain registrar

### Step 2 — S3 + CloudFront Integration (Static Website)
- Created S3 bucket: `static.acme-example.com` with static website hosting
- Created CloudFront distribution with HTTPS via ACM certificate
- Created **Alias A record**: `cdn` → CloudFront distribution

### Step 3 — Multi-Region ALB Setup (Dynamic Backend)
- Deployed ALBs in two regions:
  - `us-east-1` → `app-us-east-1`
  - `ap-south-1` → `app-ap-south-1`
- Configured **latency-based routing records** for `api.acme-example.com`
- Attached health checks:
  - `api-hc-us-east-1`
  - `api-hc-ap-south-1`

### Step 4 — Failover Configuration
- Configured **active-passive failover** setup
- Set TTL to **30–60 seconds** for rapid DNS propagation
- Validated failover by simulating primary ALB outage

### Step 5 — Private Hosted Zone (Internal Services)
- Created private hosted zone: `prod.internal`
- Added internal records:
  - `db.prod.internal` → Internal database IP
  - `cache.prod.internal` → Internal cache endpoint
- Associated with VPC for internal-only resolution

### Step 6 — Monitoring & Logging
- Enabled **Route 53 query logging** → CloudWatch Log Group: `/route53/acme-example`
- Created CloudWatch alarm: `api-dns-alarm`
  - Condition: `HealthCheckStatus < 1`
  - Action: SNS notification to ops team

### Step 7 — Cost Optimization
| Resource | TTL | Reason |
|---|---|---|
| Static content (CDN) | 3600s | Rarely changes, reduces query cost |
| API endpoint | 60s | Needs fast failover response |
| Alias records | N/A | No Route 53 query charge for alias |

### Step 8 — Validation
```bash
# Verify static website DNS
dig cdn.acme-example.com +short

# Verify API latency routing
dig api.acme-example.com +short

# Verify internal service (from within VPC only)
nslookup db.prod.internal
```

---

## 🌍 Naming Conventions

| Resource Type | Naming Pattern | Example |
|---|---|---|
| Hosted Zone | `<domain>` | `acme-example.com` |
| Subdomain Record | `<service>.<domain>` | `api.acme-example.com` |
| Alias Record | `<service>-alias` | `cdn.acme-example.com` |
| Health Check | `<service>-hc-<region>` | `api-hc-us-east-1` |
| CloudWatch Log Group | `/route53/<zone>` | `/route53/acme-example` |
| Private Hosted Zone | `<env>.internal` | `prod.internal` |

---

## ✅ Key Takeaways

- ✔️ **Public & Private hosted zones** — serving both external users and internal services
- ✔️ **Alias records** — zero-cost resolution for AWS-native resources
- ✔️ **Latency routing** — global users served from the nearest healthy region
- ✔️ **Health checks & failover** — automatic rerouting within 30–60 seconds
- ✔️ **Query logging** — full DNS audit trail in CloudWatch
- ✔️ **Cost optimization** — TTL tuning + alias records reduce unnecessary charges
- ✔️ **Disaster recovery** — simulated region outage, validated failover end-to-end

---

## 🛠️ Tech Stack

![AWS](https://img.shields.io/badge/AWS-Route53-orange?style=flat-square&logo=amazonaws)
![CloudFront](https://img.shields.io/badge/AWS-CloudFront-orange?style=flat-square&logo=amazonaws)
![S3](https://img.shields.io/badge/AWS-S3-orange?style=flat-square&logo=amazonaws)
![CloudWatch](https://img.shields.io/badge/AWS-CloudWatch-orange?style=flat-square&logo=amazonaws)
![ALB](https://img.shields.io/badge/AWS-ALB-orange?style=flat-square&logo=amazonaws)

---

## 👤 Author

**Kishore Raj**

- LinkedIn: https://www.linkedin.com/in/kishoreraj2709
- GitHub: https://github.com/kishoreraj27

---

*This project was completed as part of a hands-on AWS cloud infrastructure learning path.*
