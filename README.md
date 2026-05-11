# 🚀 AWS VPC Interface Endpoint for SQS (AWS PrivateLink)
<div align="center">

<img src="https://capsule-render.vercel.app/api?type=waving&color=gradient&customColorList=6,11,20&height=220&section=header&text=VPC%20Interface%20Endpoint%20for%20SQS&fontSize=42&fontAlignY=35&desc=Private%20Amazon%20SQS%20Access%20Without%20NAT%20Gateway%20or%20Internet&descAlignY=58&fontColor=ffffff&descSize=16" width="100%"/>

# 🔐 Private Amazon SQS Access from EC2 in a Private Subnet

[![AWS](https://img.shields.io/badge/AWS-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white)](https://aws.amazon.com/sqs/)
[![SQS](https://img.shields.io/badge/Amazon%20SQS-FF4F8B?style=for-the-badge&logo=amazonaws&logoColor=white)](https://aws.amazon.com/sqs/)
[![PrivateLink](https://img.shields.io/badge/AWS%20PrivateLink-7C3AED?style=for-the-badge&logo=amazonaws&logoColor=white)](https://aws.amazon.com/privatelink/)
[![VPC Endpoint](https://img.shields.io/badge/VPC%20Interface%20Endpoint-06B6D4?style=for-the-badge&logo=amazonaws&logoColor=white)](#)
[![IAM](https://img.shields.io/badge/IAM%20Role-DC2626?style=for-the-badge&logo=amazonaws&logoColor=white)](#)

> **Allow EC2 instances in private subnets to communicate with Amazon SQS privately using AWS PrivateLink — no NAT Gateway, no Internet Gateway, and no public IP required.**

[![Cost](https://img.shields.io/badge/Cost-Endpoint%20Hourly%20Charges-orange?style=flat-square)](#)
[![Security](https://img.shields.io/badge/Security-Traffic%20Stays%20Inside%20AWS-success?style=flat-square)](#)
[![Level](https://img.shields.io/badge/Level-Intermediate%20to%20Advanced-blue?style=flat-square)](#)

</div>

---

> 📘 This README is based on the lab screenshots and notes you provided. :contentReference[oaicite:0]{index=0}

---

# 🧠 Problem Statement

You have:

- 🖥️ **EC2-A** in a **Public Subnet** (Bastion Host)
- 🔒 **EC2-B** in a **Private Subnet**
- 📨 **Amazon SQS Queue**
- 🛡️ IAM Role attached to EC2-B with SQS permissions

You want EC2-B to send messages to SQS.

### ❌ Without Interface Endpoint

EC2-B cannot reach the public SQS API endpoint because the private subnet has:

- No Internet Gateway route
- No NAT Gateway
- No public IP

### ✅ With Interface Endpoint

AWS creates a private ENI inside your subnet, and traffic reaches SQS over the AWS backbone using **PrivateLink**.

---
# 🏗️ Architecture Diagram

```mermaid
flowchart TB
    Laptop["💻 Your Laptop"] -->|SSH| IGW["🌍 Internet Gateway"]
    IGW --> EC2A["🖥️ EC2-A (Bastion)<br/>Public Subnet"]

    subgraph VPC["🌐 VPC-A (10.100.0.0/16)"]
        subgraph PUB["🟢 Public Subnet (10.100.0.0/24)"]
            EC2A
        end

        subgraph PRIV["🔒 Private Subnet (10.100.11.0/24)"]
            EC2B["📦 EC2-B<br/>IAM Role: SQS Access"]
            ENI["🔌 Endpoint ENI<br/>Private IP"]
        end

        VPCE["🛡️ Interface Endpoint<br/>com.amazonaws.ap-south-1.sqs"]
    end

    EC2A -->|SSH Hop| EC2B
    EC2B -->|HTTPS 443| ENI
    ENI --> VPCE
    VPCE --> PL["☁️ AWS PrivateLink"]
    PL --> SQS["📨 Amazon SQS"]

    style Laptop fill:#F97316,stroke:#EA580C,color:#fff,stroke-width:2px
    style IGW fill:#8B5CF6,stroke:#7C3AED,color:#fff,stroke-width:2px
    style EC2A fill:#06B6D4,stroke:#0891B2,color:#fff,stroke-width:2px
    style EC2B fill:#3B82F6,stroke:#2563EB,color:#fff,stroke-width:2px
    style ENI fill:#A855F7,stroke:#9333EA,color:#fff,stroke-width:2px
    style VPCE fill:#10B981,stroke:#059669,color:#fff,stroke-width:2px
    style PL fill:#F59E0B,stroke:#D97706,color:#000,stroke-width:2px
    style SQS fill:#EF4444,stroke:#DC2626,color:#fff,stroke-width:2px

    style VPC fill:#1E293B,stroke:#64748B,color:#F8FAFC,stroke-width:1px,stroke-dasharray:5 5
    style PUB fill:#0F172A,stroke:#334155,color:#E2E8F0
    style PRIV fill:#0F172A,stroke:#334155,color:#E2E8F0









