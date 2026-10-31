---
title: "Week 5 Worklog"
date: 2026-10-12
weight: 5
chapter: false
pre: " <b> 1.5. </b> "
---

## Week 5 Objectives: System Security

- **AWS WAF:** Provision a Web Application Firewall associated with an Application Load Balancer (ALB), configuring Web ACL rule groups to mitigate SQL Injection (SQLi) and Cross-Site Scripting (XSS) threats.
- **AWS GuardDuty:** Enable intelligent threat detection, analyzing foundational log sources with Machine Learning to identify anomalous behaviors such as port scanning (Portscan) and Command & Control (C&C) interactions.
- **VPC Flow Logs:** Implement comprehensive IP traffic flow monitoring across VPC and Subnet tiers, archiving compressed logs to Amazon S3 to inspect and isolate rejected packets (`REJECT`) from hostile sources.
- **AWS Security Hub & AWS Config:** Activate Security Hub to benchmark the Cloud Security Posture Management (CSPM Score) against CIS AWS Foundations Benchmark, supported by continuous configuration tracking via AWS Config.
- **Data Encryption with AWS KMS:** Create a Customer Managed Key (CMK), enforcing encryption at rest across Amazon EBS Volumes and Amazon S3 Buckets utilizing Bucket Keys to optimize cost.
- **Comprehensive Lab 5:** Execute a simulated penetration testing workflow (running Nmap port scans and sending malicious SQLi payloads via curl and web browser), validating GuardDuty anomaly detection and WAF perimeter enforcement.

### Tasks Planned for This Week

| Day | Topic | Task | Start Date | End Date | Reference |
| :---: | :--- | :--- | :---: | :---: | :--- |
| **Mon** | **AWS WAF** | Configure Web ACL on ALB, establish managed rule sets blocking SQL Injection and XSS | 12/10/2026 | 12/10/2026 | [AWS WAF](https://docs.aws.amazon.com/waf/) |
| **Tue** | **AWS GuardDuty** | Activate GuardDuty, analyze security findings including C&C Activity and Outbound Portscan | 13/10/2026 | 13/10/2026 | [AWS GuardDuty](https://docs.aws.amazon.com/guardduty/) |
| **Wed** | **VPC Flow Logs** | Enable VPC Flow Logs, archive compressed logs to S3, and analyze REJECT packet metadata | 14/10/2026 | 14/10/2026 | [VPC Flow Logs](https://docs.aws.amazon.com/vpc/latest/userguide/flow-logs.html) |
| **Thu** | **Security Hub & Config** | Activate Security Hub, evaluate CSPM posture score, and configure AWS Config Recorder | 15/10/2026 | 15/10/2026 | [AWS Security Hub](https://docs.aws.amazon.com/securityhub/) |
| **Fri** | **Data Encryption (KMS)** | Create Customer Managed Key (`lab-kms-key-labt6`), encrypt EBS Volume and S3 Bucket | 16/10/2026 | 16/10/2026 | [AWS KMS](https://docs.aws.amazon.com/kms/) |
| **Sat** | **Comprehensive Lab 5** | Deploy WAF Protection Pack, execute Nmap port scanning and SQLi injection tests, analyze findings | 17/10/2026 | 17/10/2026 | [AWS Threat Detection](https://aws.amazon.com/security/) |

### Week 5 Results

- Successfully deployed AWS WAF fronting Application Load Balancer `labt2-waf-alb-1192945120.us-east-1.elb.amazonaws.com`, terminating 100% of malicious SQLi (`?username=admin' OR '1'='1`) and XSS (`?search=<script>alert(1)</script>`) requests with `403 Forbidden` responses.
- Enabled AWS GuardDuty in `us-east-1`, identifying high-severity threat detections on instance `i-034f1dad8608f0a48`, including `Backdoor:EC2/C&CActivity.B!DNS` (High) and `Recon:EC2/Portscan` (Medium).
- Configured VPC Flow Logs `fl-0626508efb0473611` streaming to S3 Bucket `labt4-vpc-flowlogs`, parsing network records that uncovered unauthorized port scanning attempts from external IPs (`85.11.167.89`, `216.180.246.118`, `192.161.49.2`) showing status `REJECT OK`.
- Established AWS Security Hub, establishing a baseline Cloud Security Posture Management (CSPM) score of 30% (7 Passed, 16 Failed across 377 checks), with AWS Config tracking configuration compliance.
- Provisioned Customer Managed Key `lab-kms-key-labt6` (`e7ae1582-7e6f-4de0-8ff0-b5aadb9ac466`), encrypting a 1 GiB gp3 EBS volume `vol-0da2c452479eef183` and enforcing SSE-KMS default encryption with Bucket Key on S3 Bucket `labt6-s3-for-kms`.
- Completed Comprehensive Lab 5: enabled WAF Essential Protection Pack, installed Nmap on `LabT7-WAF` (`i-0584ad61366a505b4`), initiated remote port scanning that triggered GuardDuty finding `c0d0beb9531b554182419977d31f37e8`, and confirmed WAF perimeter mitigation against SQL Injection queries to ALB `LabT7-ALB`.

---

#### Exercise 1: AWS WAF — Web ACL Configuration & SQL Injection / XSS Mitigation

Constructed an enterprise Layer 7 perimeter defense: Provisioned an EC2 web application instance situated behind an Application Load Balancer named **`labt2-waf-alb`**:
- **Origin Web Server Public IP:** `44.222.158.206`
- **Application Load Balancer DNS Endpoint:** `labt2-waf-alb-1192945120.us-east-1.elb.amazonaws.com`
- **Region:** `us-east-1` (N. Virginia)

Accessing the origin web server directly via its IP address returned the default page: *"Hello from AWS WAF Lab Web Server"*.

> **Screenshot:** ![Direct Web Server Origin Access Responding Normally](/images/1-Worklog/week-05/01-waf-web-server-origin.png)

Configured a **Web ACL** on **AWS WAF** associated directly with ALB `labt2-waf-alb`, deploying two managed rule groups:
- **AWSManagedRulesSQLiRuleSet:** Inspects URI paths, query arguments, and request bodies for malicious SQL injection signatures.
- **AWSManagedRulesCommonRuleSet:** Defends against Cross-Site Scripting (XSS) patterns, known bad inputs, and common web application vulnerabilities.

Conducted attack simulation testing against the ALB DNS endpoint:
- **SQL Injection Simulation:** Dispatched an authentication bypass string: `http://labt2-waf-alb-1192945120.us-east-1.elb.amazonaws.com/?username=admin%27%20OR%20%271%27=%271`. AWS WAF inspected the query string, matched the SQLi rule signature, and terminated the request with a **`403 Forbidden`** error page.

> **Screenshot:** ![AWS WAF Blocking SQL Injection Attempt with 403 Forbidden](/images/1-Worklog/week-05/02-waf-block-sql-injection.png)

- **XSS Simulation:** Dispatched an executable script vector via the query parameter: `http://labt2-waf-alb-1192945120.us-east-1.elb.amazonaws.com/?search=<script>alert(1)</script>`. The Common Rule Set intercepted the script tag and returned a **`403 Forbidden`** response immediately.

> **Screenshot:** ![AWS WAF Blocking Cross-Site Scripting XSS Attempt with 403 Forbidden](/images/1-Worklog/week-05/03-waf-block-xss.png)

---

#### Exercise 2: AWS GuardDuty — Threat Detection & Anomaly Log Analysis

Enabled **AWS GuardDuty** under AWS Account `468502414950` (`Manhdung2507`) in Region `us-east-1`. Operating completely agentless, GuardDuty applies Machine Learning models and integrated threat intelligence feeds across VPC Flow Logs, DNS Query Logs, and CloudTrail Management Events.

Following monitoring intervals, the **Findings** console flagged critical security events targeting EC2 instance **`i-034f1dad8608f0a48`**:

> **Screenshot:** ![AWS GuardDuty Security Findings Inventory Console](/images/1-Worklog/week-05/04-guardduty-findings-list.png)

Investigated the detailed security findings surfaced by GuardDuty:
- **Command & Control (C&C) Activity Threat (High Severity):**
  - **Finding Title:** `The EC2 instance i-034f1dad8608f0a48 queried a known Command & Control server domain name.`
  - **Finding ID:** `a6d0c1a1a01159c61fedac2dcccde50d`
  - **Finding Type:** `Backdoor:EC2/C&CActivity.B!DNS`
  - **Severity:** **HIGH** (Red)
  - **Occurrence Count:** `4`
  - **Impacted Resource:** `EC2 Instance: i-034f1dad8608f0a48`
  - **Malware Scan Status:** `RUNNING` (Scan ID: `4111d46d648491294be6afb8f7379081`)
  - **Security Implication:** The instance attempted outbound DNS lookups toward domains associated with botnet Command & Control infrastructure, indicating potential backdoor malware infection.

> **Screenshot:** ![High-Severity Backdoor C&C Finding Details on EC2 Instance](/images/1-Worklog/week-05/05-guardduty-finding-cnc-detail.png)

- **Outbound Portscan Threat (Medium Severity):**
  - **Finding Title:** `An outbound portscan was detected from EC2 instance i-034f1dad8608f0a48.`
  - **Finding Type:** `Recon:EC2/Portscan`
  - **Severity:** **MEDIUM** (Orange)
  - **Occurrence Count:** `5`
  - **Security Implication:** Indicates the compromised instance was repurposed to conduct reconnaissance port scans against remote network hosts.

> **Screenshot:** ![Outbound Portscan Finding Details on GuardDuty](/images/1-Worklog/week-05/06-guardduty-finding-portscan.png)

---

#### Exercise 3: VPC Flow Logs — Network Traffic Monitoring & Security Forensics

Provisioned **VPC Flow Logs** to capture detailed IP packet traffic metadata crossing network interfaces within the VPC:
- **Flow Log ID:** `fl-0626508efb0473611`
- **Scope:** All Elastic Network Interfaces (ENIs) across the VPC
- **Traffic Filter:** `ALL` (Capturing both ACCEPT and REJECT events)
- **Destination:** Amazon S3 Bucket named **`labt4-vpc-flowlogs`**

The service systematically compressed log objects as `.log.gz` files organized by prefix: `AWSLogs/468502414950/vpcflowlogs/us-east-1/2026/10/30/`.

> **Screenshot:** ![Amazon S3 Bucket Archiving Compressed VPC Flow Logs](/images/1-Worklog/week-05/07-vpc-flowlogs-s3-bucket.png)

Downloaded log archive `468502414950_vpcflowlogs_us-east-1_fl-0626508efb0473611_...log.gz`, decompressed the contents, and reviewed the records:

```
version account-id interface-id srcaddr dstaddr srcport dstport protocol packets bytes start end action log-status
2 468502414950 eni-08f0d58b5d35c8399 85.11.167.89 172.31.5.159 48994 3001 6 1 40 1793369986 1793370024 REJECT OK
2 468502414950 eni-08f0d58b5d35c8399 216.180.246.118 172.31.5.159 21413 8500 6 4 176 1793369986 1793370024 REJECT OK
2 468502414950 eni-08f0d58b5d35c8399 192.161.49.2 172.31.5.159 44718 9215 6 1 40 1793369986 1793370024 REJECT OK
2 468502414950 eni-08f0d58b5d35c8399 193.163.125.176 172.31.5.159 47360 8443 6 1 44 1793369986 1793370024 REJECT OK
```

**Forensic Analysis Breakdown:**
- **Target Network Interface:** `eni-08f0d58b5d35c8399` (Internal IP: `172.31.5.159`)
- **External Scanning Source IPs:** `85.11.167.89`, `216.180.246.118`, `192.161.49.2`, `193.163.125.176`
- **Target Destination Ports:** Port `3001`, `8500`, `9215`, `8443` (Protocol `6` - TCP)
- **Enforcement Action:** **`REJECT OK`** — Demonstrating that Security Groups and Network ACLs successfully dropped unauthorized scan probes at the VPC boundary.

> **Screenshot:** ![Detailed Forensic Inspection of Decompressed VPC Flow Log Records](/images/1-Worklog/week-05/08-vpc-flowlogs-analysis-reject.png)

---

#### Exercise 4: AWS Security Hub & AWS Config — CIS Benchmark Compliance Posture

Activated **AWS Security Hub** to establish centralized security posture visibility and automated Cloud Security Posture Management (CSPM) evaluated against the **CIS AWS Foundations Benchmark**:
- **Posture Management Metrics:**
  - **Overall CSPM Score:** **`30%`**
  - **Controls Passed:** `7` security controls compliant
  - **Controls Failed:** `16` non-compliant checks requiring remediation
  - **No Data / Not Evaluated:** `377` checks
- **Analysis:** The baseline 30% score reflects an initial lab configuration lacking enterprise controls such as automated KMS key rotation, multi-region CloudTrail auditing, and S3 access logging.

> **Screenshot:** ![AWS Security Hub Cloud Security Posture Management CSPM Score](/images/1-Worklog/week-05/09-security-hub-posture-score.png)

In conjunction, configured **AWS Config** to track, audit, and evaluate resource configurations across the AWS account:
- **Configuration Recorder:** Enabled continuous recording of Configuration Items (CIs) for EC2, S3, IAM, and Security Group resources.
- **Conformance Packs & Rules:** Prepped rule sets to enforce continuous automated compliance auditing across future infrastructure deployments.

> **Screenshot:** ![AWS Config Dashboard Tracking Compliance Status](/images/1-Worklog/week-05/10-aws-config-dashboard.png)

---

#### Exercise 5: Data Encryption with AWS KMS — Creating CMK for EBS & S3 Protection

Provisioned a dedicated Customer Managed Key (**KMS Key - CMK**) within **AWS Key Management Service (KMS)**:
- **KMS Key Alias:** **`lab-kms-key-labt6`**
- **KMS Key ID:** `e7ae1582-7e6f-4de0-8ff0-b5aadb9ac466`
- **KMS Key ARN:** `arn:aws:kms:us-east-1:468502414950:key/e7ae1582-7e6f-4de0-8ff0-b5aadb9ac466`
- **Region:** `us-east-1` (US East - N. Virginia)

Enforced this CMK across two foundational AWS storage services:
- **Amazon S3 Bucket Encryption:** Created S3 Bucket **`labt6-s3-for-kms`** configured with **Default encryption**:
  - **Encryption Type:** `Server-side encryption with AWS Key Management Service keys (SSE-KMS)`
  - **Specified Key:** `arn:aws:kms:us-east-1:468502414950:key/e7ae1582-7e6f-4de0-8ff0-b5aadb9ac466`
  - **Bucket Key:** `Enabled` (Dramatically reduces KMS request overhead by up to 99%, mitigating API call costs)
  - **Blocked Encryption Types:** `Server-Side Encryption with Customer-Provided Keys (SSE-C)`

> **Screenshot:** ![S3 Bucket labt6-s3-for-kms Default SSE-KMS Encryption Configuration](/images/1-Worklog/week-05/11-kms-s3-default-encryption.png)

- **Amazon EBS Volume Encryption:** Provisioned a new EBS block storage volume with encryption parameters:
  - **Volume ID:** `vol-0da2c452479eef183`
  - **Size:** `1 GiB` | **Volume Type:** `gp3` (General Purpose SSD)
  - **IOPS:** `3000` | **Throughput:** `125`
  - **Availability Zone:** `use1-az1` (`us-east-1a`)
  - **Encryption Status:** `Encrypted`
  - **Associated KMS Key Alias:** `lab-kms-key-labt6`
  - **Volume State:** ✅ `Available` | **Status Check:** `Okay`

> **Screenshot:** ![EBS Volume Encrypted with Customer Managed Key lab-kms-key-labt6](/images/1-Worklog/week-05/12-kms-ebs-volume-encrypted.png)

---

#### Exercise 6: Comprehensive Lab 5 — Simulated Nmap Attacks & GuardDuty / WAF Detection Analysis

Executed an integrated end-to-end security exercise: Simulating external reconnaissance and web application attacks to validate the defensive telemetry of AWS WAF and GuardDuty.

**Step 1: Provision WAF Protection Pack (Essential Rules)**
Inside **AWS WAF & Shield**, configured an Essential Protection Pack encompassing foundational security rules:
- `Layer 7 anti-DDoS:` Count
- `IP blocklist & Geolocation restriction:` Block
- `AWS core rule set:` Default Block
- `SQL injection protection:` Default Block
- `Linux and Unix protection:` Default Block

> **Screenshot:** ![AWS WAF Protection Pack Essential Rules Configuration](/images/1-Worklog/week-05/13-lab5-waf-essential-rules.png)

**Step 2: Prepare Attack Simulation Host**
Launched EC2 instance **`LabT7-WAF`** to act as a staging platform for network testing:
- **Instance ID:** `i-0584ad61366a505b4`
- **Operating System:** Amazon Linux 2023
- **Public IPv4:** `100.58.223.95`
- **Private IPv4:** `172.31.3.123`
- Installed network scanning suite **Nmap**:
  ```bash
  sudo dnf install nmap -y
  # Successfully installed 3 packages: nmap, libssh2, nmap-ncat
  ```

> **Screenshot:** ![Installing Nmap Package Suite on EC2 Instance LabT7-WAF](/images/1-Worklog/week-05/14-lab5-ec2-install-nmap.png)

**Step 3: Simulate SQL Injection via curl**
From the terminal of `LabT7-WAF`, executed `curl` sending an SQLi payload against Application Load Balancer **`LabT7-ALB`**:

```bash
[ec2-user@ip-172-31-3-123 ~]$ curl -I "http://LabT7-ALB-1504252521.us-east-1.elb.amazonaws.com/?id=1'%20OR%20'1'='1"
HTTP/1.1 403 Forbidden
Server: awselb/2.0
Date: Tue, 27 Oct 2026 09:49:06 GMT
Content-Type: text/html
Content-Length: 118
Connection: keep-alive
```

The ALB immediately returned **`HTTP/1.1 403 Forbidden`** from `awselb/2.0`, confirming WAF intercepted the malicious query parameters before they reached backend compute.

> **Screenshot:** ![curl Command Blocked by WAF with 403 Forbidden Response](/images/1-Worklog/week-05/15-lab5-curl-sqli-waf-blocked.png)

**Step 4: Execute Full TCP Port Scan via Nmap**
From the local administrative workstation using Windows PowerShell, initiated a full TCP port scan against target EC2 host `100.58.223.95`:

```powershell
PS C:\Users\Admin> nmap -Pn -A -p- -T4 100.58.223.95
Starting Nmap 7.993 ( https://nmap.org ) at 2026-10-27 19:41 +0700
Nmap scan report for ec2-100-58-223-95.compute-1.amazonaws.com (100.58.223.95)
Host is up (0.25s latency).
Not shown: 65532 filtered tcp ports (no-response)
PORT    STATE  SERVICE VERSION
22/tcp  open   ssh     OpenSSH 9.9 (protocol 2.0)
80/tcp  closed http
443/tcp closed https
```

> **Screenshot:** ![Local Administrative Workstation Executing Nmap Port Scan](/images/1-Worklog/week-05/16-lab5-powershell-nmap-scan.png)

**Step 5: Verify Threat Detection in AWS GuardDuty**
Within minutes of executing the scan, AWS GuardDuty generated a Medium-severity finding:
- **Finding Title:** `An outbound portscan was detected from EC2 instance i-0584ad61366a505b4.`
- **Finding ID:** `c0d0beb9531b554182419977d31f37e8`
- **Finding Type:** `Recon:EC2/Portscan`
- **Severity:** `MEDIUM` | **Count:** `1`
- **Behavior Detail:** GuardDuty analyzed VPC Flow Logs and identified `i-0584ad61366a505b4` actively conducting port scans targeting `100.58.223.95`.

> **Screenshot:** ![AWS GuardDuty Successfully Flagging Portscan Activity](/images/1-Worklog/week-05/17-lab5-guardduty-detected-portscan.png)

**Step 6: Validate Browser Perimeter Mitigation**
Navigated directly to `http://labt7-alb-1504252521.us-east-1.elb.amazonaws.com/?id=1%27%20OR%20%271%27=%271` via the browser. The page returned a clean **`403 Forbidden`** block.

> **Screenshot:** ![Browser Verification Confirming WAF 403 Forbidden Enforcement](/images/1-Worklog/week-05/18-lab5-browser-sqli-waf-403.png)

---

#### Challenges Encountered

| Issue | Root Cause | Resolution |
| :--- | :--- | :--- |
| WAF fails to block SQLi when targeting EC2 IP directly | AWS WAF is associated with the Application Load Balancer, not EC2 Public IPs | Restricted EC2 Security Group to accept HTTP port 80 traffic exclusively from the ALB Security Group |
| GuardDuty does not display findings immediately after Nmap scans | GuardDuty processes log streams in aggregation batches (typically 5 to 15 minutes) | Allowed a 5-10 minute observation window for telemetry aggregation across VPC Flow Logs |
| VPC Flow Log archives in S3 cannot be opened directly | Flow Logs are automatically compressed as Gzip archives (`.log.gz`) to save storage costs | Downloaded and decompressed logs using archive utilities (7-Zip, WinRAR) or command-line `gunzip` |
| Unable to attach Customer Managed Key (CMK) to EBS volume | KMS Key Policy omitted authorization for the EC2 service principal | Updated KMS Key Policy granting `ec2.amazonaws.com` permissions for `kms:CreateGrant` and `kms:GenerateDataKey` |
| Security Hub CSPM compliance score reflects only 30% | Newly provisioned account lacks multi-region CloudTrail, MFA deletion, and S3 access audit logging | Reviewed the 16 failed controls in Security Hub and initiated remediation steps aligned with CIS Benchmarks |

### Plan for Week 6: CI/CD & DevOps Automation on AWS

- Study automated continuous delivery workflows utilizing **AWS CodePipeline**, **AWS CodeBuild**, and **AWS CodeDeploy**.
- Construct automated test execution and build Docker container images.
- Manage secure container images utilizing **Amazon Elastic Container Registry (ECR)**.
- Deploy serverless containerized applications on **Amazon Elastic Container Service (ECS) with AWS Fargate**.
- Establish automated build and pipeline status notifications via **Amazon SNS** dispatching to Email/Discord.
