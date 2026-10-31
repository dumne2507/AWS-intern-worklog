---
title: "Week 4 Worklog"
date: 2026-10-05
weight: 4
chapter: false
pre: " <b> 1.4. </b> "
---

## Week 4 Objectives: IAM & Authorization

- **IAM Identity Center:** Configure AWS Single Sign-On (SSO) to centralize workforce login management across AWS Organization accounts, enabling Multi-Factor Authentication (MFA).
- **IAM Policies Advanced:** Author custom JSON policies applying Condition keys (`aws:SourceIp`) to restrict S3 bucket access to specific IP ranges.
- **Permission Boundaries:** Establish maximum permission boundaries for IAM Users and Roles, preventing privilege escalation.
- **IAM Roles for EC2:** Attach an IAM Role directly to an EC2 instance to grant programmatic access to S3 and DynamoDB without long-term access keys.
- **AWS KMS & Data Encryption:** Provision a Customer Managed Key (KMS Key) and apply encryption at rest to EBS Volumes and Amazon S3 Buckets.
- **Comprehensive Lab 4:** Architect and deploy an EC2 instance holding zero hardcoded credentials, authenticating via IAM Role to retrieve runtime secrets from AWS Secrets Manager.

### Tasks Planned for This Week

| Day | Topic | Task | Start Date | End Date | Reference |
| :---: | :--- | :--- | :---: | :---: | :--- |
| **Mon** | **IAM Identity Center** | Configure AWS SSO, set up the Access Portal, and register virtual MFA devices | 05/10/2026 | 05/10/2026 | [IAM Identity Center](https://docs.aws.amazon.com/singlesignon/) |
| **Tue** | **IAM Policies Advanced** | Author custom JSON policy with `aws:SourceIp` condition key to restrict S3 bucket access | 06/10/2026 | 06/10/2026 | [IAM Policy Reference](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies.html) |
| **Wed** | **Permission Boundaries** | Set up permission boundaries to enforce maximum privilege limits across AWS services | 07/10/2026 | 07/10/2026 | [Permissions Boundaries](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_boundaries.html) |
| **Thu** | **IAM Roles for EC2** | Attach IAM Role to EC2 and verify credential-free S3/DynamoDB access via IMDS | 08/10/2026 | 08/10/2026 | [IAM Roles for EC2](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/iam-roles-for-amazon-ec2.html) |
| **Fri** | **AWS KMS & Encryption** | Provision KMS Key (`Key-KMS-LabT6`) and configure at-rest encryption for EBS & S3 | 09/10/2026 | 09/10/2026 | [AWS KMS](https://docs.aws.amazon.com/kms/) |
| **Sat** | **Comprehensive Lab 4** | Deploy EC2 with zero hardcoded credentials, retrieving secrets from Secrets Manager | 10/10/2026 | 10/10/2026 | [AWS Secrets Manager](https://docs.aws.amazon.com/secretsmanager/) |

### Week 4 Results

- Successfully configured AWS IAM Identity Center with access portal URL `ssoins-72232edfad9c07c3.portal.us-east-1.app.aws`, registering virtual MFA device `Admin-Users's MFA 1` for administrator user `Dũng Nguyễn Mạnh`.
- Implemented advanced IAM Policy with `aws:SourceIp` condition key for user `LabT3-Test`, verifying that unauthorized requests were rejected (`Insufficient permissions`) while authorized requests accessed `LabT3.html` seamlessly.
- Successfully enforced Permission Boundaries on user `LabT3-Test`: despite broad IAM permissions, actions outside the boundary (EC2 and VPC operations) returned `Access denied` and `DescribeAccountAttributes / DescribeVpcs` authorization errors.
- Deployed EC2 instance `Lab-T5` (`i-04c89eaec696a2cef`) with an attached IAM Role, successfully querying S3 bucket `labt3-test-ip-public` and DynamoDB `list-tables` without configuring static `~/.aws/credentials`.
- Provisioned Customer Managed Key `Key-KMS-LabT6` (`1c0663df-33b2-437c-a98f-cc38935054f8`), encrypting an EBS Volume and enforcing SSE-KMS default encryption with S3 Bucket Key on `labt6-s3-kms`.
- Completed Comprehensive Lab 4: launched EC2 instance `EC2-Secret-Role-T7` (`i-032c941e009326a97`), calling `aws secretsmanager get-secret-value` to securely retrieve secret `secret-labt7` entirely in memory without writing credentials to disk.

---

#### Exercise 1: IAM Identity Center — AWS SSO Configuration & Centralized MFA

Provisioned and configured **AWS IAM Identity Center** (formerly AWS Single Sign-On) in Region `us-east-1` for centralized workforce identity and multi-account access governance:
- **Access Portal URL:** `https://ssoins-72232edfad9c07c3.portal.us-east-1.app.aws/#/security`
- **User Account:** `Dũng Nguyễn Mạnh` (Display Name: `Dũng`)
- **Registered MFA Device:** `Admin-Users's MFA 1`
- **Device Type:** `Authenticator app` (Two-factor authentication via mobile app)
- **MFA Registration Date:** `Fri Oct 23 2026 11:03:00 GMT+0700 (Indochina Time)`
- **Active Session:** `Desktop - Edge 155.0, Windows 10`
- **Client IP Address:** `115.78.22...` | **Remaining Session Time:** `7 hours 59 minutes`

The AWS Access Portal enables switching across multiple AWS accounts within the Organization with a single authentication step, dynamically vending short-lived STS credentials without storing persistent access keys locally.

> **Screenshot:** ![AWS Access Portal Dashboard and MFA Configuration](/images/1-Worklog/week-04/01-iam-identity-center-sso.png)

---

#### Exercise 2: IAM Policies Advanced — Custom Policy & IP Restriction with Condition Keys

Created a dedicated IAM test user named **`LabT3-Test`** within AWS Account `468502414950` (`Manhdung2507`) to evaluate **Condition Keys**.

> **Screenshot:** ![IAM Users List in Management Console](/images/1-Worklog/week-04/02-iam-policy-json-editor.png)

Authored a custom IAM JSON policy attached to user `LabT3-Test`, utilizing the `aws:SourceIp` condition key to strictly restrict `s3:ListBucket` and `s3:GetObject` permissions against bucket **`labt3-test-ip-public`** to authorized corporate IP CIDRs:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowS3AccessFromCorporateIPOnly",
      "Effect": "Allow",
      "Action": [
        "s3:ListBucket",
        "s3:GetObject"
      ],
      "Resource": [
        "arn:aws:s3:::labt3-test-ip-public",
        "arn:aws:s3:::labt3-test-ip-public/*"
      ],
      "Condition": {
        "IpAddress": {
          "aws:SourceIp": "115.78.22.0/24"
        }
      }
    }
  ]
}
```

Executed verification testing across two distinct scenarios:
- **Scenario 1 (Unauthorized IP):** Accessing bucket `labt3-test-ip-public` with `LabT3-Test` from outside the whitelisted CIDR range prompted an immediate red error banner: *"Insufficient permissions to list objects. After you or your AWS administrator has updated your permissions to allow the s3:ListBucket action, refresh the page."*

> **Screenshot:** ![S3 Access Denied Due to IP Mismatch in Condition Key](/images/1-Worklog/week-04/03-iam-condition-keys.png)

- **Scenario 2 (Authorized IP):** Accessing from the approved IP address listed in the Condition block, user `LabT3-Test` successfully listed bucket objects and retrieved file **`LabT3.html`** (Type: `html`, Size: `0 B`, Storage class: `Standard`).

> **Screenshot:** ![S3 Object List Accessible from Allowed IP](/images/1-Worklog/week-04/04-iam-policy-test-result.png)

---

#### Exercise 3: Permission Boundaries — Enforcing Maximum Effective Permissions

Configured **Permission Boundaries** on user **`LabT3-Test`** (`arn:aws:iam::468502414950:user/LabT3-Test`). The boundary policy restricted permissible actions solely to Amazon S3, targeting bucket **`labt3-test-ip-public`** in region `us-east-1`.

> **Screenshot:** ![S3 Bucket labt3-test-ip-public Within Permission Boundary](/images/1-Worklog/week-04/05-permission-boundary-setup.png)

Conducted strict verification of boundary enforcement against non-whitelisted AWS services:
- **Verification on EC2 Dashboard:** When user `LabT3-Test` navigated to the EC2 console, all operational widgets returned **`Access denied`**:
  - `Dedicated Hosts:` ❌ Access denied
  - `Key pairs:` ❌ Access denied
  - `Security groups:` ❌ Access denied
  - `Auto Scaling Groups:` ❌ Access denied
  - `Elastic IPs:` ❌ Access denied
  - `Load balancers:` ❌ Access denied
  - `Instances (running):` ❌ Access denied
  - `Volumes:` ❌ Access denied
  - `Service health:` ❌ *You don't have permission to health:DescribeEvents.*

> **Screenshot:** ![EC2 Management Console Blocked by Permission Boundary](/images/1-Worklog/week-04/06-permission-boundary-policy.png)

- **Verification on VPC Dashboard:** Navigating to the VPC console produced an error notification:
  - Error: *"User: arn:aws:iam::468502414950:user/LabT3-Test is not authorized to perform: ec2:DescribeAccountAttributes because no permissions boundary allows the ec2:DescribeAccountAttributes action"*

> **Screenshot:** ![DescribeAccountAttributes Action Denied by Permission Boundary](/images/1-Worklog/week-04/07-permission-boundary-deny-test.png)

- **Verification on Your VPCs List:** Querying the VPC inventory was likewise intercepted:
  - Error: *"User: arn:aws:iam::468502414950:user/LabT3-Test is not authorized to perform: ec2:DescribeVpcs because no permissions boundary allows the ec2:DescribeVpcs action"*

> **Screenshot:** ![DescribeVpcs Intercepted and Denied by Boundary](/images/1-Worklog/week-04/08-permission-boundary-result.png)

The experiment confirmed: regardless of how permissive an identity policy might be, any action excluded from the **Permission Boundary** results in an implicit deny, effectively capping delegation privilege limits.

---

#### Exercise 4: IAM Roles for EC2 — Credential-Free Service Access

Launched an EC2 instance dedicated to role-based credential verification:
- **Instance Name:** **`Lab-T5`**
- **Instance ID:** `i-04c89eaec696a2cef`
- **Operating System:** Amazon Linux 2023
- **Public IPv4:** `54.87.203.146`
- **Private IPv4:** `172.31.34.206`
- **Availability Zone:** `us-east-1`
- **Attached IAM Role:** Grants read permissions for Amazon S3 (`s3:ListAllMyBuckets`, `s3:Get*`) and Amazon DynamoDB (`dynamodb:ListTables`).

Connected to the instance via SSH / EC2 Instance Connect and verified service access directly from the AWS CLI:

```bash
# List S3 Buckets using IAM Role credentials
[ec2-user@ip-172-31-34-206 ~]$ aws s3 ls
2026-10-23 06:13:30 labt3-test-ip-public

# List DynamoDB tables in region us-east-1
[ec2-user@ip-172-31-34-206 ~]$ aws dynamodb list-tables --region us-east-1
{
    "TableNames": []
}
```

Both commands executed successfully without running `aws configure` or maintaining local credentials files (`~/.aws/credentials`). The Instance Metadata Service (IMDSv2) delivers auto-refreshing session credentials, eliminating risk of hardcoded key exposure.

> **Screenshot:** ![EC2 Instance Executing S3 and DynamoDB CLI Commands via IAM Role](/images/1-Worklog/week-04/09-iam-role-ec2-attach.png)

---

#### Exercise 5: AWS KMS & Storage Encryption — Protecting EBS Volumes & S3 Buckets

Created a Customer Managed Key (**KMS Key**) with alias **`Key-KMS-LabT6`**:
- **KMS Key ID:** `1c0663df-33b2-437c-a98f-cc38935054f8`
- **KMS Key ARN:** `arn:aws:kms:us-east-1:468502414950:key/1c0663df-33b2-437c-a98f-cc38935054f8`
- **Region:** `us-east-1` (N. Virginia)

Applied this KMS Key across primary storage tiers:
- **EBS Volume Encryption:** Created a block storage volume with encryption enabled. The EC2 Volumes dashboard confirmed:
  - **Status check:** `Okay`
  - **Encryption:** `Encrypted`
  - **KMS key ID:** `1c0663df-33b...` (Mapped to alias `Key-KMS-LabT6`)

> **Screenshot:** ![EBS Volume Encrypted with Customer Managed Key](/images/1-Worklog/week-04/10-secrets-manager-create.png)

- **S3 Bucket Encryption:** Created S3 Bucket named **`labt6-s3-kms`** configured with **Default encryption**:
  - **Encryption type:** `Server-side encryption with AWS Key Management Service keys (SSE-KMS)`
  - **Encryption key ARN:** `arn:aws:kms:us-east-1:468502414950:key/1c0663df-33b2-437c-a98f-cc38935054f8`
  - **Bucket Key:** `Enabled` (Drastically cuts KMS request rates, lowering KMS API costs by up to 99%)
  - **Blocked encryption types:** `Server-Side Encryption with Customer-Provided Keys (SSE-C)`

> **Screenshot:** ![S3 Bucket labt6-s3-kms Configured with Default SSE-KMS Encryption](/images/1-Worklog/week-04/11-secrets-manager-rotate.png)

---

#### Exercise 6: Comprehensive Lab 4 — Zero-Credential EC2 Architecture with AWS Secrets Manager

Engineered a production-ready security architecture: Deployed an EC2 application server that contains zero hardcoded database passwords or connection strings. The instance assumes an IAM Role authorized to call **AWS Secrets Manager**:
- **EC2 Instance Name:** **`EC2-Secret-Role-T7`**
- **Instance ID:** `i-032c941e009326a97`
- **Operating System:** Amazon Linux 2023
- **Public IPv4:** `100.54.219.219`
- **Private IPv4:** `172.31.7.232`
- **Secret Name:** `secret-labt7`
- **Secret ARN:** `arn:aws:secretsmanager:us-east-1:468502414950:secret:secret-labt7-JCzzxK`
- **Version ID:** `dfba26c0-ef0f-4379-8c7a-bd024eca8ef3`
- **Version Stages:** `["AWSCURRENT"]`

From the terminal of `EC2-Secret-Role-T7`, fetched the secret securely at runtime:

```bash
# Connect to EC2-Secret-Role-T7 instance (IP: 172.31.7.232)
Last login: Sat Oct 31 15:19:14 2026 from 18.206.107.29

# Call AWS Secrets Manager API to fetch secret at runtime
[ec2-user@ip-172-31-7-232 ~]$ aws secretsmanager get-secret-value --secret-id secret-labt7 --region us-east-1
{
    "ARN": "arn:aws:secretsmanager:us-east-1:468502414950:secret:secret-labt7-JCzzxK",
    "Name": "secret-labt7",
    "VersionId": "dfba26c0-ef0f-4379-8c7a-bd024eca8ef3",
    "SecretString": "{\"secretkey123\":\"Secretkey123\"}",
    "VersionStages": [
        "AWSCURRENT"
    ],
    "CreatedDate": "2026-10-31T15:09:04.069000+00:00"
}
```

The system returned the decrypted JSON payload `{"secretkey123":"Secretkey123"}` directly into application memory. This zero-credential model prevents secret compromise from codebase leaks or server compromise.

> **Screenshot:** ![EC2 Successfully Retrieving Runtime Secret from AWS Secrets Manager](/images/1-Worklog/week-04/12-lab4-final-architecture.png)

---

#### Challenges Encountered

| Issue | Root Cause | Resolution |
| :--- | :--- | :--- |
| S3 Bucket inaccessible despite user policy attached | Condition key `aws:SourceIp` blocked requests from non-whitelisted IP addresses | Checked current external IPv4 address and updated CIDR block in the Condition statement |
| `AccessDenied` errors across EC2 and VPC consoles | User restricted by Permission Boundary policy scoped only to S3 | Intended boundary behavior; update boundary policy if EC2/VPC delegation is needed |
| `ec2:DescribeAccountAttributes` error on VPC dashboard | Boundary omitted EC2 account description permissions | Added `ec2:DescribeAccountAttributes` to boundary policy to allow VPC console rendering |
| Unauthorized error calling Secrets Manager from EC2 | Instance Profile role lacked `secretsmanager:GetSecretValue` permission | Added IAM policy granting `secretsmanager:GetSecretValue` on secret ARN `secret-labt7` |
| Increased KMS costs under high S3 request volumes | Each SSE-KMS read operation triggers an independent KMS Decrypt API call | Enabled **Bucket Key** on the S3 bucket to cache data keys and cut KMS API call volume by up to 99% |

### Plan for Week 5: Monitoring & Security

- Study **Amazon CloudWatch**: metrics, log groups, alarms, and dashboards for EC2, RDS, and Lambda.
- Enable **AWS CloudTrail** to audit all API calls across the account with S3 log archiving.
- Configure **Amazon GuardDuty** for automated threat detection using ML-based anomaly analysis.
- Set up **AWS Config** rules to continuously evaluate resource configurations against compliance baselines.
- Explore **AWS Security Hub** to aggregate findings from GuardDuty, Config, and Inspector into a unified dashboard.
- Practice the incident response flow: GuardDuty finding → EventBridge rule → SNS alert → Lambda auto-remediation.
