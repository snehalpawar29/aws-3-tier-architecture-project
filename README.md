# ☁️ AWS 3-Tier Architecture

<p align="center">
  <img src="https://img.shields.io/badge/AWS-Cloud%20Architecture-FF9900?style=for-the-badge&logo=amazon-aws&logoColor=white" />
  <img src="https://img.shields.io/badge/EC2-Compute-orange?style=for-the-badge&logo=amazonec2&logoColor=white" />
  <img src="https://img.shields.io/badge/RDS-MySQL-527FFF?style=for-the-badge&logo=amazonrds&logoColor=white" />
  <img src="https://img.shields.io/badge/CloudWatch-Monitoring-8C4FFF?style=for-the-badge&logo=amazoncloudwatch&logoColor=white" />
</p>

<p align="center">
  <strong>Design • Deploy • Scale • Monitor</strong>
</p>

<p align="center">
  A highly available AWS 3-Tier web application architecture
  designed across multiple Availability Zones.
</p>

---

## 🚀 Project Overview

This project demonstrates the design and deployment of a **3-Tier Web Application Architecture on AWS**, separating the application into three independent layers:

* 🌐 **Presentation Tier** — React application served through Nginx
* ⚙️ **Application Tier** — Node.js backend running on EC2
* 🗄️ **Data Tier** — Amazon RDS MySQL database

The infrastructure is designed with **network isolation, load balancing, horizontal scaling, Multi-AZ deployment, HTTPS, DNS routing, and monitoring**.

The goal was to understand how different AWS services work together to build a scalable and highly available application rather than deploying the entire application on a single server.

---

## 🏗️ Architecture

<p align="center">
  <img src="Documentation/Architecture_Diagram_page-0001 (1).jpg" alt="AWS 3-Tier Architecture" width="900"/>
</p>

### 🔄 Request Flow

```text
                         🌍 USER
                           │
                           ▼
                    ☁️ CloudFront
                           │
                           ▼
                    🌐 Route 53
                           │
                           ▼
                  ⚖️ Public ALB
                           │
             ┌─────────────┴─────────────┐
             ▼                           ▼
       🖥️ Presentation              🖥️ Presentation
           EC2                           EC2
       Nginx + React                Nginx + React
             │                           │
             └─────────────┬─────────────┘
                           │
                           ▼
                  ⚖️ Internal ALB
                           │
             ┌─────────────┴─────────────┐
             ▼                           ▼
        ⚙️ Application               ⚙️ Application
            EC2                          EC2
        Node.js + PM2              Node.js + PM2
             │                           │
             └─────────────┬─────────────┘
                           │
                           ▼
                   🗄️ Amazon RDS
                      MySQL
                    Multi-AZ
```

---

# 🧩 Architecture Layers

## 🌐 01 — Presentation Tier

The presentation layer handles incoming web traffic and serves the frontend application.

**Components**

* Amazon EC2
* Nginx
* React
* Internet-facing Application Load Balancer
* Public subnets
* Auto Scaling

### Responsibilities

* Serve the React frontend
* Handle HTTP/HTTPS requests
* Forward API requests toward the application tier
* Distribute traffic across presentation instances
* Scale instances based on workload

---

## ⚙️ 02 — Application Tier

The application layer contains the backend business logic.

**Components**

* Amazon EC2
* Node.js
* PM2
* Internal Application Load Balancer
* Private subnets
* Auto Scaling

### Responsibilities

* Process API requests
* Execute application logic
* Communicate with the database
* Return application responses to the presentation tier
* Support horizontal scaling

The application tier is not directly exposed to the public internet.

---

## 🗄️ 03 — Data Tier

The data layer provides persistent storage for the application.

**Components**

* Amazon RDS
* MySQL
* Multi-AZ deployment
* Private database subnet

### Responsibilities

* Store application data
* Handle database requests from the application tier
* Provide database availability through Multi-AZ configuration

---

# ☁️ AWS Services Used

| Service                          | Purpose                                             |
| -------------------------------- | --------------------------------------------------- |
| 🌐 **Amazon VPC**                | Network isolation and infrastructure foundation     |
| 🔲 **Subnets**                   | Separate public, application and database resources |
| 🖥️ **Amazon EC2**               | Hosts frontend and backend workloads                |
| ⚖️ **Application Load Balancer** | Distributes application traffic                     |
| 📈 **Auto Scaling**              | Automatically adjusts EC2 capacity                  |
| 🗄️ **Amazon RDS**               | Managed MySQL database                              |
| 🔐 **IAM**                       | Identity and access management                      |
| 🌍 **Route 53**                  | DNS and domain routing                              |
| ☁️ **CloudFront**                | Content delivery and edge distribution              |
| 🔒 **AWS Certificate Manager**   | SSL/TLS certificate management                      |
| 📊 **CloudWatch**                | Metrics, alarms and application logs                |

---

# 🌍 Network Architecture

The infrastructure is deployed across **two Availability Zones** to avoid depending on a single Availability Zone.

```text
                         AWS REGION
                       ap-south-1
                            │
              ┌─────────────┴─────────────┐
              │                           │
           AZ - A                       AZ - B
              │                           │
       ┌──────┼──────┐             ┌──────┼──────┐
       │      │      │             │      │      │
     Public  App     DB          Public  App     DB
     Subnet Subnet Subnet        Subnet Subnet Subnet
       │      │      │             │      │      │
      Web    App    RDS           Web    App    RDS
```

### 🔐 Network Separation

The architecture separates resources according to their responsibilities:

**Public subnets**

* Presentation EC2 instances
* Public Application Load Balancer

**Private application subnets**

* Application EC2 instances
* Internal Application Load Balancer

**Private database subnets**

* Amazon RDS

This reduces unnecessary public exposure and provides controlled communication between tiers.

---

# ⚖️ Load Balancing

Two Application Load Balancers are used for different traffic paths.

### 🌐 Internet-Facing ALB

Handles traffic coming from users and distributes it across the presentation tier.

```text
Internet
   │
   ▼
Public ALB
   │
   ├── Presentation EC2
   └── Presentation EC2
```

### 🔒 Internal ALB

Handles communication between the presentation and application tiers.

```text
Presentation Tier
       │
       ▼
Internal ALB
       │
       ├── Application EC2
       └── Application EC2
```

This separation keeps the backend layer away from direct internet access.

---

# 📈 Auto Scaling

Auto Scaling was configured to allow the architecture to respond to workload changes.

The workload test was used to observe how the presentation tier responds when CPU utilization increases.

```text
Normal Workload
      │
      ▼
Existing EC2 Capacity
      │
      │ CPU increases
      ▼
CloudWatch Alarm
      │
      ▼
Auto Scaling
      │
      ▼
Additional EC2 Capacity
```

This demonstrates the relationship between:

**CloudWatch → Auto Scaling → EC2**

---

# 📊 Monitoring & Logging

Amazon CloudWatch was used to monitor the infrastructure and application environment.

### Monitoring included:

* 📈 EC2 CPU utilization
* 🚨 CloudWatch alarms
* 📋 Application logs
* 🔍 Workload testing
* 🔄 Auto Scaling behavior

### Application Logging

Application logging was implemented to make backend activity easier to observe and troubleshoot.

---

## 📸 CloudWatch Monitoring

<p align="center">
  <img src="Documentation/Screenshots/45_CloudWatch_Logs.png" alt="CloudWatch Alarm" width="850"/>
</p>

---

# 🔐 Security

Security was considered at the network and service levels.

### Security approach

* 🔒 Application and database resources are placed in private subnets.
* 🛡️ Security Groups control traffic between tiers.
* 🔑 IAM is used for AWS identity and access management.
* 🌐 Only required public-facing components are exposed.
* 🔒 HTTPS is configured using AWS Certificate Manager.
* 🗄️ Database access is restricted to the application layer.

The architecture follows the principle of allowing communication only where required between the different tiers.

---

# 🌍 Domain & HTTPS

A custom domain was configured using **Amazon Route 53**.

```text
User
 │
 ▼
Custom Domain
 │
 ▼
Route 53
 │
 ▼
CloudFront
 │
 ▼
AWS Application
```

HTTPS was configured using an SSL/TLS certificate managed through **AWS Certificate Manager**.

---

# ☁️ CloudFront

Amazon CloudFront was added as the content delivery layer.

### Benefits demonstrated

* 🌍 Edge-based content delivery
* ⚡ Reduced latency for cached content
* 🔒 HTTPS support
* 🔗 Integration with the application domain

---

# 🛠️ Technology Stack

### ☁️ Cloud

`AWS`

### 🖥️ Compute

`Amazon EC2`

### 🌐 Networking

`VPC` • `Subnets` • `Route Tables` • `Internet Gateway`

### ⚖️ Traffic Management

`Application Load Balancer` • `Auto Scaling`

### 🗄️ Database

`Amazon RDS` • `MySQL`

### 🌍 DNS & Delivery

`Route 53` • `CloudFront`

### 🔐 Security

`IAM` • `Security Groups` • `AWS Certificate Manager`

### 📊 Monitoring

`CloudWatch` • `CloudWatch Logs` • `CloudWatch Alarms`

### 💻 Application

`React` • `Node.js` • `Nginx` • `PM2`

### 🐧 Operating System

`Linux`

### 🔧 Version Control

`Git` • `GitHub`

---

# 📁 Repository Structure

```text
aws-3-tier-architecture-project/
│
├── 📂 Documentation/
│   ├── Architecture diagrams
│   ├── AWS implementation screenshots
│   ├── Configuration documentation
│   └── Project progress documentation
│
├── 📂 frontend/
│   └── React application
│
├── 📂 backend/
│   └── Node.js application
│
└── 📄 README.md
```

---

# 🖼️ Project Documentation

The complete implementation documentation is available inside the:

### 📂 [`Documentation`](./Documentation)

folder.

It contains the detailed project implementation, configuration references, screenshots and progress documentation.

---

# 🧪 Validation & Testing

The architecture was validated through multiple stages of testing, including:

* ✅ Application accessibility
* ✅ Presentation-to-application communication
* ✅ Application-to-database connectivity
* ✅ Load balancer health checks
* ✅ Multi-AZ resource deployment
* ✅ Auto Scaling workload testing
* ✅ CloudWatch alarm behavior
* ✅ Application logging
* ✅ Custom domain resolution
* ✅ HTTPS configuration

---

# 💡 Key DevOps & Cloud Concepts Demonstrated

This project provided hands-on experience with:

* ☁️ AWS cloud architecture
* 🏗️ 3-tier architecture design
* 🌐 VPC networking
* 🔒 Public/private subnet design
* ⚖️ Load balancing
* 📈 Horizontal scaling
* 🗄️ Managed databases
* 🌍 DNS management
* 🔐 HTTPS and certificates
* 📊 Infrastructure monitoring
* 📝 Application logging
* 🐧 Linux administration
* 🔧 Git and GitHub
* 🧩 Troubleshooting distributed applications

---

# 🎯 Project Outcome

This project demonstrates how a traditional full-stack application can be separated into independent infrastructure layers and deployed on AWS with:

**Scalability + Availability + Network Isolation + Load Balancing + Monitoring**

The implementation helped build practical understanding of how AWS compute, networking, database, DNS, security, and monitoring services work together as a complete cloud environment.

---

# 👨‍💻 Author

## Snehal Pawar

**Aspiring DevOps Engineer | AWS | Cloud | Linux**

<p>
  <a href="https://github.com/snehalpawar29">
    <img src="https://img.shields.io/badge/GitHub-snehalpawar29-181717?style=for-the-badge&logo=github" />
  </a>
  <a href="https://www.linkedin.com/in/snehalpawar29/">
    <img src="https://img.shields.io/badge/LinkedIn-Snehal%20Pawar-0A66C2?style=for-the-badge&logo=linkedin" />
  </a>
</p>

---

<p align="center">
  ☁️ <strong>Built to Learn. Designed to Scale.</strong> 🚀
</p>
