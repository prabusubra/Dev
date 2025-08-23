# 🚀 AWS Solutions Architect Associate (SAA-C03) Cheat Sheet

---

## 🌐 Core Networking
- **VPC**
  - Default VPC: public subnets in each AZ
  - Custom VPC: must add IGW for internet access
  - NACLs: stateless, rule numbered, applied at subnet
  - Security Groups: stateful, instance-level
  - VPC Peering: no transitive peering
  - Transit Gateway: hub-and-spoke, transitive routing

- **Route 53**
  - Routing policies: Simple, Weighted, Latency, Failover, Geo, Multi-Value
  - Health checks + DNS failover = HA

---

## 💻 Compute
- **EC2**
  - Pricing: On-Demand, Reserved, Spot, Savings Plans, Dedicated
  - Placement Groups:
    - Cluster → low latency, same AZ
    - Spread → across hardware, max 7 per AZ
    - Partition → large scale, HDFS/HBase

- **Auto Scaling**
  - Dynamic scaling: based on metrics
  - Predictive scaling: machine learning based
  - Cooldown period prevents thrashing

- **Elastic Beanstalk**
  - PaaS, supports EC2, RDS, S3, ELB
  - Handles provisioning, scaling, monitoring

- **Lambda**
  - Event-driven compute, up to 15 min
  - Scaling automatically
  - Max memory: 10 GB
  - VPC support with ENIs

---

## 📦 Storage
- **S3**
  - Storage classes:
    - Standard → frequent access
    - Intelligent-Tiering → auto optimize
    - IA → infrequent access
    - Glacier → archival
  - Versioning: enabled at bucket-level
  - Encryption: SSE-S3, SSE-KMS, SSE-C
  - Strong read-after-write consistency
  - S3 Cross-Region Replication requires versioning

- **EBS**
  - Types: gp3, gp2, io1/io2, st1, sc1
  - Snapshots stored in S3
  - Can be encrypted, can change volume type

- **EFS**
  - NFS protocol, multi-AZ
  - Scales automatically
  - Standard vs Infrequent Access classes

- **FSx**
  - FSx for Windows → SMB
  - FSx for Lustre → HPC

---

## 🗄 Databases
- **RDS**
  - Supports: MySQL, PostgreSQL, MariaDB, Oracle, SQL Server
  - Multi-AZ: HA, synchronous replication
  - Read Replicas: scaling, async replication
  - Backups: automated + snapshots

- **Aurora**
  - MySQL/PostgreSQL compatible
  - 6 copies across 3 AZs
  - Auto-healing, replicas (up to 15)

- **DynamoDB**
  - Key-value / NoSQL
  - Strongly consistent or eventually consistent reads
  - DAX: caching layer
  - TTL for item expiry
  - Global Tables: multi-region replication

- **ElastiCache**
  - Redis / Memcached
  - Use for caching, session management

---

## 🔐 Security
- **IAM**
  - Policies: JSON, least privilege
  - Roles: temporary credentials
  - MFA for root account
  - IAM Access Analyzer for permissions

- **KMS**
  - Customer Managed Keys (CMK)
  - Envelope encryption
  - Automatic key rotation

- **Secrets Manager vs Parameter Store**
  - Secrets Manager: automatic rotation
  - Parameter Store: free, simple secrets

- **Cognito**
  - User Pools (authentication)
  - Identity Pools (temporary AWS creds)

- **Shield / WAF**
  - Shield Standard: automatic DDoS protection
  - Shield Advanced: extra detection + support
  - WAF: block/allow by IP, headers, SQLi, XSS

---

## 📊 Monitoring & Management
- **CloudWatch**
  - Metrics, logs, dashboards, alarms
  - Custom metrics supported

- **CloudTrail**
  - Logs API calls
  - Enabled by default (90 days), store in S3 for longer retention

- **Config**
  - Tracks config changes over time
  - Can trigger remediation

- **Trusted Advisor**
  - Cost, performance, security, service limits checks

---

## 💰 Cost Optimization
- Use **S3 IA/Glacier** for archival
- Use **Reserved Instances / Savings Plans** for steady workloads
- Use **Spot Instances** for fault-tolerant, flexible jobs
- Use **Auto Scaling** to match demand
- Monitor costs with **Cost Explorer & Budgets**

---

## 🏗 High Availability & DR
- **Disaster Recovery Strategies**
  - Backup & Restore → cheapest
  - Pilot Light → minimal core always running
  - Warm Standby → scaled-down version running
  - Multi-Site Active/Active → most expensive, RTO≈0

- **Multi-AZ vs Multi-Region**
  - Multi-AZ → high availability
  - Multi-Region → disaster recovery, global apps

---

## 🎯 Exam Tips
- Look for answers that are:
  - **Secure** → least privilege, encryption
  - **Highly Available** → multi-AZ, multi-region
  - **Cost-Optimized** → Spot, S3 IA, scaling
  - **Managed Service First** → RDS, Lambda > EC2

- Eliminate wrong answers:
  - Too costly
  - Single point of failure
  - Not scalable
  - Insecure
