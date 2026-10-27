---
title: "Week 1 Worklog"
date: 2026-09-14
weight: 1
chapter: false
pre: " <b> 1.1. </b> "
---

## Week 1 Objectives

- Set up an AWS account and configure basic security (MFA, AWS CLI).
- Master IAM: Users, Groups, Policies, and configure cost alerts with AWS Budgets.
- Launch and manage an EC2 virtual server, connect via SSH, configure Elastic IP.
- Work with EBS volumes: create, attach, format, and take Snapshots.
- Create and manage S3 Buckets, configure public/private access permissions.
- Lab synthesis: build a static website on S3 and run a web server on EC2.

### Tasks Planned for This Week

| Day | Topic | Task | Start Date | End Date | Reference |
| --- | --- | --- | --- | --- | --- |
| Mon | AWS & Account | Create AWS account, enable MFA security, create IAM User, configure AWS CLI | 14/09/2026 | 14/09/2026 | [AWS Free Tier](https://aws.amazon.com/free/) |
| Tue | IAM & Budgets | Create IAM User/Group, set basic permissions. Configure cost alerts (AWS Budgets) | 15/09/2026 | 15/09/2026 | [AWS IAM](https://000002.awsstudygroup.com) |
| Wed | Amazon EC2 | Launch EC2 instance, create Key Pair, connect via SSH, practice Elastic IP | 16/09/2026 | 16/09/2026 | [Amazon EC2](https://000004.awsstudygroup.com) |
| Thu | Amazon EBS | Create, attach, format EBS volume on EC2. Take Snapshot for backup | 17/09/2026 | 17/09/2026 | [Amazon EBS](https://docs.aws.amazon.com/ebs/) |
| Fri | Amazon S3 | Create Bucket, upload files, configure public/private access | 18/09/2026 | 18/09/2026 | [Amazon S3](https://000057.awsstudygroup.com) |
| Sat | Lab Synthesis 1 | Build a static website on S3 and run a basic web server on EC2 | 19/09/2026 | 19/09/2026 | |

### Week 1 Results

- Successfully created AWS account, enabled root MFA, created dedicated IAM user for lab, configured AWS CLI successfully.
- Created `Lab-Monthly-Limit` budget ($10/month) with alert triggered at 10% threshold.
- Launched EC2 instance `Lab-Server` (t2.micro, Amazon Linux 2023), SSH connection successful, allocated static Elastic IP.
- Created and attached 2GiB gp3 EBS volume to EC2, formatted xfs, mounted to `/data_lab`, created backup Snapshot successfully.
- Created S3 Bucket `s3-bucket-lab-fptu`, uploaded files, practiced public/private access configurations.
- Completed Lab Synthesis: static website on S3 and web server on EC2.

---

#### Exercise 1: AWS Account Setup & Security

Logged into the **AWS Management Console** with account `Manhdung2507`. The Console Home displays recently visited services: EC2, Billing, CloudWatch, IAM, Aurora/RDS, Lambda, Amazon Bedrock. The **Cost and usage** widget shows `$199.93 USD` remaining credits.

> **Screenshot:** ![AWS Management Console](/images/1-Worklog/week-01/01-aws-console.png)

> **Screenshot:** ![AWS Cost and Usage Dashboard](/images/1-Worklog/week-01/02-aws-cost-dashboard.png)

Activated **MFA (Multi-Factor Authentication)** for the root account under IAM → Security credentials. Assigned a virtual MFA device named `PhoneDung-MFA` (created on 22/09/2026). MFA protects the account even if password credentials are compromised.

> **Screenshot:** ![MFA Device assigned successfully](/images/1-Worklog/week-01/03-mfa-setup.png)

Created an **Access Key** for IAM User `StudentSE194349` to configure the AWS CLI. Note: Secret Access Key is only displayed **once** at creation time and must be saved immediately.

> **Screenshot:** ![Create Access Key for IAM User](/images/1-Worklog/week-01/04-iam-access-key.png)

Configured the AWS CLI on the local machine with `aws configure`. Verified connection using `aws sts get-caller-identity` — output returned correct Account ID and ARN for `StudentSE194349`.

```bash
# Configure AWS CLI
aws configure
# AWS Access Key ID: AKIAW2FHLQZTAZ5FD2V7
# AWS Secret Access Key: ****
# Default region: us-east-1
# Default output format: json

# Verify connection
aws sts get-caller-identity
# Output:
# {
#   "UserId": "AIDAW2FHLQZTO3MK6ZNBK",
#   "Account": "468502414950",
#   "Arn": "arn:aws:iam::468502414950:user/StudentSE194349"
# }

# Test S3 command
aws s3 ls
```

> **Screenshot:** ![AWS CLI configured and tested successfully](/images/1-Worklog/week-01/05-aws-cli-config.png)

IAM User `StudentSE194349` was assigned **2 policies**:
- `AmazonEC2ReadOnlyAccess` — inherited via Group `Lab-Admins`
- `AmazonS3ReadOnlyAccess` — attached directly

> **Screenshot:** ![IAM User Permissions Policies](/images/1-Worklog/week-01/06-iam-policy.png)

---

#### Exercise 2: AWS Budgets Configuration

Created Budget named **`Lab-Monthly-Limit`** with a `$10.00/month` limit. Budgets track and control AWS costs, automatically alerting when spending exceeds defined thresholds.

> **Screenshot:** ![Budget Lab-Monthly-Limit created successfully](/images/1-Worklog/week-01/07-budget-created.png)

Configured **Alert #1**: Triggers when actual cost (`Actual`) exceeds **10% of budget** (`$1.00`). Notifications sent via Email, AWS SNS, and AWS Chatbot.

> **Screenshot:** ![Budget details and alert configuration](/images/1-Worklog/week-01/08-budget-detail.png)

---

#### Exercise 3: Launch Amazon EC2

Launched EC2 instance with the following specifications:

- **Name:** `Lab-Sever`
- **AMI:** Amazon Linux 2023
- **Instance type:** `t2.micro` (Free Tier eligible)
- **Instance ID:** `i-08023703be10a8d23`
- **Public IPv4:** `54.147.198.241`
- **Private IPv4:** `172.31.92.66`
- **Availability Zone:** `us-east-1b`
- **Status:** ✅ Running

> **Screenshot:** ![EC2 Instance running - Status Running](/images/1-Worklog/week-01/09-ec2-running.png)

Connected via **EC2 Instance Connect** in the web browser (Public subnet access with Managed SSH keys). Status check passed 2/2.

> **Screenshot:** ![Connect to Linux instance UI](/images/1-Worklog/week-01/10-ec2-connect.png)

Connected successfully via SSH from the local terminal using key pair `EC2-Lab.pem`. The terminal displayed the **Amazon Linux 2023** banner and the prompt `[ec2-user@ip-172-31-26-62 ~]$`.

```bash
# Connect to EC2 via SSH
ssh -i "EC2-Lab.pem" ec2-user@100.58.113.158

# Output:
#        #_
#  ~\_  ####_        Amazon Linux 2023
# ~~  \_#####\
# ~~     \###|       https://aws.amazon.com/linux/amazon-linux-2023
# ~~       \#/ ___
# ~~       V~' '->
# ~~~         /
#  ~~._.   _/
#     _/ _/
#   _/m/'
# [ec2-user@ip-172-31-26-62 ~]$
```

> **Screenshot:** ![SSH connected successfully to EC2](/images/1-Worklog/week-01/11-ssh-connected.png)

Allocated **Elastic IP** `3.211.17.203` (static Public IP) to ensure EC2 retains its address across reboots.

> **Screenshot:** ![Elastic IP allocated](/images/1-Worklog/week-01/12-elastic-ip.png)

---

#### Exercise 4: Amazon EBS — Create, Attach, and Format Disk

Created an EBS Volume with specs:

- **Volume ID:** `vol-01b6945e9ce043db8`
- **Size:** 2 GiB
- **Type:** gp3 (General Purpose SSD)
- **IOPS:** 3000 | **Throughput:** 125
- **Availability Zone:** `us-east-1c`
- **Volume state:** ✅ In-use
- **Attached to:** `i-05441fcc8d06a176 (EC2-Lab)` at `/dev/sdb`

> **Screenshot:** ![EBS Volume created and attached to EC2](/images/1-Worklog/week-01/13-ebs-volume.png)

Formatted and mounted the EBS volume to EC2. Formatted with xfs and mounted to `/data_lab`. Verified with `df -h` that `/dev/xvdb` (2.0G) was mounted successfully.

```bash
# Format filesystem as xfs
sudo mkfs -t xfs /dev/xvdb

# Create mount directory
sudo mkdir /data_lab

# Mount volume
sudo mount /dev/xvdb /data_lab

# Verify
df -h
# Filesystem      Size  Used Avail Use% Mounted on
# /dev/xvda1      8.0G  1.6G  6.4G  20% /
# /dev/xvdb       2.0G   47M  1.9G   3% /data_lab
```

> **Screenshot:** ![EBS mount successful - df -h showing /data_lab](/images/1-Worklog/week-01/14-ebs-mount.png)

Created an **EBS Snapshot** for backup:

- **Snapshot ID:** `snap-0d1423504beac3d88`
- **Description:** "Ban Sao O Cung 2GB"
- **Size:** 66.5 MiB | **Volume size:** 2 GiB
- **Status:** ✅ Completed

> **Screenshot:** ![EBS Snapshot completed](/images/1-Worklog/week-01/15-ebs-snapshot.png)

---

#### Exercise 5: Amazon S3 — Bucket Management & Access Control

Uploaded file to S3 Bucket `s3-bucket-lab-fptu` successfully. Configured **Default encryption** using SSE-S3 (Server-side encryption with Amazon S3 managed keys).

> **Screenshot:** ![S3 Upload succeeded - Default Encryption](/images/1-Worklog/week-01/16-s3-upload-success.png)

Bucket contains object: `Screenshot 2026-09-15 122113.png` (40.3 KB, Standard storage class).

> **Screenshot:** ![S3 Bucket with uploaded file](/images/1-Worklog/week-01/17-s3-bucket.png)

Without public configuration, accessing the object URL directly returned **AccessDenied** (403) — S3's secure default.

> **Screenshot:** ![S3 Access Denied error](/images/1-Worklog/week-01/18-s3-access-denied.png)

Configured **Make public** for the object via ACL.

> **Screenshot:** ![Make public configuration for S3 object](/images/1-Worklog/week-01/19-s3-make-public.png)

---

#### Exercise 6: Lab Synthesis 1

**Part A: Static Website on S3**

Created `index.html` with: *"Xin chao, day la website tinh tren AWS S3! He thong phong thu mang dang hoat dong tot."* Uploaded to S3 and enabled **Static Website Hosting**.

> **Screenshot:** ![Static website running on S3](/images/1-Worklog/week-01/21-s3-static-website.png)

**Part B: Web Server on EC2**

Configured **Security Group** Inbound rules:
- SSH (TCP port 22) — Source: `0.0.0.0/0`
- HTTP (TCP port 80) — Source: `0.0.0.0/0`

> **Screenshot:** ![Security Group Inbound Rules SSH + HTTP](/images/1-Worklog/week-01/23-security-group.png)

Initially, accessing EC2 via `100.58.113.158` timed out because port 80 was closed. After updating Inbound rules, the web server loaded successfully.

> **Screenshot:** ![Timeout error before opening port 80](/images/1-Worklog/week-01/22-ec2-web-server.png)

> **Screenshot:** ![Web Server running successfully on EC2](/images/1-Worklog/week-01/24-ec2-web-success.png)

---

#### Challenges Encountered

| Issue | Resolution |
| :--- | :--- |
| S3 returned `AccessDenied` (403) | Granted **Make public** via ACL or added Bucket Policy |
| EC2 web server timed out in browser | Security Group missing **Inbound port 80 (HTTP)** — added HTTP rule for `0.0.0.0/0` |
| SSH port 22 open to `0.0.0.0/0` | Changed source to **My IP** to restrict access |
| EBS volume not visible after attach | Ran `lsblk` and waited a few seconds for OS device detection |

### Plan for Week 2

- Study VPC: Subnets, Route Tables, Internet Gateway, NAT Gateway.
- Create custom VPC with public and private subnets.
- Deploy EC2 instances inside VPC.
- Explore Load Balancers and Auto Scaling Groups.
