---
title: "Worklog Week 2"
date: 2026-09-21
weight: 2
chapter: false
pre: " <b> 1.2. </b> "
---

## Week 2 Objectives: Networking & CDN

- **Amazon VPC Fundamentals:** Initialize VPC, configure IPv4 CIDR blocks, and segment isolated Public and Private Subnets.
- **Routing & NAT:** Master route tables, attach an Internet Gateway (IGW) for public ingress/egress, and provision a NAT Gateway for private egress.
- **Amazon Route 53:** Manage DNS records, create a Public Hosted Zone, and map domain A records to EC2 instance IP addresses.
- **CloudFront:** Deploy a Content Delivery Network (CDN), configure S3 as Origin storage, and accelerate content delivery via global Edge Locations.
- **Lambda@Edge:** Author Lambda functions intercepting viewer requests/responses at Edge Locations, injecting HTTP Security Headers.
- **Comprehensive Lab 2:** Build a secure multi-tier web architecture: Web server (Public Subnet) communicating securely with backend instances (Private Subnet).

### Tasks to be completed this week

| Day | Topic | Task Details | Start Date | Completion Date | Reference Materials |
| :---: | :--- | :--- | :---: | :---: | :--- |
| **Mon** | **Amazon VPC Fundamentals** | Initialize VPC, configure IPv4 CIDR, segment Public and Private Subnets | 21/09/2026 | 21/09/2026 | [Amazon VPC](https://docs.aws.amazon.com/vpc/) |
| **Tue** | **Routing & NAT** | Configure Route Tables, Internet Gateway (IGW), and NAT Gateway for Private Subnet | 22/09/2026 | 22/09/2026 | [AWS NAT Gateway](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-gateway.html) |
| **Wed** | **Amazon Route 53** | Manage DNS, create Hosted Zone, point A Record to EC2 IP address | 23/09/2026 | 23/09/2026 | [Amazon Route 53](https://docs.aws.amazon.com/route53/) |
| **Thu** | **CloudFront** | Provision CDN, configure S3 Origin to accelerate page loading | 24/09/2026 | 24/09/2026 | [Amazon CloudFront](https://docs.aws.amazon.com/cloudfront/) |
| **Fri** | **Lambda@Edge** | Author Lambda script intercepting requests/responses at Edge Locations | 25/09/2026 | 25/09/2026 | [Lambda@Edge](https://docs.aws.amazon.com/lambda/latest/dg/lambda-edge.html) |
| **Sat** | **Comprehensive Lab 2** | Deploy web architecture: Web server (Public Subnet) securely communicating with Private Subnet | 26/09/2026 | 26/09/2026 | [AWS Security Groups](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_SecurityGroups.html) |

### Week 2 Key Accomplishments

- Successfully initialized custom VPC (`My-Lab-VPC`), allocated IPv4 CIDR, and partitioned distinct `Public-subnet` and `Private-subnet`.
- Configured dynamic routing: attached Internet Gateway (IGW) to `Public-RT` and configured default egress `0.0.0.0/0` via NAT Gateway (`LabT3-NAT`) on `Private-RT`.
- Managed DNS routing with Amazon Route 53: created Public Hosted Zone `labt4-thuchanh.com`, mapped A Record to EC2 instance IP `98.86.220.109`, verified resolution using `nslookup`.
- Deployed global content caching with CloudFront Distribution (`d2vfkjpwxqkl8v.cloudfront.net`) backed by S3 Origin `bucketlabt5`, delivering low latency content.
- Implemented edge compute with Lambda@Edge: wrote Node.js 24.x handler bound to CloudFront `Viewer response` trigger, injecting HTTP Security Headers (`X-Frame-Options: DENY`, `X-Fpt-Sec-Lab: Hoan-thanh-Lab-T6`).
- Successfully completed Comprehensive Lab 2: designed tiered Security Groups (`Web-SG`, `Private-SG`), verifying internal inter-subnet ICMP reachability (`10.0.2.214`) at ~0.3ms latency without exposing private assets to the public internet.

---

#### Exercise 1: Amazon VPC Fundamentals — VPC Initialization, IPv4 CIDR & Subnet Segmentation

Created a dedicated custom Virtual Private Cloud (VPC) named **`My-Lab-VPC`**. Inside this VPC, partitioned IPv4 CIDR block into two isolated subnets:
- `Public-subnet`: Associated with an Internet Gateway, receiving external incoming user traffic.
- `Private-subnet`: Completely isolated from direct internet ingress, reserved for backend databases and services.

Launched an Apache web server on `Public-subnet` assigned with Public IPv4 `13.223.85.49`. Accessing the public IP via web browser displayed the welcome page: *"Chao mung den voi Web Server tren VPC rieng! - Manhdung2507"*.

> **Screenshot:** ![Web Server on Public Subnet within Custom VPC](/images/1-Worklog/week-02/01-vpc-web-server-public.png)

Provisioned a second EC2 instance named **`Test-Private-EC2`** (`i-0a2a4b868f760e445`) inside the `Private-subnet`. This instance is allocated only an internal Private IPv4 address (`10.0.2.91`), with no public IPv4 address.

> **Screenshot:** ![EC2 Instance Details in Private Subnet](/images/1-Worklog/week-02/02-private-ec2-instance.png)

Conducted network isolation verification: Attempted direct HTTP access from the internet to private IP `10.0.2.91`. The browser returned **`ERR_CONNECTION_TIMED_OUT`** ("This site can't be reached"). This validates that instances deployed within the private subnet are completely shielded from unsolicited internet scanning and threats.

> **Screenshot:** ![Verification of Inaccessibility to Private Subnet from Internet](/images/1-Worklog/week-02/03-private-ec2-connection-timeout.png)

---

#### Exercise 2: Routing & NAT — Route Tables, Internet Gateway (IGW) & NAT Gateway

Created and managed custom routing tables for VPC network traffic. Established **`Private-RT`** (`rtb-05b1c5a9be77d018d`) dedicated to private resources in `LabT3-VPC`.

> **Screenshot:** ![Private Route Table Configuration](/images/1-Worklog/week-02/04-private-route-table.png)

Segmented IP CIDR blocks for subnets within `LabT3-VPC`:
- `Public-Subnet`: `10.0.1.0/24` in Availability Zone `us-east-1a`
- `Private-Subnet`: `10.0.2.0/24` in Availability Zone `us-east-1a`

> **Screenshot:** ![Public and Private Subnet List in VPC](/images/1-Worklog/week-02/05-vpc-subnets-list.png)

Configured Subnet Associations: Associated the routing table **`Public-RT`** (`rtb-0d32fcfc31405b51b`) with `Public-Subnet` (`subnet-0d86b9a896d860d5b`), featuring a default route targeting the Internet Gateway (`igw`).

> **Screenshot:** ![Subnet Association for Public Route Table](/images/1-Worklog/week-02/06-public-subnet-association.png)

Created a managed **NAT Gateway** named **`LabT3-NAT`** (`nat-160fbcccddaa27944`). The NAT Gateway is placed inside `Public-Subnet` and allocated an Elastic IP (`18.209.181.106`) for translating outbound traffic. The NAT Gateway reached status **Available**.

> **Screenshot:** ![NAT Gateway Available with Elastic IP](/images/1-Worklog/week-02/07-nat-gateway-available.png)

Configured routes on `Private-RT`: Added default egress route `0.0.0.0/0` targeting NAT Gateway `nat-160fbcccddaa27944`. This enables instances in the private subnet to securely fetch software updates and dependencies from the internet while maintaining strict blocking against inbound connections.

```bash
# Verify outbound connectivity from Private EC2 instance via NAT Gateway
curl -I https://aws.amazon.com
# HTTP/2 200
# content-type: text/html; charset=utf-8
```

> **Screenshot:** ![Outbound NAT Route Configured in Private Route Table](/images/1-Worklog/week-02/08-private-rt-nat-route.png)

---

#### Exercise 3: Amazon Route 53 — DNS Management, Hosted Zone & A Record

Launched a test EC2 instance **`LabT4-Test`** (`i-0a86d42df283fe32e`) assigned with Public IPv4 `98.86.220.109`.

> **Screenshot:** ![EC2 Target Instance for Route 53 DNS](/images/1-Worklog/week-02/09-ec2-route53-instance.png)

Created a **Public Hosted Zone** in **Amazon Route 53** for domain `labt4-thuchanh.com` (Hosted Zone ID: `Z012720835D8Y5T663A3Y`). The service automatically allocated 4 authoritative AWS Name Servers (NS) and an SOA record.

> **Screenshot:** ![Public Hosted Zone Created in Route 53](/images/1-Worklog/week-02/10-route53-hosted-zone.png)

Created an **A Record** for the domain:
- **Record name:** `www.labt4-thuchanh.com`
- **Record type:** `A - Routes traffic to an IPv4 address and some AWS resources`
- **Value:** `98.86.220.109`
- **TTL (Seconds):** `300`
- **Routing policy:** Simple

> **Screenshot:** ![A Record Created Pointing to EC2 Public IP](/images/1-Worklog/week-02/11-route53-record-created.png)

Verified authoritative DNS resolution: Executed a query directly against Route 53's designated Name Server (`ns-388.awsdns-48.com`) using Command Prompt:

```cmd
nslookup www.labt4-thuchanh.com ns-388.awsdns-48.com
```

The lookup resolved to the expected IPv4 address `98.86.220.109`, confirming proper DNS routing setup.

> **Screenshot:** ![nslookup Query Verifying Accurate DNS Resolution](/images/1-Worklog/week-02/12-nslookup-dns-verification.png)

---

#### Exercise 4: CloudFront — CDN Deployment & S3 Origin Page Load Acceleration

Provisioned an **Amazon CloudFront Distribution** named **`Labt5`** (Distribution ID: `E3G4O9N747F2M4`). CloudFront assigned the global distribution domain name `d2vfkjpwxqkl8v.cloudfront.net` with status **Enabled**.

> **Screenshot:** ![CloudFront Distribution Provisioned and Enabled](/images/1-Worklog/week-02/13-cloudfront-distribution.png)

Created S3 Bucket **`bucketlabt5`** (`arn:aws:s3:::bucketlabt5`) in Region `us-east-1` configured as the Origin storage for the CloudFront distribution to accelerate static page delivery.

> **Screenshot:** ![S3 Bucket Configured as CloudFront Origin](/images/1-Worklog/week-02/14-s3-bucket-cloudfront.png)

Created and uploaded `index.html` into S3 Bucket `bucketlabt5`, containing custom CDN deployment verification text.

> **Screenshot:** ![index.html File Uploaded into S3 Origin](/images/1-Worklog/week-02/15-s3-objects-index-html.png)

Accessed the static site via the global CloudFront edge URL `https://d2vfkjpwxqkl8v.cloudfront.net/index.html`. The page rendered with low latency and displayed: *"Triển khai CDN Thành công! Trang web đang được phân phối qua mạng lưới Edge Locations toàn cầu của AWS CloudFront."*

> **Screenshot:** ![Static Website Live via CloudFront Global CDN](/images/1-Worklog/week-02/16-cloudfront-website-live.png)

---

#### Exercise 5: Lambda@Edge — Intercepting Requests/Responses at Edge Locations

Created a serverless function named **`Edge-Security-Header`** running on `Node.js 24.x` with `x86_64` architecture in Region `us-east-1` (the mandatory region for Lambda@Edge distributions).

> **Screenshot:** ![Lambda Function Created for Lambda@Edge](/images/1-Worklog/week-02/17-lambda-create-function.png)

Configured IAM Execution Role: Attached existing role **`lambda-edge-role`** which includes trusted entities for both `lambda.amazonaws.com` and `edgelambda.amazonaws.com`.

> **Screenshot:** ![IAM Execution Role Selection for Lambda@Edge](/images/1-Worklog/week-02/18-lambda-role-selection.png)

Configured permissions via IAM Policy Statement Editor, ensuring CloudWatch Logs execution write access across edge points of presence.

> **Screenshot:** ![IAM Policy Statement Editor Configuration](/images/1-Worklog/week-02/19-lambda-policy-statement.png)

Authored Node.js logic to intercept responses and inject HTTP Security Headers; configured the CloudFront trigger:
- **Distribution:** `E3G4O9N747F2M4` (CloudFront)
- **CloudFront event:** `Viewer response`
- Deployed a published version to Lambda@Edge.

```javascript
export const handler = async (event) => {
    const response = event.Records[0].cf.response;
    const headers = response.headers;

    // Intercept and inject HTTP Security Headers
    headers['strict-transport-security'] = [{ key: 'Strict-Transport-Security', value: 'max-age=63072000; includeSubdomains; preload' }];
    headers['x-content-type-options'] = [{ key: 'X-Content-Type-Options', value: 'nosniff' }];
    headers['x-frame-options'] = [{ key: 'X-Frame-Options', value: 'DENY' }];
    headers['x-xss-protection'] = [{ key: 'X-XSS-Protection', value: '1; mode=block' }];
    headers['x-fpt-sec-lab'] = [{ key: 'X-Fpt-Sec-Lab', value: 'Hoan-thanh-Lab-T6' }];

    return response;
};
```

> **Screenshot:** ![CloudFront Viewer Response Trigger Configuration](/images/1-Worklog/week-02/20-lambda-edge-trigger.png)

Inspected HTTP Response Headers using browser DevTools (Network tab) when requesting `https://d2vfkjpwxqkl8v.cloudfront.net/index.html`. Confirmed that custom security headers injected by Lambda@Edge were present: `X-Fpt-Sec-Lab: Hoan-thanh-Lab-T6` and `X-Frame-Options: DENY`.

> **Screenshot:** ![Verified Security Response Headers via Browser DevTools](/images/1-Worklog/week-02/21-lambda-edge-headers-verified.png)

---

#### Exercise 6: Comprehensive Lab 2 — Web Architecture: Web Server (Public) & Secure Communication (Private)

Configured Security Group for the public web tier named **`Web-SG`** in VPC `Mylabt7`, opening ingress rules for client traffic:
- **HTTP (TCP 80)**: Source `0.0.0.0/0`
- **HTTPS (TCP 443)**: Source `0.0.0.0/0`
- **SSH (TCP 22)**: Source `0.0.0.0/0` (administrative access)

> **Screenshot:** ![Web-SG Security Group Inbound Rules](/images/1-Worklog/week-02/22-security-group-web-sg.png)

Configured Security Group for backend private instances **`Private-SG`** (`sg-00f606d0da92bd89c`). For strict isolation, Inbound rules permitted only internal communication:
- **All ICMP - IPv4**: Network reachability checks within internal subnet.
- **SSH (TCP 22)**: Secure administrative access originating from the Web Server (Bastion Host) in the Public Subnet.

> **Screenshot:** ![Private-SG Inbound Rules Configuration](/images/1-Worklog/week-02/24-private-security-group-rules.png)

Executed Inter-Subnet Connectivity Verification: Connected to public instance **`LabT7-Web-Server`** (`10.0.1.123`) via EC2 Instance Connect and initiated an ICMP `ping` toward private instance `10.0.2.214`. Packet transmission succeeded with ~0.3ms latency, demonstrating secure multi-tier networking and proper Security Group rule enforcement.

```bash
# Ping private backend instance from public web server
ping 10.0.2.214
# PING 10.0.2.214 (10.0.2.214) 56(84) bytes of data.
# 64 bytes from 10.0.2.214: icmp_seq=1 ttl=127 time=1.31 ms
# 64 bytes from 10.0.2.214: icmp_seq=2 ttl=127 time=0.319 ms
# ...
```

> **Screenshot:** ![Successful Inter-Subnet ICMP Ping Test](/images/1-Worklog/week-02/23-inter-subnet-ping-test.png)

---

#### Troubleshooting & Challenges Encountered

| Issue | Root Cause | Resolution |
| :--- | :--- | :--- |
| Inability to connect to Private EC2 instance via browser | Private Subnet lacks an Internet Gateway route and public IP addressing | Intended architectural design; manage through a Bastion Host or configure NAT Gateway for egress |
| Private Route Table instances unable to reach internet dependencies | Missing default egress route `0.0.0.0/0` pointing to NAT Gateway | Added route `0.0.0.0/0` with target `nat-160fbcccddaa27944` in `Private-RT` |
| Route 53 A record not immediately resolving on local workstation | Local recursive DNS resolvers caching old NXDOMAIN records | Queried AWS Authoritative Name Server directly (`ns-388.awsdns-48.com`) using `nslookup` or flushed DNS via `ipconfig /flushdns` |
| CloudFront trigger option missing in Lambda console | Lambda function created outside Region `us-east-1` (N. Virginia) | Recreated function in `us-east-1` and verified execution role trusted `edgelambda.amazonaws.com` |
| Inter-subnet ping times out between instances | Target instance Security Group missing inbound ICMP rule | Added `All ICMP - IPv4` rule in target Security Group inbound configuration scoped to VPC CIDR |

### Week 3 Plan

- Explore and deploy AWS Elastic Load Balancing (Application Load Balancer - ALB).
- Configure Auto Scaling Groups (ASG) for automated dynamic compute scaling.
- Set up target groups, path routing, and target health checks.
- Establish system monitoring and automated alerting using Amazon CloudWatch Alarms.
