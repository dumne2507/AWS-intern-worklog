---
title: "Week 4 Worklog"
date: 2026-10-05
weight: 4
chapter: false
pre: " <b> 1.4. </b> "
---

{{% notice warning %}}
⚠️ **Note:** The following information is for reference purposes only. Please **do not copy verbatim** for your own report, including this warning.
{{% /notice %}}

## Week 4 Objectives: IAM & Authorization

| # | Goal |
| :---: | :--- |
| 1 | Configure AWS IAM Identity Center (SSO) for centralized login management |
| 2 | Write custom JSON IAM policies with Condition keys to restrict access by IP and time |
| 3 | Set up Permission Boundaries to cap maximum permissions for users and roles |
| 4 | Attach IAM Roles to EC2 instances to securely access S3/DynamoDB without hardcoded keys |
| 5 | Store and auto-rotate database passwords using AWS Secrets Manager |
| 6 | Complete Lab 4: Deploy a zero-credential EC2 architecture using Roles + Secrets Manager |

---

### Tasks Planned for This Week

| Day | Task | Start Date | Completion Date | Reference Material |
| :---: | :--- | :---: | :---: | :---: |
| **2** | IAM Identity Center | 05/10/2026 | 05/10/2026 | AWS Docs – IAM Identity Center |
| **3** | IAM Policies Advanced | 06/10/2026 | 06/10/2026 | AWS Docs – IAM Policy Reference |
| **4** | Permission Boundaries | 07/10/2026 | 07/10/2026 | AWS Docs – Permissions Boundaries |
| **5** | IAM Roles for EC2 | 08/10/2026 | 08/10/2026 | AWS Docs – IAM Roles for EC2 |
| **6** | AWS Secrets Manager | 09/10/2026 | 09/10/2026 | AWS Docs – Secrets Manager |
| **7** | Lab 4 – Integrated | 10/10/2026 | 10/10/2026 | All previous references |

---

### Results Achieved in Week 4

#### Exercise 1: IAM Identity Center — Centralized SSO Login

AWS IAM Identity Center (formerly AWS SSO) provides a single sign-on entry point for all AWS accounts and cloud applications within an AWS Organization. I began by enabling IAM Identity Center in the **us-east-1** region and linking it to the AWS Organization root account. The service creates a dedicated identity store that acts as the authoritative source for user identities.

After enabling the service, I created a Permission Set named `DevReadOnly` that includes `ReadOnlyAccess` as a managed policy. A Permission Set defines what an IAM Identity Center user can do once they log in to a specific account. I then created a user, assigned the Permission Set to that user for the development account, and tested the login flow via the generated AWS access portal URL.

The SSO portal URL follows the pattern `https://<subdomain>.awsapps.com/start`. From this portal, the user can directly switch between AWS accounts with a single click, and temporary session credentials are automatically generated in the background — no long-term access keys are needed.

> **Screenshot:** ![IAM Identity Center SSO Dashboard](/images/1-Worklog/week-04/01-iam-identity-center-sso.png)

---

#### Exercise 2: IAM Policies Advanced — Condition Keys

Standard IAM policies grant or deny actions on resources, but **Condition keys** make policies dynamic. I created a custom policy that allows S3 `GetObject` only when the requester's IP falls within the corporate CIDR block `203.0.113.0/24` using the condition key `aws:SourceIp`.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::my-intern-bucket/*",
      "Condition": {
        "IpAddress": {
          "aws:SourceIp": "203.0.113.0/24"
        }
      }
    }
  ]
}
```

I also experimented with `aws:RequestedRegion` to restrict all API calls to `ap-southeast-1` only, and `aws:CurrentTime` to allow console access only during office hours (08:00–18:00 UTC+7). These Condition keys are evaluated in real time by the IAM policy engine before any action is permitted.

> **Screenshot:** ![IAM Policy JSON Editor with Condition Keys](/images/1-Worklog/week-04/02-iam-policy-json-editor.png)

To verify the policies, I used the **IAM Policy Simulator** tool available in the AWS Console. The simulator lets you select a user or role, pick an action and resource, and see exactly which policy statement matches — whether Allow or Deny — without making a real API call.

> **Screenshot:** ![Condition Key Configuration Detail](/images/1-Worklog/week-04/03-iam-condition-keys.png)

After simulating several scenarios (correct IP, wrong IP, inside office hours, outside office hours), all results matched expectations: requests from outside the allowed CIDR or outside the allowed time window returned `ImplicitDeny`.

> **Screenshot:** ![Policy Simulator Test Result](/images/1-Worklog/week-04/04-iam-policy-test-result.png)

---

#### Exercise 3: Permission Boundaries — Capping Maximum Permissions

A **Permission Boundary** is an advanced IAM feature that sets the maximum permissions an identity-based policy can grant to an IAM entity (user or role). Even if the entity's policy allows an action, if the permission boundary does not allow it, the action is denied. This is especially useful for delegated administration — allowing junior admins to create IAM roles but preventing privilege escalation.

I created a boundary policy named `DeveloperBoundary` that permits only `s3:*`, `dynamodb:*`, and `cloudwatch:GetMetric*`. I then created a new IAM role `AppRole-Dev` and attached this boundary via the AWS CLI:

```bash
aws iam create-role \
  --role-name AppRole-Dev \
  --assume-role-policy-document file://trust-policy.json \
  --permissions-boundary arn:aws:iam::123456789012:policy/DeveloperBoundary
```

> **Screenshot:** ![Permission Boundary Setup in Console](/images/1-Worklog/week-04/05-permission-boundary-setup.png)

After attaching the boundary, I also attached `AdministratorAccess` as the identity-based policy to the same role (intentionally over-permissive). Testing an `ec2:DescribeInstances` call via this role returned `AccessDenied` because EC2 is not in the boundary — demonstrating that the boundary caps permissions regardless of the identity policy.

> **Screenshot:** ![Permission Boundary Policy Details](/images/1-Worklog/week-04/06-permission-boundary-policy.png)

I then modified the boundary to include `ec2:DescribeInstances` and tested again — the call succeeded this time, confirming the boundary is the gating factor.

> **Screenshot:** ![Deny Test Result with Boundary](/images/1-Worklog/week-04/07-permission-boundary-deny-test.png)

Finally, I ran the IAM Policy Simulator with the boundary in scope and confirmed the effective permissions matched only the intersection of the identity policy and the boundary policy.

> **Screenshot:** ![Permission Boundary Final Result](/images/1-Worklog/week-04/08-permission-boundary-result.png)

---

#### Exercise 4: IAM Roles for EC2 — Credential-Free Architecture

Hardcoding AWS credentials (`aws_access_key_id`, `aws_secret_access_key`) into EC2 application code or environment variables is a critical security anti-pattern. The correct approach is to attach an **IAM Role** to the EC2 instance. The EC2 instance metadata service (IMDS) then vends temporary, auto-rotating credentials to any process on the instance.

I created an IAM role named `EC2-S3-DynamoDB-Role` with an EC2 trust policy and attached two managed policies: `AmazonS3ReadOnlyAccess` and `AmazonDynamoDBReadOnlyAccess`. I then launched a new EC2 instance and attached this role during launch, or used the AWS CLI to associate it with a running instance:

```bash
aws ec2 associate-iam-instance-profile \
  --instance-id i-0abcdef1234567890 \
  --iam-instance-profile Name=EC2-S3-DynamoDB-Role
```

> **Screenshot:** ![IAM Role Attached to EC2 Instance](/images/1-Worklog/week-04/09-iam-role-ec2-attach.png)

From inside the EC2 instance, I verified that the AWS CLI automatically uses the role credentials without any configuration:

```bash
# Inside EC2 — no aws configure needed
aws s3 ls s3://my-intern-bucket
aws dynamodb list-tables --region ap-southeast-1
```

Both commands succeeded, and `aws sts get-caller-identity` confirmed the caller identity was the IAM Role, not any IAM user. Temporary credentials are automatically refreshed by the metadata service before expiry.

---

#### Exercise 5: AWS Secrets Manager — Secure Credential Storage & Rotation

Storing database connection strings, API keys, or passwords in plaintext (environment variables, config files, source code) is a major security risk. **AWS Secrets Manager** solves this by storing secrets in encrypted form (using AWS KMS) and providing a programmatic API to retrieve them at runtime.

I created a new secret of type **RDS credentials** containing the username and password for a MySQL RDS instance. Secrets Manager encrypted the value and stored it under the name `prod/myapp/db-password`. From an EC2 instance with the appropriate role, the application retrieves it at startup:

```python
import boto3, json

client = boto3.client('secretsmanager', region_name='ap-southeast-1')
response = client.get_secret_value(SecretId='prod/myapp/db-password')
secret = json.loads(response['SecretString'])
DB_PASSWORD = secret['password']
```

> **Screenshot:** ![Secrets Manager Secret Creation](/images/1-Worklog/week-04/10-secrets-manager-create.png)

I then enabled **automatic rotation** with a 30-day schedule. Secrets Manager uses a Lambda function (auto-created for RDS types) to generate a new password, update it in the RDS instance, and store the new value — all without downtime. The application code does not need to be redeployed; it simply calls `get_secret_value` each time it needs the password.

> **Screenshot:** ![Secrets Manager Rotation Configuration](/images/1-Worklog/week-04/11-secrets-manager-rotate.png)

---

#### Exercise 6: Lab 4 — Zero-Credential EC2 Architecture

The final lab integrated all week's concepts into a production-grade security architecture: an EC2 instance that holds no hardcoded credentials, authenticates via IAM Role, and retrieves its database password dynamically from Secrets Manager at runtime.

The architecture flow is:
1. EC2 instance starts → IMDS provides temporary credentials for `EC2-AppRole`
2. Application calls `secretsmanager:GetSecretValue` for `prod/myapp/db-password`
3. Secrets Manager validates the role's IAM policy → returns encrypted secret
4. KMS decrypts the secret value → application receives plaintext password in memory
5. Application connects to RDS using the retrieved password — never stored to disk

All components were deployed via the AWS CLI and verified end-to-end. The `aws sts get-caller-identity` output confirmed no IAM user credentials were present on the instance.

> **Screenshot:** ![Lab 4 Final Architecture Diagram](/images/1-Worklog/week-04/12-lab4-final-architecture.png)

---

### Challenges & Solutions

| Issue | Root Cause | Solution |
| :---: | :---: | :---: |
| SSO login loop after enabling Identity Center | Missing DNS propagation for the SSO subdomain | Waited 5 minutes and cleared browser cache |
| `AccessDenied` when testing Permission Boundary | Condition key `aws:SourceIp` conflict with boundary | Removed `aws:SourceIp` from boundary scope; kept it only in resource policy |
| EC2 `aws configure` credentials overriding role | Local `~/.aws/credentials` file had static keys | Deleted `~/.aws/credentials`; verified role via IMDS |
| `ResourceNotFoundException` in Secrets Manager | Wrong secret name (case-sensitive) | Corrected name from `prod/myapp/DB-password` to `prod/myapp/db-password` |

---

### Plan for Week 5: Monitoring & Security

Next week's focus shifts to **observability and threat detection** — ensuring the AWS infrastructure is continuously monitored and security events are automatically detected and responded to.

- Study **Amazon CloudWatch**: metrics, log groups, alarms, and dashboards for EC2, RDS, and Lambda
- Enable **AWS CloudTrail** to audit all API calls across the account with S3 log archiving
- Configure **Amazon GuardDuty** for automated threat detection using ML-based anomaly analysis
- Set up **AWS Config** rules to continuously evaluate resource configurations against compliance baselines
- Explore **AWS Security Hub** to aggregate findings from GuardDuty, Config, and Inspector into a unified dashboard
- Practice the **incident response flow**: GuardDuty finding → EventBridge rule → SNS alert → Lambda auto-remediation
