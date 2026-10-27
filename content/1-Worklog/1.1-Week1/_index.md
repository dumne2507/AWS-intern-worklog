---
title: "Week 1 Worklog"
date: 2026-09-14
weight: 1
chapter: false
pre: " <b> 1.1. </b> "
---

## Week 1 Objectives

- Set up an AWS Free Tier account and configure basic security (MFA, AWS CLI).
- Master IAM: Users, Groups, Policies, and configure cost alerts with AWS Budgets.
- Launch and manage an EC2 virtual server, connect via SSH, configure Elastic IP.
- Work with EBS volumes: create, attach, format, and take Snapshots.
- Create and manage S3 Buckets, configure public/private access permissions.
- Lab synthesis: build a static website on S3 or run a web server on EC2.

### Tasks Planned for This Week

| Day | Topic | Task | Start Date | End Date | Reference |
| --- | --- | --- | --- | --- | --- |
| Mon | AWS & Account | Create AWS Free Tier account, enable MFA security, configure AWS CLI | 14/09/2026 | 14/09/2026 | [AWS Free Tier](https://aws.amazon.com/free/) |
| Tue | IAM & Budgets | Create IAM User/Group, set basic permissions. Configure cost alerts (AWS Budgets) | 15/09/2026 | 15/09/2026 | [AWS IAM](https://000002.awsstudygroup.com) |
| Wed | Amazon EC2 | Launch EC2 instance, create Key Pair, connect via SSH, practice Elastic IP | 16/09/2026 | 16/09/2026 | [Amazon EC2](https://000004.awsstudygroup.com) |
| Thu | Amazon EBS | Create, attach, format EBS volume on EC2. Take Snapshot for backup | 17/09/2026 | 17/09/2026 | [Amazon EBS](https://docs.aws.amazon.com/ebs/) |
| Fri | Amazon S3 | Create static bucket, upload files, configure public/private access | 18/09/2026 | 18/09/2026 | [Amazon S3](https://000057.awsstudygroup.com) |
| Sat | Lab Synthesis 1 | Build a static website on S3 or run a basic web server on EC2 | 19/09/2026 | 19/09/2026 | |

### Week 1 Results

- Successfully created an AWS Free Tier account, enabled MFA, and configured AWS CLI.
- Mastered the IAM model: User, Group, Role, Policy and the Least Privilege Principle.
- Launched an EC2 instance, connected via SSH, and configured Elastic IP.
- Created and attached an EBS volume to EC2, formatted and mounted the disk, and took a Snapshot.
- Created an S3 Bucket, uploaded files, and configured public/private access.
- Completed Lab Synthesis: built a static website hosted on S3.

#### Exercise 1: AWS & Account Setup

Created an AWS Free Tier account with a new email. Enabled MFA (Multi-Factor Authentication) on the root account for enhanced security. Installed and configured the AWS CLI on the local machine.

> **Screenshot:** ![Create AWS Free Tier](/images/1-Worklog/week-01/01-aws-free-tier.png)

> **Screenshot:** ![Enable MFA](/images/1-Worklog/week-01/02-mfa-setup.png)

> **Screenshot:** ![Configure AWS CLI](/images/1-Worklog/week-01/03-aws-cli-config.png)

#### Exercise 2: IAM & Budgets

Mastered key IAM concepts:

- **User:** A specific individual with their own credentials
- **Group:** A collection of users sharing a common policy
- **Role:** Temporary permissions granted to a service or application
- **Policy:** A JSON document that defines permissions

Created IAM Users/Groups and assigned permissions following the Least Privilege Principle. Configured AWS Budgets to receive alerts when costs approach Free Tier limits.

> **Screenshot:** ![Create IAM User/Group](/images/1-Worklog/week-01/04-iam-user-group.png)

> **Screenshot:** ![Configure AWS Budgets](/images/1-Worklog/week-01/05-aws-budgets.png)

#### Exercise 3: Amazon EC2

Launched a virtual EC2 instance with the following configuration:

- AMI: Amazon Linux 2023 (Free Tier)
- Instance type: `t2.micro` (1 vCPU, 1GB RAM)
- Key Pair: Created new for SSH access
- Security Group: Opened port 22 (SSH), 80 (HTTP)

Connected via SSH and configured an Elastic IP to maintain a static public address.

```bash
# Connect to EC2 via SSH
ssh -i "key-pair.pem" ec2-user@<Elastic-IP>
```

> **Screenshot:** ![Launch EC2](/images/1-Worklog/week-01/06-ec2-launch.png)

> **Screenshot:** ![SSH Connected](/images/1-Worklog/week-01/07-ssh-connected.png)

> **Screenshot:** ![Elastic IP Configuration](/images/1-Worklog/week-01/08-elastic-ip.png)

#### Exercise 4: Amazon EBS

Created a new EBS volume and attached it to the EC2 instance. Formatted and mounted the disk for use:

```bash
# List block devices
lsblk

# Format the volume
sudo mkfs -t xfs /dev/xvdf

# Create mount point and mount
sudo mkdir /data
sudo mount /dev/xvdf /data
```

Took a Snapshot of the EBS volume for data backup.

> **Screenshot:** ![Create and Attach EBS](/images/1-Worklog/week-01/09-ebs-attach.png)

> **Screenshot:** ![Take Snapshot](/images/1-Worklog/week-01/10-ebs-snapshot.png)

#### Exercise 5: Amazon S3

Created an S3 Bucket and practiced access management:

- Created a bucket with a globally unique name
- Uploaded files to the bucket
- Configured **public access** for specific files
- Configured **private access** and used Presigned URL for temporary sharing

> **Screenshot:** ![Create S3 Bucket](/images/1-Worklog/week-01/11-s3-bucket.png)

> **Screenshot:** ![Upload and Configure Permissions](/images/1-Worklog/week-01/12-s3-permissions.png)

#### Exercise 6: Lab Synthesis 1

Built a static website on S3 using the **Static Website Hosting** feature:

- Enabled Static Website Hosting in bucket Properties
- Uploaded `index.html`
- Configured Bucket Policy to allow public read
- Accessed the website via the S3 endpoint URL

> **Screenshot:** ![Configure Static Website Hosting](/images/1-Worklog/week-01/13-s3-static-website.png)

> **Screenshot:** ![Website Live](/images/1-Worklog/week-01/14-website-live.png)

#### Challenges Encountered

| Issue | Resolution |
| :--- | :--- |
| Could not SSH into EC2 | Checked Security Group — opened inbound port 22 for My IP |
| S3 returned 403 Forbidden | Disabled "Block all public access" first, then added Bucket Policy |
| EBS not visible after attach | Needed to wait a few seconds or run `lsblk` again for OS to detect the device |

### Plan for Week 2

- Study VPC: Subnets, Route Tables, Internet Gateway.
- Create a VPC with public/private subnets.
- Configure NAT Gateway for private subnets.
- Deploy EC2 instances inside a custom VPC.
