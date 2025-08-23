# 📅 30-Day AWS SAA-C03 Study Plan

---

## Week 1: AWS Foundations & Core Compute/Networking

**Day 1**
- 📘 [AWS Exam Guide (SAA-C03)](https://aws.amazon.com/certification/certified-solutions-architect-associate/)
- 🎥 [AWS Training – Exam Readiness (Free)](https://skillbuilder.aws/)
- 📘 Study: IAM basics (users, groups, policies, MFA)
- 🧪 Lab: [IAM Hands-on](https://docs.aws.amazon.com/IAM/latest/UserGuide/getting-started_create-admin-group.html)

**Day 2**
- 📘 Study: IAM advanced (roles, cross-account access, federation)
- 🎥 Udemy (Stephane Maarek) → IAM section *(optional paid)*
- 🧪 Lab: Create IAM roles for EC2, test least-privilege access

**Day 3**
- 📘 Study: EC2 basics (instances, AMIs, pricing models)
- 🎥 [AWS EC2 Deep Dive – FreeCodeCamp](https://youtu.be/Ia-UEYYR44s?t=2815)
- 🧪 Lab: Launch EC2, attach EBS, stop/start, snapshot

**Day 4**
- 📘 Study: EC2 advanced (placement groups, Auto Recovery, load balancing)
- 🧪 Lab: Create EC2 Auto Scaling Group

**Day 5**
- 📘 Study: VPC fundamentals (subnets, route tables, IGW, NAT)
- 🎥 FreeCodeCamp VPC Section (same video as above)
- 🧪 Lab: Build custom VPC with public/private subnets

**Day 6**
- 📘 Study: VPC security (SGs, NACLs, Peering, VPN)
- 🧪 Lab: Create private EC2 and connect via bastion host

**Day 7 (Weekend Deep Dive)**
- 📘 Study: Load Balancing + Auto Scaling (ALB, NLB)
- 🧪 Lab: Deploy EC2 web app with ALB + ASG across AZs
- 📘 Read: [Well-Architected Framework – Reliability Pillar](https://docs.aws.amazon.com/wellarchitected/latest/reliability-pillar/welcome.html)

---

## Week 2: Storage, Databases, and High Availability

**Day 8**
- 📘 Study: S3 basics (storage classes, versioning, lifecycle)
- 🧪 Lab: Create S3 bucket, enable versioning + lifecycle

**Day 9**
- 📘 Study: S3 advanced (replication, encryption, presigned URLs)
- 🧪 Lab: Enable cross-region replication

**Day 10**
- 📘 Study: CloudFront (CDN, signed URLs, caching)
- 🧪 Lab: S3 static website with CloudFront + Route 53

**Day 11**
- 📘 Study: RDS basics (engines, Multi-AZ, Read Replicas)
- 🧪 Lab: Launch MySQL RDS Multi-AZ DB

**Day 12**
- 📘 Study: DynamoDB (basics, GSIs/LSIs, DAX)
- 🧪 Lab: Create DynamoDB table with Auto Scaling

**Day 13 (Weekend)**
- 📘 Study: EFS + FSx (when to use, performance modes)
- 🧪 Lab: Mount EFS on EC2

**Day 14 (Weekend)**
- 📘 Study: Disaster Recovery Strategies
- 📘 Read: [AWS Disaster Recovery Whitepaper](https://docs.aws.amazon.com/whitepapers/latest/disaster-recovery-workloads-on-aws/disaster-recovery-workloads-on-aws.pdf)

---

## Week 3: Security, Monitoring & Optimization

**Day 15**
- 📘 Study: KMS, Secrets Manager, CloudHSM
- 🧪 Lab: Encrypt S3 bucket with KMS

**Day 16**
- 📘 Study: Cognito, WAF, Shield
- 🧪 Lab: Add Cognito auth to API Gateway

**Day 17**
- 📘 Study: CloudWatch basics (metrics, logs, alarms)
- 🧪 Lab: Create EC2 CloudWatch alarm

**Day 18**
- 📘 Study: CloudTrail & Config
- 🧪 Lab: Enable CloudTrail, track changes

**Day 19**
- 📘 Study: Cost Optimization (Reserved vs Spot vs On-Demand, Savings Plans)
- 📘 Read: [AWS Cost Optimization Pillar](https://docs.aws.amazon.com/wellarchitected/latest/cost-optimization-pillar/welcome.html)

**Day 20 (Weekend)**
- 📝 Mock Test #1 (Tutorials Dojo / Jon Bonso recommended)
- 🔎 Review all wrong answers & revise weak areas

**Day 21 (Weekend)**
- 📘 Study weak topics (example: ElastiCache, Step Functions)
- 🎥 Quick recap videos (FreeCodeCamp or Maarek’s summary)

---

## Week 4: Final Revision & Exam Prep

**Day 22–23**
- 📘 Study: Advanced Networking (Transit Gateway, Direct Connect, VPN, PrivateLink)
- 🧪 Lab: VPC Peering with 2 regions

**Day 24–25**
- 📘 Read AWS Whitepapers (skim for key takeaways):
  - [Well-Architected Framework](https://docs.aws.amazon.com/wellarchitected/latest/framework/welcome.html)
  - [Storage Options](https://docs.aws.amazon.com/whitepapers/latest/aws-overview/storage-services.html)
  - [Security Best Practices](https://docs.aws.amazon.com/whitepapers/latest/aws-overview/security.html)

**Day 26**
- 📝 Mock Test #2 (timed, 65 questions)
- 🔎 Review mistakes

**Day 27**
- 📝 Mock Test #3
- 📘 Revise weak services

**Day 28**
- 📘 Quick revision: service limits, FAQs, key use cases
- 📝 Flashcards (Anki or self-made notes)

**Day 29**
- 📝 Mock Test #4 (final simulation)
- 🛌 Rest, no heavy study

**Day 30 (Exam Day)**
- ✅ Light review: IAM, S3, EC2, RDS, VPC
- ✅ Remember elimination strategy: secure, cost-optimized, highly available, well-architected
- ✅ Go crush it 🚀
